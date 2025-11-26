---
name: nanobanana
description: Guia para gerar e editar imagens usando IA generativa com a CLI nanobanana
license: MIT
---

# Nanobanana

Este é um guia para gerar e editar imagens usando a ferramenta de linha de comando (CLI) nanobanana.

Nanobanana é uma interface de linha de comando para a API de geração de imagem Nano Banana, que utiliza os modelos de IA generativa do Google.

## Instalação

Se a CLI nanobanana ainda não estiver instalada, instale-a usando:

```bash
go install maragu.dev/nanobanana@latest
```

## Pré-requisitos

A variável de ambiente `GOOGLE_API_KEY` deve estar configurada, ou um arquivo `.env` com a chave deve estar presente no diretório de trabalho.

## Gerando imagens

Para gerar uma única imagem:

```bash
nanobanana generate output.png "um belo pôr do sol sobre as montanhas"
```

A saída pode ser um arquivo `.png` ou `.jpg`.

## Gerando múltiplas variações

Para gerar múltiplas variações de uma imagem:

```bash
nanobanana generate -count 3 output.png "arte abstrata com cores vibrantes"
```

Ao gerar múltiplas imagens, o nome do arquivo de saída será modificado para incluir um sufixo numérico (ex: `output-1.png`, `output-2.png`, `output-3.png`).

## Editando imagens existentes

Para editar ou modificar uma imagem existente usando um prompt de texto:

```bash
nanobanana generate -i input.png output.png "faça o céu roxo e adicione estrelas"
```

Isso é útil para fazer alterações específicas em imagens existentes com base em instruções em linguagem natural.

## Dicas para prompts eficazes

- Seja específico e descritivo sobre o que você deseja na imagem
- Inclua detalhes sobre estilo, cores, clima, composição e assunto
- Para edições, descreva claramente o que deve mudar enquanto o restante permanece o mesmo
- Você pode referenciar estilos de arte, artistas ou estéticas visuais específicas

## Exemplos

Gerar um logotipo:
```bash
nanobanana generate logo.png "logotipo de empresa de tecnologia minimalista com formas geométricas em azul e branco"
```

Criar arte conceitual:
```bash
nanobanana generate concept.png "horizonte de cidade futurista à noite com luzes de neon e veículos voadores, estilo cyberpunk"
```

Editar uma foto existente:
```bash
nanobanana generate -i photo.png enhanced.png "realce as cores e faça parecer com a hora dourada"
```

Gerar múltiplas opções:
```bash
nanobanana generate -count 5 icon.png "ícone simples de uma banana, design flat, amarelo e branco"
```
