# AVALIAÇÃO — GitHub Actions com IA

**Disciplina:** Desenvolvimento Web  
**Repositório:** [gabriel240705/oauth-aula-equipe-02](https://github.com/gabriel240705/oauth-aula-equipe-02)  
**Equipe:** Laura Gois Casteleins e Gabriel Henrique Mendes (Engenharia de Software)  
**Base da avaliação:** enunciado *Avaliação: GitHub Actions com IA* (tarefas 1 a 4, quatro perguntas e evidências).

> **Critério de registro:** as informações abaixo distinguem implementação conferida nos arquivos e fatos verificáveis nos PRs de evidências ainda não documentadas. Os campos **[PENDENTE]** devem ser preenchidos com o link real da execução (run), trecho de log ou saída do comando; não representam testes já aprovados.

## Tarefa 1 — CI a cada commit (2,5 pontos)

**Arquivo:** [`.github/workflows/ci.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/ci.yml)

O workflow `ci`, com job `verificar`, foi configurado para executar em todo `push` e em `pull_request` direcionado à `main`. Suas três etapas são: (1) busca por padrões de possíveis segredos versionados com `git grep`; (2) aplicação sequencial dos scripts de `migrations/*.sql` ao banco SQLite local `teste.db`, com `sqlite3`; e (3) validação de arquivos HTML com `npx --yes htmlhint "**/*.html"`.

O [PR #1 — correção do CI e teste de proteção da main](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) registra os commits de criação de uma falha intencional em SQL e sua correção. Isso evidencia o histórico da tentativa de teste, mas, para pontuação integral, os **links das execuções** e a **linha exata do erro** precisam constar abaixo.

- **Execução CI aprovada (verde):** **[PENDENTE — inserir URL do run]**.
- **Execução CI reprovada propositalmente (vermelho):** **[PENDENTE — inserir URL do run]**.
- **Linha exata do log que causou a falha:** **[PENDENTE — copiar apenas a mensagem de erro, sem segredos]**.
- **Histórico do teste:** [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), commits `0eadd42` (teste de falha) e `f0f06f8` (correção).

**Por que testar as migrações em SQLite antes de chegar ao D1?** Para detectar erros de sintaxe SQL e inconsistências na sequência das alterações antes de aplicá-las ao banco remoto, reduzindo o risco de interromper a atualização do esquema em produção; esse teste não substitui a verificação final no D1.

## Tarefa 2 — Proteção da `main` pelo status check (1,5 ponto)

A tarefa exige uma regra de proteção ou ruleset que obrigue pull request e aprovação do status check **`verificar`** antes do merge. Essa configuração fica nas definições do repositório e **não é comprovada apenas pelos arquivos do ZIP**.

- **PR usado no experimento de CI:** [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1).
- **Evidência do PR com merge bloqueado por falha no check `verificar`:** **[PENDENTE — link para PR/check e confirmação visual do bloqueio]**.
- **Evidência do mesmo PR liberado após a correção:** **[PENDENTE — link para PR/check aprovado e liberação de merge]**.
- **Saída do `git push` direto para a `main` rejeitado pelo GitHub:** **[PENDENTE — transcrever a mensagem de recusa, sem credenciais]**.

**Interpretação:** o bloqueio deve impedir o merge enquanto a verificação obrigatória falhar. Após a correção e aprovação do check, a regra deve permitir o merge pelo PR, sem autorizar push direto à `main`.

## Tarefa 3 — Revisão de pull request por IA (3,5 pontos)

**Arquivo:** [`.github/workflows/revisao-ia.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/revisao-ia.yml)

O workflow `revisao-ia` responde aos eventos `pull_request` `opened` e `synchronize`, estabelece `contents: read` e `pull-requests: write`, prepara um prompt com o diff, utiliza `actions/ai-inference@v3` e publica a resposta por `gh pr comment`. A versão `v3` é a existente no projeto analisado, embora o exemplo do enunciado mostre `@v1`.

No [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), há comentários da conta `github-actions` que permitem comparar a mudança vulnerável e a correção:

1. **Antes da correção:** [comentário da IA sobre SQL vulnerável](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) — a IA identifica interpolação do nome do aluno na consulta SQL e recomenda parâmetros vinculados.
2. **Depois da correção:** [comentário posterior da IA](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) — o texto reconhece o emprego de `prepare().bind(nome)` e registra outros riscos, como a ausência de autenticação/autorização no endpoint de teste.

**Teste de permissão:** o histórico do [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) contém commits associados ao teste e à restauração de permissão (`37eb278` e `a68931b`). Entretanto, o histórico, isoladamente, não demonstra qual mensagem foi retornada pelo job quando `pull-requests: write` foi removida.

- **Link da execução da revisão por IA após abertura do PR:** **[PENDENTE — URL do run]**.
- **Link da execução após o commit de correção (`synchronize`):** **[PENDENTE — URL do run]**.
- **Log do erro ao remover `pull-requests: write`:** **[PENDENTE — URL do run e trecho exato do erro]**.
- **Restauração da permissão:** ver commit `a68931b` no [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1).

### Avaliação crítica de uma afirmação da IA

Em um comentário do [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), a IA classificou como risco de SQL injection uma concatenação de SQL em `public/app.js`. Porém, a expressão identificada estava no script executado no navegador e fazia referência a `req`, variável que não existe naquele contexto. **Sem evidência de que a string seja executada como consulta no banco, não é correto afirmar uma injeção SQL explorável nesse ponto.** O achado comprovável é o erro de JavaScript (`ReferenceError`), se a expressão for executada. Outros comentários da própria IA fizeram essa distinção. Esse caso demonstra por que a revisão automatizada é auxiliar, e não substitui a análise técnica ou os checks determinísticos.

## Tarefa 4 — Migrações D1 após merge (1,5 ponto)

**Arquivo:** [`.github/workflows/migrar.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/migrar.yml)

O workflow `migrar` tem gatilho `push` na `main` com filtro `paths: ['migrations/**']`. Seu job usa `npx --yes wrangler d1 migrations apply oauth-sessions-equipe-02 --remote` e recebe as variáveis `CLOUDFLARE_API_TOKEN` e `CLOUDFLARE_ACCOUNT_ID` de `secrets`, sem colocá-las literalmente no arquivo.

A pasta `migrations/` contém `0001_inicial.sql`, que cria a tabela `notas`, e `0002_adiciona_coluna.sql`, que adiciona a coluna `observacao TEXT` à tabela. O `wrangler.toml` declara o binding `DB`, o banco `oauth-sessions-equipe-02` e o diretório de migrações.

- **PR/merge que alterou `migrations/`:** consultar [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) e identificar no histórico o merge específico que incluiu `0002_adiciona_coluna.sql`: **[PENDENTE — link exato, se não foi o PR #1]**.
- **Execução de `migrar` após o merge que alterou `migrations/`:** **[PENDENTE — URL do run com sucesso]**.
- **Comprovação da coluna remota:** executar o comando abaixo e inserir a saída que apresente `observacao`:

```bash
npx --yes wrangler d1 execute oauth-sessions-equipe-02 --remote --command "PRAGMA table_info(notas)"
```

**Saída do `PRAGMA table_info(notas)`: [PENDENTE — colar resultado real, omitindo segredos].**

- **Merge sem mudanças em `migrations/`:** [PR #4 — Teste de merge sem alterações no D1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/4). O conteúdo de `TESTE-SEM-MIGRACAO.md` documenta a finalidade desse ensaio, e o PR foi integrado à `main`.
- **Prova de que esse merge não disparou `migrar`:** **[PENDENTE — indicar commit de merge e a conferência na lista de runs do workflow]**. Conferir [execuções de `migrar`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/workflows/migrar.yml).

**Bônus:** não foi identificado no `migrar.yml` um step com `actions/ai-inference` que escreva resumo em `$GITHUB_STEP_SUMMARY`. Portanto, o bônus não está implementado no ZIP analisado.

## Perguntas obrigatórias (1,0 ponto)

### 1. O `ci.yml` roda duas vezes quando você faz push numa ramificação com PR aberto. Por quê, e qual execução o status check da `main` considera?

O arquivo possui dois gatilhos independentes: `push` (novo commit na ramificação) e `pull_request` (atualização de um PR dirigido à `main`). Por isso, uma atualização pode iniciar duas execuções; a proteção da `main` considera o check exigido para o commit/PR que está sendo avaliado, normalmente a execução associada ao `pull_request`, conforme a regra de branch e os checks configurados.

### 2. Por que um PR vindo de fork não consegue usar `secrets.COPILOT_PAT`? Qual ataque essa regra impede?

Em eventos `pull_request` originados de forks, os segredos do repositório de destino não são disponibilizados ao código não confiável da contribuição. Isso reduz o risco de alguém abrir um PR malicioso que execute comandos para ler e exfiltrar o token `COPILOT_PAT` ou outras credenciais armazenadas nos secrets.

### 3. Por que o workflow da Tarefa 3 declara `permissions` em vez de usar as permissões padrão do `GITHUB_TOKEN`?

A declaração explícita aplica o princípio do menor privilégio: `contents: read` permite consultar o código e `pull-requests: write` permite publicar comentários no PR. Como as permissões padrão do `GITHUB_TOKEN` podem depender da configuração do repositório, especificá-las torna o requisito verificável e evita conceder acesso maior do que o necessário.

### 4. A revisão da IA pode substituir o status check da Tarefa 2? Justifique com o que você observou no item 4 da Tarefa 3.

Não. A revisão por IA pode produzir alertas úteis, mas também inferências incorretas: no PR #1, um trecho de SQL construído no JavaScript do navegador foi tratado em certo comentário como SQL injection, apesar de não haver execução demonstrada desse SQL no D1. O status check `verificar` executa testes concretos e reproduzíveis e, quando obrigatório na regra da `main`, impede o merge em caso de falha; a análise da IA deve complementá-lo, não substituí-lo.

## Checklist de evidências antes da entrega

- [ ] T1: link do run verde, link do run vermelho e linha exata do erro.
- [ ] T2: PR bloqueado e depois liberado; texto de `git push` direto recusado.
- [x] T3: PR público com comentários da IA antes e depois da correção e parágrafo crítico.
- [ ] T3: link do run e log do erro da ausência de `pull-requests: write`.
- [ ] T4: run do merge com migração, saída remota do `PRAGMA` e comprovação da ausência de run no merge sem migração.
- [x] Quatro respostas correspondentes às perguntas do professor.
- [ ] Criar PR de entrega do `AVALIACAO.md` e conferir CI e revisão por IA nesse PR.

**Observação de segurança:** nunca copiar PATs, tokens Cloudflare nem valores de `secrets` para este relatório, commits ou logs. Os nomes das variáveis e os links de runs são suficientes.
