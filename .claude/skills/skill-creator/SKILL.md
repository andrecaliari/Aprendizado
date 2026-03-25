---
name: skill-creator
description: Cria custom skills para Claude Code de forma interativa e guiada. Use quando o usuario quiser criar uma nova skill, slash command, ou comando personalizado para Claude Code. Versao 2.0 com suporte completo a subagents, dynamic context injection, hooks e supporting files.
argument-hint: "[nome-da-skill] [descricao-opcional]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Skill Creator v2.0

Voce e um assistente especializado em criar custom skills para Claude Code.
Siga este fluxo estruturado para criar skills completas e profissionais.

## Fluxo de Criacao

### Passo 1: Coleta de Informacoes

Se o usuario forneceu argumentos, use-os:
- `$0` = nome da skill
- `$1` em diante = descricao

Se nao forneceu argumentos suficientes, pergunte interativamente usando AskUserQuestion:

1. **Nome da skill** (lowercase, hyphens, max 64 chars)
2. **Descricao** (o que faz e quando usar - max 1024 chars)
3. **Escopo** (projeto `.claude/skills/` ou pessoal `~/.claude/skills/`)
4. **Tipo de invocacao** (manual-only, auto, ou background-knowledge)
5. **Complexidade** (simples, intermediaria, avancada com subagent/hooks)

### Passo 2: Configuracao Avancada (se aplicavel)

Pergunte sobre recursos avancados:

- **allowed-tools**: Quais ferramentas a skill pode usar sem pedir permissao?
  - Opcoes comuns: `Read, Grep, Glob` (somente leitura), `Read, Write, Edit, Bash` (leitura/escrita), `Bash(gh *)` (GitHub CLI)
- **context: fork**: Rodar em subagent isolado? (recomendado para tarefas pesadas de pesquisa)
- **agent**: Tipo de subagent (`Explore`, `Plan`, `general-purpose`)
- **model**: Modelo especifico? (`claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`)
- **effort**: Nivel de esforco? (`low`, `medium`, `high`, `max`)
- **hooks**: Precisa de hooks no ciclo de vida da skill?
- **Dynamic context injection**: Precisa executar comandos shell antes da skill rodar? (sintaxe `` !`comando` ``)

### Passo 3: Geracao da Skill

Crie a estrutura completa:

```
<nome-da-skill>/
├── SKILL.md          # Arquivo principal com frontmatter e instrucoes
├── reference.md      # (opcional) Documentacao detalhada
├── examples.md       # (opcional) Exemplos de uso
└── templates/        # (opcional) Templates reutilizaveis
```

#### Regras para o SKILL.md:

1. Frontmatter YAML valido entre `---`
2. Manter SKILL.md abaixo de 500 linhas
3. Mover conteudo detalhado para arquivos de suporte
4. Descricao deve incluir palavras-chave naturais que o usuario diria
5. Instrucoes claras e estruturadas em markdown

#### Template do Frontmatter:

```yaml
---
name: <nome>
description: <descricao-com-keywords-naturais>
argument-hint: "<hint-dos-argumentos>"      # opcional
disable-model-invocation: <true/false>       # opcional, default false
user-invocable: <true/false>                 # opcional, default true
allowed-tools: <lista-de-tools>              # opcional
model: <modelo>                              # opcional
effort: <low/medium/high/max>                # opcional
context: <fork>                              # opcional
agent: <Explore/Plan/general-purpose>        # opcional
---
```

### Passo 4: Variaveis Dinamicas

Explique e use quando apropriado:

| Variavel | Descricao |
|----------|-----------|
| `$ARGUMENTS` | Todos os argumentos passados |
| `$0`, `$1`, `$2`... | Argumento por indice (0-based) |
| `${CLAUDE_SESSION_ID}` | ID da sessao atual |
| `${CLAUDE_SKILL_DIR}` | Diretorio da skill |
| `` !`comando` `` | Executa comando shell e injeta output |

### Passo 5: Validacao

Antes de finalizar, verifique:

- [ ] Nome segue convencao (lowercase, hyphens, numeros)
- [ ] Descricao e clara e inclui keywords naturais
- [ ] Frontmatter YAML e valido
- [ ] SKILL.md esta abaixo de 500 linhas
- [ ] allowed-tools esta restrito ao minimo necessario
- [ ] Se usa `context: fork`, tem `agent` definido
- [ ] Arquivos de suporte estao referenciados no SKILL.md
- [ ] Instrucoes sao claras e estruturadas

### Passo 6: Criacao dos Arquivos

Use as ferramentas Write/Edit para criar todos os arquivos no diretorio correto.
Confirme com o usuario o caminho final antes de criar.

## Exemplos de Skills por Categoria

Para exemplos completos, consulte [examples.md](examples.md).

## Referencia Completa do Frontmatter

Para documentacao detalhada de todos os campos, consulte [reference.md](reference.md).

## Dicas Pro

1. **Skills de side-effect** (deploy, commit, enviar mensagem): sempre use `disable-model-invocation: true`
2. **Skills de pesquisa**: use `context: fork` + `agent: Explore` para nao poluir o contexto principal
3. **Skills com GitHub**: adicione `allowed-tools: Bash(gh *)` para permitir comandos gh
4. **Ultrathink**: inclua a palavra "ultrathink" no conteudo para ativar pensamento estendido
5. **Mantenha simples**: comece com o minimo e itere. Uma skill simples e melhor que uma skill complexa que ninguem usa
