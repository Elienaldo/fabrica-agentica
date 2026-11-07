# Gerenciamento de seções por feature

**Data:** 2026-11-06 — ligação por volta de 23h22 (America/Bahia, UTC−3).  
**Estado:** decisões registradas; implementação e viabilidade da automação ainda pendentes.  
**Escopo desta entrega:** somente este documento e o diagrama `../../excalidraw/gerenciamento-secoes-por-feature.excalidraw`. Nenhuma regra oficial, bot, seção, rotina ou automação foi alterada.

## Vocabulário

Neste registro, **seção** é o agrupamento de bots na barra lateral (sidebar). Na ligação, “seção” e “sessão” foram usados de forma intercambiável; a intenção registrada aqui é **seção da sidebar**, não sessão de execução do Cursor. A destruição descrita abaixo não promete encerrar sessões do Cursor.

## Decisões da ligação

### 1. Criação na chegada da feature

1. O Supervisor recebe a feature nova.
2. O Supervisor solicita ao Dr. Eggbot a criação da equipe completa da feature de uma vez na chegada. Assim que o nome da feature estiver criado, passa esse nome como parâmetro ao futuro script de criação, com a equipe disponível para agrupamento.
3. O script de criação cria uma seção com **o mesmo nome da feature** e move **todos os bots daquela equipe** para ela.

O nome é um parâmetro lógico do desenho; não foi definida sintaxe de comando, flag, CLI, chamada de API ou implementação de script.

### 2. Mudança de organização da equipe

- **Situação anterior relatada na ligação:** a equipe de review só era criada depois da implementação.
- **Nova decisão da ligação:** o Supervisor solicita ao Dr. Eggbot a criação de toda a equipe de uma vez na chegada da feature, para que o script possa mover todos os bots dessa equipe para a seção.

Essa mudança está apenas documentada aqui. Os modelos e as instruções atuais do Supervisor e dos demais bots não foram editados. A composição exata da equipe e a compatibilização técnica com o mecanismo atual de criação continuam pendentes; não se presume uma lista nova de papéis.

### 3. Destruição somente com autorização humana

No final da feature:

1. **Sem autorização humana, aguardar e não excluir nada.**
2. Com autorização humana, o Supervisor executa o futuro script de destruição, passando o nome da feature/seção como parâmetro.
3. O script exclui primeiro **todos os bots que estão na seção-alvo**.
4. Só depois de confirmar que todos esses bots foram excluídos, exclui a própria seção.
5. Se algum bot falhar na exclusão, **interrompe antes de apagar a seção**, mantendo a seção e o rastro disponível.

A abrangência é exclusivamente a **seção-alvo** identificada pelo parâmetro. Não há autorização para excluir todos os bots do usuário, outras seções ou bots fora da seção-alvo. Esta documentação também não constitui autorização antecipada para uma execução real de destruição: o gate humano continua obrigatório no final de cada feature.

### 4. Caminho técnico escolhido e origem da informação

Conforme a **resposta do suporte colada pelo usuário em t895**, foi informado que não há API documentada para bots criarem, renomearem, excluírem ou moverem seções, nem prazo informado para essa capacidade. Esta frase registra o relato do suporte apresentado pelo usuário; **não é resultado de nova auditoria de API ou consulta ao suporte nesta tarefa**.

Diante disso, a decisão da ligação foi usar **scripts por automação da interface desktop** para criação e destruição. Nenhum script foi implementado ou testado e a disponibilidade/viabilidade dessas operações na UI não foi verificada nesta entrega. Renomear aparece somente no contexto do relato sobre API; não foi definido um fluxo de renomeação.

### 5. Atalho opcional

Foi registrada a opção de uma tecla de atalho para acionar a criação, usando o **mesmo parâmetro: nome da feature**. A combinação de teclas **não foi escolhida** e nenhum atalho foi ativado. É uma entrada opcional do desenho, não uma automação em funcionamento.

## Responsabilidades no desenho

| Responsável | Ação registrada |
|---|---|
| Supervisor | Receber a feature, solicitar ao Dr. Eggbot a criação da equipe completa na chegada e passar o nome ao script de criação. No final, somente após autorização humana, passar o nome ao script de destruição. |
| Futuro script de criação | Criar a seção de mesmo nome e mover a equipe daquela feature. |
| Humano | Autorizar ou não a destruição ao final da feature. |
| Futuro script de destruição | Excluir apenas os bots da seção-alvo; verificar o resultado; excluir a seção somente se todos foram excluídos; interromper e manter seção/rastro em caso de falha. |
| Atalho opcional | Eventual entrada para criação com o mesmo nome; combinação e funcionamento ainda não definidos. |

## Pendências técnicas — não são decisões adicionais

- Definir a composição exata da equipe completa e como a criação conjunta será compatibilizada com o mecanismo atual, sem presumir alteração já aplicada às regras.
- Definir como o nome recebido identificará a seção correta e os bots corretos, inclusive em casos de nomes duplicados ou conflito. Essas verificações ainda não foram projetadas ou testadas.
- Verificar a viabilidade real da automação UI desktop, seletores/controles disponíveis, permissões e comportamento de erros. Não há garantia técnica de funcionamento nesta documentação.
- Projetar e implementar os dois scripts e sua interface de parâmetros. Não existe comando executável definido aqui.
- Definir como detectar e registrar falhas de exclusão, preservando o rastro conforme a decisão humana. Nenhuma política de repetição automática foi aprovada.
- Decidir a combinação de teclas e o modo de fornecimento do parâmetro no atalho opcional, caso ele venha a ser adotado.
- Definir a forma técnica de registrar e conferir a autorização humana por feature antes de executar destruição.

Esses itens são trabalho técnico pendente, não recomendações apresentadas como decisões já tomadas pelo usuário. A sequência aprovada não deve ser confundida com disponibilidade de implementação.

## Diagrama e limites da entrega

O diagrama separa as trilhas de criação e encerramento. Mostra o nome como parâmetro, a entrada opcional por atalho, o gate de autorização humana e o gate de sucesso da exclusão dos bots antes da exclusão da seção. O caminho “Não” da autorização aguarda sem exclusão; o caminho “Não” de “Todos excluídos?” interrompe e mantém a seção/rastro.

Foram criados apenas arquivos novos de documentação e desenho. Não foram criados, movidos ou excluídos bots/seções; não houve automação desktop, implementação de scripts, alteração de rotinas, perfis, skills oficiais ou documentos existentes. Nenhuma sessão de execução do Cursor foi encerrada por esta tarefa.
