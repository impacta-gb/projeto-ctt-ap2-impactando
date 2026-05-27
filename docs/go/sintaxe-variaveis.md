---
icon: lucide/code-2
---

# Sintaxe Básica e Variáveis

Go possui uma sintaxe limpa e direta. Nesta seção, exploraremos os fundamentos da linguagem: declaração de variáveis, tipos primitivos, constantes e operadores.

## Declaração de Variáveis

Go oferece duas formas principais de declarar variáveis:

=== "Declaração explícita (var)"
    ```go
    var nome string = "Go"
    var idade int = 15
    var ativo bool = true
    ```

=== "Declaração curta (:=)"
    ```go
    nome := "Go"       // string inferido
    idade := 15        // int inferido
    ativo := true      // bool inferido
    ```

!!! info "Regra de escopo"
    O operador `:=` só pode ser usado **dentro de funções**. Para variáveis no escopo de pacote, use `var`.

### Declaração múltipla

```go title="variaveis.go"
var (
    nome    string  = "Gopher"
    versao  float64 = 1.22
    estavel bool    = true
)

// Ou com inferência
linguagem, ano := "Go", 2009
```

## Tipos Primitivos

| Categoria    | Tipos                                                     | Tamanho (bits)     |
|--------------|-----------------------------------------------------------|--------------------|
| **Inteiros** | `int8`, `int16`, `int32`, `int64`, `int`                  | 8 / 16 / 32 / 64   |
| **Unsigned** | `uint8`, `uint16`, `uint32`, `uint64`, `uint`             | 8 / 16 / 32 / 64   |
| **Float**    | `float32`, `float64`                                      | 32 / 64            |
| **Byte**     | `byte` (alias de `uint8`)                                 | 8                  |
| **Rune**     | `rune` (alias de `int32`, caractere Unicode)              | 32                 |
| **String**   | `string` (sequência imutável de bytes)                    | variável           |
| **Bool**     | `bool` (`true` ou `false`)                                | 1                  |

```go
var inteiro int64 = 9_223_372_036_854_775_807
var pi float64 = 3.14159265358979
var letra rune = 'A'
var texto string = "Olá, 世界"
var flag bool = false
```

## Zero Values

Em Go, toda variável não inicializada recebe um **valor zero** de seu tipo:

```go
var i int        // 0
var f float64    // 0.0
var b bool       // false
var s string     // "" (string vazia)
var p *int       // nil
```

!!! tip "Vantagem importante"
    Não há variáveis "undefined" em Go. Isso elimina uma classe inteira de bugs comuns em outras linguagens.

## Constantes

```go
const Pi = 3.14159
const MaxRetries = 3

// Enumerador automático com iota
const (
    Segunda = iota + 1  // 1
    Terca               // 2
    Quarta              // 3
    Quinta              // 4
    Sexta               // 5
)
```

### iota para potências de 2

```go
const (
    KB = 1 << (10 * (iota + 1))  // 1024
    MB                            // 1048576
    GB                            // 1073741824
)
```

## Conversão de Tipos

Go **não faz conversão implícita**. Toda conversão é explícita:

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)

texto := "Olá"
bytes := []byte(texto)
volta := string(bytes)
```

!!! warning "Truncamento em conversões"
    Converter `float64` para `int` **trunca** o valor (não arredonda):
    ```go
    x := 3.99
    y := int(x)  // y == 3, não 4!
    ```
