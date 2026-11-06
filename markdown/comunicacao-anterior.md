# Comunicação da fábrica agêntica

Versão de trabalho. Define quem fala com quem, em quais grupos do Grok Bot, por onde tudo passa e como os grupos nascem, encerram e são apagados. Os demais papéis e fases estão em `fábrica-agêntica.md`.

## 1. Regra de criação dos grupos (permissão fixa)

- **Permissão fixa dada pelo Elienaldo:** sempre que o ThreadBot criar uma nova sessão da fábrica, ele cria automaticamente os grupos correspondentes aos pares definidos neste documento, sem precisar de pedido a cada feature.
- Cada grupo é um canal do Grok Bot com no máximo 6 membros. Canais não se aninham.
- O Dr. Eggbot cria canais e muda membros, mas não esvazia canal (sempre sobra pelo menos 1 membro) e não apaga canal. Só o humano apaga, pela barra lateral.

## 2. Grupos

| Grupo | Membros (agentes) | Quando existe |
|---|---|---|
| Permanente | Supervisor e Dr. Eggbot (o humano acompanha) | Sempre |
| Direção | Supervisor, Principal Product Engineer e Orquestrador de pipeline | Toda feature |
| Research | Principal Product Engineer, Product Engineer, Design de front-end e Advogado do diabo | Só quando o Principal classificar a feature como SIM para research |
| Review | Orquestrador de pipeline, Cirurgião, Curador amigável e Supervisor | Toda feature |

O Supervisor entra nos grupos de direção e de review. Não entra no de research: recebe o resultado pelo Principal, no grupo de direção.

## 3. Pares de comunicação

| Par | Onde acontece |
|---|---|
| Supervisor e humano (Elienaldo) | Chat direto do app do Grok Bot, fora dos grupos da fábrica |
| Supervisor e Dr. Eggbot | Grupo permanente |
| Supervisor e Principal | Grupo de direção |
| Principal e Orquestrador | Grupo de direção |
| Orquestrador e Supervisor (falhas e escaladas) | Grupo de direção |
| Principal, Product Engineer, Design de front-end e Advogado do diabo | Grupo de research |
| Orquestrador, Cirurgião e Curador (com o Supervisor) | Grupo de review |
| Principal e sessões do Cursor (fase research) | **Fora do Grok Bot**: o Cursor não é agente do Grok Bot, então não entra em grupo |
| Orquestrador e sessões do Cursor (fases plan, implement e review) | **Fora do Grok Bot**, pelo mesmo motivo |

## 4. Regras de roteamento

1. Tudo que vai para o humano passa pelo Supervisor, sem exceção.
2. O Curador entrega HTML e documentos ao Supervisor, que os repassa ao humano.
3. A decisão do humano volta pelo mesmo caminho: humano, Supervisor, grupo em que a pergunta nasceu.
4. Quem fala com o Cursor: nas fases plan, implement e review, o Orquestrador; na fase research, o Principal. Quem fala com o Cursor posta o resumo do resultado no grupo da fase.

## 5. Encerramento e destruição

1. Ao encerrar uma feature, o grupo recebe uma mensagem final em maiúsculas, com o link do registro de decisões.
2. Os bots saem do canal, deixando um membro. Assim não recebem mais nada.
3. Canal nunca é reaproveitado entre features, porque mistura contexto.
4. Nome padronizado do canal, para filtrar e apagar em lote por prefixo: `feat-<slug>-AAAAMMDD.<tipo>`, ou `fix-<slug>-AAAAMMDD.<tipo>` nas sessões `fix/`. O tipo é `direção`, `research` ou `review`. Exemplo: `feat-nome-20261101.direção`.

## 6. Histórico

- O canal fica arquivado por 30 dias. O humano apaga em lote ou na hora, como preferir.
- A conversa do canal não vale como histórico. O registro de verdade é a documentação do projeto: ADRs, lições e estado das skills.
- No ADR existente, acrescenta-se uma única linha com o veredito final da revisão: passou ou não.

## 7. Pontos em aberto (a confirmar com o Elienaldo)

- **ThreadBot:** o nome não aparece nos demais arquivos da fábrica, onde quem cria os bots e os chats da sessão é o Dr. Eggbot. Confirmar se é o mesmo agente ou outro.
- **Prefixo:** o ditado dizia `fit`; foi gravado `feat`, para casar com a sessão `feat/`. Confirmar.
- **Dono do ADR:** quem escreve a linha do veredito no ADR não está definido.
- **Abertura e fechamento de cada grupo:** quem posta a primeira mensagem e a mensagem final de cada grupo não está definido.
