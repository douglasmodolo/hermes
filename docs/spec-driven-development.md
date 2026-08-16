# Spec-Driven Development (SDD) no Hermes

No Hermes você **nunca escreve código antes de escrever uma spec**. A spec é onde você
transforma uma *dor* numa *need* clara e num *comportamento* definido. É a disciplina que
te impede de codar no vácuo — a mesma ideia sobre a qual o projeto inteiro é construído,
aplicada por feature.

Pense na spec como a resposta a três coisas antes de qualquer código existir:
- **Por quê** — a need. Que dor isso resolve? Por que agora?
- **O quê** — o comportamento. O que precisa ser verdade quando estiver pronto? (não o
  *como*)
- **Quanto custa** — os trade-offs que você está aceitando.

Só depois disso você toca no *como* (design), e só então no código.

## Por que SDD combina com este projeto

- Força a **need** a existir no papel antes de você estudar ou construir — matando a
  obesidade mental.
- Separa o **o quê** do **como**, que é exatamente o músculo que um sênior/arquiteto
  precisa.
- A spec vira o contrato contra o qual você testa e, depois, a documentação do seu
  raciocínio.
- Para um sistema distribuído, escrever a spec traz à tona as perguntas difíceis (failure
  modes, idempotency, consistência) *antes* que elas te mordam em produção.

## O workflow (por dor)

1. **Escolha uma dor** em `pain-backlog.md`.
2. **Escreva a spec** usando o template abaixo. Salve como
   `docs/specs/NN-short-name.spec.md` (ex.: `04-overselling.spec.md`).
3. **Revise a spec** — traga-a para um segundo par de olhos antes de codar. Uma revisão de
   spec pega o erro de design enquanto ele ainda é um parágrafo, não 300 linhas de código.
4. **Fabrique a dor** como a spec descreve (o failure scenario).
5. **Implemente** para satisfazer a spec.
6. **Verifique** contra os acceptance criteria.
7. **Registre** no `progress-journal.md` e anote onde a spec errou (specs são ferramentas
   de aprendizado — uma previsão imprecisa é uma lição).

> **Idioma:** as specs são escritas em **português (pt-BR)**, como os demais docs. Mas os
> nomes de domínio e termos técnicos (Packing, saga, idempotency, throughput, ...)
> permanecem em inglês, para ficarem consistentes com o código.

## Template de spec

Copie isto em cada `docs/specs/NN-short-name.spec.md`.

```markdown
# Spec NN — <short name>

## 1. Need (por quê)
- Qual dor (backlog #):
- A need em uma frase: "Eu preciso disso porque ..."
- Meta de aprendizado / curiosidade que serve:

## 2. Contexto
- Onde vive (domain/serviço):
- Comportamento atual / o que existe hoje:
- A dor a fabricar (o failure scenario que vou provocar):

## 3. Comportamento (o quê — não o como)
- Cenários Given / When / Then:
  - Given ... When ... Then ...
  - Given ... When ... Then ...
- Non-goals explícitos (o que esta spec NÃO vai fazer):

## 4. Acceptance criteria (como vou saber que está pronto)
- [ ] Critério observável 1 (mensurável: latência, sem stock negativo, sem duplicata, ...)
- [ ] Critério observável 2
- A medição/experimento que prova isso:

## 5. Failure modes & edge cases
- O que acontece quando <dependência> cai / fica lenta / responde duas vezes?
- Preocupações de concorrência / ordenação:
- Preocupações de consistência de dados:

## 6. Trade-offs aceitos
- O que esta solução custa (staleness, complexidade, novo ponto de falha, ...):
- O que estou explicitamente escolhendo NÃO otimizar ainda:

## 7. Design sketch (o como — curto)
- A abordagem em poucos bullets. Não é código completo. O suficiente para começar.

## 8. Perguntas em aberto
- Coisas de que não tenho certeza e quero resolver na revisão.
```

## Regras práticas

- **A spec descreve comportamento e restrições, não implementação.** Se você está
  escrevendo nomes de classe na seção 3, você pulou para frente — isso pertence à seção 7,
  brevemente.
- **Acceptance criteria têm que ser observáveis.** "Funciona" não é um critério. "Stock
  nunca fica abaixo de zero sob 100 compras concorrentes" é.
- **Toda spec tem que nomear um failure mode.** Num sistema distribuído, o happy path é os
  5% chatos. A spec ganha o pão dela na seção 5.
- **Mantenha as specs pequenas.** Uma dor, uma spec. Se uma spec espalha, a dor está grande
  demais — quebre-a.
- **Deixe a spec estar errada.** Quando a realidade a contradiz, essa lacuna é o
  aprendizado de mais alto valor da sessão. Registre.

## Layout de diretório para specs

```
docs/
├── specs/
│   ├── 01-bootstrap.spec.md
│   ├── 02-slow-queries.spec.md
│   ├── 04-overselling.spec.md
│   └── ...
```

Os números batem com os números das dores em `pain-backlog.md` para os dois andarem em
lockstep.
