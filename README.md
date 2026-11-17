# Estudo de Caso — Parte 2: A Defesa Arquitetural

**Disciplina:** Arquitetura de Software

> Entrega 100% teórica/modelagem: este repositório não contém código. Reúne os requisitos priorizados, os registros de decisão arquitetural (ADRs), os slides da apresentação para a "diretoria", o diagrama C4 e a simulação de crise.

Na Parte 1 escolhemos o estilo arquitetural de **três sistemas fictícios**. Agora **formalizamos e defendemos** essas decisões: refinamos os requisitos não funcionais, registramos cada decisão em um ADR com seus prós **e contras**, e preparamos a defesa para quem decide dinheiro e prazo.

## Estrutura do repositório

| Caminho | Descrição |
|---|---|
| [`README.md`](README.md) | Este documento: requisitos priorizados, ADRs resumidos, apresentação e reflexão |
| [`adr/adr-001-ecommerce.md`](adr/adr-001-ecommerce.md) | ADR completo do E-commerce |
| [`adr/adr-002-app-bancario.md`](adr/adr-002-app-bancario.md) | ADR completo do App Bancário |
| [`adr/adr-003-jogo-multiplayer.md`](adr/adr-003-jogo-multiplayer.md) | ADR completo do Jogo Multiplayer |
| [`docs/simulacao-de-crise.md`](docs/simulacao-de-crise.md) | "E se o sistema crescer 10x em 1 mês?", para os três sistemas |
| [`docs/matriz-de-decisao.md`](docs/matriz-de-decisao.md) | Comparação ponderada dos estilos candidatos (da Parte 1) |
| [`docs/parte1-estudo-de-caso.md`](docs/parte1-estudo-de-caso.md) | Documento da Parte 1 (contexto, requisitos e justificativas iniciais) |
| [`slides/apresentacao-parte2.pdf`](slides/apresentacao-parte2.pdf) | Slides da apresentação da Parte 2 |
| [`slides/roteiro-de-fala-parte2.md`](slides/roteiro-de-fala-parte2.md) | Roteiro de apoio para a apresentação oral |
| [`slides/apresentacao.pdf`](slides/apresentacao.pdf) | Slides da Parte 1 |
| [`diagramas/jogo-multiplayer-c4.png`](diagramas/jogo-multiplayer-c4.png) | Diagrama C4 do jogo multiplayer (PNG; também em [SVG](diagramas/jogo-multiplayer-c4.svg)) |

## Resumo das decisões

| Sistema | Estilo definitivo | Atributo ISO 25010 mais importante | ADR |
|---|---|---|---|
| **E-commerce** | Microsserviços com eventos | Eficiência de desempenho › Capacidade (escalabilidade elástica) | [ADR-001](adr/adr-001-ecommerce.md) |
| **App bancário** | Monolito modular | Segurança (integridade e responsabilização) | [ADR-002](adr/adr-002-app-bancario.md) |
| **Jogo multiplayer** | Cliente-servidor autoritativo | Eficiência de desempenho › Comportamento no tempo (latência) | [ADR-003](adr/adr-003-jogo-multiplayer.md) |

---

## Parte 1 — Refinamento dos requisitos

### Como classificamos

- **Prioridade:**
  - **P1 (crítico):** define a arquitetura. Se falhar, o sistema falha no seu propósito.
  - **P2 (importante):** muito relevante, mas a arquitetura pode atendê-lo com mais de um caminho.
  - **P3 (desejável):** vale buscar, mas cede em caso de conflito com os de cima.
- **Característica ISO/IEC 25010:** associamos cada requisito a uma característica do modelo de qualidade de produto da norma, usando a terminologia mais difundida (a edição de 2023 atualizou alguns nomes).
- **Meta:** valores de exemplo escolhidos neste estudo para tornar o requisito **mensurável**. São premissas do estudo de caso, e não medições reais.

### 1.1 E-commerce (MercadoGrande)

