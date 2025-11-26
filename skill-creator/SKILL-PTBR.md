---
name: skill-creator
description: Guia para criar habilidades eficazes. Esta habilidade deve ser usada quando os usuários desejam criar uma nova habilidade (ou atualizar uma existente) que estenda as capacidades do Claude com conhecimento especializado, fluxos de trabalho ou integrações de ferramentas.
license: Termos completos em LICENSE.txt
---

# Criador de Habilidades (Skill Creator)

Esta habilidade fornece orientação para criar habilidades eficazes.

## Sobre Habilidades

Habilidades são pacotes modulares e independentes que estendem as capacidades do Claude fornecendo conhecimento especializado, fluxos de trabalho e ferramentas. Pense nelas como "guias de integração" para domínios ou tarefas específicas — elas transformam o Claude de um agente de propósito geral em um agente especializado equipado com conhecimento procedural que nenhum modelo pode possuir totalmente.

### O Que as Habilidades Fornecem

1. Fluxos de trabalho especializados - Procedimentos de várias etapas para domínios específicos
2. Integrações de ferramentas - Instruções para trabalhar com formatos de arquivo ou APIs específicas
3. Expertise de domínio - Conhecimento específico da empresa, esquemas, lógica de negócios
4. Recursos agrupados - Scripts, referências e ativos para tarefas complexas e repetitivas

### Anatomia de uma Habilidade

Toda habilidade consiste em um arquivo `SKILL.md` obrigatório e recursos agrupados opcionais:

```
skill-name/
├── SKILL.md (obrigatório)
│   ├── Metadados YAML frontmatter (obrigatório)
│   │   ├── name: (obrigatório)
│   │   └── description: (obrigatório)
│   └── Instruções Markdown (obrigatório)
└── Recursos Agrupados (opcional)
    ├── scripts/          - Código executável (Python/Bash/etc.)
    ├── references/       - Documentação destinada a ser carregada no contexto conforme necessário
    └── assets/           - Arquivos usados na saída (modelos, ícones, fontes, etc.)
```

#### SKILL.md (obrigatório)

**Qualidade dos Metadados:** O `name` e `description` no frontmatter YAML determinam quando o Claude usará a habilidade. Seja específico sobre o que a habilidade faz e quando usá-la. Use a terceira pessoa (ex: "Esta habilidade deve ser usada quando..." em vez de "Use esta habilidade quando...").

#### Recursos Agrupados (opcional)

##### Scripts (`scripts/`)

Código executável (Python/Bash/etc.) para tarefas que requerem confiabilidade determinística ou são reescritas repetidamente.

- **Quando incluir**: Quando o mesmo código está sendo reescrito repetidamente ou confiabilidade determinística é necessária
- **Exemplo**: `scripts/rotate_pdf.py` para tarefas de rotação de PDF
- **Benefícios**: Eficiente em tokens, determinístico, pode ser executado sem carregar no contexto
- **Nota**: Scripts ainda podem precisar ser lidos pelo Claude para correções ou ajustes específicos do ambiente

##### Referências (`references/`)

Documentação e material de referência destinados a serem carregados conforme necessário no contexto para informar o processo e pensamento do Claude.

- **Quando incluir**: Para documentação que o Claude deve referenciar enquanto trabalha
- **Exemplos**: `references/finance.md` para esquemas financeiros, `references/mnda.md` para modelo de NDA da empresa, `references/policies.md` para políticas da empresa, `references/api_docs.md` para especificações de API
- **Casos de uso**: Esquemas de banco de dados, documentação de API, conhecimento de domínio, políticas da empresa, guias de fluxo de trabalho detalhados
- **Benefícios**: Mantém o SKILL.md enxuto, carregado apenas quando o Claude determina que é necessário
- **Melhor prática**: Se os arquivos forem grandes (>10k palavras), inclua padrões de busca grep no SKILL.md
- **Evite duplicação**: A informação deve viver no SKILL.md ou nos arquivos de referência, não em ambos. Prefira arquivos de referência para informações detalhadas, a menos que seja verdadeiramente central para a habilidade — isso mantém o SKILL.md enxuto enquanto torna a informação descobrível sem ocupar a janela de contexto. Mantenha apenas instruções procedurais essenciais e orientação de fluxo de trabalho no SKILL.md; mova material de referência detalhado, esquemas e exemplos para arquivos de referência.

##### Ativos (`assets/`)

