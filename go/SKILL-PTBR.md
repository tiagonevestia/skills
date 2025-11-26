---
name: go
description: Guia sobre como desenvolver aplicativos e módulos/bibliotecas Go. Use sempre esta habilidade ao ler ou escrever código Go.
license: MIT
---

# Go

Este é um guia sobre como desenvolver aplicativos e módulos/bibliotecas em Go.

Parte dele é aplicável apenas para aplicativos, não para módulos usados como bibliotecas em outros projetos, como acesso a banco de dados e execução de um servidor.

## Estrutura da aplicação

Geralmente, eu construo aplicativos web e bibliotecas/módulos.

Estes são os pacotes tipicamente presentes em aplicativos (alguns podem estar faltando, o que geralmente significa que não preciso deles no projeto).

- `main`: contém o ponto de entrada principal da aplicação (no diretório `cmd/app`)
- `model`: contém o modelo de domínio usado em todos os outros pacotes
- `sql`/`sqlite`/`postgres`: contém lógica relacionada a banco de dados SQL, bem como migrações de banco de dados (no subdiretório `migrations/`) e fixtures de teste (no subdiretório `testdata/fixtures/`). O banco de dados usado é SQLite ou PostgreSQL.
- `sqltest`/`sqlitetest`/`postgrestest`: pacote usado em testes, para configurar e desmontar bancos de dados de teste
- `s3`: lógica para interagir com Amazon S3 ou armazenamentos de objetos compatíveis
- `s3test`: pacote usado em testes, para configurar e desmontar buckets S3 de teste
- `llm`: clientes para interagir com grandes modelos de linguagem (LLMs) e modelos de fundação
- `llmtest`: pacote usado em testes, para configurar clientes LLM para testes
- `http`: manipuladores (handlers) HTTP para a aplicação
- `html`: templates HTML para a aplicação, escritos com a biblioteca gomponents (veja https://www.gomponents.com/llms.txt para saber como usar se precisar)

## Estilo de código

### Injeção de dependência

Faço uso intenso de injeção de dependência entre componentes. Isso é tipicamente feito com interfaces privadas no lado do receptor. Note o uso de `userGetter` neste exemplo:

```go user.go
package http

import (
	"net/http"

	"github.com/go-chi/chi/v5"
	"maragu.dev/httph"

	"model"
)

type UserResponse struct {
	Name string
}

type userGetter interface {
	GetUser(ctx context.Context, id model.ID) (model.User, error)
}

func User(r chi.Router, db userGetter) {
	r.Get("/user", httph.JSONHandler(func(w http.ResponseWriter, r *http.Request, _ any) (UserResponse, error) {
		id := r.URL.Query().Get("id")
		user, err := db.GetUser(r.Context(), model.ID(id))
		if err != nil {
			return UserResponse{}, httph.HTTPError{Code: http.StatusInternalServerError, Err: errors.New("error getting user")}
		}
		return UserResponse{Name: user.Name}, nil
	}))
}
```

### Testes

Escrevo testes para a maioria das funções e métodos. Quase sempre uso subtestes com uma boa descrição do que está acontecendo e qual é o resultado esperado.

Aqui está um exemplo:

```go example.go
package example

type Thing struct {}

func (t *Thing) DoSomething() (bool, error) {
	return true, nil
}
```

```go example_test.go
package example_test

import (
	"testing"

	"maragu.dev/is"

	"example"
)

func TestThing_DoSomething(t *testing.T) {
	t.Run("should do something and return a nil error", func(t *testing.T) {
		thing := &example.Thing{}

		ok, err := thing.DoSomething()
		is.NotError(t, err)
		is.True(t, ok)
	})
}
```

Às vezes uso testes baseados em tabelas (table-driven tests):

```go example.go
package example

import "errors"

type Thing struct {}

var ErrChairNotSupported = errors.New("chairs not supported")

func (t *Thing) DoSomething(with string) error {
	if with == "chair" {
		return ErrChairNotSupported
	}
	return nil
}
```

```go example_test.go
package example_test

import (
	"testing"

	"maragu.dev/is"

	"example"
)

func TestThing_DoSomething(t *testing.T) {
	tests := []struct {
		name     string
		input    string
		expected error
	}{
		{name: "should do something with the table and return a nil error", input: "table", expected: nil},
		{name: "should do something with the chair and return an ErrChairNotSupported", input: "chair", expected: example.ErrChairNotSupported},
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			thing := &example.Thing{}

			err := thing.DoSomething(test.input)
			if test.expected != nil {
				is.Error(t, test.expected, err)
			} else {
				is.NotError(t, err)
			}
		})
	}
}
```

Prefiro testes de integração com dependências reais em vez de mocks, porque nada substitui a coisa real. Dependências são tipicamente executadas em contêineres Docker. Você pode assumir que as dependências estão rodando ao executar testes.

Faz sentido usar mocks quando a parte importante de um teste não é a dependência, mas ela desempenha um papel menor. Mas, por exemplo, ao testar métodos de banco de dados, um banco de dados subjacente real deve ser usado.

Uso asserções de teste com o módulo `maragu.dev/is`. Funções disponíveis: `is.True`, `is.Equal`, `is.Nil`, `is.NotNil`, `is.EqualSlice`, `is.NotError`, `is.Error`. Todas elas aceitam uma mensagem opcional como último parâmetro.

Como os testes são embaralhados (shuffled), não confie na ordem dos testes, mesmo para subtestes.

Sempre que os helpers de teste `postgrestest.NewDatabase(t)`/`sqlitetest.NewDatabase(t)` são chamados, o banco de dados está em um estado limpo (sem sobras de outros testes etc.).

Você pode usar fixtures de banco de dados para testes. Prefira usá-las para configurações de dados de teste quando múltiplos testes dependem dos mesmos dados ou dados muito similares, para que cada teste não tenha que configurar os mesmos dados. Elas ficam em `sqlite/testdata/fixtures`/`postgres/testdata/fixtures`. Use-as com `sqlitetest.NewDatabase(t, sqlitetest.WithFixtures("fixture one", "fixture two"))`. Elas são aplicadas na ordem fornecida.

Funções helper de teste devem chamar `testing.T.Helper()`.

Em testes, use `t.Context()` em vez de `context.Background()`, e sempre use inline em vez de extrair para uma variável `ctx`.

### Diversos

- Nomeação de variáveis:
  - `req` para requisições, `res` para respostas
- Existem helpers SQL disponíveis em `Database.H.Select`, `Database.H.Exec`, `Database.H.Get`, `Database.H.InTx`.
- Use o builtin `any` em Go em vez de `interface{}`
- Existe um alias para `sql.ErrNoRows` da stdlib em `maragu.dev/glue/sql.ErrNoRows`, então você não precisa importar ambos
- Todos os botões HTML precisam da classe CSS `cursor-pointer`
- O formato de tempo do SQLite é sempre uma string retornada por `strftime('%Y-%m-%dT%H:%M:%fZ')`. Use `maragu.dev/glue/model.Time` (geralmente com alias em `model.Time` no projeto) em vez de `time.Time` da stdlib ao trabalhar com o banco de dados.
- Lembre-se que funções privadas em Go são de nível de pacote, então você pode usá-las entre arquivos no mesmo pacote
- A documentação deve seguir o estilo Go de ter o nome do identificador como a primeira palavra da frase, e então completar a frase sem se repetir. Exemplo: "// SearchProducts using the given search query and result limit." NÃO: "// SearchProducts searches products using the given search query and result label."
- Identificadores de nível de pacote devem começar com minúscula por padrão, ou seja, ter visibilidade de nível de pacote, para tornar a área de superfície da API menor para outros pacotes.

## Testes, linting, avaliações

Execute `make test` ou `go test -shuffle on ./...` para rodar todos os testes. Para rodar testes em apenas um pacote, use `go test -shuffle on ./path/to/package`. Para rodar um teste específico, use `go test ./path/to/package -run TestName`.

Execute `make lint` ou `golangci-lint run` para rodar linters. Eles devem sempre ser executados no nível do pacote/diretório, não funcionará com arquivos únicos.

Execute `make eval` ou `go test -shuffle on -run TestEval ./...` para rodar avaliações de LLM.

Execute `make fmt` para formatar todo o código no projeto, o que é útil como um toque final.

Você pode acessar o banco de dados usando `psql` ou `sqlite3` no shell.

## Documentação

Você geralmente pode consultar a documentação para um módulo Go usando `go doc` com o nome do módulo. Por exemplo, `go doc net/http` para algo na biblioteca padrão, ou `go doc maragu.dev/gai` para um módulo de terceiros. Você também pode consultar documentação mais específica para um identificador com algo como `go doc maragu.dev/gai.ChatCompleter`, para a interface `ChatCompleter`.

## Verificando aplicativos em um navegador

Você pode assumir que o aplicativo está rodando e disponível em um navegador usando a ferramenta Chrome Dev Tools MCP. Ela recarrega automaticamente nas mudanças de código para que você não precise fazer isso.
A saída de log do aplicativo em execução está em `app.log` na raiz do projeto.
