# Domains (Bounded Contexts)

Cada domain é um lugar para plantar problemas. Eles começam como **módulos/pacotes dentro
do monólito** e viram **microserviços só quando uma dor força a separação** — nunca antes.

Os domains estão ordenados grosso modo ao longo da jornada pós-compra.

## 1. Catalog (fino)
Products, SKUs, dimensões, peso. Mantido mínimo — existe para alimentar os orders e para
dar ao Packing as dimensões de item de que ele precisa.
- *Dores naturais:* milhões de SKUs, search lenta, full-text, caching.

## 2. Storefront / Cart (fino)
Só o suficiente para montar um cart e iniciar um checkout. Não faça gold-plating.
- *Dores naturais:* quase nenhuma por design; um lugar de onde gerar carga de orders.

## 3. Orders
Checkout, ciclo de vida do order (created → paid → fulfilling → dispatched → delivered).
O coordenador da saga da jornada.
- *Dores naturais:* transactions, consistência, sagas, idempotency, state machines.

## 4. Payments (simulado)
Authorize, capture, refund — tudo falso para você poder injetar falhas e latência.
- *Dores naturais:* comunicação assíncrona, resiliência, circuit breaker, timeouts,
  retries.

## 5. Inventory / Stock
Reservar e decrementar stock entre warehouses.
- *Dores naturais:* concorrência, race conditions, locks, overselling, TTLs de
  reservation.

## 6. Fulfillment Planning
Decidir **qual warehouse** faz o fulfillment de um order e se o stock pode ser reservado
lá.
- *Dores naturais:* lógica de routing, consistência multi-warehouse, split shipments.

## 7. Warehouse / Picking
Gerar picking tasks, modelar shelves/bins/locations, rastrear a retirada de itens.
- *Dores naturais:* atribuição de tasks, throughput, ordenação de operações.

## 8. Packing  ⭐ (centro de gravidade)
Escolher a box, registrar peso & dimensões, gerar labels, confirmar o parcel.
- *Dores naturais:*
  - **Box selection** — bin-packing / otimização: encaixar itens na menor box viável.
  - **Label generation** — idempotency (nunca imprimir um label duas vezes), integração.
  - **Throughput** — milhares de packs/hora; onde é que engasga?
  - **Consistência** — o conteúdo do packing precisa bater com o conteúdo do picking e com
    o order.
  - **Modelagem de station** — muitas packing stations trabalhando em paralelo
    (concorrência).

## 9. Shipping / Last-mile
Hand-off para o carrier, route/tracking, eventos de entrega (out-for-delivery, delivered,
failed).
- *Dores naturais:* APIs de carrier externas e lentas, latência, caching de quotes,
  streams de eventos.

## 10. Notifications
"Seu pedido foi empacotado", "saiu para entrega". Reage a eventos de outros domains.
- *Dores naturais:* messaging, event-driven, retries, dead-letter queues.

## 11. Identity / Users (fino)
Signup, auth, sessions. Uma entrada suave na nova stack Java antes dos domains mais
difíceis.
- *Dores naturais:* auth, sessions; mantenha simples.

## 12. Observability (cross-cutting)
Não é um domain de negócio — é o sistema nervoso. Logs, metrics, distributed traces.
- *Dores naturais:* rastrear um request através de muitos serviços, correlacionar
  latência, achar o gargalo verdadeiro.

---

## Como domains viram serviços

Não faça pre-split. O gatilho para extrair um domain para o seu próprio serviço é sempre
uma **dor**: ele precisa escalar de forma independente, ou um acoplamento síncrono não para
de derrubar o sistema, ou os dados dele passam do tamanho do banco compartilhado. Quando
isso acontece, a separação vira a sua própria sessão de estudo — veja o backlog.
