# Matriz de decisão dos estilos arquiteturais

Para que a escolha não seja só opinião, comparamos os estilos candidatos de cada sistema contra os **requisitos não funcionais críticos**.

**Como ler:**
- Cada critério tem um **peso** (a soma é 100), definido pelo grupo conforme a importância para o negócio daquele sistema.
- Cada estilo recebe uma **nota de 1 a 5** em cada critério (5 = atende muito bem).
- A **nota final** é a média ponderada (de 1 a 5).

> **Aviso:** as notas são um **julgamento do grupo**, baseado em conhecimento geral de arquitetura, e não uma medição. O valor da matriz é deixar os trade-offs explícitos e discutíveis. Se mudarem os pesos (por exemplo, uma startup que valoriza muito a velocidade), o resultado pode mudar. Esse é justamente o ponto da reflexão no README: o contexto de negócio influencia a escolha.

## 1. E-commerce

| Critério | Peso | Monolito simples | Monolito modular | Microsserviços + eventos | SOA com barramento (ESB) |
|---|:-:|:-:|:-:|:-:|:-:|
| Escalabilidade seletiva (picos como a Black Friday) | 25 | 2 | 3 | 5 | 3 |
| Tolerância a falhas / isolamento de falhas | 20 | 2 | 2 | 5 | 3 |
| Autonomia dos times (deploy independente) | 20 | 1 | 3 | 5 | 3 |
| Simplicidade operacional | 15 | 5 | 4 | 1 | 2 |
| Consistência forte | 10 | 5 | 5 | 2 | 3 |
| Velocidade para lançar a primeira versão | 10 | 5 | 4 | 2 | 2 |
| **Nota final (1 a 5)** | **100** | 2.85 | 3.25 | **3.80 ✔** | 2.75 |

## 2. App bancário

| Critério | Peso | Monolito simples (sem módulos) | Monolito modular | Microsserviços | Orientado a eventos puro (event sourcing) |
|---|:-:|:-:|:-:|:-:|:-:|
| Consistência forte (ACID) | 30 | 5 | 5 | 2 | 3 |
| Segurança e superfície de ataque | 20 | 4 | 4 | 3 | 3 |
| Auditoria e rastreabilidade | 15 | 4 | 4 | 3 | 5 |
| Disponibilidade | 10 | 3 | 3 | 5 | 4 |
| Simplicidade operacional | 10 | 5 | 4 | 1 | 1 |
| Facilidade de evoluir e manter | 10 | 1 | 4 | 5 | 4 |
| Escalabilidade horizontal extrema | 5 | 2 | 2 | 5 | 4 |
| **Nota final (1 a 5)** | **100** | 3.90 | **4.10 ✔** | 3.00 | 3.35 |

## 3. Jogo multiplayer

| Critério | Peso | Cliente-servidor com servidor autoritativo | Peer-to-Peer (P2P) | Microsserviços via HTTP | Orientado a eventos com broker |
|---|:-:|:-:|:-:|:-:|:-:|
| Latência ultrabaixa | 30 | 5 | 4 | 1 | 2 |
| Sincronização de estado e anti-trapaça | 25 | 5 | 2 | 2 | 3 |
| Escalabilidade por partida | 15 | 4 | 5 | 4 | 4 |
| Simplicidade | 10 | 4 | 2 | 1 | 2 |
| Custo de infraestrutura | 10 | 3 | 5 | 2 | 3 |
| Resiliência a falhas | 10 | 3 | 3 | 4 | 4 |
| **Nota final (1 a 5)** | **100** | **4.35 ✔** | 3.45 | 2.10 | 2.85 |

## O que a matriz mostra

- **E-commerce:** Microsserviços com eventos vencem porque os critérios de escala seletiva, isolamento de falhas e autonomia dos times somam 65% do peso. Perdem nos critérios de simplicidade e consistência forte, e esse é o preço que a equipe aceita pagar (veja o README).
- **App bancário:** o Monolito modular vence porque a consistência forte e a segurança somam 50% do peso. Ele supera o monolito simples na facilidade de evoluir, sem abrir mão do controle transacional.
- **Jogo multiplayer:** o Cliente-servidor autoritativo vence com folga porque latência e sincronização somam 55% do peso. Microsserviços via HTTP ficam em último lugar justamente nesses critérios.
