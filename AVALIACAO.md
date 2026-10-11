# AVALIAÇÃO — GitHub Actions com IA

**Disciplina:** Desenvolvimento Web  
**Repositório:** [gabriel240705/oauth-aula-equipe-02](https://github.com/gabriel240705/oauth-aula-equipe-02)  
**Equipe:** Laura Gois Casteleins e Gabriel Henrique Mendes (Engenharia de Software)  
**Base da avaliação:** enunciado *Avaliação: GitHub Actions com IA* (tarefas 1 a 4, quatro perguntas e evidências).

> **Atualização das evidências:** 11/10/2026.

> **Critério de registro:** as informações abaixo distinguem implementação conferida nos arquivos e fatos verificáveis nos PRs de evidências ainda não documentadas. Os campos **[PENDENTE]** devem ser preenchidos com o link real da execução (run), trecho de log ou saída do comando; não representam testes já aprovados.

## Tarefa 1 — CI a cada commit (2,5 pontos)

**Arquivo:** [`.github/workflows/ci.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/ci.yml)

O workflow `ci`, com job `verificar`, foi configurado para executar em todo `push` e em `pull_request` direcionado à `main`. Suas três etapas são: (1) busca por padrões de possíveis segredos versionados com `git grep`; (2) aplicação sequencial dos scripts de `migrations/*.sql` ao banco SQLite local `teste.db`, com `sqlite3`; e (3) validação de arquivos HTML com `npx --yes htmlhint "**/*.html"`.

O [PR #1 — correção do CI e teste de proteção da main](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) registra os commits de criação de uma falha intencional em SQL e sua correção. As capturas das execuções `ci #10` e `ci #66` comprovam, respectivamente, uma falha intencional de SQL e um resultado bem-sucedido. A linha do erro está reproduzida abaixo. Os URLs individuais dos runs devem ser copiados da barra de endereço do GitHub, pois o número sequencial do workflow não corresponde ao identificador numérico da URL.

- **Execução CI aprovada (verde):** `ci #66`, job `verificar`, branch `main`, commit `02158d7`, status **Success**. **[PENDENTE — colar URL individual do run #66]**.
- **Execução CI reprovada (vermelho):** `ci #10`, job `verificar`, evento `pull_request` do PR #1, status **Failure**. **[PENDENTE — colar URL individual do run #10]**.
- **Trecho comprovado do log de falha**, etapa `Aplica as migrações D1 em um SQLite local`:

```text
Parse error near line 2: near ")": syntax error
id INTEGER PRIMARY KEY, nome TEXT NOT NULL, );
Error: Process completed with exit code 1.
```

A vírgula indevida antes do fechamento de `CREATE TABLE` tornou a migração inválida. A etapa de validação HTML não foi executada naquela tentativa porque a verificação das migrações falhou; no run aprovado, o job `verificar` terminou com sucesso.
- **Histórico do teste:** [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), commits `0eadd42` (teste de falha) e `f0f06f8` (correção).

**Por que testar as migrações em SQLite antes de chegar ao D1?** Para detectar erros de sintaxe SQL e inconsistências na sequência das alterações antes de aplicá-las ao banco remoto, reduzindo o risco de interromper a atualização do esquema em produção; esse teste não substitui a verificação final no D1.

## Tarefa 2 — Proteção da `main` pelo status check (1,5 ponto)

A tarefa exige uma regra de proteção ou ruleset que obrigue pull request e aprovação do status check **`verificar`** antes do merge. Essa configuração depende das definições administrativas do repositório e **não é comprovada apenas pelos arquivos do ZIP**. Na tentativa de consulta, a conta da integrante Laura recebeu a mensagem `You don't have access to repository options`, portanto não foi possível visualizar ou editar a regra de proteção. Isso **não demonstra** que a regra inexiste.

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

**Teste de permissão — verificado no histórico e nos logs:**

- [Commit `37eb278`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/37eb278): retirada da linha `pull-requests: write` no workflow de revisão.
- **Run `revisao-ia #10`** (PR #1, evento `synchronize`): a ação de IA concluiu, mas o step `Comenta no PR` falhou com:

```text
GraphQL: Resource not accessible by integration (addComment)
Error: Process completed with exit code 1.
```

- [Commit `a68931b`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/a68931b): restauração de `pull-requests: write`.
- **Run `revisao-ia #26`** (PR #9): execução aprovada, job `revisar`, 51 segundos.
- **Links individuais dos runs:** **[PENDENTE — colar URLs de `revisao-ia #10` e `#26` e, se disponível, dos runs relacionados aos comentários do PR #1]**.
- **Comentários antes e depois da correção:** [conversa do PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1). A IA primeiro apontou interpolação direta de `nome` em `functions/api/teste-sql.js`, recomendando `WHERE aluno = ?` com `.bind(nome)`; depois reconheceu a ausência de SQL inseguro por concatenação. Outros comentários apontaram eventual falha de controle de acesso no endpoint temporário, que foi removido posteriormente.

### Avaliação crítica de uma afirmação da IA

Em comentário posterior à correção no [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), a IA dedicou parte da resposta a explicar a opção `model: ""` da ação `actions/ai-inference@v3` e o comportamento de escolha automática do modelo. Embora essa observação possa ser pertinente à configuração do workflow, **não constitui por si só segredo exposto, SQL inseguro nem erro de lógica no código da aplicação**, que eram os tópicos centrais solicitados no prompt. Trata-se, portanto, de uma digressão de pouca relevância para a revisão do diff; a análise humana é necessária para distinguir problemas acionáveis de comentários acessórios. As demais afirmações de segurança devem igualmente ser confrontadas com o código, antes de serem aceitas como vulnerabilidades efetivas.

## Tarefa 4 — Migrações D1 após merge (1,5 ponto)

**Arquivo:** [`.github/workflows/migrar.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/migrar.yml)

O workflow `migrar` tem gatilho `push` na `main` com filtro `paths: ['migrations/**']`. Seu job usa `npx --yes wrangler d1 migrations apply oauth-sessions-equipe-02 --remote` e recebe as variáveis `CLOUDFLARE_API_TOKEN` e `CLOUDFLARE_ACCOUNT_ID` de `secrets`, sem colocá-las literalmente no arquivo.

A pasta `migrations/` contém `0001_inicial.sql`, que cria a tabela `notas`, e `0002_adiciona_coluna.sql`, que adiciona a coluna `observacao TEXT` à tabela. O `wrangler.toml` declara o binding `DB`, o banco `oauth-sessions-equipe-02` e o diretório de migrações.

- **PR/merge com migrações:** [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), commit de merge `6559a46`, associado ao único run de `migrar` visível nas capturas.
- **Execução de `migrar` após o merge:** `migrar #1`, branch `main`, job `aplicar`, resultado **Success**; step `Aplica migrações no D1 remoto` concluído com sucesso. **[PENDENTE — colar URL individual do run #1]**.
- **Resultado comprovado do log do Wrangler:** o comando `npx --yes wrangler d1 migrations apply oauth-sessions-equipe-02 --remote` indicou `Resource location: remote`; `0001_inicial.sql` e `0002_adiciona_coluna.sql` apareceram com status de sucesso na tabela final do log. Isso comprova **a aplicação remota das duas migrações**, mas não substitui a saída solicitada do `PRAGMA`.
- **Confirmação da coluna remota:** ainda **[PENDENTE]**. O painel Cloudflare acessível à integrante não pertence à conta que hospeda o banco deste repositório; o proprietário ou outra pessoa autorizada deverá executar e registrar:

```bash
npx --yes wrangler d1 execute oauth-sessions-equipe-02 --remote --command "PRAGMA table_info(notas)"
```

**Saída real do `PRAGMA table_info(notas)`: [PENDENTE — anexar o resultado que liste a coluna `observacao`, do tipo `TEXT`; não simular o retorno].**

- **Merge sem mudanças em `migrations/`:** [PR #4 — arquivos alterados](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/4/files), incorporado à `main` pelo commit `91218dc`. A aba **Files changed** registra somente a criação de `TESTE-SEM-MIGRACAO.md`, sem alterações na pasta `migrations/`.
- **Prova de ausência de disparo:** [histórico de `migrar`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/workflows/migrar.yml). Na captura posterior ao merge do PR #4, o histórico apresentava **apenas uma execução**, ligada ao merge do PR #1; não havia nova execução para o commit `91218dc`. O resultado é consistente com o filtro `paths: ['migrations/**']`.

**Bônus:** não foi identificado no `migrar.yml` um step com `actions/ai-inference` que escreva resumo em `$GITHUB_STEP_SUMMARY`. Portanto, o bônus não está implementado no ZIP analisado.

## Perguntas obrigatórias (1,0 ponto)

### 1. O `ci.yml` roda duas vezes quando você faz push numa ramificação com PR aberto. Por quê, e qual execução o status check da `main` considera?

O arquivo possui dois gatilhos independentes: `push` (novo commit na ramificação) e `pull_request` (atualização de um PR dirigido à `main`). Por isso, uma atualização pode iniciar duas execuções; a proteção da `main` considera o check exigido para o commit/PR que está sendo avaliado, normalmente a execução associada ao `pull_request`, conforme a regra de branch e os checks configurados.

### 2. Por que um PR vindo de fork não consegue usar `secrets.COPILOT_PAT`? Qual ataque essa regra impede?

Em eventos `pull_request` originados de forks, os segredos do repositório de destino não são disponibilizados ao código não confiável da contribuição. Isso reduz o risco de alguém abrir um PR malicioso que execute comandos para ler e exfiltrar o token `COPILOT_PAT` ou outras credenciais armazenadas nos secrets.

### 3. Por que o workflow da Tarefa 3 declara `permissions` em vez de usar as permissões padrão do `GITHUB_TOKEN`?

A declaração explícita aplica o princípio do menor privilégio: `contents: read` permite consultar o código e `pull-requests: write` permite publicar comentários no PR. Como as permissões padrão do `GITHUB_TOKEN` podem depender da configuração do repositório, especificá-las torna o requisito verificável e evita conceder acesso maior do que o necessário.

### 4. A revisão da IA pode substituir o status check da Tarefa 2? Justifique com o que você observou no item 4 da Tarefa 3.

Não. A revisão por IA pode produzir alertas úteis, mas também inferências incorretas: no PR #1, a IA desviou parte da análise para a opção `model: ""`, uma configuração que não demonstrava por si só vulnerabilidade no código da aplicação. O status check `verificar` executa testes concretos e reproduzíveis e, quando obrigatório na regra da `main`, impede o merge em caso de falha; a análise da IA deve complementá-lo, não substituí-lo.

## Checklist de evidências antes da entrega

- [x] T1: runs verde e vermelho identificados e erro SQL transcrito.
- [ ] T1: colar URLs individuais dos runs `ci #66` e `ci #10`.
- [ ] T2: PR bloqueado e depois liberado; texto de `git push` direto recusado.
- [x] T3: PR público com comentários da IA antes e depois da correção e parágrafo crítico.
- [x] T3: log do erro de permissão e commits de retirada/restauração verificados.
- [ ] T3: colar URLs individuais dos runs e, se possível, links diretos dos comentários.
- [x] T4: aplicação remota de `0001` e `0002` e merge do PR #4 sem nova execução de `migrar` documentados.
- [ ] T4: colar URL individual do run `migrar #1` e resultado real do `PRAGMA` remoto.
- [x] Quatro respostas correspondentes às perguntas do professor.
- [ ] Criar PR de entrega do `AVALIACAO.md` e conferir CI e revisão por IA nesse PR.


