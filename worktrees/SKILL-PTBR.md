---
name: worktrees
description: Guia para usar git worktrees para paralelizar o desenvolvimento com agentes de codificação. Use esta habilidade quando o usuário solicitar trabalhar em uma nova worktree ou quiser trabalhar em uma feature separada isoladamente (ex: "Trabalhe em uma nova worktree", "Crie uma worktree para a feature X").
---

# Git Worktrees para Desenvolvimento Paralelo

## Visão Geral

Esta habilidade permite o desenvolvimento paralelo usando git worktrees. Cada worktree fornece um diretório de trabalho isolado com sua própria branch, permitindo que múltiplos agentes trabalhem em diferentes funcionalidades simultaneamente sem conflitos.

## Quando Usar Esta Habilidade

Use esta habilidade quando:
- O usuário solicitar explicitamente trabalhar em uma nova worktree (ex: "Trabalhe em uma nova worktree")
- O usuário quiser desenvolver uma funcionalidade isoladamente enquanto preserva o diretório de trabalho principal
- Múltiplos agentes precisarem trabalhar em tarefas diferentes em paralelo

## Fluxo de Trabalho

### 1. Determinar o Nome da Branch

Escolha um nome de branch descritivo para a funcionalidade ou tarefa a ser trabalhada. O nome da branch deve seguir as convenções de nomenclatura padrão do git (minúsculas, separadas por hífens, ex: `add-user-authentication`, `fix-login-bug`).

### 2. Criar Worktree

Crie uma nova worktree no diretório `.worktrees/` dentro do projeto atual:

```bash
git worktree add .worktrees/<nome-da-branch> -b <nome-da-branch>
```

Este comando:
- Cria um novo diretório em `.worktrees/<nome-da-branch>`
- Cria e faz checkout de uma nova branch chamada `<nome-da-branch>`
- Vincula a worktree ao repositório atual

### 3. Mudar para a Worktree

Altere o diretório de trabalho para a worktree recém-criada:

```bash
cd .worktrees/<nome-da-branch>
```

### 4. Trabalhar em Isolamento

Prossiga com as tarefas de desenvolvimento na worktree. Este ambiente é completamente isolado do diretório de trabalho principal, permitindo trabalho independente sem interferência.

Todas as operações padrão do git (commit, push, pull, etc.) funcionam normalmente dentro da worktree.

**Nota:** Se este projeto executa serviços (aplicações web, docker-compose, etc.), veja [apps.md](apps.md) para etapas de configuração, incluindo cópia de arquivos de ambiente, alocação de portas e inicialização de serviços.

### 5. Listar Worktrees Ativas (Opcional)

Para visualizar todas as worktrees ativas:

```bash
git worktree list
```

Isso exibe todas as worktrees, seus caminhos e as branches em que estão.

### 6. Remover Worktree (Opcional)

Quando terminar com uma worktree, você pode removê-la:

```bash
git worktree remove .worktrees/<nome-da-branch>
```

**Nota:** Não remova worktrees automaticamente. Deixe essa decisão para o usuário. Se a worktree estiver executando serviços (veja [apps.md](apps.md)), certifique-se de parar esses serviços antes de remover a worktree.

## Notas Importantes

- O diretório `.worktrees/` deve ser adicionado ao `.gitignore` se ainda não estiver presente
- Cada worktree mantém seu próprio diretório de trabalho, mas compartilha o mesmo repositório git
- Worktrees permitem verdadeiro desenvolvimento paralelo sem a necessidade de stashing ou troca de branch
- Após criar e mudar para uma worktree, informe ao usuário o novo caminho do diretório de trabalho