Arquivos não destinados a serem carregados no contexto, mas sim usados dentro da saída que o Claude produz.

- **Quando incluir**: Quando a habilidade precisa de arquivos que serão usados na saída final
- **Exemplos**: `assets/logo.png` para ativos de marca, `assets/slides.pptx` para modelos de PowerPoint, `assets/frontend-template/` para boilerplate HTML/React, `assets/font.ttf` para tipografia
- **Casos de uso**: Modelos, imagens, ícones, código boilerplate, fontes, documentos de exemplo que são copiados ou modificados
- **Benefícios**: Separa recursos de saída da documentação, permite que o Claude use arquivos sem carregá-los no contexto

### Princípio de Design de Divulgação Progressiva

As habilidades usam um sistema de carregamento de três níveis para gerenciar o contexto de forma eficiente:

1. **Metadados (nome + descrição)** - Sempre no contexto (~100 palavras)
2. **Corpo do SKILL.md** - Quando a habilidade é acionada (<5k palavras)
3. **Recursos agrupados** - Conforme necessário pelo Claude (Ilimitado*)

*Ilimitado porque scripts podem ser executados sem ler na janela de contexto.

## Processo de Criação de Habilidade

Para criar uma habilidade, siga o "Processo de Criação de Habilidade" em ordem, pulando etapas apenas se houver uma razão clara pela qual não são aplicáveis.

### Passo 1: Entendendo a Habilidade com Exemplos Concretos

Pule este passo apenas quando os padrões de uso da habilidade já forem claramente compreendidos. Ele permanece valioso mesmo ao trabalhar com uma habilidade existente.

Para criar uma habilidade eficaz, entenda claramente exemplos concretos de como a habilidade será usada. Esse entendimento pode vir de exemplos diretos do usuário ou exemplos gerados que são validados com feedback do usuário.

Por exemplo, ao construir uma habilidade de editor de imagens, perguntas relevantes incluem:

- "Que funcionalidade a habilidade de editor de imagens deve suportar? Edição, rotação, algo mais?"
- "Você pode dar alguns exemplos de como esta habilidade seria usada?"
- "Eu posso imaginar usuários pedindo coisas como 'Remova os olhos vermelhos desta imagem' ou 'Gire esta imagem'. Existem outras maneiras que você imagina esta habilidade sendo usada?"
- "O que um usuário diria que deveria acionar esta habilidade?"

Para evitar sobrecarregar os usuários, evite fazer muitas perguntas em uma única mensagem. Comece com as perguntas mais importantes e faça o acompanhamento conforme necessário para melhor eficácia.

Conclua este passo quando houver um senso claro da funcionalidade que a habilidade deve suportar.

### Passo 2: Planejando o Conteúdo Reutilizável da Habilidade

Para transformar exemplos concretos em uma habilidade eficaz, analise cada exemplo:

1. Considerando como executar o exemplo do zero
2. Identificando quais scripts, referências e ativos seriam úteis ao executar esses fluxos de trabalho repetidamente

Exemplo: Ao construir uma habilidade `pdf-editor` para lidar com consultas como "Ajude-me a girar este PDF", a análise mostra:

1. Girar um PDF requer reescrever o mesmo código a cada vez
2. Um script `scripts/rotate_pdf.py` seria útil para armazenar na habilidade

Exemplo: Ao projetar uma habilidade `frontend-webapp-builder` para consultas como "Construa um app de tarefas" ou "Construa um painel para rastrear meus passos", a análise mostra:

1. Escrever um webapp frontend requer o mesmo boilerplate HTML/React a cada vez
2. Um modelo `assets/hello-world/` contendo os arquivos de projeto boilerplate HTML/React seria útil para armazenar na habilidade

Exemplo: Ao construir uma habilidade `big-query` para lidar com consultas como "Quantos usuários fizeram login hoje?", a análise mostra:

1. Consultar o BigQuery requer redescobrir os esquemas de tabela e relacionamentos a cada vez
2. Um arquivo `references/schema.md` documentando os esquemas de tabela seria útil para armazenar na habilidade

Para estabelecer o conteúdo da habilidade, analise cada exemplo concreto para criar uma lista dos recursos reutilizáveis a incluir: scripts, referências e ativos.

### Passo 3: Inicializando a Habilidade

Neste ponto, é hora de realmente criar a habilidade.

Pule este passo apenas se a habilidade sendo desenvolvida já existir, e iteração ou empacotamento for necessário. Neste caso, continue para o próximo passo.

