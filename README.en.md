# Hermes

[Português (pt-BR)](README.md) · **English**

> An "impossible project": a from-scratch clone of a fulfillment-centric marketplace,
> built to learn distributed systems by feeling the pain, not reading about it.

Hermes — messenger of the gods, patron of roads and commerce — covers the **entire
post-purchase journey of an order**: checkout → warehouse (picking, packing, dispatch) →
delivery / last-mile. The storefront (catalog, cart) exists only to feed orders into the
part that matters: **fulfillment and delivery operations**.

This is a personal learning laboratory. It is never "done". It grows one pain at a time.
What drives it is curiosity about how logistics and fulfillment systems work under the
hood, and the wish to go deep on distributed systems in practice.

## Why this project exists

Two problems kill most self-study:

1. **Mental obesity** — hoarding knowledge with no purpose. It never sticks, never gets
   practiced. The brain discards what has no utility, context, emotion, repetition,
   problem-solving or survival attached to it.
2. **No practice** — studying right but never applying it. You forget it just the same.

Hermes fixes both. Every topic you study must (a) answer a real *need*, and (b) be
applied here, against a pain you deliberately manufacture.

## How to work on Hermes (non-negotiable rules)

- **You build everything, from scratch.** These docs are a map, not a solution. No code
  is written for you unless you explicitly ask.
- **Code, comments, commits, and domain language in English.** The docs and specs are
  written in **Portuguese (pt-BR)** — but domain names and technical terms (Packing,
  Picking, SKU, saga, throughput, ...) stay in English so they remain consistent with the
  code.
- **Spec-Driven Development.** Before writing code for any feature or pain, you write a
  spec first (see `docs/spec-driven-development.md`). The spec is where you materialize
  the *need* before the *solution*. No spec, no code.
- **One pain at a time.** Pick a pain from `docs/pain-backlog.md`, write its spec,
  manufacture the pain, feel it, then solve it. Log it in `docs/progress-journal.md`.
- **Let the architecture emerge.** Hermes starts as a **monolith** with **MySQL**. It
  does not start distributed. Microservices, new databases, message brokers — they enter
  only when a pain demands them. Deliberately starting on MySQL means you will later feel
  the pain of migrating to Postgres, which is one of the most valuable lessons there is.

## Target stack

- **Language / framework:** Java + Spring Boot.
- **Initial database:** MySQL (on purpose — the migration pain is a feature, not a bug).
- **Everything else** (Redis, Kafka/RabbitMQ, Postgres, Elasticsearch, containers, load
  generators, tracing) enters **only when a pain requires it**. Nothing is added "because
  it's important".

## The documents

> The docs live in `docs/` and are written in Portuguese (pt-BR), with domain terms in
> English.

| File | What it is |
|---|---|
| `docs/vision-and-scope.md` | What Hermes is, the post-purchase journey, boundaries. |
| `docs/domains.md` | The bounded contexts (Catalog, Orders, Warehouse, Packing, Shipping, ...). Where each pain lives. |
| `docs/pain-backlog.md` | **The heart.** The ordered list of pains to manufacture and beat, each an implementation idea. |
| `docs/spec-driven-development.md` | How to do SDD properly — the spec template and workflow you follow before coding. |
| `docs/progress-journal.md` | The ruler of your evolution. One entry per session. |
| `docs/glossary.md` | Fulfillment/logistics terms in English, so the domain language stays consistent. |

## Suggested first steps

1. `git init` this folder, first commit the docs.
2. Read `docs/vision-and-scope.md` and `docs/domains.md` to hold the whole picture.
3. Open `docs/pain-backlog.md`, pick **Pain #1**, and read `docs/spec-driven-development.md`.
4. Write the spec for Pain #1. Bring it for review before you write code.
