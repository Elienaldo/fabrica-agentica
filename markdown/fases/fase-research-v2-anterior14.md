# Fábrica agêntica - Fase RESEARCH (v2, fundo preto)

Arquivo: `fase-research-v2.excalidraw`. Link: https://excalidraw.com/#json=rkxT364u6o0qaDD5niw6_,_otW52o-7OxLAZqe8bFtLA (anterior aos diagramas de 01/11; a anterior ao ajuste de tamanho e cor está em `fase-research-v2-anterior.excalidraw`)

```mermaid
flowchart LR
  PE[PRINCIPAL PRODUCT ENGINEER] -- pedido de feature --> D1{3 perguntas: alguma SIM?}
  D1 -- SIM: vai pro research --> T
  subgraph T[TIME DE RESEARCH]
    E1[PRINCIPAL PRODUCT ENGINEER]
    E2[PRODUCT ENGINEER]
    FE[DESIGN DE FRONT-END]
    AD[ADVOGADO DO DIABO - não vota]
  end
  GM[[SKILL grill-me]] -.-> T
  WF[[SKILL wayfinder opcional: decide o Principal]] -.-> T
  PR[[SKILL prototype]] -.-> T
  T -- posições por área --> D2{CONSENSO por área?}
  D2 -- NÃO: nova rodada, máx. 2 --> T
  D2 -- NÃO após 2 rodadas --> SUP[SUPERVISOR]
  SUP -- repassa --> H((HUMANO))
  H -- decisão --> SUP
  SUP -- decisão humana --> DD
  D2 -- SIM: consenso --> DD[SAÍDA: DESIGN DOC]
  DD -- design doc --> PL[FASE PLAN]
  D1 -- NÃO: feature pequena, atalho --> PL
```

| Peça | Forma | O que faz |
| --- | --- | --- |
| Principal Product Engineer | caixa | Recebe o pedido e classifica a feature com 3 perguntas. |
| 3 perguntas | losango | Muda arquitetura? Mexe em mais de um serviço? Requisito ambíguo? Qualquer SIM vai pro research. |
| Principal Product Engineer e Product Engineer | caixa | Mesmo peso de voto, modelos de famílias diferentes; decidem escopo, prioridade e o como técnico. |
| Design de front-end | caixa | Desenha a experiência e decide o front. |
| Advogado do diabo | caixa | Só levanta objeções; não vota. |
| Skills grill-me, wayfinder (opcional), prototype | caixa tracejada de cantos retos | Perguntas duras; mapa de projeto muito grande (decide o Principal; humano só se ele não conseguir, via Supervisor); protótipo descartável. |
| Consenso por área | losango | Produto e engenharia decidem escopo, prioridade e como técnico; design decide o front. O advogado do diabo fica fora. |
| Supervisor | caixa | Único canal com o humano; recebe a escalada e repassa a decisão. |
| Humano | círculo | Decide quando não há consenso após 2 rodadas, via Supervisor. |
| Design doc | caixa | Saída: problema, opções, decisão e trade-offs. |
| Fase PLAN | caixa | Recebe o design doc (quando a feature passou pelo research) ou a solicitação direta (quando é pequena e pula o research). |

Itens que eu propus e você não pediu: limite de 2 rodadas e o humano no desempate.