| Prioridade | Requisito não funcional | Meta (exemplo) | ISO 25010 |
|---|---|---|---|
| **P1** | **Escalabilidade elástica** (picos como a Black Friday) | Absorver picos de até ~10x o tráfego médio sem intervenção manual | Eficiência de desempenho › Capacidade |
| P2 | Disponibilidade e tolerância a falhas | 99,95% no fluxo de compra; falha em recomendações não impede a compra | Confiabilidade › Disponibilidade, tolerância a falhas |
| P2 | Desempenho de leitura (busca e produto) | 95% das respostas em menos de 300 ms | Eficiência de desempenho › Comportamento no tempo |
| P2 | Autonomia de evolução dos times | Cada time publica sem depender de um lançamento conjunto | Manutenibilidade › Modularidade, modificabilidade |
| P2 | Segurança dos dados de pagamento | Dados de cartão tratados pelo parceiro de pagamento, nunca guardados por nós | Segurança › Confidencialidade |
| P3 | Consistência eventual (exceto pagamento e estoque) | Atraso de poucos segundos aceitável na vitrine; rigor no pagamento | Adequação funcional › Corretude funcional |

> **Atributo mais importante:** **Eficiência de desempenho (Capacidade)**, com a Confiabilidade logo atrás. O negócio vive de absorver picos sem cair.

### 1.2 App bancário (BancoDigital Alfa)

| Prioridade | Requisito não funcional | Meta (exemplo) | ISO 25010 |
|---|---|---|---|
| **P1** | **Segurança** (autenticação, criptografia, antifraude) | Autenticação forte (MFA); criptografia em trânsito e em repouso; controle de acesso por perfil | Segurança › Confidencialidade, autenticidade |
| **P1** | **Consistência forte (ACID)** | Zero perda ou duplicação de transações; saldo sempre correto | Segurança › Integridade |
| **P1** | **Auditoria** | 100% das operações registradas, de forma que não possa ser alterada | Segurança › Responsabilização, não repúdio |
| P2 | Disponibilidade 24x7 (Pix) | 99,99%; recuperação em minutos (RTO) e sem perda de dados (RPO = 0) | Confiabilidade › Disponibilidade, recuperabilidade |
| P2 | Desempenho previsível | 95% das transferências confirmadas em menos de 2 s | Eficiência de desempenho › Comportamento no tempo |
| P3 | Escalabilidade | Crescimento previsível e planejado | Eficiência de desempenho › Capacidade |
| P3 | Facilidade de manutenção | Módulos com fronteiras claras e testáveis | Manutenibilidade › Modularidade, testabilidade |

> **Atributo mais importante:** **Segurança**, principalmente a **integridade** (o saldo nunca errado) e a **responsabilização** (auditoria). Aqui, errar custa mais do que ser lento.

### 1.3 Jogo multiplayer (Arena Online)

| Prioridade | Requisito não funcional | Meta (exemplo) | ISO 25010 |
|---|---|---|---|
| **P1** | **Latência ultrabaixa** | Atraso de ida e volta de até ~50 ms para a região do jogador; atualização do jogo de 30 a 60 vezes por segundo | Eficiência de desempenho › Comportamento no tempo |
| P2 | Sincronização de estado em tempo real | Todos os jogadores veem o mesmo estado, com diferença imperceptível | Adequação funcional › Corretude funcional |
| P2 | Resistência a trapaças | Decisões do jogo tomadas só no servidor | Segurança › Integridade |
| P2 | Escalabilidade por partida | Subir e descer servidores conforme o número de partidas | Eficiência de desempenho › Capacidade |
| P3 | Tolerância a falhas | Queda de um servidor afeta só a partida dele | Confiabilidade › Tolerância a falhas |
| P3 | Custo de infraestrutura | Pagar proporcionalmente às partidas em andamento | Eficiência de desempenho › Utilização de recursos |

> **Atributo mais importante:** **Eficiência de desempenho (Comportamento no tempo)**. Num jogo em tempo real, a latência é o produto.

---

## Parte 2 — A defesa arquitetural (ADRs)

Um **ADR (Architecture Decision Record)** registra uma decisão importante, o contexto em que foi tomada e suas consequências. Abaixo está a versão resumida de cada um. As versões completas, com alternativas, mitigações e gatilhos de revisão, estão na pasta [`adr/`](adr/).

### ADR-001 — E-commerce: Microsserviços com eventos

