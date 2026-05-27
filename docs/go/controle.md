---
icon: lucide/git-branch
---

# Estruturas de Controle

Go possui estruturas de controle simples mas poderosas. Diferente de outras linguagens, **não há `while` ou `do-while`** — o `for` é o único loop, mas extremamente versátil.

## if / else

A condição não usa parênteses e as chaves são obrigatórias:

```go title="condicional.go"
x := 10

if x > 5 {
    fmt.Println("Maior que 5")
} else if x == 5 {
    fmt.Println("Igual a 5")
} else {
    fmt.Println("Menor que 5")
}
```

### if com inicialização

Go permite inicializar uma variável antes da condição no mesmo `if`:

```go
// A variável 'err' existe apenas no escopo do if/else
if err := fazerAlgo(); err != nil {
    fmt.Println("Erro:", err)
} else {
    fmt.Println("Sucesso!")
}
```

!!! tip "Padrão idiomático"
    Esse padrão é muito comum para verificar erros em Go. A variável fica encapsulada no bloco, mantendo o código limpo.

## for — O único loop em Go

### Loop clássico (estilo C)

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

### Loop condicional (estilo while)

```go
n := 1
for n < 100 {
    n *= 2
}
fmt.Println(n) // 128
```

### Loop infinito

```go
for {
    // Executa para sempre (use break para sair)
    if condicao {
        break
    }
}
```

### for range — iteração sobre coleções

```go
// Slice
frutas := []string{"maçã", "banana", "laranja"}
for i, fruta := range frutas {
    fmt.Printf("[%d] %s\n", i, fruta)
}

// Map
capitais := map[string]string{"BR": "Brasília", "US": "Washington"}
for pais, capital := range capitais {
    fmt.Printf("%s → %s\n", pais, capital)
}

// String (itera por rune/caractere Unicode)
for i, ch := range "Go 🐹" {
    fmt.Printf("%d: %c\n", i, ch)
}
```

!!! note "Descartando valores com _"
    Se não precisar do índice ou do valor, use `_`:
    ```go
    for _, fruta := range frutas {
        fmt.Println(fruta)
    }
    ```

### break e continue

```go
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue  // Pula números pares
    }
    if i > 7 {
        break     // Para quando > 7
    }
    fmt.Println(i) // Imprime: 1, 3, 5, 7
}
```

## switch

O `switch` em Go não precisa de `break` — só executa o caso correspondente:

```go title="switch.go"
diaSemana := "terça"

switch diaSemana {
case "segunda", "terça", "quarta", "quinta", "sexta":
    fmt.Println("Dia útil")
case "sábado", "domingo":
    fmt.Println("Fim de semana!")
default:
    fmt.Println("Dia inválido")
}
```

### switch sem expressão (substitui if/else longo)

```go
nota := 85

switch {
case nota >= 90:
    fmt.Println("A")
case nota >= 80:
    fmt.Println("B") // Imprime "B"
case nota >= 70:
    fmt.Println("C")
default:
    fmt.Println("Reprovado")
}
```

### switch com inicialização

```go
switch os := runtime.GOOS; os {
case "linux":
    fmt.Println("Linux")
case "darwin":
    fmt.Println("macOS")
default:
    fmt.Printf("Outro SO: %s\n", os)
}
```

### fallthrough

Para executar o próximo caso explicitamente:

```go
switch 2 {
case 1:
    fmt.Println("um")
    fallthrough
case 2:
    fmt.Println("dois")  // Imprime
    fallthrough
case 3:
    fmt.Println("três")  // Também imprime (por causa do fallthrough acima)
case 4:
    fmt.Println("quatro") // Não imprime
}
```

!!! warning "Use fallthrough com moderação"
    `fallthrough` executa o próximo bloco **sem checar a condição**. Use apenas quando realmente necessário.
