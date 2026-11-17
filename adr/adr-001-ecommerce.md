# ADR-001 — Estilo arquitetural do E-commerce (MercadoGrande)

| | |
|---|---|
| **Status** | **Aceito** |
| **Data** | 16/11/2026 |
| **Decisores** | Grupo (arquitetura de software) |
| **Atributo de qualidade principal (ISO 25010)** | Eficiência de desempenho › **Capacidade** (escalabilidade elástica) |

## Contexto

O MercadoGrande é uma loja online de varejo, com milhões de produtos e **vários times** de desenvolvimento, cada um responsável por uma área (catálogo, busca, carrinho, pedidos, pagamentos e logística).

**Problema de negócio:**
- O tráfego é estável na maior parte do ano, mas em datas como a Black Friday ele **multiplica várias vezes**, principalmente em catálogo, busca e carrinho. Lentidão ou indisponibilidade nesses dias significam **vendas perdidas** e danos à marca.
- Cada área precisa **lançar novidades no seu ritmo**. Se todos os times dependem de um mesmo lançamento, o prazo de todos fica refém do mais lento.
- Pagar por servidores dimensionados para o pico o ano inteiro é caro.

**Requisitos que guiam a decisão** (detalhados no [README](../README.md)): escalabilidade elástica (prioridade 1), disponibilidade e tolerância a falhas, desempenho de leitura, autonomia dos times e consistência eventual aceitável (exceto em pagamento e estoque).

**Premissas:** cerca de 6 times de produto, e uma equipe que já tem (ou vai formar antes do lançamento) prática com automação de implantação, monitoramento e operação de sistemas distribuídos.

## Decisão

Adotar **Microsserviços**, com **comunicação assíncrona orientada a eventos** (via message broker) entre os serviços. Detalhes:

- **Um serviço por área de negócio:** catálogo, busca, carrinho, pedidos, pagamentos, entregas e notificações, cada um com **seu próprio banco de dados**.
- **Chamadas síncronas (REST) apenas onde a resposta é imediata e indispensável**, como a autorização do pagamento. O restante usa eventos (por exemplo, "pedido confirmado").
- **Saga** para o fluxo pedido → pagamento → estoque: uma sequência de passos em que cada passo tem uma ação de desfazer (por exemplo, estornar o pagamento se o estoque acabar).
- **API Gateway** como porta única de entrada, com **cache e CDN** para absorver a leitura de catálogo e busca.
- **Escala elástica automática** por serviço, com limites de taxa e degradação planejada (por exemplo, desligar recomendações sob pressão extrema).

## Alternativas consideradas

| Alternativa | Por que foi descartada |
|---|---|
| **Monolito simples** | Escala tudo junto (desperdício no pico) e acopla o lançamento de todos os times. Um defeito pode derrubar a loja inteira. |
| **Monolito modular** | Melhor ponto de partida para um produto novo, mas continua escalando e publicando como uma peça só. Para o porte e os picos do MercadoGrande, vira gargalo. |
| **SOA com barramento central (ESB)** | O barramento vira ponto único de falha e de gargalo organizacional. |

## Consequências

### Positivas (o que ganhamos)

- **Escalabilidade independente:** só busca, catálogo e carrinho crescem na Black Friday. Os demais ficam pequenos, o que reduz o custo do pico.
- **Isolamento de falhas:** se as recomendações caírem, o cliente continua conseguindo comprar.
- **Deploy independente:** cada time publica no seu ritmo, com menos coordenação e menos risco de "quebrar o outro time".
- **Resiliência a picos:** filas e eventos amortecem a carga em pedidos e notificações. O pedido é aceito e processado conforme a capacidade.
- **Liberdade tecnológica** por serviço, quando fizer sentido (por exemplo, um mecanismo de busca específico).

### Negativas (dívidas técnicas e desafios que assumimos)

- **Maior complexidade operacional:** muitos serviços, bancos e filas para monitorar, atualizar e proteger. Depurar um problema exige rastrear uma requisição por vários serviços.
- **Comunicação de rede:** chamadas entre serviços são mais lentas e menos confiáveis do que chamadas dentro de um mesmo programa. É preciso prever tempo limite, novas tentativas e falhas parciais.
- **Consistência eventual:** o sistema passa por estados intermediários (por exemplo, "pedido recebido, aguardando confirmação"). A experiência do cliente precisa ser desenhada para isso.
- **Sagas e testes mais difíceis:** os fluxos que atravessam serviços são mais complexos de projetar e de testar de ponta a ponta.
- **Governança de contratos:** mudar a API ou um evento pode quebrar outros serviços. Exige versionamento e acordos entre times.
- **Custo fixo de plataforma:** a infraestrutura de implantação, monitoramento e mensageria custa mesmo fora do pico.
- **Exige uma equipe madura.** Se ela não estiver preparada, a decisão perde o sentido.

### Mitigações previstas

- Monitoramento centralizado, métricas e rastreamento distribuído **desde o primeiro serviço**, e não depois.
- Testes de contrato entre serviços e versionamento de eventos.
- **Começar com poucos serviços, bem delimitados por área de negócio**, em vez de dividir demais.
- Roteiros de operação (runbooks) e treinamento da equipe de plantão.
- Telas que mostram o status do pedido de forma clara durante os estados intermediários.

## Gatilhos de revisão

Reavaliar esta decisão se: (a) a equipe encolher a ponto de não conseguir operar a plataforma; (b) o custo fixo da plataforma superar o ganho de escala; (c) incidentes causados por falhas de integração entre serviços se tornarem frequentes. Para uma empresa **menor ou em validação de produto**, a recomendação seria **começar por um monolito modular** e extrair serviços aos poucos, conforme a dor aparecer.

## Analogia para a diretoria

Um **food truck** (monolito) é ótimo para começar: barato, simples, uma pessoa resolve tudo. Mas, se a fila dobra, não dá para "duplicar só a chapa". Uma **praça de alimentação** (microsserviços) tem várias cozinhas independentes: se a da pizza lota, abre-se mais um balcão de pizza sem mexer no resto, e, se uma cozinha fecha, as outras seguem vendendo. O preço é a praça ser mais cara de administrar.
