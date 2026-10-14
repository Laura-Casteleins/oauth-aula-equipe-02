# AVALIAÇÃO — GitHub Actions com IA

**Disciplina:** Desenvolvimento Web  
**Equipe:** Laura Gois Casteleins e Gabriel Henrique Mendes — Engenharia de Software  
**Repositório:** [gabriel240705/oauth-aula-equipe-02](https://github.com/gabriel240705/oauth-aula-equipe-02)  
**Data de revisão das evidências:** 14/10/2026

## Visão geral

O projeto utiliza GitHub Actions para executar integração contínua (CI), proteger a branch `main`, revisar pull requests com inteligência artificial e aplicar migrações ao banco Cloudflare D1. Os testes foram realizados em branches de trabalho e documentados com histórico de commits, pull requests, logs e consultas ao banco remoto.

| Item avaliado | Pontuação | Evidência principal |
|---|---:|---|
| Tarefa 1 — CI a cada commit | 2,5 | Execuções [aprovada](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38139609439) e [reprovada](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/37999911145) |
| Tarefa 2 — Proteção da `main` | 1,5 | [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) e recusa `GH013` |
| Tarefa 3 — Revisão por IA | 3,5 | [Comentários no PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1) e [teste de permissão](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38002448463) |
| Tarefa 4 — Migrações D1 | 1,5 | [Workflow `migrar #1`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38005176585) e consulta `PRAGMA` |
| Quatro perguntas obrigatórias | 1,0 | Respostas na última seção |

---

## Tarefa 1 — CI a cada commit (2,5 pontos)

**Workflow:** [`.github/workflows/ci.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/ci.yml)  
**Job:** `verificar`

O workflow é acionado em eventos `push` e em eventos `pull_request` direcionados à `main`. O job reúne três verificações:

1. **Segredos versionados:** procura, com `git grep`, padrões que possam indicar credenciais expostas no repositório.
2. **Migrações SQL:** executa sequencialmente os arquivos `migrations/*.sql` sobre um banco SQLite local, verificando se os scripts podem ser aplicados sem erro.
3. **Validação HTML:** utiliza `npx --yes htmlhint "**/*.html"` para analisar os arquivos HTML.

### Evidências de sucesso e falha

- **Execução aprovada:** [`ci #66 — Success`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38139609439), job `verificar`, na `main`.
- **Execução reprovada:** [`ci #10 — Failure`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/37999911145), associada ao [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1).
- **Commits de teste:** [`0eadd42`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/0eadd42) (falha intencional) e [`f0f06f8`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/f0f06f8) (correção).

Na execução reprovada, foi acrescentada intencionalmente uma vírgula indevida na definição de uma tabela. O log registrou:

```text
Parse error near line 2: near ")": syntax error
id INTEGER PRIMARY KEY, nome TEXT NOT NULL, );
Error: Process completed with exit code 1.
```

A falha fez o job parar antes da validação HTML. Após a correção do SQL, o CI voltou a passar. **Testar primeiro em SQLite local** permite identificar erros de sintaxe e de sequência de migrações antes da aplicação no D1 remoto; essa validação não substitui o teste efetivo no ambiente remoto.

---

## Tarefa 2 — Proteção da `main` pelo status check (1,5 ponto)

Foi configurado o ruleset **`Protecao-main`**, aplicado à branch `main`, com exigência de pull request e aprovação do status check **`verificar`** antes do merge.

### Teste: bloqueio e liberação do PR

No [PR #1 — Correção do CI e teste de proteção da main](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), o SQL propositalmente inválido levou à reprovação de `ci / verificar`. O GitHub exibiu o check como **Required** e manteve o botão de merge desabilitado. Depois da correção da migração na **mesma branch**, os checks obrigatórios passaram, não havia conflitos e o botão **Merge pull request** foi habilitado. O PR foi posteriormente integrado à `main` pelo commit [`6559a46`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/6559a46).

### Teste: tentativa de push direto

Foi criado localmente o commit `8e27d2a` com o arquivo temporário `TESTE-PUSH-DIRETO.txt`. O comando `git push origin main` foi recusado pelo GitHub, conforme a saída real do Terminal:

```text
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - Changes must be made through a pull request.
remote: - Required status check "verificar" is expected.
! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs
```

O commit de teste **não foi enviado** à `main` e foi descartado da cópia local com `git reset --hard origin/main`. Em seguida, `git status` confirmou que a cópia local estava sincronizada e sem alterações pendentes.

**Conclusão:** o ruleset impediu a alteração direta da `main` e exigiu que a integração passasse por pull request e pela verificação obrigatória.

---

## Tarefa 3 — Revisão de pull requests por IA (3,5 pontos)

**Workflow:** [`.github/workflows/revisao-ia.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/revisao-ia.yml)

O workflow é acionado ao abrir ou atualizar um PR (`opened` e `synchronize`). Ele coleta o diff, prepara um prompt para identificar segredos expostos, SQL construído por concatenação e possíveis erros de lógica, executa a revisão usando `actions/ai-inference@v3` e publica o resultado com `gh pr comment`.

Foram definidas permissões explícitas: `contents: read` para ler o repositório e `pull-requests: write` para permitir comentários no PR. A versão `v3` foi adotada após problemas encontrados na execução da versão anterior; foi configurado `model: ""` para permitir que o Copilot CLI utilize sua escolha padrão de modelo.

### Experimento 1 — SQL vulnerável identificado

No [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), foi criado temporariamente o arquivo `functions/api/teste-sql.js`, contendo uma consulta SQL com interpolação direta de `nome`. A IA apontou uma vulnerabilidade de **alta severidade**, referenciou as linhas da consulta e recomendou o uso de parâmetros associados à instrução SQL.

Trecho representativo da correção recomendada:

```javascript
const resultado = await env.DB.prepare(
  "SELECT * FROM notas WHERE aluno = ?"
).bind(nome).all();
```

### Experimento 2 — Revisão após correção

O mesmo arquivo foi corrigido com `prepare().bind(nome)`, e a IA reconheceu que a consulta não era mais montada por concatenação. Ela também observou que o endpoint de demonstração continuava sem autenticação/autorização, uma questão distinta da injeção SQL. **O endpoint temporário foi removido antes de integrar o PR à `main`.**

- **Histórico com os dois comentários automáticos:** [PR #1 — Conversation](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1).

### Experimento 3 — Falha por falta de permissão

A permissão `pull-requests: write` foi removida temporariamente no commit [`37eb278`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/37eb278). A IA conseguiu realizar sua análise, mas a etapa **Comenta no PR** não pôde publicar a mensagem:

```text
GraphQL: Resource not accessible by integration (addComment)
Error: Process completed with exit code 1.
```

- **Execução reprovada:** [`revisao-ia #10 — Failure`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38002448463).
- **Restauração da permissão:** commit [`a68931b`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/a68931b).
- **Execução aprovada após restauração:** [`revisao-ia #26 — Success`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38139535047), no PR #9.

Esse teste demonstrou a diferença entre **conseguir gerar a análise** e **ter autorização para publicá-la** no pull request.

### Avaliação crítica da IA

Em uma revisão, a IA afirmou que `model: ""` poderia causar falhas por não indicar um modelo válido. Entretanto, o workflow com essa configuração executou e publicou comentários; posteriormente, outra revisão explicou que a string vazia permite o uso do modelo padrão do Copilot CLI. A primeira observação, portanto, não deveria ser tratada como uma falha confirmada sem confrontá-la com a implementação e o resultado real dos testes.

Além disso, a configuração do modelo não representa, por si só, um segredo exposto ou uma consulta SQL vulnerável. **A revisão por IA é útil como apoio, mas seus achados exigem verificação humana.**

---

## Tarefa 4 — Migrações D1 após merge (1,5 ponto)

**Workflow:** [`.github/workflows/migrar.yml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/.github/workflows/migrar.yml)  
**Configuração:** [`wrangler.toml`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/wrangler.toml)  
**Banco D1:** `oauth-sessions-equipe-02`  
**Binding:** `DB`

O workflow `migrar` executa no evento `push` da `main` somente quando há mudanças em `migrations/**`. Ele utiliza os secrets `CLOUDFLARE_API_TOKEN` e `CLOUDFLARE_ACCOUNT_ID` e aplica as migrações com:

```bash
npx --yes wrangler d1 migrations apply oauth-sessions-equipe-02 --remote
```

O projeto possui as migrações:

- [`0001_inicial.sql`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/migrations/0001_inicial.sql): cria a tabela `notas`.
- [`0002_adiciona_coluna.sql`](https://github.com/gabriel240705/oauth-aula-equipe-02/blob/main/migrations/0002_adiciona_coluna.sql): acrescenta a coluna `observacao TEXT`.

### Experimento 1 — Merge com mudança em `migrations/`

Após integrar o [PR #1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/1), commit `6559a46`, o GitHub Actions disparou o workflow `migrar`.

- **Execução:** [`migrar #1 — Success`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/runs/38005176585).
- **Resultado:** o job `aplicar` foi aprovado, e o log do Wrangler indicou aplicação no banco remoto, incluindo `0001_inicial.sql` e `0002_adiciona_coluna.sql`.

### Experimento 2 — Verificação no D1 remoto

A consulta abaixo foi executada diretamente no **Console do Cloudflare D1**, no banco `oauth-sessions-equipe-02`, em 09/10/2026:

```sql
PRAGMA table_info(notas);
```

**Resultado observado:**

| Coluna | Tipo | Observação |
|---|---|---|
| `id` | INTEGER | Chave primária |
| `aluno` | TEXT | Obrigatória |
| `nota` | REAL | Obrigatória |
| `observacao` | TEXT | Nova coluna adicionada pela migração |

A consulta confirmou a presença da coluna `observacao`, de tipo `TEXT`, no banco remoto. O esquema foi consultado sem modificar os registros existentes.

### Experimento 3 — Merge sem mudança em `migrations/`

O [PR #4 — Teste de merge sem alterações no D1](https://github.com/gabriel240705/oauth-aula-equipe-02/pull/4/files) adicionou somente o arquivo `TESTE-SEM-MIGRACAO.md`. O merge gerou o commit [`91218dc`](https://github.com/gabriel240705/oauth-aula-equipe-02/commit/91218dc), com CI aprovado.

No [histórico do workflow `migrar`](https://github.com/gabriel240705/oauth-aula-equipe-02/actions/workflows/migrar.yml), a verificação realizada após esse merge mostrou que **não houve nova execução** de `migrar` para o commit `91218dc`. Isso é consistente com o filtro `paths: ['migrations/**']`: alterações fora dessa pasta não acionam o workflow.

---

## Perguntas obrigatórias (1,0 ponto)

### 1. Por que o `ci.yml` roda duas vezes quando fazemos push em uma branch com PR aberto? Qual execução o status check da `main` considera?

O arquivo possui dois gatilhos independentes: `push`, quando um commit é enviado à branch, e `pull_request`, quando o PR direcionado à `main` é atualizado. Por isso uma mesma alteração pode iniciar duas execuções; a proteção exige que o status check `verificar` esteja aprovado no contexto do commit/PR avaliado, considerando os checks configurados no ruleset.

### 2. Por que um PR vindo de fork não consegue acessar `secrets.COPILOT_PAT`? Qual ataque essa regra impede?

Nos eventos `pull_request` originados de forks, os segredos do repositório de destino não são fornecidos ao código não confiável da contribuição. Essa proteção dificulta que alguém envie um PR malicioso para executar comandos e extrair o token `COPILOT_PAT` ou outras credenciais.

### 3. Por que o workflow de revisão por IA declara `permissions` em vez de usar as permissões padrão do `GITHUB_TOKEN`?

Para aplicar o princípio do menor privilégio e declarar explicitamente os acessos necessários: `contents: read` para ler o código e `pull-requests: write` para comentar no PR. As permissões padrão do `GITHUB_TOKEN` variam conforme as configurações do repositório; declará-las deixa o comportamento mais previsível e verificável.

### 4. A revisão da IA pode substituir o status check da Tarefa 2? Justifique com os erros observados na Tarefa 3.

Não. A IA pode gerar alertas úteis, mas também observações incorretas ou pouco relevantes, como o alerta inicial sobre `model: ""`, que não representava uma falha confirmada. O status check `verificar` executa testes objetivos e reproduzíveis e, quando obrigatório, bloqueia o merge em caso de falha; a IA deve complementar essa verificação, não substituí-la.

---

## Conclusão

Os experimentos demonstraram o funcionamento das quatro etapas: CI capaz de detectar uma migração inválida, proteção da `main` contra merge com check reprovado e contra push direto, revisão automática com IA e controle explícito de permissões, e migração remota ao D1 acionada apenas quando a pasta `migrations/` foi alterada. Os testes foram documentados com links para pull requests e execuções, além do log de recusa `GH013` e do resultado real da consulta ao banco.

**Observação:** o bônus opcional de resumo por IA em `$GITHUB_STEP_SUMMARY` não foi implementado. A pontuação efetiva será definida pelo professor com base no enunciado e nas evidências apresentadas.
