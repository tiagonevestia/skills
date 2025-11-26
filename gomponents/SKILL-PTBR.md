---
name: gomponents
description: Guia para trabalhar com gomponents, uma biblioteca de componentes HTML puramente em Go. Use esta habilidade ao ler ou escrever código gomponents, ou ao construir views HTML em aplicações Go.
license: MIT
---

# gomponents

## Visão Geral

gomponents é uma biblioteca de componentes HTML puramente em Go que trata elementos HTML como valores Go combináveis. Tudo é construído sobre a interface `Node`, tornando a construção de HTML type-safe e combinável.

## Quando Usar Esta Habilidade

Use esta habilidade quando:
- Ler ou escrever código gomponents
- Construir views HTML no lado do servidor em aplicações Go
- Criar componentes HTML reutilizáveis em Go

## Interface Principal

Tudo em gomponents implementa a interface `Node`:

```go
type Node interface {
    Render(w io.Writer) error
}
```

## Funções Essenciais

### Criação de Elementos e Atributos

- `El(name string, children ...Node)` - Cria elementos HTML personalizados
- `Attr(name string, value ...string)` - Cria atributos personalizados

A maioria dos elementos e atributos padrão do HTML5 estão disponíveis como funções no pacote `html`:
- Elementos: `Div()`, `Span()`, `P()`, `A()`, etc.
- Atributos: `Class()`, `ID()`, `Href()`, `Src()`, etc.

**Nota:** Nodes `nil` são ignorados durante a renderização, então é seguro passar nodes nil para elementos.

### Conteúdo de Texto

- `Text(string)` - Conteúdo de texto com escape HTML
- `Textf(format string, args...)` - Texto formatado e com escape
- `Raw(string)` - HTML sem escape
- `Rawf(format string, args...)` - Conteúdo formatado sem escape

### Composição

- `Group([]Node)` - Combina múltiplos nodes
- `Map[T]([]T, func(T) Node)` - Transforma slices em sequências de nodes
- `If(condition bool, node Node)` - Renderização condicional
- `Iff(condition bool, func() Node)` - Renderização condicional preguiçosa (avaliação adiada)

## Convenção de Importação

Ao contrário dos idiomas comuns de Go, **dot imports são recomendados** para gomponents para alcançar uma sintaxe semelhante a DSL:

```go
import (
    . "maragu.dev/gomponents"
    . "maragu.dev/gomponents/html"
    . "maragu.dev/gomponents/components"
)
```

Isso permite escrever código limpo, semelhante a HTML:

```go
Div(Class("container"),
    H1(Text("Hello World")),
    P(Text("Welcome to gomponents")),
)
```

## Organização de Pacotes

- `maragu.dev/gomponents` - Interface principal e funções auxiliares
- `maragu.dev/gomponents/html` - Todos os elementos e atributos HTML5
- `maragu.dev/gomponents/http` - Auxiliares HTTP para renderizar componentes como respostas
- `maragu.dev/gomponents/components` - Utilitários de nível superior (estrutura de documento HTML5, classes dinâmicas)

## Padrões Comuns

### Componente Básico

```go
func UserCard(name, email string) Node {
    return Div(Class("user-card"),
        H2(Text(name)),
        P(Text(email)),
    )
}
```

### Renderização Condicional

```go
func Alert(message string, isError bool) Node {
    return Div(
        If(isError, Class("error")),
        If(!isError, Class("info")),
        P(Text(message)),
    )
}
```

Use `If` quando for sempre seguro avaliar o node. Use `Iff` quando o node puder ser nil e não deve ser avaliado a menos que a condição seja verdadeira.

```go
func UserProfile(user *User) Node {
    return Div(
        H1(Text(user.Name)),
        // Use Iff para evitar desreferência de ponteiro nil quando user.Avatar for nil
        Iff(user.Avatar != nil, func() Node {
            return Img(Src(user.Avatar.URL))
        }),
    )
}
```

### Agrupamento Sem Elemento Pai

Use `Group` para agrupar múltiplos nodes sem envolvê-los em um elemento pai:

```go
func FormFields(required bool) Node {
    return Group{
        Label(For("email"), Text("Email")),
        Input(Type("email"), ID("email")),
        If(required, Span(Class("required"), Text("*"))),
    }
}
```

### Renderização de Lista

```go
func TodoList(todos []Todo) Node {
    return Ul(Class("todo-list"),
        Map(todos, func(t Todo) Node {
            return Li(Text(t.Title))
        }),
    )
}
```

### Documento HTML

```go
func Page(title string, body Node) Node {
    return HTML5(HTML5Props{
        Title:    title,
        Language: "en",
        Head: []Node{
            Link(Rel("stylesheet"), Href("/styles.css")),
        },
        Body: []Node{body},
    })
}
```

### Handler HTTP

```go
import ghttp "maragu.dev/gomponents/http"

func HomeHandler(w http.ResponseWriter, r *http.Request) (Node, error) {
    return Page("My App",
        Div(Class("container"),
        H1(Text("Hello, World!")),
        ),
    ), nil
}

// In main:
http.HandleFunc("/", ghttp.Adapt(HomeHandler))
```

O pacote `http` fornece:
- Tipo `Handler` - assinatura de função que retorna `(Node, error)`
- `Adapt(Handler)` - converte Handler para `http.HandlerFunc`
- Tratamento de erro com códigos de status personalizados via interface `StatusCode() int`