- **Status:** Aceito
- **Contexto:** loja online com muitos times e picos sazonais (Black Friday). Lentidão no pico significa vendas perdidas. Cada time precisa lançar no seu ritmo, e pagar por servidores dimensionados para o pico o ano todo é caro.
- **Decisão:** Microsserviços, um por área de negócio, cada um com seu banco. Comunicação por **eventos** (message broker), com chamadas síncronas só onde a resposta é imediata (pagamento). **Saga** no fluxo pedido → pagamento → estoque. API Gateway com cache e CDN, e escala elástica por serviço.
- **Consequências:**
  - ✅ **Prós:** escalabilidade independente (só busca e carrinho crescem no pico, o que reduz custo), isolamento de falhas, deploy independente por time, filas que amortecem picos.
  - ❌ **Contras (o que assumimos):** maior **complexidade de monitoramento e de comunicação de rede**, consistência eventual (estados intermediários), Sagas e testes de ponta a ponta mais difíceis, governança de contratos entre times, custo fixo de plataforma, e exige equipe madura.
- **Quando reavaliar:** se a equipe não conseguir operar a plataforma, ou se o custo fixo superar o ganho. Para uma empresa pequena, **começar por um monolito modular**.

### ADR-002 — App bancário: Monolito modular

- **Status:** Aceito
- **Contexto:** banco digital que movimenta dinheiro real, sob exigências regulatórias e de proteção de dados. Errar um saldo custa mais do que ser lento. Pix 24x7. Equipe de porte médio.
- **Decisão:** Monolito Modular em camadas, com **um único banco relacional**. Módulos com fronteiras claras (contas, transferências/Pix, cartões, extrato, antifraude), **trilha de auditoria somente de inclusão** na mesma transação, várias instâncias atrás de balanceador, banco com failover e réplicas de leitura.
- **Consequências:**
  - ✅ **Prós:** transações ACID simples e confiáveis, menor superfície de ataque, auditoria na mesma transação, menor custo e risco operacional, evolução segura pelas fronteiras entre módulos.
  - ❌ **Contras (o que assumimos):** escala principalmente **vertical** (o banco único tem teto), publicação do sistema inteiro a cada versão, efeito dominó em falhas, risco de erosão da modularidade, tecnologia única, e o banco como ponto único de falha (exige failover).
- **Quando reavaliar:** se um módulo (como o Pix) precisar de escala ou ritmo próprios, ele é **extraído como serviço**.

### ADR-003 — Jogo multiplayer: Cliente-servidor autoritativo

- **Status:** Aceito
- **Contexto:** jogo competitivo em tempo real, partidas curtas de ~10 jogadores. A experiência (fluidez e justiça) decide a retenção. Trapaça afasta jogadores honestos. A demanda varia muito, principalmente no lançamento.
- **Decisão:** **Cliente-servidor com servidor autoritativo**: um servidor de jogo por partida, comunicação em tempo real por **WebSockets/UDP**, predição de movimento no cliente. Serviços de apoio (autenticação, matchmaking, ranking) separados e fora do tempo real, via HTTPS. Um **orquestrador** liga e desliga servidores conforme a demanda.
- **Consequências:**
  - ✅ **Prós:** latência mínima, anti-trapaça (só o servidor decide), sincronização consistente, escala simples por partida, custo proporcional ao uso.
  - ❌ **Contras (o que assumimos):** custo de servidores dedicados, latência para quem está longe (exige várias regiões), a queda de um servidor derruba a partida, **netcode complexo no cliente**, orquestrador de frota a manter, e servidores como alvo de DDoS.
- **Quando reavaliar:** modos cooperativos casuais (onde P2P ou híbrido reduziria custo) ou crescimento dos serviços de apoio (que podem virar microsserviços, mantendo o servidor de jogo).

---

## Parte 3 — Apresentação para a "diretoria"

- **Slides:** [`slides/apresentacao-parte2.pdf`](slides/apresentacao-parte2.pdf), com o [roteiro de fala](slides/roteiro-de-fala-parte2.md) como apoio. Usam **analogias visuais** (food truck × praça de alimentação, cozinha única com estações, árbitro de futebol) e traduzem cada decisão técnica em **dinheiro, prazo e risco**.
- **Diagrama C4 (nível de contêineres) do jogo multiplayer**, presente nos slides: a seta laranja é o caminho de tempo real entre o jogador e o servidor de jogo.

![Diagrama C4 do jogo multiplayer](diagramas/jogo-multiplayer-c4.png)

### Bônus — Simulação de crise: "E se este sistema crescer 10x em 1 mês?"

A análise completa está em [`docs/simulacao-de-crise.md`](docs/simulacao-de-crise.md). Resumo:

