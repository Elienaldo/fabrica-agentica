# Fábrica agêntica - Fase PR

Registro em texto do diagrama `fase-pr.excalidraw`. Entra depois do handoff da fase IMPLEMENT e termina com o PR mergeado.

```mermaid
flowchart LR
  A[PR aberto, vem da fase IMPLEMENT] --> J[the-judge: review do PR]
  J -- resultado --> C[Curador review: HTML visual simples]
  C -. decisão informada .-> H[HUMANO]
  J -- limpo --> M[Mergeador]
  J -- should fix ou nit --> S[Cirurgião]
  S -- devolve --> J
  J -- blocker --> H
  S -- complexo ou com risco --> H
  H -- flag autoriza blocker --> S
  H -- ok humano --> M
  M --> F[PR mergeado]
```

## Peças

| Peça | O que faz |
| --- | --- |
| the-judge | Review de PR. Classifica findings em blocker, should fix e nit. No máximo 3 rodadas por execução. Se estourar 3 rodadas com blocker, para e escala pro humano. Nova rodada, depois de uma correção, só revisa o diff novo. |
| Curador review (bot novo) | Pega o resultado da the-judge e gera um HTML visual, com linguagem direta e simples, para facilitar a decisão de dev júnior, pleno ou novo no projeto. O HTML é visto no app do Grok Bot. Blockers e itens não bloqueantes (should fix e nit) ficam em destaque. |
| Cirurgião (agente, não skill) | Corrige should fix e nit sozinho. Should fix complexo ou com risco de efeito colateral vai pro humano. Blocker nunca é corrigido sem flag de autorização humana. Depois de corrigir, devolve pra the-judge. |
| Mergeador (bot) | Merge automático só se não houver blocker pendente e o veredito da última rodada da the-judge passou limpo. Nit pode permanecer. Se o cirurgião usou flag de blocker autorizado, o merge espera ok humano. |
| Humano | Decide blocker, should fix complexo ou com risco e as travas anti-loop. Autoriza correção de blocker com flag. |
| PR mergeado | Handoff final da fábrica. |

## Travas anti-loop e escalada

- 3 rodadas da the-judge com blocker: para e escala pro humano.
- Should fix gerado 3 vezes seguidas após correções do cirurgião: trava para intervenção humana.
- Blocker novo: sempre vai pro humano.
- Should fix complexo ou com risco de efeito colateral: humano.
- Blocker com decisão humana que inclui sugestão de correção: o cirurgião corrige com a flag, a the-judge valida, e se passou o merge é automático, sem nova avaliação humana.
