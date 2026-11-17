# ADR-003 — Estilo arquitetural do Jogo Multiplayer (Arena Online)

| | |
|---|---|
| **Status** | **Aceito** |
| **Data** | 16/11/2026 |
| **Decisor** | Autor do estudo de caso (arquitetura de software) |
| **Atributo de qualidade principal (ISO 25010)** | Eficiência de desempenho › **Comportamento no tempo** (latência) |

## Contexto

A Arena Online é um jogo competitivo em tempo real, com partidas curtas entre cerca de 10 jogadores. Todos precisam ver o **mesmo mundo, quase ao mesmo tempo**, e o resultado da partida precisa ser justo.

**Problema de negócio:**
- A **experiência de jogo** (fluidez e justiça) decide se o jogador continua jogando e pagando. Atrasos de dezenas de milissegundos já são sentidos.
- **Trapaça afasta jogadores honestos** e prejudica a reputação do jogo.
- O número de partidas simultâneas varia muito ao longo do dia e, principalmente, no lançamento. Pagar por servidores ociosos é desperdício.

**Requisitos que guiam a decisão** (detalhados no [README](../README.md)): latência ultrabaixa (prioridade 1), sincronização de estado em tempo real e resistência a trapaças, escalabilidade por partida.

## Decisão

Adotar o estilo **Cliente-Servidor com servidor autoritativo**, usando **WebSockets/UDP** na comunicação de tempo real. Detalhes:

- **Um servidor de jogo por partida:** é o "juiz". Recebe os comandos dos jogadores, valida, calcula o novo estado do jogo e envia o resultado a todos várias vezes por segundo.
- **O cliente não decide nada:** ele só mostra o jogo e **prevê o movimento localmente** para parecer fluido. O servidor corrige se houver divergência.
- **Conexão direta e contínua** entre o cliente e o servidor da sua partida, sem saltos intermediários.
- **Serviços de apoio separados, fora do tempo real:** autenticação, matchmaking e ranking, via HTTPS.
- **Orquestrador de servidores** que liga e desliga servidores de jogo conforme a demanda de partidas.
- O diagrama C4 desta arquitetura está em [`diagramas/jogo-multiplayer-c4.png`](../diagramas/jogo-multiplayer-c4.png).

## Alternativas consideradas

| Alternativa | Por que foi descartada |
|---|---|
| **Microsserviços para o laço do jogo** | Cada chamada entre serviços soma atraso, o que mata a latência. Usamos serviços separados **apenas** fora do tempo real. |
| **Peer-to-Peer (P2P)** | Reduz o custo de servidor, mas, sem um juiz central, é fácil trapacear, difícil sincronizar o estado e há problemas de conexão entre jogadores (como NAT). |
| **Orientado a eventos com broker** | A fila intermediária acrescenta atraso. Serve bem para o que acontece **depois** da partida (como enviar o resultado ao ranking), não durante. |

## Consequências

### Positivas (o que ganhamos)

- **Latência mínima:** caminho curto e direto entre o jogador e o servidor da partida.
- **Justiça e anti-trapaça:** como só o servidor decide, adulterar o cliente não dá vantagem.
- **Sincronização consistente:** todos os jogadores recebem o estado de uma única fonte de verdade.
- **Escala simples por partida:** cada partida é independente, então a capacidade cresce adicionando servidores.
- **Custo proporcional ao uso:** servidores sobem quando há partidas e descem quando não há.

### Negativas (dívidas técnicas e desafios que assumimos)

- **Custo de servidores dedicados:** é preciso manter uma capacidade mínima ligada, mesmo nos horários de pouco movimento.
- **Latência geográfica:** quem está longe do servidor sente atraso. Exige servidores em várias regiões, o que aumenta custo e complexidade.
- **A queda de um servidor derruba a partida dele:** o estado do jogo fica em memória.
- **Complexidade do código de rede (netcode) no cliente:** a predição de movimento e a correção de divergências são difíceis de acertar.
- **Orquestração de frota:** o orquestrador é mais um componente crítico a desenvolver e manter.
- **Alvo de ataques:** servidores de jogo expostos são alvos de ataques de negação de serviço (DDoS).

### Mitigações previstas

- **Partidas curtas** e retorno rápido ao matchmaking caso um servidor caia. Opcionalmente, guardar fotos periódicas do estado.
- **Servidores em várias regiões** e escolha do mais próximo no matchmaking.
- **Orquestrador com escala automática** e uma reserva de servidores já ligados (pré-aquecidos) para picos como o lançamento.
- **Proteção contra DDoS** na borda da rede e limites de taxa por jogador.
- Testes de qualidade da experiência com **atraso e perda de pacotes simulados**.

## Gatilhos de revisão

Reavaliar esta decisão se: (a) o jogo ganhar modos cooperativos casuais, sem competição, em que P2P ou um modelo híbrido reduziriam muito o custo; (b) o ecossistema de serviços de apoio crescer muito (loja, clãs, eventos). Nesse caso, os **serviços de apoio** podem evoluir para microsserviços, **mantendo o servidor de jogo como está**.

## Analogia para a diretoria

O servidor autoritativo é o **árbitro de uma partida de futebol**: os jogadores jogam, mas quem decide o que valeu é o árbitro, e todos aceitam o mesmo resultado. Num jogo sem árbitro (P2P), cada jogador "conta" o placar do seu jeito, e é fácil alguém trapacear. O custo é pagar o árbitro de cada partida.