Ao criar uma nova habilidade do zero, sempre execute o script `init_skill.py`. O script gera convenientemente um diretório de modelo de nova habilidade que inclui automaticamente tudo o que uma habilidade requer, tornando o processo de criação de habilidade muito mais eficiente e confiável.

Uso:

```bash
scripts/init_skill.py <nome-da-habilidade> --path <diretório-de-saída>
```

O script:

- Cria o diretório da habilidade no caminho especificado
- Gera um modelo SKILL.md com frontmatter adequado e marcadores TODO
- Cria diretórios de recursos de exemplo: `scripts/`, `references/` e `assets/`
- Adiciona arquivos de exemplo em cada diretório que podem ser personalizados ou excluídos

Após a inicialização, personalize ou remova o SKILL.md gerado e os arquivos de exemplo conforme necessário.

### Passo 4: Editar a Habilidade

Ao editar a habilidade (recém-gerada ou existente), lembre-se de que a habilidade está sendo criada para outra instância do Claude usar. Concentre-se em incluir informações que seriam benéficas e não óbvias para o Claude. Considere que conhecimento procedural, detalhes específicos do domínio ou ativos reutilizáveis ajudariam outra instância do Claude a executar essas tarefas de forma mais eficaz.

#### Comece com Conteúdo Reutilizável da Habilidade

Para começar a implementação, comece com os recursos reutilizáveis identificados acima: arquivos `scripts/`, `references/` e `assets/`. Note que este passo pode exigir entrada do usuário. Por exemplo, ao implementar uma habilidade `brand-guidelines`, o usuário pode precisar fornecer ativos de marca ou modelos para armazenar em `assets/`, ou documentação para armazenar em `references/`.

Além disso, exclua quaisquer arquivos e diretórios de exemplo não necessários para a habilidade. O script de inicialização cria arquivos de exemplo em `scripts/`, `references/` e `assets/` para demonstrar a estrutura, mas a maioria das habilidades não precisará de todos eles.

#### Atualizar SKILL.md

**Estilo de Escrita:** Escreva toda a habilidade usando **forma imperativa/infinitiva** (instruções verbo-primeiro), não segunda pessoa. Use linguagem objetiva e instrucional (ex: "Para realizar X, faça Y" em vez de "Você deve fazer X" ou "Se você precisar fazer X"). Isso mantém a consistência e clareza para o consumo da IA.

Para completar o SKILL.md, responda às seguintes perguntas:

1. Qual é o propósito da habilidade, em poucas frases?
2. Quando a habilidade deve ser usada?
3. Na prática, como o Claude deve usar a habilidade? Todo o conteúdo reutilizável da habilidade desenvolvido acima deve ser referenciado para que o Claude saiba como usá-lo.

### Passo 5: Empacotando uma Habilidade

Uma vez que a habilidade esteja pronta, ela deve ser empacotada em um arquivo zip distribuível que é compartilhado com o usuário. O processo de empacotamento valida automaticamente a habilidade primeiro para garantir que ela atenda a todos os requisitos:

```bash
scripts/package_skill.py <caminho/para/pasta-da-habilidade>
```

Especificação opcional de diretório de saída:

```bash
scripts/package_skill.py <caminho/para/pasta-da-habilidade> ./dist
```

O script de empacotamento irá:

1. **Validar** a habilidade automaticamente, verificando:
   - Formato do frontmatter YAML e campos obrigatórios
   - Convenções de nomenclatura de habilidade e estrutura de diretório
   - Completude e qualidade da descrição
   - Organização de arquivos e referências de recursos

2. **Empacotar** a habilidade se a validação passar, criando um arquivo zip nomeado após a habilidade (ex: `minha-habilidade.zip`) que inclui todos os arquivos e mantém a estrutura de diretório adequada para distribuição.

Se a validação falhar, o script relatará os erros e sairá sem criar um pacote. Corrija quaisquer erros de validação e execute o comando de empacotamento novamente.

### Passo 6: Iterar

Após testar a habilidade, os usuários podem solicitar melhorias. Frequentemente isso acontece logo após o uso da habilidade, com contexto fresco de como a habilidade performou.

**Fluxo de trabalho de iteração:**
1. Use a habilidade em tarefas reais
2. Note dificuldades ou ineficiências
3. Identifique como o SKILL.md ou recursos agrupados devem ser atualizados
4. Implemente mudanças e teste novamente
