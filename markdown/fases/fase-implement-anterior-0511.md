# Fábrica agêntica - Fase IMPLEMENT

Registro em texto do diagrama `fase-implement.excalidraw`. Entra depois do handoff da fase PLAN.

```mermaid
flowchart LR
  A[HANDOFF do PLAN: caminho da pasta da feature] --> B[Orquestrador de pipeline]
  B -- pede ao Supervisor que dispare o modelo do Cursor --> S
  subgraph S[TLC Spec Lean, modo IMPLEMENT, via Cursor, modelo menos custoso; Verifier e commits internos]
    BU[Build: executa o que foi planejado] --> TU[Testes unitários locais]
    TU --> V{Verificação interna, Verifier: tudo verde?}
    V -- NÃO --> CO[Rodada de correção, até 3]
    CO --> TU
  end
  V -- SIM --> H[HANDOFF ao orquestrador: caminho da pasta da feature]
  H --> PU[Push no GitHub cria o PR]
  PU --> RV[Fase REVIEW]
  CO -- após 3 rodadas --> SUP[Supervisor]
  SUP -- repassa --> E((Humano))
```

## Peças

| Peça | O que faz |
| --- | --- |
| HANDOFF do PLAN | Entrega o caminho da pasta da feature (com `plan.md` e `checks.md`). |
| Orquestrador de pipeline | Pede ao Supervisor que solicite ao modelo do Cursor que dispare a skill (só o Supervisor dispara o Cursor). Não escreve código. |
| TLC Spec Lean (modo implement) | Disparada via Cursor, escolhendo o modelo menos custoso. Recebe o caminho da pasta da feature e executa o que foi planejado. |
| Build | A skill executa o que foi planejado. |
| Testes unitários | Rodam localmente, a partir dos checks do plan. |
| Verificação | Subtarefa interna da skill (Verifier, parte da TLC Spec Lean, não é fase nem agente): confere se tudo está verde e gera o `verification.md`. Os commits também são internos da skill. |
| Rodada de correção | Até 3 rodadas, voltando a testar. |
| Escalar | Depois de 3 rodadas sem ficar verde, escala ao Supervisor, que repassa ao humano. |
| Regra | Teste vermelho é parada. Não se enfraquece a asserção. |
| Handoff final | Caminho da pasta da feature, entregue ao orquestrador, que pede o push; o push cria o PR e a fase REVIEW começa. |

## Nota

A skill real é `tlc-spec-lean`. A escalada vai ao Supervisor, único canal com o humano.
