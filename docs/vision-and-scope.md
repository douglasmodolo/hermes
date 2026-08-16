# Visão & Escopo

## Visão em uma linha

Um marketplace centrado em fulfillment cujo assunto real é **o que acontece com um pedido
depois que ele é pago** — dentro do armazém e na rua — e não a vitrine.

## A jornada pós-compra (a espinha dorsal do Hermes)

É este o fluxo em torno do qual tudo gira. Cada estágio é uma fonte de dor de sistemas
distribuídos.

```
  [ Storefront ]        fino — só o suficiente para criar pedidos
        |
        v
  [ Checkout / Order created ]
        |
        v
  [ Payment authorized ]        assíncrono, pode falhar, precisa de resiliência
        |
        v
  [ Fulfillment planning ]      qual armazém? o stock está reservado?
        |
        v
  [ Picking ]                   puxar os itens das prateleiras
        |
        v
  [ Packing ]                   escolher a box, peso, dimensões, label
        |          
        v
  [ Dispatch ]                  entregar ao carrier
        |
        v
  [ Shipping / Last-mile ]      rotas, tracking, eventos de entrega
        |
        v
  [ Delivered / Returned ]
```

## Por que "packing" é o centro de gravidade

Packing é o momento em que um pedido deixa de ser uma linha de banco de dados e vira um
parcel físico. É enganosamente rico em problemas de engenharia:

- Escolher a box certa para um conjunto de itens (um problema de otimização de verdade).
- Registrar peso e dimensões (alimenta o custo de shipping e as regras do carrier).
- Gerar e imprimir labels (integração, idempotency — você não pode imprimir duas vezes).
- Throughput sob carga (um armazém faz o packing de milhares de pedidos por hora).
- Consistência com stock e orders (o que foi feito o picking precisa bater com o que foi
  feito o packing).

## Dentro do escopo

- Toda a jornada pós-compra acima, de ponta a ponta.
- Preocupações operacionais: throughput, concorrência, resiliência, latência,
  observabilidade.
- Fulfillment multi-warehouse (introduzido como uma dor, não no primeiro dia).
- Integração com carrier simulada com serviços externos falsos/lentos que você controla.

## Fora do escopo (mantido fino de propósito)

- Vitrine rica / recomendações / ads — o catálogo e o cart existem apenas para produzir
  pedidos. Não faça gold-plating neles.
- Provedores de pagamento reais — os payments são simulados para você poder injetar
  falhas à vontade.
- Dinheiro real, PII real, carriers reais — tudo é dado falso que você gera.

## Não-objetivos

- Estar "terminado". Hermes é um laboratório permanente.
- Parecer impressionante num currículo listando tecnologias. A régua do progresso é o
  `progress-journal.md`, não uma lista de stack.

## A medida de sucesso

Daqui a três anos, olhar o git history do Hermes deve contar a história da sua evolução:
as migrações que você sobreviveu, os outages que você causou e consertou, os gargalos que
você caçou. Esse histórico — e não uma contagem de certificados — é o portfólio.
