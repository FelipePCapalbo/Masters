# Modelo de Otimização: Planejamento Agregado com Estoque e Capacidade Restrita

Este documento apresenta a modelagem matemática para um problema de planejamento agregado da produção com múltiplos produtos, horizonte mensal e restrições de capacidade por máquina. O objetivo é minimizar faltas de atendimento à demanda, considerando a possibilidade de antecipar produção e formar estoque para compensar a insuficiência de capacidade em períodos futuros. O modelo permite atrasos de atendimento da demanda, até um limite máximo de semanas (convertido para períodos mensais) definido por parâmetro.

## Índices

* $p \in P$: conjunto de produtos
* $t \in T$: conjunto de períodos (mês/ano)
* $m \in M$: conjunto de máquinas
* $\tau \in T$: índice de períodos anteriores a $t$

## Conjuntos

* **P**: conjunto de produtos
* **T**: conjunto de períodos mensais
* **M**: conjunto de máquinas
* **$P_m \subseteq P$**: subconjunto de produtos que podem ser produzidos na máquina $m$

## Parâmetros

| Parâmetro          | Descrição                                                                  |
| ------------------ | -------------------------------------------------------------------------- |
| $D_{p,t}$          | Demanda prevista do produto $p$ no período $t$ (em kg)                     |
| $a_{p,m}$          | Taxa de processamento na máquina $m$ para o produto $p$ (em kg/min)        |
| $\text{Cap}_{m,t}$ | Capacidade produtiva disponível da máquina $m$ no período $t$ (em minutos) |
| $I_{p,0}$          | Estoque inicial do produto $p$ (em kg)                                     |
| $C^{\text{sh}}$    | Custo por unidade de falta (atraso acima do limite permitido)              |
| $\delta$           | Número máximo de semanas que um pedido pode ser atendido com atraso        |
| $w$                | Número de semanas por período (ex: 4 semanas por mês)                      |

**Nota**: o número de períodos de atraso permitidos é dado por $\Delta = \lfloor \delta / w \rfloor$, ou seja, o maior número inteiro de meses dentro do limite de semanas.

## Variáveis de decisão

| Variável              | Descrição                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------- |
| $x_{p,t} \geq 0$      | Quantidade produzida do produto $p$ no período $t$ (em kg)                                        |
| $i_{p,t} \geq 0$      | Estoque final do produto $p$ no período $t$ (em kg)                                               |
| $y_{p,t,\tau} \geq 0$ | Quantidade da demanda de $\tau$ atendida no período $t$ (em kg)                                   |
| $b_{p,t} \geq 0$      | Quantidade de demanda originalmente de $t$ que não foi atendida dentro do prazo permitido (falta) |

## Função objetivo

Minimizar o total de faltas:

$$
\min \sum_{p \in P} \sum_{t \in T} C^{\text{sh}} \cdot b_{p,t}
$$

## Restrições

### 1. Balanço de atendimento da demanda com atraso máximo

Cada demanda $D_{p,t}$ pode ser atendida em até $\Delta$ períodos futuros:

$$
\sum_{\tau = t}^{t + \Delta} y_{p,\tau,t} + b_{p,t} = D_{p,t}, \quad \forall p \in P,\; t \in T
$$

### 2. Balanço de estoque

A produção somada ao estoque anterior deve suprir os atendimentos do período:

$$
i_{p,t-1} + x_{p,t} = \sum_{\tau = t - \Delta}^{t} y_{p,t,\tau} + i_{p,t}, \quad \forall p \in P,\; t \in T
$$

Com:

$$
i_{p,0} = I_{p,0}
$$

### 3. Capacidade produtiva por máquina

$$
\sum_{p \in P_m} \frac{x_{p,t}}{a_{p,m}} \leq \text{Cap}_{m,t}, \quad \forall m \in M,\; t \in T
$$

## Considerações adicionais

* O parâmetro $\delta$ (em semanas) é convertido para o número inteiro de períodos mensais permitidos $\Delta$ com base em $w$, o número de semanas por mês.
* A modelagem permite que parte da demanda de um mês seja atendida em meses seguintes, desde que dentro do limite $\Delta$.
* Não há custos de estoque nem custos de setup.
* Todas as produções são obrigatórias (não há lotes mínimos nem decisões binárias de ativação).
* Cada máquina só pode produzir determinados produtos, conforme $P_m$.
