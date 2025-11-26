---
name: git
description: Guia para usar git de acordo com minhas preferências. Use isso quando for solicitado a commitar algo.
license: MIT
---

# git

A maior parte do uso do git é o que você já conhece, então baseie-se nisso. Esta habilidade é apenas um refinamento.

## Nomeação de branches

Apenas nomeie a branch com uma frase curta separada por hífens. Exemplo: `add-some-feature`. Não use prefixos como `feat/`, `hotfix/` etc.

## Mensagens de commit

- Sempre envolva identificadores de código com crases. Exemplo: "Add `html.UserPage` component"
- Sempre refira-se a identificadores de código Go incluindo o nome do pacote, como em `html.UserPage` acima. Campos e métodos em structs podem ser referidos com `model.User.Name`.
- Pergunte-me sobre quaisquer issues do Github que devam ser referenciadas. Referencie-as no final da mensagem de commit assim: "See #123, #234". Se o commit corrigir uma ou mais issues, use "Fixes #123, fixes #234" (o duplo "fixes" é importante para o Github realmente fechar a issue).
- Não mencione que você atualizou testes, isso já é assumido.

## Realizando Commits

- Não faça amend em commits anteriores a menos que instruído. Ao commitar após o primeiro commit em uma branch, apenas faça o commit com uma mensagem simples (ex: "fixing …"), porque a branch na maioria das vezes sofrerá squash no Github de qualquer forma.
