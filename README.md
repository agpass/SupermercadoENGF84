# Simulador de Filas — Operação de Caixa de Supermercado

**Grupo 6 · ENGF84 — Modelagem e Otimização de Sistemas de Produção · UFBA 2026.1**

Análise e simulação de filas de caixa de supermercado, em dois arquivos HTML/JS de página única, **sem build e sem dependências obrigatórias**. O sistema é modelado como **M(t)/G/c com abandono**: chegadas de Poisson não-homogêneas, tempo de serviço geral (função do tamanho da cesta), `c` caixas em paralelo e fila com paciência limitada (*reneging*).

O projeto faz duas coisas encadeadas:

1. **Da planilha de dados reais a um modelo matemático generalizável** — os dados observados viram parâmetros de um modelo de filas; e como o modelo é paramétrico, ele permite **gerar novos dados** para simular realidades diferentes.
2. **Da simulação de eventos discretos à análise em dashboard** — esses dados (reais ou gerados) alimentam uma simulação de eventos discretos, com indicadores e relatório segundo a **teoria das filas**.

## As duas ferramentas

| Arquivo | O que faz |
|---|---|
| **`index.html`** | O simulador: carrega a planilha real, calcula e reporta os parâmetros, visualiza o fluxo dia a dia e re-simula sob configurações de caixa editáveis. |
| **`gerador.html`** | O anexo: gera planilhas novas a partir de parâmetros escolhidos (λ por hora, serviço, coleta, cesta, caixas escalonados, paciência). |

Os dois compartilham o mesmo núcleo de modelagem (`sim_core.js`, embutido em cada um). As referências de código abaixo apontam para **funções** buscáveis no `<script>` de cada arquivo.

---

## Como funciona

### Parte I — Da planilha ao modelo matemático generalizável

**Leitura e contrato de esquema (`parseData`).** Toda planilha precisa ter as colunas do contrato:

```
id, dia, itens, chegada, coleta_inicio, coleta_fim, caixa_inicio, caixa_fim, abandonou
```

`parseData` valida essas colunas antes de qualquer cálculo e converte os carimbos `AAAA-MM-DD HH:MM:SS` em minutos desde a meia-noite (a unidade escalar interna).

**Limpeza (`limpar`).** Remove duplicatas e registros temporalmente impossíveis, contabiliza coletas faltantes e **mantém os abandonos** (dado legítimo, não ruído), devolvendo um relatório de auditoria.

**Estimação — o modelo (`estimar`).** Transforma dados em modelo. Cada fonte de variabilidade do M(t)/G/c é estimada de uma coluna:

- **λ(h)** — chegadas por hora (Poisson não-homogêneo), contagem por faixa dividida pelos dias;
- **μ e serviço G = f(itens)** — regressão de `caixa_fim − caixa_inicio` sobre `itens`, dando **serviço(s) ≈ b0 + b1·itens** (36 + 3·itens, r ≈ 0,99 no dataset real);
- **coleta** — log-normal por momentos;
- **itens** — distribuição empírica reamostrada;
- **c** — concorrência máxima dos atendimentos (varredura *sweep-line*, separando os dias).

O objeto devolvido **é o modelo**: um M(t)/G/c com abandono cujos parâmetros descrevem aquele supermercado. Por serem parâmetros (e não valores fixos), o mesmo código recalibra o modelo para qualquer planilha do mesmo esquema — daí a **generalização**.

**Geração de novas realidades (`gerador.html`).** Como o modelo é paramétrico, dá para o caminho inverso: dado um conjunto de parâmetros, sintetizar uma planilha nova. `gerarChegadas` produz um processo de Poisson não-homogêneo (intervalos exponenciais com λ(h)), sorteando itens e coleta por cliente. Por ser Poisson, a contagem de cada hora **oscila em torno da média** (para 70, valores como 65, 71, 68). Uma **variação entre dias (±%)** reescala a demanda de cada dia, e um **escalonamento de caixas** (cada caixa com seu horário) atende os clientes via `runCenario`. A página reestima o CSV gerado e mostra uma tabela *pedido × medido*.

