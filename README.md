# Simulador de Filas ao Vivo — Supermercado (Grupo 6)

ENGF84 · Modelagem e Otimização de Sistemas de Produção · UFBA 2026.1

Simulação de eventos discretos da operação de caixa de um supermercado, com **animação em tempo real**, num **arquivo único HTML/JS** (`index.html`) — sem build e sem dependências obrigatórias. O modelo é **orientado a dados**: ingere um log de eventos, estima os próprios parâmetros e simula. Trocar a planilha troca o sistema simulado.

Sistema modelado: **M(t)/G/c com abandono** — chegadas Poisson não-homogêneas, coleta como atraso individual (sem fila), fila FIFO do caixa com paciência limitada (*reneging*) e serviço dependente do tamanho da cesta.

## O que dá para fazer

- **Assistir à operação ao vivo**: clientes chegam, coletam itens, entram na fila, são atendidos nos caixas (com barra de progresso) ou abandonam. Cada cliente é uma pílula colorida por fase, com o tamanho da cesta.
- **Controlar o tempo**: iniciar / pausar / reiniciar e acelerar — **1×, 2×, 10×, 100×, 1000×** (1× = tempo real, 1 segundo simulado por segundo). Um relógio mostra a hora do dia e a barra de progresso o avanço.
- **Ajustar o único parâmetro que o gerente controla**: o **número de caixas abertos**. Demanda, serviço, coleta e paciência vêm dos dados e não são editáveis — é o supermercado real, não um brinquedo de sliders.
- **Ler os indicadores em destaque**, no estilo das "5 fórmulas" do M/M/1, atualizados ao vivo: **Abandono**, **Utilização ρ**, **Espera na fila Wq** em destaque; **Fila Lq**, **Sistema L**, **Tempo W** e **Throughput** logo abaixo.
- **Consultar o registro de eventos**: log filtrável por tipo (chegada, fila, atendimento, saída, abandono), buscável por cliente, sincronizado com o relógio, com download em `.csv`.
- **Conferir a análise técnica** (recolhível): baseline analítico M/M/c (Erlang-C) e a bateria de verificação/validação (conservação, Lei de Little, comparação com os dados reais).

## Contrato de esquema (colunas exigidas)

```
id, dia, itens, chegada, coleta_inicio, coleta_fim, caixa_inicio, caixa_fim, abandonou
```

Datas em `AAAA-MM-DD HH:MM:SS`; `abandonou` em `True`/`False`. Qualquer log com este esquema é aceito e recalibra o modelo sozinho.

## Dados de exemplo (sempre funciona)

O botão **"usar dados de exemplo"** tenta carregar o CSV hospedado e, se não conseguir (offline, ou aberto via `file://`), **gera um dataset válido de 25 dias** rodando o próprio motor — estatisticamente equivalente ao real (μ≈33,8/h, serviço≈36+3·itens, abandono≈10%, com um dia atípico mais cheio). Assim a demonstração nunca quebra.

## Como rodar localmente

Abrir `index.html` direto no navegador. Anexar planilha .csv com entrada de dados.

## Estrutura da repo

```
index.html                       <- o simulador inteiro (motor + animacao + painel)
ENGF84_dados_supermercado.csv    <- log de exemplo (opcional; o botao gera dados se faltar)
README.md
```

## Notas de validação

- O motor da animação é o mesmo verificado: conservação de entidades, Lei de Little (erro ~2%), e abandono/throughput do modelo batem com os dados reais.
- Os KPIs ao vivo são médias acumuladas desde a abertura e convergem para os valores finais do dia; pequenas diferenças de borda no fechamento (serviços que terminam após o horário) são esperadas.
- A bateria de convergência ao M/M/c (no cenário simplificado) fica abaixo de ~4% de desvio para os pontos de operação relevantes.
