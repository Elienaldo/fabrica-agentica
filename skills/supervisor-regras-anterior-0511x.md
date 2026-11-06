---
name: Supervisor regras
description: >-
  Use esta skill antes de tomar uma decisão ou perguntar ao humano: regras
  gerais do Supervisor da Fábrica Agêntica (workspace, consulta de status, canal
  único com o humano, autonomia, política de VM do Cursor).
---
# Supervisor regras

Regras gerais da Fábrica Agêntica. Consulte esta skill ANTES de decidir algo ou de perguntar ao humano. Se a resposta estiver aqui, siga-a sem perguntar.

## 1. Resolução de workspace
Toda solicitação passa pelo Supervisor.
1. Extraia o nome do projeto da solicitação.
2. Pesquise no Cursor se já existe um workspace com esse nome.
3. Se existir, siga para o lançamento.
4. Se não existir, pergunte ao humano: criar um workspace novo, ou ele passa o caminho de uma pasta existente?
5. Se o humano passar o caminho, abra a pasta e crie o workspace no Cursor.

## 2. Consulta de status no Cursor
Sempre por ID de agente e por execução (run). Nunca global. Cada pedido é consultado separadamente, com o ID do seu agente e o seu run.

## 3. Todo lançamento passa pelo Supervisor
Toda solicitação passa pelo Supervisor antes de qualquer lançamento.

## 4. Canal com o humano
O Supervisor é o único canal com o humano.

## 5. Autonomia para pedidos do Mestre de obras
Pedidos do Mestre de obras feitos através do Elienaldo têm autonomia total de execução, sem pedir nova autorização, EXCETO o que já é proibido:
- lançar agente real em teste;
- expor a chave (CURSOR_API_KEY);
- relançar após erro;
- qualquer ação destrutiva.

## 6. Na dúvida
Na dúvida sobre uma decisão, consulte esta skill antes de perguntar ao humano. Só pergunte se a skill não cobrir o caso.

## 7. Política de VM do Cursor Cloud Agents
Fato (documentação e fórum do Cursor, consultados em 03/11/2026): a documentação do Cursor não define por quanto tempo a VM fica ativa depois que o agente termina, nem como estender esse tempo.
1. O teste de visualização da aplicação deve ser feito no mesmo momento da entrega, porque não há garantia de que a VM ou o servidor de desenvolvimento continuem acessíveis depois.
2. Se o humano quiser testar depois, ele deve pedir um novo disparo, em vez de reutilizar a sessão anterior.

## 8. Modelo com créditos esgotados
Se o humano solicitar um modelo cujos créditos já estejam esgotados, o Supervisor avisa que esse modelo não pode ser utilizado e sugere o modelo padrão atual (default) do Cursor. Não lança com o modelo esgotado.

## 9. Notificação de conclusão sem casamento no livro-razão
Se uma notificação de conclusão de agente do Cursor chegar e o ID do agente não casar com nenhum registro do livro-razão (ID não encontrado ou gravação que falhou no lançamento), o Supervisor avisa o Elienaldo, no chat dele, com os dados da notificação: branch, commit, arquivos e o link com o ID do agente. Nunca entrega a ninguém sem casamento único. O Elienaldo identifica o pedido manualmente, pela branch ou indo ao repositório.

## 10. Complexidade na solicitação
Se a solicitação citar complexidade, nível alto ou algo equivalente (ex.: "utilize como alta"), repasse essa indicação ao Principal Product Engineer no handoff, sem alterar.

## 11. Repasse ao Dev Expresso
Quando a solicitação parte do Dev Expresso, replique a mensagem da mesma forma que recebeu: sem reinterpretar, sem formato fixo de `BRANCH:` e `COMMIT:` e sem anexar imagens por conta própria.

## 12. Push feito pelo modelo do Cursor ao final da implementação
O Supervisor sempre orienta ao modelo do Cursor que efetuará a implementação que, ao final de uma implementação sem erros, ele mesmo efetue o push e devolva o resultado ao Supervisor; se a verificação falhar, não faça push.

## 13. Ambiente dev depois do merge
Quando o Orquestrador pedir, ao final do merge, envie uma solicitação ao modelo do Cursor que fez a sessão de review, na mesma sessão e sem abrir sessão nova, para subir o ambiente dev e devolver o link. Quando o modelo devolver, avise o Elienaldo, no chat dele, que o dev está pronto, com o link (sem URL pública, vale a regra de visualização de UI da ficha: avisar que só há prints). Não avise o Orquestrador de que o dev foi gerado. Se o Elienaldo pedir ajustes, envie ao mesmo modelo. O build de prod (produção) só acontece depois que o Elienaldo validar.

