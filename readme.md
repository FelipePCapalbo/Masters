# Modelo de Otimização: Planejamento Agregado com Estoque e Capacidade Restrita

## Índices

* $p \in P$: produtos
* $t \in T$: períodos (semana/mês/ano)
* $m \in M$: máquinas
* $\tau \in T$: período de origem da demanda

## Conjuntos

* **P**: conjunto de produtos
* **T**: conjunto de períodos mensais
* **M**: conjunto de máquinas
* **$P_m \subseteq P$**: subconjunto de produtos que podem ser produzidos na máquina $m$

## Parâmetros

| Parâmetro           | Descrição                                                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| $D_{p,t}$           | Demanda prevista do produto $p$ no período $t$ (em kg)                                                                    |
| $a_{p,m}$           | Taxa de processamento na máquina $m$ para o produto $p$ (em kg/min)                                                       |
| $\text{Cap}_{m,t}$  | Capacidade produtiva disponível da máquina $m$ no período $t$ (em minutos)                                                |
| $I_{p,0}$           | Estoque inicial do produto $p$ (em kg)                                                                                    |
| $C^{\text{f}}_{p}$ | Custo por unidade (kg) de falta do produto $p$ (atraso acima do limite permitido)                                              |
| $\delta$            | Número máximo de semanas que um pedido pode ser atendido com atraso                                                       |
| $w$                 | Número de subperíodos por período (ex: 4 semanas por mês; 7 dias na semana)                                                                     |
| $\Delta$            | Número máximo de períodos que um pedido pode ser atendido com atraso, definido como $\Delta = \lfloor \delta / w \rfloor$ |

## Variáveis de decisão

| Variável         | Descrição                                                                                         |
| ---------------- | ------------------------------------------------------------------------------------------------- |
| $x_{p,t} \geq 0$ | Quantidade produzida do produto $p$ no período $t$ (em kg)                                        |
| $i_{p,t} \geq 0$ | Estoque final do produto $p$ no período $t$ (em kg)                                               |
| $b_{p,t} \geq 0$ | Quantidade de demanda originalmente de $t$ que não foi atendida dentro do prazo permitido (falta) |

## Função objetivo

Minimizar o total de faltas ponderado pelo custo por produto:

$$
\min \sum_{p \in P} \sum_{t \in T} C^{\text{f}}_{p} \cdot b_{p,t}
$$

## Restrições

### 1. Atendimento da demanda com atraso máximo

Cada demanda $D_{p,t}$ deve ser atendida até $\Delta$ períodos após $t$, ou será considerada falta. O total acumulado de produção e estoque nos períodos $t$ até $t+\Delta$ deve cobrir a demanda do período $t$:

$$
\sum_{\tau = t}^{t + \Delta} x_{p,\tau} + i_{p,t-1} \geq D_{p,t} - b_{p,t}, \quad \forall p \in P,\; t \in T
$$

Assume-se que $i_{p,0} = I_{p,0}$ e que $x_{p,t} = 0$ e $i_{p,t} = 0$ para períodos fora do horizonte.

### 2. Balanço de estoque

A evolução do estoque é dada por:

$$
i_{p,t} = i_{p,t-1} + x_{p,t} - D_{p,t} + b_{p,t}, \quad \forall p \in P,\; t \in T
$$

### 3. Capacidade produtiva por máquina

$$
\sum_{p \in P_m} \frac{x_{p,t}}{a_{p,m}} \leq \text{Cap}_{m,t}, \quad \forall m \in M,\; t \in T
$$

## Observações

* O parâmetro $\Delta = \lfloor \delta / w \rfloor$ define o número máximo de períodos mensais que um pedido pode ser atrasado, sendo sempre um numero inteiro.
* A modelagem permite que parte da demanda de um mês seja atendida em periodos seguintes, desde que dentro do limite $\Delta$, constituindo o conceito de "atraso".
* Custos de estoque não foram adicionados.
* Custos, ou tempos, de setup não foram adicionados.
* Todas as produções são obrigatórias, pois trata-se de um planejamento agregado tático (não há lotes mínimos nem decisões binárias de ativação).
* Cada máquina só pode produzir determinados produtos, conforme $P_m$.
* O custo de cancelamento varia por produto e está refletido no parâmetro $C^{\text{f}}_{p}$.
