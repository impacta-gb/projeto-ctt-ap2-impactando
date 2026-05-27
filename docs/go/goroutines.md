---
icon: lucide/zap
---

# Concorrência I: Goroutines

Uma das características mais poderosas do Go é seu suporte nativo à **concorrência leve** através de **goroutines** — funções executadas de forma concorrente, gerenciadas pelo próprio runtime do Go.

## O que é uma Goroutine?

Uma goroutine é uma **thread leve** gerenciada pelo runtime do Go (não pelo sistema operacional). Você pode ter milhões delas simultâneas com consumo mínimo de memória (~2KB cada, contra ~1MB de uma thread OS).

```go title="goroutines.go"
package main

import (
    "fmt"
    "time"
)

func tarefa(nome string) {
    for i := 0; i < 3; i++ {
        fmt.Printf("[%s] passo %d\n", nome, i)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    go tarefa("goroutine-1") // (1)!
    go tarefa("goroutine-2")

    tarefa("main") // Executa na goroutine principal
}
```

1. A palavra-chave `go` antes de uma chamada de função lança uma goroutine.

!!! warning "Cuidado com o fim do main"
    Quando `main` termina, **todas as goroutines são encerradas**. Use mecanismos de sincronização para aguardar a conclusão.

## sync.WaitGroup — Aguardando goroutines

O `WaitGroup` é a forma idiomática de esperar múltiplas goroutines:

```go
package main

import (
    "fmt"
    "sync"
)

func processar(id int, wg *sync.WaitGroup) {
    defer wg.Done() // Decrementa o contador ao finalizar
    fmt.Printf("Goroutine %d iniciada\n", id)
    // ... trabalho
    fmt.Printf("Goroutine %d concluída\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1) // Incrementa o contador
        go processar(i, &wg)
    }

    wg.Wait() // Bloqueia até o contador chegar a zero
    fmt.Println("Todas as goroutines concluídas!")
}
```

!!! tip "Sempre passe WaitGroup por ponteiro"
    Passe `*sync.WaitGroup` (ponteiro), nunca por valor. Copiar um WaitGroup causa comportamento indefinido.

## sync.Mutex — Exclusão mútua

Quando múltiplas goroutines acessam dados compartilhados, é necessário sincronização:

```go
package main

import (
    "fmt"
    "sync"
)

type Contador struct {
    mu    sync.Mutex
    valor int
}

func (c *Contador) Incrementar() {
    c.mu.Lock()   // Adquire o lock
    defer c.mu.Unlock() // Garante liberação
    c.valor++
}

func (c *Contador) Valor() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.valor
}

func main() {
    c := &Contador{}
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            c.Incrementar()
        }()
    }

    wg.Wait()
    fmt.Println("Valor final:", c.Valor()) // 1000 (sempre!)
}
```

!!! warning "Race condition sem Mutex"
    Sem o Mutex, o resultado seria imprevisível (poderia ser 823, 991, etc.). Execute `go run -race main.go` para detectar race conditions.

## sync.RWMutex — Leituras concorrentes

Quando há muito mais leituras que escritas, `RWMutex` é mais eficiente:

```go
type Cache struct {
    mu   sync.RWMutex
    dados map[string]string
}

func (c *Cache) Get(chave string) (string, bool) {
    c.mu.RLock()  // Múltiplos leitores simultâneos permitidos
    defer c.mu.RUnlock()
    v, ok := c.dados[chave]
    return v, ok
}

func (c *Cache) Set(chave, valor string) {
    c.mu.Lock()   // Escritas são exclusivas
    defer c.mu.Unlock()
    c.dados[chave] = valor
}
```

## sync/atomic — Operações atômicas

Para contadores simples, operações atômicas são mais eficientes que mutex:

```go
import "sync/atomic"

var contador int64

// Incremento atômico (thread-safe)
atomic.AddInt64(&contador, 1)

// Leitura atômica
valor := atomic.LoadInt64(&contador)
```

## Goroutines vs Threads

| Característica     | Goroutine             | Thread OS           |
|--------------------|-----------------------|---------------------|
| **Tamanho inicial**| ~2 KB (stack dinâmica)| ~1 MB               |
| **Criação**        | Muito rápida          | Cara (syscall)      |
| **Quantidade**     | Milhões possíveis     | Milhares no máximo  |
| **Escalonamento**  | Runtime do Go (M:N)   | Kernel do SO        |
| **Comunicação**    | Channels (idiomático) | Shared memory       |

!!! info "Modelo M:N"
    O Go mapeia **M goroutines** em **N threads OS**, usando um escalonador cooperativo. Isso torna a criação de goroutines extremamente barata.
