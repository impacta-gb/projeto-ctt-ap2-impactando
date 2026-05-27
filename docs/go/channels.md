---
icon: lucide/arrow-right-left
---

# Concorrência II: Channels

**Channels** são a forma idiomática de comunicação entre goroutines em Go. O mantra da concorrência em Go é:

> *"Do not communicate by sharing memory; instead, share memory by communicating."*

## O que é um Channel?

Um channel é um **duto tipado** pelo qual goroutines enviam e recebem valores:

```go title="channels.go"
package main

import "fmt"

func main() {
    ch := make(chan int)      // Channel de inteiros (unbuffered)

    go func() {
        ch <- 42              // Envia 42 para o channel
    }()

    valor := <-ch             // Recebe do channel (bloqueia até receber)
    fmt.Println(valor)        // 42
}
```

!!! info "Channels são bloqueantes por padrão"
    Um channel **unbuffered** bloqueia o remetente até que o receptor esteja pronto, e vice-versa. Isso garante sincronização automática.

## Channels Buffered

Um channel com buffer não bloqueia até que o buffer esteja cheio:

```go
ch := make(chan string, 3) // Buffer de 3 elementos

ch <- "primeiro"   // Não bloqueia
ch <- "segundo"    // Não bloqueia
ch <- "terceiro"   // Não bloqueia
// ch <- "quarto"  // Bloquearia! Buffer cheio

fmt.Println(<-ch) // "primeiro"
fmt.Println(<-ch) // "segundo"
fmt.Println(<-ch) // "terceiro"
```

## Fechando channels e range

Use `close(ch)` para sinalizar que não há mais valores a enviar:

```go
func gerador(nums ...int) <-chan int {
    ch := make(chan int)
    go func() {
        for _, n := range nums {
            ch <- n
        }
        close(ch) // Sinaliza fim dos dados
    }()
    return ch
}

func main() {
    for n := range gerador(2, 3, 5, 7, 11) { // range detecta close()
        fmt.Println(n)
    }
}
```

!!! warning "Regras sobre close"
    - Apenas o **remetente** deve fechar um channel
    - Enviar para um channel fechado causa **panic**
    - Receber de um channel fechado retorna o zero value
    - Use `v, ok := <-ch` para verificar se o channel está fechado (`ok == false`)

## select — Multiplexação de channels

`select` permite aguardar em múltiplos channels simultaneamente:

```go
func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() { time.Sleep(1 * time.Second); ch1 <- "um" }()
    go func() { time.Sleep(2 * time.Second); ch2 <- "dois" }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Recebido de ch1:", msg1)
        case msg2 := <-ch2:
            fmt.Println("Recebido de ch2:", msg2)
        }
    }
}
```

### select com default (não-bloqueante)

```go
select {
case msg := <-ch:
    fmt.Println("Recebido:", msg)
default:
    fmt.Println("Nenhuma mensagem disponível")
}
```

### select com timeout

```go
select {
case resultado := <-ch:
    fmt.Println("Resultado:", resultado)
case <-time.After(2 * time.Second):
    fmt.Println("Timeout! Operação demorou demais.")
}
```

## Padrões de concorrência

### Pipeline

```go
// Estágio 1: gera números
func gerar(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums { out <- n }
        close(out)
    }()
    return out
}

// Estágio 2: eleva ao quadrado
func elevarAoQuadrado(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in { out <- n * n }
        close(out)
    }()
    return out
}

func main() {
    // Pipeline: gerar → elevarAoQuadrado
    for n := range elevarAoQuadrado(gerar(2, 3, 4)) {
        fmt.Println(n) // 4, 9, 16
    }
}
```

### Fan-out / Fan-in

```go
// Fan-out: distribui trabalho para múltiplas goroutines
// Fan-in: combina resultados de múltiplos channels em um
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    merged := make(chan int)

    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c { merged <- n }
    }

    wg.Add(len(cs))
    for _, c := range cs { go output(c) }

    go func() { wg.Wait(); close(merged) }()
    return merged
}
```

## Channels direcionais

Para segurança de tipo, declare a direção do channel:

```go
func produtor(ch chan<- int) { // Só pode enviar
    ch <- 42
}

func consumidor(ch <-chan int) { // Só pode receber
    fmt.Println(<-ch)
}

func main() {
    ch := make(chan int)
    go produtor(ch)
    consumidor(ch)
}
```

!!! tip "Use channels direcionais em assinaturas de funções"
    Isso documenta a intenção e permite que o compilador detecte usos incorretos.