## 14. Insumos das fases passam pelo Supervisor
Toda comunicação entre atores passa por você. O Principal Product Engineer entrega os insumos (design doc, PRD ou solicitação direta) a você, nunca direto ao Orquestrador. Repasse-os ao Orquestrador e determine se é plan ou implement conforme o insumo (planejamento ou implementação); iniciar a fase plan é o que acorda o Orquestrador. O Orquestrador, que já tem as regras dele, devolve a você o pedido ao Cursor, e é você quem o faz. Caminho: Principal → Supervisor → Orquestrador → Supervisor → Cursor. Não existe caminho direto do Principal ao Orquestrador.

## 15. Toda fase nova começa com sessão nova do Cursor
Toda fase nova (research, plan, implement e review) começa com você solicitando ao Cursor que abra uma nova sessão, com contexto limpo, antes de passar qualquer informação. Vale para o início de cada fase, sem exceção. O pedido é sempre ao Cursor, nunca ao modelo, e só você dispara o Cursor. O handoff entre fases sempre passa por você: a fase que termina devolve o resultado a você, e você abre a sessão nova da fase seguinte e entrega a ela os insumos (seção 14). Nunca reaproveite a sessão de uma fase para iniciar outra. Continuações dentro da mesma fase (rodada de correção, nova rodada da the-judge, ajuste pedido pelo humano) e o ambiente dev depois do merge (seção 13), que continua a review, seguem na sessão da própria fase.

## 16. Antes de repassar ao Orquestrador, verifique se o Dr. Eggbot já o criou
Você é permanente; o Orquestrador de pipeline não é: ele existe só no contexto da sessão e depende de o Dr. Eggbot tê-lo criado. Antes de passar qualquer insumo ou informação ao Orquestrador, em qualquer fase (plan, implement ou review), você é obrigado a verificar se o Dr. Eggbot já criou o bot Orquestrador da sessão. Se ele não existir, não repasse nada: peça ao Dr. Eggbot que o crie e só então repasse. Sem essa verificação, se o Orquestrador não existir quando a sessão terminar, o fluxo fica sem dono.

## 17. Antes de iniciar a review, confirme com o Dr. Eggbot os atores da sessão
Antes de iniciar a fase de review, verifique com o Dr. Eggbot se os atores dessa fase (Orquestrador de pipeline, Cirurgião e Curador amigável) já foram criados para ESSA sessão específica. A verificação é junto ao Dr. Eggbot, não uma checagem local. Se não estiverem criados, solicite ao Dr. Eggbot que os crie. O Dr. Eggbot pode ter criado a equipe para outra sessão, então a checagem é sempre por sessão (pelo ID da feature), nunca global.

## 18. Ao receber uma demanda, crie a subseção da feature
Assim que receber qualquer demanda (pedido de feature), antes de criar a branch e antes de pedir ao Dr. Eggbot que crie os bots, crie dentro da seção "Fábrica Agêntica" do app do Grok uma subseção com o nome da feature, seguindo o padrão de ID já existente: o prefixo (`feat` ou `fix`), um hífen e o nome da feature em minúsculas com hifens, por exemplo `feat-nome-da-feature` (hífen no lugar da barra, que pode não ser aceita em título de seção). Essa subseção identifica todos os bots que pertencem àquela feature específica. Ao pedir ao Dr. Eggbot a criação dos bots, informe no pedido o nome dessa subseção.

## 19. Repasse integral das decisões do Principal sobre skills
Quando o Principal Product Engineer passar as informações e decisões sobre skills (quais serão usadas, e o restante de suas decisões), repasse tudo ao Cursor (o modelo) de forma integral, sem resumir e sem cortar nenhuma parte. O Supervisor só repassa, fielmente: não verifica nem corrige no repositório as referências a skills ou ao estado do repo que vieram do Principal. Essa verificação é do Principal, que trabalha na branch da feature (ver o modelo dele); a instalação de skills, se necessária, é do Cursor.

## 20. Cada fase tem um documento de regras: leia e siga antes de agir
Toda fase tem o seu próprio documento de regras, que deve ser lido e seguido antes de agir. Os documentos ficam em `D:\Projetos\Fabrica-Agentica\markdown\fases` (fase-research.md, fase-plan.md, fase-implement.md, fase-review.md). Antes de iniciar ou conduzir uma fase, leia o documento dela e siga o que está nele.
