# Roteiro de fala — apresentação oral (~10 minutos)

Apoio para a apresentação. **Não leia os slides**: eles têm pouco texto de propósito. Use o roteiro como lembrete do que falar e foque em **defender por que a escolha faz sentido para o negócio**.

| Slide | Tempo | O que dizer |
|---|---|---|
| **1. Capa** | 0:30 | Apresentar o desafio: escolher o estilo arquitetural de três sistemas bem diferentes. Adiantar a ideia central: **cada sistema pede um estilo diferente**. |
| **2. Método** | 1:00 | Explicar que não escolhemos por moda. Primeiro olhamos os **requisitos críticos**, depois comparamos estilos e, por fim, declaramos o **preço** de cada escolha. A matriz de decisão com os pesos está no repositório. |
| **3. E-commerce: requisitos** | 1:00 | O MercadoGrande tem muitos times e picos como a Black Friday. O ponto-chave: **não precisa escalar tudo, só o que está sob pressão**. Citar o exemplo das recomendações que podem cair sem derrubar a compra. |
| **4. E-commerce: decisão** | 1:30 | Microsserviços com eventos. Falar do **negócio**: times publicam sozinhos, custo menor no pico, falha isolada. **Assumir o preço**: complexidade operacional e consistência eventual (Saga no pedido). Frisar a condição: só vale com equipe madura. Se a empresa fosse pequena, começaríamos por monolito modular. |
| **5. Banco: requisitos** | 0:45 | O BancoDigital Alfa mexe com dinheiro real. Explicar ACID com o exemplo de transferência: débito e crédito acontecem **juntos ou não acontecem**. Mencionar segurança e auditoria. |
| **6. Banco: decisão** | 1:30 | Monolito modular. Mostrar que a escolha **vai contra a "moda"** de propósito: aqui, consistência e segurança valem mais do que escala extrema. "Modular" significa organizado, com fronteiras claras, e deixa a porta aberta para extrair um módulo depois. Preço: escala mais vertical, publicação mais cuidadosa. |
| **7. Jogo: requisitos** | 0:45 | Arena Online: partidas curtas e competitivas. Latência de dezenas de milissegundos já é sentida pelo jogador. Todos precisam ver o mesmo mundo, e o jogador não pode ser o juiz do próprio jogo. |
| **8. Jogo: decisão** | 1:15 | Cliente-servidor com servidor autoritativo. Explicar o "juiz": o servidor valida e distribui o estado, o que dá sincronização e impede trapaça. Responder antes que perguntem: **por que não microsserviços?** Porque cada chamada HTTP entre serviços soma atraso. Só usamos serviços separados fora do tempo real (login, ranking). |
| **9. Diagrama C4 (bônus)** | 1:00 | Seguir o caminho: o jogador entra no cliente, faz login, pede partida ao matchmaking, que pede um servidor ao orquestrador. Depois o cliente conversa **direto** com o servidor de jogo pela seta laranja. No fim da partida, o resultado vai para o ranking. |
| **10. Resumo** | 0:45 | Passar a tabela pelas colunas "o que priorizamos" e "o que sacrificamos". Mensagem: nenhum estilo ganhou em tudo. |
| **11. Bala de prata** | 1:00 | Fechar a reflexão: todo estilo é um trade-off. Equipe, orçamento e tempo de mercado pesam tanto quanto a técnica. Terminar com a frase do slide: a pergunta certa é **"qual a melhor arquitetura para este problema, agora?"** |
| **12. Fechamento** | 0:15 | Agradecer, abrir para perguntas e lembrar que tudo está no repositório. |

## Perguntas que o professor pode fazer (preparem as respostas)

- **"Por que não usar microsserviços no banco, que é o que os bancos grandes fazem?"** Bancos grandes têm equipes enormes e escala extrema, que é um contexto diferente. Para o nosso banco fictício, de equipe média, a consistência forte e a simplicidade de auditoria pesam mais. E o monolito modular permite extrair módulos quando for necessário.
- **"E se o e-commerce for uma startup?"** Começaríamos por um monolito modular e extrairíamos serviços conforme a dor aparecesse. A decisão depende do momento da empresa.
- **"Por que não P2P no jogo, se sai mais barato?"** Sem um servidor como juiz, é fácil trapacear e difícil sincronizar o estado. Para um jogo competitivo, a confiança no resultado vale o custo dos servidores.
- **"O que é uma Saga?"** Uma sequência de passos em que cada passo tem uma ação de desfazer. Por exemplo: se o pagamento foi feito, mas o estoque acabou, estorna-se o pagamento.
- **"O que é consistência eventual?"** O sistema pode mostrar dados levemente desatualizados por um instante, mas todos acabam concordando. Serve para o que não é crítico, como o estoque exibido, mas não para o saldo bancário.
