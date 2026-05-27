---
icon: lucide/check-circle
---

# Testes Automatizados em Go

Go possui um framework de testes integrado (`testing`) sem necessidade de bibliotecas externas. Bons testes são parte fundamental da cultura Go.

## Estrutura de um arquivo de teste

Arquivos de teste terminam com `_test.go` e são excluídos do build de produção:

```
calculadora/
├── calculadora.go       # Código de produção
└── calculadora_test.go  # Testes
```

```go title="calculadora.go"
package calculadora

func Somar(a, b int) int    { return a + b }
func Subtrair(a, b int) int { return a - b }
func Dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("divisão por zero")
    }
    return a / b, nil
}
```

## Testes unitários básicos

```go title="calculadora_test.go"
package calculadora

import "testing"

func TestSomar(t *testing.T) {
    resultado := Somar(2, 3)
    esperado := 5

    if resultado != esperado {
        t.Errorf("Somar(2, 3) = %d; esperado %d", resultado, esperado)
    }
}

func TestDividir(t *testing.T) {
    resultado, err := Dividir(10, 2)
    if err != nil {
        t.Fatalf("Não esperava erro: %v", err)
    }
    if resultado != 5 {
        t.Errorf("Esperado 5, obtido %f", resultado)
    }
}

func TestDividirPorZero(t *testing.T) {
    _, err := Dividir(10, 0)
    if err == nil {
        t.Error("Esperava erro de divisão por zero, mas não ocorreu")
    }
}
```

### Executando testes

```bash
go test ./...                    # Todos os pacotes
go test ./calculadora/           # Pacote específico
go test -v ./...                 # Verbose (mostra todos os testes)
go test -run TestSomar ./...     # Filtra por nome
go test -count=1 ./...           # Desabilita cache
```

!!! tip "t.Fatal vs t.Error"
    | Função           | Comportamento                          |
    |------------------|----------------------------------------|
    | `t.Error()`      | Marca como falho, **continua** o teste |
    | `t.Errorf()`     | Como Error, mas com formatação         |
    | `t.Fatal()`      | Marca como falho, **para** o teste     |
    | `t.Fatalf()`     | Como Fatal, mas com formatação         |
    | `t.Skip()`       | Pula o teste                           |

## Table-Driven Tests (padrão idiomático)

O padrão mais usado em Go para testar múltiplos cenários:

```go
func TestSomarTabela(t *testing.T) {
    casos := []struct {
        nome     string
        a, b     int
        esperado int
    }{
        {"positivos", 2, 3, 5},
        {"negativos", -1, -2, -3},
        {"zero", 0, 0, 0},
        {"misto", -5, 10, 5},
    }

    for _, tc := range casos {
        t.Run(tc.nome, func(t *testing.T) { // (1)!
            resultado := Somar(tc.a, tc.b)
            if resultado != tc.esperado {
                t.Errorf("Somar(%d, %d) = %d; esperado %d",
                    tc.a, tc.b, resultado, tc.esperado)
            }
        })
    }
}
```

1. `t.Run` cria um subtest nomeado, executável individualmente com `-run TestSomarTabela/positivos`.

## Benchmarks

```go
func BenchmarkSomar(b *testing.B) {
    for i := 0; i < b.N; i++ { // b.N é ajustado automaticamente
        Somar(100, 200)
    }
}
```

```bash
go test -bench=. ./...           # Executa todos os benchmarks
go test -bench=BenchmarkSomar -benchmem ./...  # Com uso de memória
```

Saída típica:
```
BenchmarkSomar-8    1000000000    0.3 ns/op    0 B/op    0 allocs/op
```

## Testes de exemplo (Exemplos)

Exemplos aparecem na documentação gerada pelo `go doc` e são validados pelo compilador:

```go
func ExampleSomar() {
    fmt.Println(Somar(2, 3))
    // Output:
    // 5
}
```

## Cobertura de testes

```bash
# Mostra percentual de cobertura
go test -cover ./...

# Gera relatório HTML detalhado
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

!!! tip "Meta de cobertura"
    Não existe uma regra universal, mas 70-80% de cobertura é um bom objetivo. Foque em cobrir **caminhos críticos** e casos de erro, não apenas o caminho feliz.

## Testify — biblioteca popular

A biblioteca `testify` simplifica as asserções:

```bash
go get github.com/stretchr/testify
```

```go
import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestComTestify(t *testing.T) {
    resultado := Somar(2, 3)

    assert.Equal(t, 5, resultado, "Soma deve ser 5")
    assert.NoError(t, err)

    // require para ao primeiro falho (como t.Fatal)
    require.NotNil(t, resultado)
}
```

## Mocks com interfaces

```go
// Interface para facilitar mocks
type Repositorio interface {
    BuscarUsuario(id int) (*Usuario, error)
}

// Mock manual
type MockRepositorio struct {
    usuarios map[int]*Usuario
}

func (m *MockRepositorio) BuscarUsuario(id int) (*Usuario, error) {
    if u, ok := m.usuarios[id]; ok {
        return u, nil
    }
    return nil, fmt.Errorf("usuário %d não encontrado", id)
}

func TestServico(t *testing.T) {
    mock := &MockRepositorio{
        usuarios: map[int]*Usuario{1: {Nome: "Alice"}},
    }
    servico := NovoServico(mock)
    usuario, err := servico.ObterUsuario(1)
    assert.NoError(t, err)
    assert.Equal(t, "Alice", usuario.Nome)
}
```

!!! warning "Teste o comportamento, não a implementação"
    Bons testes verificam **o que** o código faz, não **como** ele faz. Evite testar detalhes internos que podem mudar sem afetar o comportamento externo.
