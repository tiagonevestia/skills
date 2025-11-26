---
name: decisions
description: Guia para registrar decisões arquiteturais e de design significativas em docs/decisions.md. Use esta habilidade quando decisões arquiteturais claramente significativas forem tomadas (escolhas de banco de dados, frameworks, padrões de design principais) ou quando for explicitamente solicitado a documentar uma decisão. Seja conservador - sugira apenas para decisões importantes, não para detalhes menores de implementação.
license: MIT
---

# Decisões

## Visão Geral

Esta habilidade mantém um registro cronológico de decisões significativas do projeto em `docs/decisions.md`. Ela captura decisões importantes de arquitetura e design, especialmente aquelas que envolvem compensações (tradeoffs), para criar um registro permanente de por que as principais escolhas foram feitas.

## Quando Usar Esta Habilidade

**Proativo (Conservador):**
Sugira registrar decisões apenas quando houver uma escolha arquitetural claramente significativa, como:
- Escolha entre sistemas de banco de dados (ex: SQLite vs PostgreSQL)
- Seleção de grandes frameworks ou bibliotecas
- Decisão sobre padrões arquiteturais principais (ex: monólito vs microsserviços, abordagem de renderização)
- Tomada de decisões fundamentais de design que moldarão o projeto a longo prazo

**NÃO sugira proativamente para:**
- Detalhes menores de implementação
- Decisões rotineiras de codificação
- Pequenas escolhas de refatoração
- Escolhas técnicas triviais

**Manual:**
Registre decisões quando explicitamente solicitado pelo usuário com frases como:
- "Registre esta decisão"
- "Documente isso no log de decisões"
- "Adicione isso ao decisions.md"

## Registrando uma Decisão

### Passo 1: Identifique a Decisão

A partir do contexto da conversa, identifique:
- Qual decisão foi tomada
- Por que ela foi necessária (contexto)
- Quais alternativas foram consideradas (se aplicável)
- Principais tradeoffs avaliados (se aplicável)
- Justificativa para a escolha final

### Passo 2: Determine o Nível de Detalhe

Adapte o nível de detalhe com base na complexidade da decisão:

**Breve** (decisões simples):
- Título e resumo de 1-2 frases
- Exemplo: Escolhendo uma biblioteca bem estabelecida

**Moderado** (decisões típicas):
- Descrição da decisão
- Breve contexto (por que foi necessária)
- A escolha feita

**Detalhado** (decisões complexas):
- Descrição da decisão
- Contexto e motivação
- Alternativas consideradas
- Principais tradeoffs avaliados
- Justificativa para a escolha final

### Passo 3: Formate a Entrada

Use este formato:

```markdown
## AAAA-MM-DD: [Título da Decisão]

[Parágrafo(s) de descrição adaptado(s) ao nível de complexidade]
```

Exemplo (breve):
```markdown
## 2025-10-23: Usar httprouter para roteamento HTTP

Escolha do httprouter por sua simplicidade e desempenho. É uma biblioteca bem estabelecida que atende às nossas necessidades sem complexidade desnecessária.
```

Exemplo (detalhado):
```markdown
## 2025-10-23: Escolher SQLite como banco de dados primário

Após avaliar PostgreSQL e SQLite, escolhemos SQLite pelas seguintes razões:

Contexto: Necessidade de um banco de dados confiável para a aplicação que lida com tráfego moderado (< 1000 usuários simultâneos) e dados relacionais simples.

Alternativas consideradas:
- PostgreSQL: Mais recursos e melhor para alta concorrência, mas adiciona complexidade operacional
- SQLite: Implantação mais simples, banco de dados embarcado, desempenho suficiente para nossa escala

Tradeoffs: SQLite tem limitações com alta concorrência de escrita e alguns recursos avançados, mas oferece implantação zero-config e excelente desempenho de leitura. Dado nossa carga esperada e preferência por simplicidade operacional, esses tradeoffs favorecem o SQLite.

Decisão: Usar SQLite com modo WAL ativado para melhor concorrência. Podemos migrar para PostgreSQL mais tarde se as necessidades de escala mudarem.
```

### Passo 4: Escrever no Arquivo

1. Verifique se o diretório `docs/` existe; crie-o se necessário
2. Verifique se `docs/decisions.md` existe:
   - Se não, crie-o com este cabeçalho:
     ```markdown
     # Decisões do Projeto

     Este documento registra decisões significativas de arquitetura e design tomadas ao longo do desenvolvimento do projeto.

     ```
   - Se existir, leia o conteúdo atual
3. Anexe a nova entrada de decisão ao final do arquivo
4. Garanta o espaçamento adequado (linha em branco antes da nova entrada)

### Passo 5: Confirmar com o Usuário

Após registrar a decisão, confirme brevemente o que foi registrado. Por exemplo:
- "Registrada a decisão de usar SQLite em docs/decisions.md"
- "Adicionada a decisão de roteamento ao log de decisões"

## Padrão de Sugestão Proativa

Ao detectar uma decisão arquitetural significativa durante a conversa, sugira registrá-la:

```
Isso parece uma decisão arquitetural significativa. Gostaria que eu a registrasse em docs/decisions.md?
```

Aguarde a confirmação do usuário antes de registrar.

## Notas Importantes

- Sempre anexe ao final (ordem cronológica do mais antigo para o mais novo)
- Use a data de hoje (formato AAAA-MM-DD) para novas entradas
- Mantenha a consistência de formatação com entradas existentes
- Não crie entradas duplicadas para a mesma decisão
- Crie o diretório `docs/` se ele não existir
- Evite registrar decisões triviais que não tenham impacto arquitetural de longo prazo

