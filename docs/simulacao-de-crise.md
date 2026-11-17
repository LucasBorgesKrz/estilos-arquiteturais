# Simulação de crise — "E se este sistema crescer 10x em 1 mês?"

Pensar no pior cenário mostra até onde cada arquitetura aguenta e onde ela vai doer. Para cada sistema usamos o cenário de crise mais realista para ele. Para o crescimento, a regra é a mesma: **o que não guarda estado escala em minutos, mas o que guarda dados (bancos) precisa de preparo de semanas**. Por isso o "1 mês" é curto e pede ação imediata.

> As respostas descrevem a arquitetura do grupo em um cenário hipotético. Os números do enunciado (10x em 1 mês) são o cenário de teste, e não medições reais.

## 1. E-commerce — Black Friday permanente (10x de usuários)

**Estilo:** Microsserviços com eventos ([ADR-001](../adr/adr-001-ecommerce.md))

| | O que acontece |
|---|---|
| ✅ **Aguenta bem** | • **Busca, catálogo e carrinho** escalam sozinhos, adicionando instâncias só desses serviços. <br>• **Cache e CDN** absorvem a maior parte da leitura, que é a mais pesada. <br>• **Filas e eventos** amortecem pedidos e notificações: o pedido é aceito e processado conforme a capacidade. <br>• Uma falha em um serviço (por exemplo, recomendações) **fica isolada**. |
| ⚠️ **Aperta** | • **Banco de pedidos:** é onde 10x de escrita dói primeiro, e bancos não escalam em minutos. <br>• **Pagamentos:** depende de um parceiro externo, que tem seus próprios limites de volume. <br>• **Custo de nuvem** cresce junto com a escala. <br>• **Monitoramento e plantão:** 10x de tráfego em dezenas de serviços exige observabilidade madura. |
| 🛠️ **O que fazemos agora** | • **Testes de carga** com 10x o tráfego atual, antes do pico. <br>• **Particionar** o banco de pedidos (por cliente ou região) e usar **réplicas de leitura**. <br>• Combinar **limites de volume com o parceiro de pagamento**. <br>• **Degradação planejada:** desligar recomendações e recursos secundários sob pressão. <br>• **Fila de espera virtual** no checkout para proteger o pagamento. <br>• Acompanhar o custo por pedido (para que o crescimento não corroa a margem). |

**Veredito:** a arquitetura **foi feita para este cenário** e aguenta, **desde que** os pontos que guardam dados (banco de pedidos, parceiro de pagamento) sejam preparados com antecedência. É a hora de cobrar o preço da complexidade que aceitamos no ADR.

## 2. App bancário — ataque hacker e pico de uso (10x de transações)

**Estilo:** Monolito Modular ([ADR-002](../adr/adr-002-app-bancario.md))

| | O que acontece |
|---|---|
| ✅ **Aguenta bem** | • **A camada de aplicação** (sem guardar estado) escala **horizontalmente**: basta adicionar instâncias atrás do balanceador. <br>• **Extrato e consultas** vão para as **réplicas de leitura**, aliviando o banco principal. <br>• **Consistência e auditoria** continuam corretas, porque dependem do banco transacional, e não do número de instâncias. <br>• Em um **ataque hacker**, a superfície de ataque é menor (um ponto de entrada), mais fácil de proteger e de investigar. |
| ⚠️ **Aperta** | • **Banco principal único:** as escritas (transferências, Pix) têm um teto. Escalar na vertical ajuda, mas não infinitamente. <br>• **O módulo do Pix** pode competir por recursos com os demais. <br>• **Publicar versões** em pleno pico é arriscado, porque cada release mexe no sistema todo. <br>• Um **ataque de negação de serviço** pode saturar a entrada. |
| 🛠️ **O que fazemos agora** | • **Congelar publicações** de risco durante o pico e usar implantação gradual. <br>• Aumentar o banco na vertical e **planejar o particionamento por conta**. <br>• **Extrair o módulo do Pix** como serviço próprio (o plano de extração já existe, conforme o ADR). <br>• **Proteção na borda** (WAF, limites de taxa, mitigação de DDoS) e **bloqueio automático** de comportamento suspeito. <br>• Simular a recuperação (**RTO/RPO**) com o volume 10x. |

**Veredito:** aguenta **até certo ponto**. 10x em 1 mês estica o monolito, e o banco único é o limite. A arquitetura foi escolhida pensando em **correção primeiro**, e o plano para este cenário é **extrair o módulo mais pressionado**, e não reescrever o sistema. É exatamente o gatilho de revisão registrado no ADR.

## 3. Jogo multiplayer — lançamento com 10x de jogadores

**Estilo:** Cliente-servidor autoritativo ([ADR-003](../adr/adr-003-jogo-multiplayer.md))

| | O que acontece |
|---|---|
| ✅ **Aguenta bem** | • **Cada partida é independente:** 10x de jogadores significa 10x de servidores de jogo, e a capacidade cresce **quase linearmente**. <br>• **A latência de cada partida não piora** com mais partidas, porque o jogador conversa só com o servidor da sua partida. <br>• O **orquestrador** liga novos servidores automaticamente. |
| ⚠️ **Aperta** | • **Tempo para ligar servidores:** no minuto do lançamento, a demanda chega antes dos servidores. <br>• **Autenticação, matchmaking e banco de jogadores** viram gargalos, porque todos entram ao mesmo tempo. <br>• **Limite e custo de máquinas** na nuvem em muitas regiões. <br>• **Ataques de negação de serviço** aproveitam a visibilidade do lançamento. |
| 🛠️ **O que fazemos agora** | • **Reserva de servidores pré-aquecidos** antes do lançamento e escala preventiva por horário. <br>• **Fila de login** ("você é o número X") para proteger a autenticação. <br>• **Cache** de contas e **réplicas** do banco de jogadores. <br>• **Reservar cota de máquinas** antecipadamente nas regiões principais. <br>• **Proteção contra DDoS** e limites de taxa por jogador. |

**Veredito:** é o sistema que **melhor aguenta** o crescimento, porque o servidor de jogo escala por partida. O risco está nos **serviços de entrada** (login e matchmaking) e no **minuto exato do lançamento**, e ambos se resolvem com preparo, e não com mudança de arquitetura.

## Comparativo

| Sistema | Aguenta 10x em 1 mês? | Principal gargalo | Resposta |
|---|---|---|---|
| E-commerce | **Sim**, com preparo do banco e do parceiro | Banco de pedidos e pagamento externo | Particionar, testar carga, combinar limites |
| App bancário | **Até certo ponto** | Banco único de escrita | Extrair o módulo do Pix e particionar |
| Jogo multiplayer | **Sim**, com servidores pré-aquecidos | Login e matchmaking no lançamento | Fila de login e reserva de servidores |