| Sistema | Cenário de crise | Aguenta? | Principal gargalo | Resposta |
|---|---|---|---|---|
| **E-commerce** | Black Friday permanente (10x de usuários) | **Sim**, com preparo prévio | Banco de pedidos e parceiro de pagamento | Testes de carga, particionar o banco, combinar limites com o parceiro, degradação planejada |
| **App bancário** | Ataque hacker e pico de transações | **Até certo ponto** | Banco único de escrita | Extrair o módulo do Pix, particionar por conta, proteção na borda |
| **Jogo multiplayer** | Lançamento com 10x de jogadores | **Sim**, com servidores pré-aquecidos | Login e matchmaking no minuto do lançamento | Fila de login, reserva de servidores e de cota de máquinas |

A lição comum: **o que não guarda estado escala em minutos, mas o que guarda dados precisa de preparo de semanas.**

---

## Parte 4 — Reflexão

**Por que a documentação de uma decisão arquitetural (ADR) é tão importante quanto o diagrama em si? Como defender uma escolha técnica para um stakeholder que só se importa com dinheiro e prazos?**

### Por que o ADR importa tanto quanto o diagrama

O diagrama mostra **o quê**: quais são as peças e como elas se conectam. Mas ele não explica **por que** o sistema é assim, e é justamente o "porquê" que se perde com o tempo. Pessoas saem da equipe, chegam outras, e daqui a um ano alguém olha o diagrama e pergunta: "por que usamos um único banco aqui? Isso não é um erro?". Sem registro, a equipe nova tende a **desfazer boas decisões** (porque não entende o motivo) ou a **repetir discussões antigas** (porque ninguém lembra que as alternativas já foram avaliadas).

O ADR guarda três coisas que o diagrama não guarda:

1. **O contexto:** em que situação a decisão foi tomada (equipe, prazo, requisitos, orçamento daquele momento).
2. **As alternativas descartadas e por quê:** evita rediscutir o que já foi pensado.
3. **As consequências, inclusive as ruins:** a "dívida" que assumimos de propósito. Com isso, quando o contra aparece (por exemplo, o banco único chegando ao limite), a equipe sabe que **não é uma surpresa nem uma falha**, e sim algo previsto, com um gatilho de revisão já definido.

Por isso, o ADR é como o **registro de obras** de uma casa, e o diagrama é só a planta. A planta mostra onde estão as paredes, mas só o registro conta por que aquela parede é mais grossa: ela sustenta o andar de cima. Quem reformar sem ler o registro pode derrubar a casa.

### Como defender uma escolha para quem só pensa em dinheiro e prazos

A regra é **traduzir o argumento técnico em impacto de negócio**. O stakeholder não precisa entender "consistência eventual" ou "latência de rede", mas entende **custo, prazo e risco**. Foi isso que fizemos na apresentação:

| Argumento técnico | Tradução para a diretoria |
|---|---|
| E-commerce: microsserviços com escala independente | "Pagamos só pelos servidores que o pico realmente usa, em vez de manter o ano todo uma estrutura gigante ociosa. E as equipes lançam sem esperar umas pelas outras, então as novidades chegam **mais rápido** ao cliente." |
| App bancário: monolito modular com banco único | "Reduzimos o risco de **erro de saldo, multa e fraude**, que custam muito mais do que qualquer economia. E operar uma peça só é **mais barato** para uma equipe do nosso tamanho." |
| Jogo: servidor autoritativo | "O jogo fica fluido e justo, e jogador que se diverte **continua jogando e pagando**. Os servidores só ficam ligados quando há partidas, então o custo acompanha a receita." |

Quatro práticas ajudam a defender a escolha com credibilidade:

- **Falar do custo da escolha antes de ser perguntado.** "Esta decisão tem um preço: mais complexidade de monitoramento. Veja como vamos pagá-lo." Quem mostra o lado ruim ganha confiança. Quem esconde perde.
- **Usar analogias.** Food truck × praça de alimentação explica em 10 segundos o que 10 minutos de jargão não explicam.
- **Comparar cenários em dinheiro e tempo:** "Se escolhermos X, entregamos em N meses e custa Y. Se escolhermos Z, é mais rápido, mas ficaremos presos a W."
- **Mostrar o pior cenário e o plano:** a simulação de "e se crescermos 10x em 1 mês?" mostra maturidade, porque o stakeholder vê que a equipe já pensou no que pode dar errado e no que fará.

No fim, **um bom arquiteto é um tradutor**: converte a complexidade da engenharia em decisões de negócio, e registra o raciocínio (no ADR) para que a decisão sobreviva às pessoas que a tomaram.
