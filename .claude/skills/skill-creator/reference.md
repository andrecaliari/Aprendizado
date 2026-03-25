# Referencia Completa - Skill Creator v2.0

## Campos do Frontmatter

### name (string)
- **Obrigatorio**: Nao, mas altamente recomendado
- **Formato**: lowercase, hyphens e numeros apenas
- **Max**: 64 caracteres
- **Uso**: Define o nome do slash command (`/nome-da-skill`)
- **Exemplo**: `name: deploy-api`

### description (string)
- **Obrigatorio**: Altamente recomendado
- **Max**: 1024 caracteres
- **Uso**: Claude usa para decidir quando invocar automaticamente. Inclua keywords naturais.
- **Exemplo**: `description: Deploy da API para producao. Use quando estiver pronto para publicar mudancas.`

### argument-hint (string)
- **Obrigatorio**: Nao
- **Uso**: Dica mostrada no autocomplete ao digitar o slash command
- **Exemplo**: `argument-hint: "[ambiente] [versao]"`

### disable-model-invocation (boolean)
- **Default**: false
- **Uso**: Quando `true`, apenas o usuario pode invocar (via `/nome`). Claude nao invoca automaticamente.
- **Quando usar**: Skills com side-effects (deploy, commit, envio de mensagens, delecao)

### user-invocable (boolean)
- **Default**: true
- **Uso**: Quando `false`, skill nao aparece no menu `/`. Apenas Claude pode invocar.
- **Quando usar**: Conhecimento de background que Claude deve usar automaticamente.

### allowed-tools (string list)
- **Obrigatorio**: Nao
- **Uso**: Ferramentas que Claude pode usar sem pedir permissao quando a skill esta ativa
- **Opcoes comuns**:
  - `Read, Grep, Glob` - somente leitura
  - `Read, Write, Edit, Grep, Glob` - leitura e escrita de arquivos
  - `Read, Write, Edit, Bash` - acesso completo
  - `Bash(gh *)` - apenas comandos GitHub CLI
  - `Bash(npm *), Bash(npx *)` - apenas comandos npm

### model (string)
- **Obrigatorio**: Nao
- **Uso**: Modelo especifico para executar a skill
- **Opcoes**:
  - `claude-opus-4-6` - mais capaz, mais lento
  - `claude-sonnet-4-6` - equilibrio entre capacidade e velocidade
  - `claude-haiku-4-5-20251001` - mais rapido, menor custo

### effort (string)
- **Obrigatorio**: Nao
- **Opcoes**: `low`, `medium`, `high`, `max`
- **Nota**: `max` disponivel apenas com Opus 4.6
- **Uso**: Controla profundidade de raciocinio do modelo

### context (string)
- **Obrigatorio**: Nao
- **Opcoes**: `fork`
- **Uso**: Executa a skill em um subagent isolado, protegendo o contexto principal
- **Quando usar**: Skills de pesquisa intensiva, analise de PRs, tarefas que geram muito output

### agent (string)
- **Obrigatorio**: Nao (mas recomendado quando `context: fork`)
- **Opcoes**:
  - `Explore` - otimizado para explorar codebases (Glob, Grep, Read)
  - `Plan` - otimizado para planejamento de implementacao
  - `general-purpose` - agente generico com todas as ferramentas
- **Uso**: Define o tipo de subagent quando `context: fork` esta ativo

### hooks (object)
- **Obrigatorio**: Nao
- **Uso**: Hooks que executam durante o ciclo de vida da skill
- **Exemplo**:
```yaml
hooks:
  PreToolUse:
    - matcher: Bash
      hooks:
        - type: command
          command: echo "Executando bash..."
```

---

## Locais de Armazenamento

| Escopo | Caminho | Quando usar |
|--------|---------|-------------|
| **Projeto** | `.claude/skills/<nome>/SKILL.md` | Skills especificas do projeto |
| **Pessoal** | `~/.claude/skills/<nome>/SKILL.md` | Skills que voce usa em todos os projetos |
| **Enterprise** | Via managed settings | Skills organizacionais |
| **Plugin** | `<plugin>/skills/<nome>/SKILL.md` | Skills distribuidas como plugin |

---

## Variaveis Dinamicas

### Substituicao de Argumentos
- `$ARGUMENTS` - todos os argumentos como string
- `$0`, `$1`, `$2`, ... - argumento por indice (0-based)
- `$ARGUMENTS[0]`, `$ARGUMENTS[1]`, ... - sintaxe alternativa

### Variaveis de Ambiente
- `${CLAUDE_SESSION_ID}` - ID unico da sessao
- `${CLAUDE_SKILL_DIR}` - caminho absoluto do diretorio da skill

### Injecao Dinamica de Contexto
Sintaxe: `` !`comando` ``

O comando shell e executado ANTES de Claude processar a skill. O output substitui o placeholder.

**Exemplo**:
```markdown
## Contexto do Branch
Branch atual: !`git branch --show-current`
Ultimos commits: !`git log --oneline -5`
Arquivos modificados: !`git diff --name-only`
```

---

## Matriz de Invocacao

| Configuracao | Usuario pode invocar | Claude pode invocar |
|-------------|---------------------|---------------------|
| Padrao | Sim (`/nome`) | Sim (automatico) |
| `disable-model-invocation: true` | Sim (`/nome`) | Nao |
| `user-invocable: false` | Nao | Sim (automatico) |

---

## Restricoes de Ferramentas (allowed-tools)

### Padroes Comuns

**Somente leitura (mais seguro)**:
```yaml
allowed-tools: Read, Grep, Glob
```

**Leitura e escrita de arquivos**:
```yaml
allowed-tools: Read, Write, Edit, Grep, Glob
```

**Acesso completo com bash**:
```yaml
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
```

**Bash restrito a comandos especificos**:
```yaml
allowed-tools: Bash(git *), Bash(npm *)
```

**GitHub CLI**:
```yaml
allowed-tools: Bash(gh *)
```

### Prefixos de Permissao

Dentro de `allowed-tools`, voce pode usar prefixos para restringir:
- `Bash(git *)` - apenas comandos git
- `Bash(npm test)` - apenas `npm test`
- `Bash(docker *)` - apenas comandos docker
