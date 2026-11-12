# Convenções da fábrica agêntica

**Data:** 2026-11-02

## Branches

Toda nova solicitação cria uma branch nova. Nunca se reutiliza branch de outra solicitação.

**Padrão de nome:** `feat/YYYYMMDD-nome-da-task`

- `feat` é a abreviação de feature.
- `YYYYMMDD` é ano, mês e dia.
- `nome-da-task` é o slug gerado automaticamente pelo supervisor a partir das palavras principais da solicitação, sem artigos e preposições, com até uns três termos e hífen entre eles. O slug é o mesmo nome da task.

**Exemplo:** `feat/20261102-botao-login`

**Por quê:** isola uma solicitação da outra e deixa o merge limpo.

## Roteamento

O grupo de chat amarra o projeto. Se o pedido for de outro projeto, o supervisor pergunta antes de rotear.

## Campos que a notificação de conclusão não traz

A notificação de conclusão do agente de nuvem não traz o campo tipo nem o slug de origem. Os dois vêm do Supervisor, que os registra no livro-razão ao lançar o agente e os recupera pelo ID do agente. Quem preenche esses campos é o Supervisor, não o agente que roda. Não há webhook no fluxo.

## Criação de bots: sempre pelo modelo

Existe um modelo para cada bot da fábrica, em `C:\Users\Elienaldo\Projetos\Fabrica-Agentica\markdown\modelos\` (por exemplo `supervisor.md`, `dr-eggbot.md`, `principal-product-engineer.md`, `orquestrador-de-pipeline.md`, `cirurgiao.md`, `curador-amigavel.md`).

**Regra (decidida pelo Elienaldo em 2026-11-03):** sempre que o Dr. Eggbot receber uma solicitação de criação de bot na fábrica, ele usa o modelo correspondente àquele bot e o aplica na criação: nome, job único, anti-jobs, voz, quando acorda e como concreto. Não cria um bot da fábrica de cabeça nem a partir de descrição solta no pedido.

- Quem pede a criação indica o modelo pelo nome do arquivo.
- Se não existir modelo para o bot pedido, o Dr. Eggbot não cria: devolve o pedido e avisa o Elienaldo.
- Se o pedido contradisser o modelo, vale o modelo; a divergência vai ao Elienaldo antes de criar.
- O Dr. Eggbot não acessa diretamente a pasta local da Fábrica (`C:\Users\Elienaldo\Projetos\Fabrica-Agentica`) na máquina do Elienaldo. Quem faz o pedido (Supervisor ou Mestre de obras) cola o texto do modelo na própria solicitação.
- Só os arquivos sem sufixo `-anterior` valem; os com `-anterior` são backup.

**Por quê:** mantém cada bot igual ao que foi desenhado e aprovado, e evita bots com dois jobs ou sem anti-jobs.

## Modelo padrão

O modelo padrão da fábrica é o grok-4.7 com reasoning_effort high, fast false e context 500k (ver seção 30 de supervisor-regras): toda solicitação ao Cursor sem modelo explícito usa esse padrão. Se o humano solicitar um modelo específico no início do fluxo, ele segue em todas as fases até o final. A troca de modelo no meio do fluxo não é automática. Se o grok-4.7 estiver com créditos esgotados, usa outro modelo disponível, nunca o esgotado.

## Regra de ouro de atualização

**Regra de ouro de atualização (decisão do Elienaldo; dono: Mestre de obras):** toda atualização de regra de um bot da fábrica é conduzida pelo Mestre de obras, que no mesmo trabalho atualiza a documentação do projeto (`C:\Users\Elienaldo\Projetos\Fabrica-Agentica`) e a biblioteca correspondente (skill ou perfil do bot), com backup antes e hash antes/depois. Documento e biblioteca não podem ficar com versões diferentes. Vale para todos os trabalhos futuros, sem precisar reescrever cada perfil.

## Economia de contexto (regra 28) vale para todos os bots

Decisao do Elienaldo (12/11/2026): a regra 28 (conversa nova por fase, handoff enxuto, economia de contexto) aplica-se a TODOS os bots da fabrica, sem excecao, inclusive Dr. Eggbot. Cada bot carrega o bloco da regra 28 na sua fonte real (skill ou descricao) e na sua documentacao de modelo, em versoes iguais. O texto canonico da regra 28 esta em supervisor-regras secao 28 e em _modelo-base-bot.md.