### Parte II — Simulação de eventos discretos e análise em dashboard

**Preparação (`prepararReais`).** Agrupa os clientes reais por dia, cada um com chegada, entrada na fila, itens, tempo de serviço real (quando atendido) e desfecho.

**Motor de eventos discretos (`runCenario`).** Uma re-simulação *trace-driven*: recebe o fluxo real de clientes e uma configuração de caixas (servidores com `{abre, fecha}`) e recomputa a fila. Características:

- lista de eventos futuros numa *heap* binária (o relógio salta para o próximo evento);
- disciplina **FIFO**; caixas só atendem dentro da própria janela (escalonamento);
- **tempo de serviço** real de cada cliente quando disponível, senão modelado por `b0 + b1·itens`;
- **abandono** (reneging) com paciência exponencial e evento obsoleto (garante conservação);
- **acumuladores integrais no tempo** para Lq, ρ e L (médias temporais corretas).

A paciência é ajustada por `calibraPacienciaReal` para que a **configuração real reproduza o abandono observado** — assim o cenário-base valida contra a realidade, e mudar caixas vira um cenário hipotético sobre a mesma demanda.

**Teoria das filas na análise.** Notação de Kendall (M(t)/G/c com abandono); grandezas λ, μ, **ρ**; as **5 fórmulas** (ρ, Lq, L, Wq, W) medidas por integração; **Lei de Little** como verificação; e o baseline analítico **M/M/c (Erlang-C)** via `mmc`, que sinaliza instabilidade quando ρ ≥ 1.

---

## Funcionalidades

**Simulador (`index.html`):**

- Relatório dos parâmetros calculados a partir de toda a planilha.
- Visualização animada do fluxo real, cliente a cliente, com navegação **dia a dia**.
- **Configuração de caixas editável**: número de caixas e **horário de abertura de cada um** (ex.: caixa 1 das 8–22h, caixa 2 das 18–22h).
- Velocidade 1×/10×/100×/1000×/5000× e um campo personalizado.
- **Gráfico dos parâmetros ao longo do dia** (λ, atendidos, abandonos, Lq, ρ, Wq).
- **Comparação dia a dia** do mesmo indicador — para ver se o pico do dia 1 coincide com o do dia 2.
- **Emitir relatório**: snapshot em HTML (imprimível em PDF) com parâmetros, configuração de caixas, KPIs médios e tabela por dia.

**Gerador (`gerador.html`):**

- λ por hora com preenchimento rápido base/pico; contagem por hora oscila em torno da média (Poisson).
- Variação de demanda entre dias (±%).
- Escalonamento de caixas (horário por caixa).
- Serviço, coleta, cesta e paciência configuráveis; tabela *pedido × medido* e download do CSV.

---

## Como rodar localmente

Abrir `index.html` ou `gerador.html` direto no navegador já funciona. Para o botão "usar dados de exemplo" do simulador (que busca o CSV por `fetch`), sirva a pasta:

```bash
python3 -m http.server 8000     # e abra http://localhost:8000
```

Arrastar-e-soltar a planilha funciona offline. O leitor de `.xlsx` precisa de internet (carrega a biblioteca sob demanda); para uso 100% offline, exporte como `.csv`.

## Verificação e validação

- **Conservação de entidades** em todos os dias (entram = atendidos + abandonos).
- Com a configuração de caixas real, a re-simulação **reproduz o abandono observado** no log.
- **Convergência ao M/M/c** em cenário simplificado (baixo desvio nos pontos de operação).
- **Lei de Little** (L ≈ λ·W) satisfeita nos resultados.
- **Fidelidade do gerador**: o CSV gerado, reestimado, devolve os parâmetros pedidos.

---

*Grupo 6 · ENGF84 · UFBA 2026.1*
