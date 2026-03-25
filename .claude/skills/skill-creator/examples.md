# Exemplos de Skills - Skill Creator v2.0

## 1. Skill Simples - Revisao de Codigo

```yaml
---
name: review-code
description: Revisa codigo para qualidade, bugs e boas praticas. Use quando quiser revisar mudancas antes de commitar.
argument-hint: "[arquivo-ou-diretorio]"
allowed-tools: Read, Grep, Glob
---
```

```markdown
# Code Review

Revise o codigo em `$ARGUMENTS` (ou os arquivos modificados se nenhum argumento for passado).

## Checklist de Revisao
- Bugs e erros logicos
- Seguranca (OWASP top 10)
- Performance
- Legibilidade e manutencao
- Testes ausentes

## Contexto
Arquivos modificados: !`git diff --name-only`

Forneca feedback estruturado com severidade (critico, importante, sugestao).
```

---

## 2. Skill com Side-Effect - Deploy

```yaml
---
name: deploy
description: Faz deploy da aplicacao para o ambiente especificado. Use para publicar mudancas em staging ou producao.
argument-hint: "[staging|production]"
disable-model-invocation: true
allowed-tools: Read, Bash(npm *), Bash(git *)
---
```

```markdown
# Deploy

Faca o deploy para o ambiente `$0` (default: staging).

## Pre-requisitos
Branch atual: !`git branch --show-current`
Status: !`git status --porcelain`

## Passos
1. Verifique se nao ha mudancas uncommitted
2. Execute os testes: `npm test`
3. Faca o build: `npm run build`
4. Execute o deploy: `npm run deploy:$0`
5. Confirme que o deploy foi bem-sucedido
```

---

## 3. Skill de Background Knowledge

```yaml
---
name: project-conventions
description: Convencoes e padroes do projeto. Regras de estilo, estrutura de pastas e padroes de codigo.
user-invocable: false
allowed-tools: Read, Grep, Glob
---
```

```markdown
# Convencoes do Projeto

Sempre siga estas convencoes ao escrever codigo neste projeto:

## Estilo
- TypeScript strict mode
- Funcoes puras quando possivel
- Nomes descritivos em ingles
- Arquivos em kebab-case

## Estrutura
- `src/components/` - Componentes React
- `src/hooks/` - Custom hooks
- `src/utils/` - Funcoes utilitarias
- `src/types/` - Tipos TypeScript

## Testes
- Testes junto ao arquivo: `*.test.ts`
- Minimo 80% de cobertura
- Use `vitest` para testes unitarios
```

---

## 4. Skill com Subagent - Analise de PR

```yaml
---
name: analyze-pr
description: Analisa um pull request em profundidade. Use para revisao detalhada de PRs com analise de impacto.
argument-hint: "[numero-do-pr]"
allowed-tools: Read, Grep, Glob, Bash(gh *)
context: fork
agent: Explore
---
```

```markdown
# Analise de Pull Request

Analise o PR #$0 em profundidade.

## Informacoes do PR
!`gh pr view $0 --json title,body,files,additions,deletions`

## Analise Requerida
1. **Resumo**: O que o PR faz?
2. **Impacto**: Quais areas do codigo sao afetadas?
3. **Riscos**: Identifique possiveis problemas
4. **Testes**: Os testes cobrem as mudancas?
5. **Sugestoes**: Melhorias recomendadas

Explore o codebase para entender o contexto completo das mudancas.
```

---

## 5. Skill com Hooks

```yaml
---
name: safe-refactor
description: Refatora codigo com validacao automatica. Executa testes antes e depois da refatoracao.
argument-hint: "[arquivo] [descricao-da-refatoracao]"
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(npm test)
hooks:
  PreToolUse:
    - matcher: Edit
      hooks:
        - type: command
          command: echo "Editando arquivo..."
  PostToolUse:
    - matcher: Edit
      hooks:
        - type: command
          command: npm test 2>&1 | tail -5
---
```

```markdown
# Safe Refactor

Refatore `$0` conforme descrito: $1

## Processo
1. Leia e entenda o codigo atual
2. Execute os testes para garantir que passam
3. Faca a refatoracao incrementalmente
4. Verifique que os testes continuam passando apos cada mudanca

## Regras
- Nao mude comportamento externo (mesma interface publica)
- Mantenha ou melhore a cobertura de testes
- Commits atomicos por mudanca logica
```

---

## 6. Skill com Dynamic Context Injection

```yaml
---
name: fix-ci
description: Diagnostica e corrige falhas no CI/CD. Use quando o pipeline falhar.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(gh *), Bash(npm *)
---
```

```markdown
# Fix CI

Diagnostique e corrija a falha no CI.

## Contexto Atual
Branch: !`git branch --show-current`
Ultimo commit: !`git log --oneline -1`
Status do CI: !`gh run list --limit 1 --json status,conclusion,name`

## Logs de Erro
!`gh run list --limit 1 --json databaseId --jq '.[0].databaseId' | xargs -I{} gh run view {} --log-failed 2>/dev/null | tail -50`

## Processo
1. Analise os logs de erro acima
2. Identifique a causa raiz
3. Faca a correcao necessaria
4. Verifique localmente se a correcao resolve o problema
```

---

## 7. Skill com Modelo Especifico - Triagem Rapida

```yaml
---
name: triage
description: Triagem rapida de issues e bugs. Classifica severidade e sugere responsavel.
argument-hint: "[numero-da-issue]"
model: claude-haiku-4-5-20251001
effort: low
allowed-tools: Bash(gh *)
---
```

```markdown
# Triagem de Issue

Faca uma triagem rapida da issue #$0.

!`gh issue view $0 --json title,body,labels,assignees`

Classifique:
- **Severidade**: critica / alta / media / baixa
- **Tipo**: bug / feature / melhoria / documentacao
- **Esforco estimado**: pequeno / medio / grande
- **Sugestao de labels**: liste as labels apropriadas
```

---

## 8. Skill com Planejamento - Arquitetura

```yaml
---
name: plan-feature
description: Planeja a arquitetura e implementacao de uma nova feature. Use antes de comecar a implementar algo grande.
argument-hint: "[descricao-da-feature]"
context: fork
agent: Plan
model: claude-opus-4-6
effort: high
allowed-tools: Read, Grep, Glob
---
```

```markdown
# Planejamento de Feature

Planeje a implementacao de: $ARGUMENTS

## Analise o Codebase
Explore a estrutura do projeto para entender:
- Arquitetura atual
- Padroes existentes
- Pontos de integracao

## Entregaveis
1. **Visao geral**: Resumo da abordagem
2. **Arquitetura**: Componentes e suas interacoes
3. **Plano de implementacao**: Passos ordenados com dependencias
4. **Arquivos afetados**: Lista de arquivos a criar/modificar
5. **Riscos**: Pontos de atencao e trade-offs
6. **Estimativa**: Tamanho relativo (P/M/G)
```
