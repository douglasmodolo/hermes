# Glossary — fulfillment & logística (linguagem de domínio em inglês)

Mantenha a linguagem de domínio consistente e em inglês no código, nas specs e nos commits.
Os termos (headwords) ficam em inglês; as definições estão em português.

- **Fulfillment** — tudo o que acontece para transformar um order pago num parcel entregue.
- **Picking** — puxar os itens pedidos das shelves/bins do warehouse.
- **Packing** — colocar os itens do picking numa box, pesar, medir, gerar o label. ⭐
- **Parcel** — o pacote físico produzido pelo packing.
- **Dispatch** — entregar o parcel a um carrier para transporte.
- **Last-mile** — a perna final da entrega até o endereço do cliente.
- **Carrier** — a empresa que transporta o parcel (simulada aqui).
- **SKU** (Stock Keeping Unit) — um identificador único de uma variação de produto vendável.
- **Bin / Location** — um slot físico no warehouse onde o stock fica.
- **Reservation** — segurar o stock de um order para que ele não seja vendido duas vezes.
- **Overselling** — vender mais unidades do que existem (um bug de concorrência a
  reproduzir).
- **Split shipment** — um order atendido a partir de múltiplos warehouses / em múltiplos
  parcels.
- **Reconciliation** — checar se sistemas diferentes concordam sobre o estado do mesmo
  order.
- **Idempotency** — uma operação que você pode repetir com segurança sem efeito extra (ex.:
  não imprimir dois labels para um parcel).
- **Outbox pattern** — publicar um evento de forma confiável como parte de uma transação de
  DB.
- **Saga** — uma sequência de passos entre serviços com ações compensatórias em caso de
  falha.
- **Backpressure** — reduzir a entrada quando um estágio downstream não consegue
  acompanhar.
- **Throughput** — unidades processadas por unidade de tempo (ex.: packs/hora).
- **P99 latency** — a latência que o 1% mais lento dos requests sofre (a cauda).
- **Replication lag** — o atraso entre um DB primário e sua read replica.
- **Cutover** — o momento em que você troca o tráfego do sistema/DB antigo para o novo
  (ex.: MySQL→Postgres).
- **Circuit breaker** — corta chamadas a uma dependência que está falhando, para não
  propagar a falha nem insistir no que já caiu.
- **Exponential backoff** — retentar com esperas crescentes entre tentativas, para não
  formar um retry storm que amplifica o incêndio.
- **Fallback** — resposta degradada quando a dependência falha (ex.: "frete indisponível" +
  valor padrão), em vez de um erro duro que trava o fluxo.
- **Rate limiting / Throttling** — limitar requests por usuário/IP por unidade de tempo,
  para proteger o downstream de um spike.
- **Read replica** — cópia somente-leitura do DB primário para onde se roteiam leituras;
  paga-se com replication lag.
- **Canary deployment** — subir a versão nova para uma fração pequena do tráfego (1–5%) e
  só avançar se as métricas de saúde seguirem boas.
- **Blue-green deployment** — manter o ambiente antigo (blue) de pé enquanto o novo (green)
  sobe; o load balancer volta ao blue na hora se o green falhar.
- **CAP theorem** — sob partição de rede, você escolhe entre consistência e disponibilidade.
- **PACELC** — extensão do CAP: mesmo sem partição (Else), há troca entre latência e
  consistência.
- **N+1 problem** — bug de ORM em que carregar N registros dispara 1+N queries por causa de
  lazy fetch.
- **Virtual Threads** — threads baratíssimas do Java 21 (Project Loom) que podem bloquear
  sem o custo das threads de plataforma.
- **SLI / SLO** — Service Level Indicator (métrica medida, ex.: % de requests não-5xx) e
  Service Level Objective (a meta sobre ela, ex.: 99.9%).

Adicione termos aqui à medida que o domínio cresce, para o naming ficar consistente.
