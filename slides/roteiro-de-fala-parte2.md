# Roteiro de fala — Parte 2: defesa para a "diretoria" (~10 minutos)

Apoio para a apresentação. Você **não está convencendo um computador, e sim pessoas**: o professor e a turma fazem o papel da diretoria, que se importa com **dinheiro, prazo e risco**. Fale nessa linguagem, use as analogias e **não leia os slides**.

| Slide | Tempo | O que dizer |
|---|---|---|
| **1. Capa** | 0:30 | "Hoje vamos pedir a aprovação de três decisões de arquitetura. Para cada uma, vamos mostrar o que ganhamos **e o que pagamos**." |
| **2. Quatro imagens** | 1:15 | Contar as analogias com calma, é o momento de ganhar a plateia. **Food truck**: barato e simples, mas não dá para duplicar só a chapa. **Praça de alimentação**: cada cozinha cresce sozinha, mas custa mais para administrar (e-commerce). **Cozinha única em estações**: tudo num lugar, tudo rastreável (banco). **Árbitro**: quem decide o que valeu (jogo). |
| **3. Requisitos priorizados** | 1:00 | O que mais importa em cada um: pico no e-commerce, segurança e consistência no banco, latência no jogo. Mencionar o atributo da ISO 25010 de cada um sem entrar em detalhe. Dizer que cada requisito tem prioridade e meta mensurável no README. |
| **4. ADR-001: E-commerce** | 1:30 | Explicar rapidamente o que é um **ADR**: o registro da decisão (contexto, decisão, status, consequências). Defender a decisão pelo **negócio**: paga só pelo pico, times lançam mais rápido. **Falar do preço antes de ser perguntado**: complexidade de monitoramento e de rede, e a exigência de uma equipe madura. |
| **5. ADR-002: Banco** | 1:15 | Mostrar que a escolha **vai contra a moda** de propósito. Argumento de dinheiro: errar saldo custa mais do que qualquer economia. Preço: o banco único tem teto, e cada versão mexe em tudo. Dizer que há um plano para extrair o módulo do Pix quando for preciso. |
| **6. ADR-003: Jogo** | 1:15 | O servidor é o árbitro. Argumento de negócio: jogador que se diverte continua pagando, e trapaça afasta quem joga limpo. Preço: servidores dedicados e várias regiões. |
| **7. Diagrama C4** | 1:00 | Seguir o caminho: o jogador entra no cliente, faz login, pede partida, e o cliente conversa **direto** com o servidor de jogo pela seta laranja. Explicar que a decisão do ADR está **visível** no desenho. |
| **8. Crise: e-commerce** | 1:15 | "O que acontece se crescermos 10x em 1 mês?" Aguenta: busca e carrinho escalam sozinhos, cache e filas amortecem. Aperta: banco de pedidos e parceiro de pagamento. O que faremos agora: teste de carga, particionar o banco, combinar limites. |
| **9. Crise: banco e jogo** | 0:45 | Banco: aguenta até certo ponto, e o plano é extrair o Pix. Jogo: aguenta bem, mas o minuto do lançamento exige servidores pré-aquecidos e fila de login. Frase final: **o que não guarda estado escala em minutos, o que guarda dados precisa de semanas.** |
| **10. Dinheiro, prazo e risco** | 0:45 | Ler a tabela em voz alta como se fosse para um diretor: o que ele ganha e o que ele paga em cada sistema. |
| **11. O ADR importa** | 0:45 | O diagrama mostra o quê, o ADR guarda o porquê. Fechar com a imagem da planta e do registro de obras. |
| **12. Fechamento** | 0:20 | "Pedimos a aprovação das três decisões." Abrir para perguntas. |

## Perguntas que a "diretoria" pode fazer (preparem as respostas)

- **"Microsserviços não são mais caros? Por que não o monolito?"** São mais caros de operar, e dissemos isso na apresentação. Mas, com muitos times e picos como a Black Friday, o custo de **não** poder escalar só o que precisa e de todos dependerem do mesmo lançamento é maior. Para uma empresa pequena, a nossa recomendação é outra: começar pelo monolito modular.
- **"Por que o banco não usa microsserviços, como os grandes bancos?"** Grandes bancos têm equipes enormes e escala extrema. No nosso contexto (porte médio, regulação, dinheiro real), consistência forte e simplicidade de auditoria pesam mais. O monolito modular deixa a porta aberta para extrair um módulo depois.
- **"O que acontece se o banco de dados único cair?"** Por isso há failover automático em mais de uma zona e simulações regulares de recuperação. O banco único é um risco assumido e registrado no ADR.
- **"Por que não P2P no jogo, se sai mais barato?"** Sem um juiz central, é fácil trapacear e difícil sincronizar o estado. Para um jogo competitivo, a confiança no resultado vale o custo dos servidores.
- **"Qual é o custo real disso?"** Os números são hipotéticos neste estudo de caso, mas a lógica é a que mostramos: o e-commerce paga mais caro pela plataforma e economiza no pico, o banco paga em crescimento mais lento e evita risco de erro, e o jogo paga por servidores e retém jogadores.
- **"E se a equipe não estiver preparada para microsserviços?"** Então a decisão perde o sentido, e esse é um gatilho de revisão do ADR-001. Começaríamos por um monolito modular e extrairíamos serviços aos poucos.
