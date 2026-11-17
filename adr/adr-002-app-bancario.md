# ADR-002 — Estilo arquitetural do App Bancário (BancoDigital Alfa)

| | |
|---|---|
| **Status** | **Aceito** |
| **Data** | 16/11/2026 |
| **Decisor** | Autor do estudo de caso (arquitetura de software) |
| **Atributo de qualidade principal (ISO 25010)** | **Segurança** (integridade dos dados e responsabilização/auditoria) |

## Contexto

O BancoDigital Alfa é um banco digital com conta corrente, extrato, transferências, Pix e cartões. O sistema movimenta **dinheiro real** e está sujeito a **exigências regulatórias** e de proteção de dados (como a LGPD).

**Problema de negócio:**
- **Errar um saldo custa mais do que ser lento:** multas, perda de confiança dos clientes e risco regulatório.
- Fraudes e vazamentos são danos graves e muito difíceis de reverter na imagem da marca.
- O Pix funciona 24 horas por dia, então o sistema precisa se recuperar rápido de falhas.
- A equipe é de porte médio, com perfil mais conservador. Operar muitas peças em rede aumentaria o custo e o risco.

**Requisitos que guiam a decisão** (detalhados no [README](../README.md)): segurança extrema, consistência forte (ACID), auditoria completa (prioridade 1), disponibilidade 24x7 e desempenho previsível (prioridade 2).

**Premissa:** o escopo é o de um banco digital de porte médio, e não o de um grande banco com milhares de engenheiros. Em outro contexto, a conclusão poderia ser diferente.

## Decisão

Adotar um **Monolito Modular**, organizado em camadas e com **um único banco de dados relacional**. Detalhes:

- **Módulos com fronteiras claras:** contas, transferências/Pix, cartões, extrato e antifraude. Cada módulo só conversa com os outros por interfaces definidas, sem acessar os dados internos uns dos outros.
- **Transações ACID** no banco único: uma transferência (débito + crédito) é uma única transação, que acontece completa ou não acontece.
- **Trilha de auditoria somente de inclusão (append-only):** o registro de quem fez o quê e quando é gravado **na mesma transação** da operação, então nunca existe operação sem registro.
- **Várias instâncias da aplicação** (sem guardar estado em memória) atrás de um balanceador, e **banco com failover automático** em mais de uma zona de disponibilidade.
- **Réplicas de leitura** para extrato e consultas pesadas, deixando o banco principal livre para as escritas.
- Um único ponto de entrada protegido, com autenticação forte e controle de acesso rígido.

## Alternativas consideradas

| Alternativa | Por que foi descartada |
|---|---|
| **Microsserviços** | Resolvem escala horizontal extrema, que não é a maior dor deste banco. Em troca, exigem transações distribuídas (mais difíceis de garantir), ampliam a superfície de ataque e aumentam o custo operacional. |
| **Orientado a eventos puro (event sourcing)** | Excelente para auditoria, mas muito complexo para o porte da equipe. Aproveitamos a ideia de forma simples, com a trilha de auditoria somente de inclusão. |
| **Monolito sem módulos** | É simples no início, mas tende a virar uma "bola de lama" difícil de manter e de evoluir. |

## Consequências

### Positivas (o que ganhamos)

- **Consistência forte com simplicidade:** o saldo nunca fica incorreto, e o controle transacional é um recurso nativo do banco, e não algo que precisamos construir.
- **Menor superfície de ataque:** menos serviços conversando pela rede significa menos pontos para proteger e monitorar.
- **Auditoria confiável:** registro e operação na mesma transação, o que facilita a prestação de contas ao regulador.
- **Menor custo e risco operacional:** uma peça para implantar e monitorar, ideal para uma equipe de porte médio.
- **Evolução com segurança:** as fronteiras entre módulos permitem refatorar e, se necessário, extrair um módulo no futuro.

### Negativas (dívidas técnicas e desafios que assumimos)

- **Escala principalmente vertical:** o banco único tem um teto. Um crescimento muito acima do previsto exigirá particionar os dados ou extrair módulos.
- **Publicação do sistema inteiro:** cada nova versão mexe em tudo, o que aumenta o risco e a necessidade de janelas controladas.
- **Efeito dominó em falhas:** um defeito grave em um módulo pode afetar o processo inteiro.
- **Risco de erosão da modularidade:** com o tempo, atalhos entre módulos podem transformar a organização em bagunça.
- **Tecnologia única:** todos os módulos usam a mesma base tecnológica.
- **Ponto único de falha no banco:** exige investimento em failover e em backups testados.

### Mitigações previstas

- **Testes automatizados fortes**, implantações graduais e janelas de publicação controladas.
- **Verificações automáticas** de dependências entre módulos, para barrar atalhos proibidos.
- **Limites de recursos por módulo**, para que um módulo com problema não consuma tudo.
- **Banco com failover automático**, múltiplas zonas de disponibilidade e **simulações regulares de recuperação** (RTO/RPO).
- **Plano de extração** já desenhado para os módulos mais prováveis de precisar de escala própria (Pix e notificações).

## Gatilhos de revisão

Reavaliar esta decisão se: (a) um módulo (por exemplo, o gateway do Pix) precisar de escala ou ritmo de entrega muito diferentes dos demais; (b) o banco único se aproximar do limite de capacidade; (c) o número de times crescer a ponto de a publicação conjunta virar um gargalo. Nesses casos, o módulo é **extraído como serviço**, sem refazer o resto.

## Analogia para a diretoria

Imagine uma **cozinha única, bem organizada em estações** (grelha, saladas, sobremesas), com um chefe que enxerga tudo. Os pedidos saem corretos e rastreáveis, porque tudo acontece no mesmo lugar. A cozinha não cresce infinitamente, mas, se uma estação (o Pix, por exemplo) ficar muito grande, ela pode ganhar um espaço próprio sem refazer o restaurante inteiro.
