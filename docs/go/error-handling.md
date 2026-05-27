---
icon: lucide/alert-triangle
---

# Tratamento de Erros (Error Handling)

Go adota uma abordagem explícita e idiomática para erros: ao invés de exceções, funções retornam um valor de erro como **último valor de retorno**. Isso torna o fluxo de erros visível e controlado.

## O tipo error

`error` é uma interface built-in em Go:

```go
type error interface {
    Error() string
}
```

Funções que podem falhar retornam `(resultado, error)`:

```go title="erros.go"
import (
    "errors"
    "fmt"
    "strconv"
)

func dividir(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("divisão por zero não é permitida")
    }
    return a / b, nil  // nil indica ausência de erro
}

func main() {
    resultado, err := dividir(10, 2)
    if err != nil {
        fmt.Println("Erro:", err)
        return
    }
    fmt.Println("Resultado:", resultado) // 5
}
```

!!! tip "Padrão idiomático"
    Sempre verifique o erro imediatamente após a chamada. O padrão `if err != nil` é onipresente em código Go.

## Criando erros personalizados

### errors.New

```go
var ErrNaoEncontrado = errors.New("item não encontrado")

func buscarUsuario(id int) (*Usuario, error) {
    if id <= 0 {
        return nil, ErrNaoEncontrado
    }
    // ...
}
```

### fmt.Errorf com contexto

```go
func processarArquivo(path string) error {
    dados, err := os.ReadFile(path)
    if err != nil {
        return fmt.Errorf("processarArquivo: falha ao ler %s: %w", path, err)
    }
    // ...
    return nil
}
```

!!! info "O verbo %w (wrap)"
    `%w` envolve o erro original, permitindo inspecioná-lo mais tarde com `errors.Is` e `errors.As`. Prefira `%w` a `%v` quando quiser preservar o tipo do erro.

## Tipos de erro personalizados

Para erros com dados adicionais, implemente a interface `error`:

```go
type ErrValidacao struct {
    Campo   string
    Mensagem string
}

func (e *ErrValidacao) Error() string {
    return fmt.Sprintf("validação falhou no campo '%s': %s", e.Campo, e.Mensagem)
}

func validarIdade(idade int) error {
    if idade < 0 {
        return &ErrValidacao{Campo: "idade", Mensagem: "não pode ser negativa"}
    }
    if idade > 150 {
        return &ErrValidacao{Campo: "idade", Mensagem: "valor improvável"}
    }
    return nil
}
```

## errors.Is e errors.As

Para inspecionar a cadeia de erros:

```go
var ErrPermissao = errors.New("permissão negada")

// errors.Is — verifica se o erro (ou algum na cadeia) é o alvo
err := fmt.Errorf("operação falhou: %w", ErrPermissao)
if errors.Is(err, ErrPermissao) {
    fmt.Println("Sem permissão!") // imprime
}

// errors.As — extrai um tipo específico de erro da cadeia
err2 := validarIdade(-5)
var ve *ErrValidacao
if errors.As(err2, &ve) {
    fmt.Println("Campo com problema:", ve.Campo)
}
```

## defer, panic e recover

### defer

`defer` agenda a execução de uma função para quando a função atual retornar:

```go
func lerArquivo(path string) (string, error) {
    f, err := os.Open(path)
    if err != nil {
        return "", err
    }
    defer f.Close() // Garante fechamento mesmo em caso de erro

    // ... processar arquivo
}
```

!!! tip "Múltiplos defers"
    Múltiplos `defer` executam em ordem **LIFO** (último a entrar, primeiro a sair — como uma pilha).

### panic e recover

`panic` interrompe a execução normal. Use apenas para erros irrecuperáveis:

```go
func dividirComPanic(a, b int) int {
    if b == 0 {
        panic("divisão por zero!")
    }
    return a / b
}

// recover captura um panic dentro de um defer
func executarComSeguranca() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Panic capturado:", r)
        }
    }()

    dividirComPanic(10, 0) // causa panic
    fmt.Println("Esta linha nunca executa")
}
```

!!! warning "panic não é para controle de fluxo"
    Ao contrário de exceções em Java/Python, `panic` é para situações verdadeiramente inesperadas (bugs de programação). Para erros esperados, retorne `error`.

## Boas Práticas

| Prática                              | Recomendado |
|--------------------------------------|-------------|
| Retornar `error` como último valor   | ✅ Sim       |
| Verificar `err != nil` imediatamente | ✅ Sim       |
| Usar `fmt.Errorf` com `%w` para contexto | ✅ Sim  |
| Usar `panic` para erros de negócio   | ❌ Não      |
| Ignorar erros com `_`                | ❌ Evitar   |
