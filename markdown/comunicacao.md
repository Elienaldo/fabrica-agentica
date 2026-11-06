# Comunicação da fábrica

**Nome:** Comunicação da fábrica

**Responsável:** Dr. Eggbot. Quem lê este documento e cria sessões é ele.

> **OBRIGATÓRIO:** ao criar uma sessão, o Dr. Eggbot SEMPRE cria os grupos de chat **direção** e **review**, e o grupo **research** junto com os agentes de research. Isso é permissão fixa dada pelo Elienaldo: vale para toda feature, sem exceção, e o Dr. Eggbot não espera pedido do humano a cada feature. Sessão sem os grupos não está criada.

**Job único:** definir quem fala com quem, em quais grupos do Grok Bot, por onde tudo passa e como os grupos nascem, encerram e são apagados. Os papéis e as fases estão em `fábrica-agêntica.md`.

**Anti-jobs:** o Dr. Eggbot nunca cria sessão sem os grupos; nunca espera pedido do humano para criar os grupos; nunca reaproveita canal entre features; nunca apaga canal (não consegue); nunca coloca o Cursor em grupo; nunca deixa um bot falar com o humano fora do Supervisor.

**Voz:** regra imperativa, sem exceção.

**Acordar:** sob demanda, quando o Supervisor pede a criação ou a destruição de uma sessão.

## 1. Criação dos grupos (permissão fixa)

1. Ao criar a sessão, o Dr. Eggbot cria os grupos abaixo, com o nome padronizado da seção 5.
2. O grupo permanente (Supervisor e Dr. Eggbot) existe antes de qualquer sessão e não é recriado a cada feature.
3. O grupo de research nasce quando o Principal classifica a feature como SIM para research e o Dr. Eggbot cria os agentes de research. Se a feature pega o atalho, não existe grupo de research.
4. Cada grupo é um canal do Grok Bot com no máximo 6 membros. Canais não se aninham.
5. O Dr. Eggbot cria canais e muda membros. Não esvazia canal (sempre sobra pelo menos 1 membro) e não apaga canal. Só o humano apaga, pela barra lateral.

## 2. Grupos

| Grupo | Membros (agentes) | Quando existe |
|---|---|---|
| Permanente | Supervisor e Dr. Eggbot (o humano acompanha) | Sempre |
| Direção | Supervisor, Principal Product Engineer e Orquestrador de pipeline | Toda feature, criado com a sessão |
| Research | Principal Product Engineer, Product Engineer, Design de front-end e Advogado do diabo | Só quando o Principal classificar a feature como SIM para research |
| Review | Orquestrador de pipeline, Cirurgião, Curador amigável e Supervisor | Toda feature, criado com a sessão |

O Supervisor entra nos grupos de direção e de review. Não entra no de research: recebe o resultado pelo Principal, no grupo de direção.

## 3. Pares de comunicação

| Par | Onde acontece |
|---|---|
| Supervisor e humano (Elienaldo) | Chat direto do app do Grok Bot, fora dos grupos da fábrica |
| Supervisor e Dr. Eggbot | Grupo permanente |
| Supervisor e Principal | Grupo de direção |
| Principal e Orquestrador | **Não se falam direto**: o Principal entrega os insumos ao Supervisor, que os repassa ao Orquestrador (grupo de direção) |
| Orquestrador e Supervisor (insumos recebidos, pedidos ao Cursor, falhas e escaladas) | Grupo de direção |
| Principal, Product Engineer, Design de front-end e Advogado do diabo | Grupo de research |
| Orquestrador, Cirurgião e Curador (com o Supervisor) | Grupo de review |
| Principal e sessões do Cursor (fase research) | **Fora do Grok Bot**: o Cursor não é agente do Grok Bot, então não entra em grupo |
| Orquestrador e sessões do Cursor (fases plan, implement e review) | **Fora do Grok Bot**, pelo mesmo motivo |

## 4. Roteamento

1. Tudo que vai para o humano passa pelo Supervisor, sem exceção.
2. O Curador entrega HTML e documentos ao Supervisor, que os repassa ao humano.
3. A decisão do humano volta pelo mesmo caminho: humano, Supervisor, grupo em que a pergunta nasceu.
4. Quem fala com o Cursor: só o Supervisor. Nas fases plan, implement e review, o Orquestrador pede ao Supervisor que dispare; na fase research, o Principal pede ao Supervisor que dispare. O Supervisor posta o resumo do resultado no grupo da fase.
5. Toda comunicação entre atores passa pelo Supervisor. O Principal entrega os insumos (design doc, PRD ou solicitação direta) ao Supervisor; o Supervisor os repassa ao Orquestrador e determina se é plan ou implement; o Orquestrador devolve ao Supervisor o pedido ao Cursor, e o Supervisor o faz (Principal → Supervisor → Orquestrador → Supervisor → Cursor). Não existe caminho direto do Principal ao Orquestrador. A conversa interna do time de research, no grupo de research (onde o Supervisor não entra), é da própria fase.
6. Toda fase nova (research, plan, implement e review) começa com o Supervisor solicitando ao Cursor que abra uma nova sessão, com contexto limpo, sem exceção. O handoff entre fases passa sempre pelo Supervisor, que dispara a sessão nova. Nenhuma fase reaproveita a sessão da anterior.

## 5. Nome dos canais e encerramento

1. Nome padronizado, para filtrar e apagar em lote por prefixo: `feat-<slug>-AAAAMMDD.<tipo>`, ou `fix-<slug>-AAAAMMDD.<tipo>` nas sessões `fix/`. O tipo é `direção`, `research` ou `review`. Exemplo: `feat-nome-20261101.direção`.
2. Ao encerrar uma feature, cada grupo recebe uma mensagem final em maiúsculas, com o link do registro de decisões.
3. Em seguida os bots saem do canal, deixando um membro. Assim não recebem mais nada.
4. Canal nunca é reaproveitado entre features, porque mistura contexto.

## 6. Histórico

1. O canal fica arquivado por 30 dias. O humano apaga em lote ou na hora, como preferir.
2. A conversa do canal não vale como histórico. O registro de verdade é a documentação do projeto: ADRs, lições e estado das skills.
3. No ADR existente, acrescenta-se uma única linha com o veredito final da revisão: passou ou não.

**Como concreto:** canais do Grok Bot (criar canal e mudar membros). Nenhuma skill da fábrica.

**Ciclo de vida:** permanente; vale para toda sessão.

## 7. Pontos em aberto (a confirmar com o Elienaldo)

- **Prefixo:** o ditado dizia `fit`; foi gravado `feat`, para casar com a sessão `feat/`. Confirmar.
- **Dono do ADR:** quem escreve a linha do veredito não está definido.
- **Abertura e fechamento de cada grupo:** quem posta a primeira mensagem e a mensagem final de cada grupo não está definido.
