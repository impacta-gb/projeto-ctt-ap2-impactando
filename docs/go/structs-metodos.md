---
icon: lucide/box
---

# Structs e Métodos

Go é uma linguagem orientada a objetos de forma diferente — não há classes, mas **structs** com **métodos** cumprem esse papel de forma mais explícita e simples.

## Structs

Uma struct é um tipo composto que agrupa campos relacionados:

```go title="structs.go"
type Pessoa struct {
    Nome  string
    Idade int
    Email string
}

// Criação
p1 := Pessoa{Nome: "Alice", Idade: 30, Email: "alice@go.dev"}
p2 := Pessoa{"Bob", 25, "bob@go.dev"}  // posicional (não recomendado)

fmt.Println(p1.Nome)  // Alice
fmt.Println(p2.Idade) // 25
```

### Struct anônima

```go
config := struct {
    Host string
    Port int
}{
    Host: "localhost",
    Port: 8080,
}
```

### Ponteiros para structs

```go
p := &Pessoa{Nome: "Carlos", Idade: 28}
p.Idade = 29  // Go automaticamente desreferencia: (*p).Idade = 29
fmt.Println(p.Idade) // 29
```

### new()

```go
p := new(Pessoa) // Cria um *Pessoa com zero values
p.Nome = "Diana"
```

## Métodos

Métodos são funções com um **receiver** (receptor) — o objeto ao qual o método pertence:

```go
type Retangulo struct {
    Largura float64
    Altura  float64
}

// Método com value receiver
func (r Retangulo) Area() float64 {
    return r.Largura * r.Altura
}

// Método com pointer receiver (modifica o original)
func (r *Retangulo) Escalar(fator float64) {
    r.Largura *= fator
    r.Altura *= fator
}

func main() {
    rect := Retangulo{Largura: 10, Altura: 5}
    fmt.Println(rect.Area())  // 50

    rect.Escalar(2)
    fmt.Println(rect.Area())  // 200
}
```

!!! tip "Value vs Pointer Receiver"
    | Receiver     | Modifica original? | Uso recomendado               |
    |--------------|-------------------|-------------------------------|
    | `(r Tipo)`   | Não               | Structs pequenas, leitura     |
    | `(r *Tipo)`  | Sim               | Structs grandes, mutação      |
    
    **Regra geral**: se um método precisa modificar a struct, use pointer receiver. Se a struct for grande (muitos campos), prefira pointer por performance.

## Composição (Embedding)

Go não tem herança, mas tem **embedding** — composição de tipos:

```go
type Animal struct {
    Nome string
}

func (a Animal) Falar() string {
    return a.Nome + " faz um som"
}

type Cachorro struct {
    Animal            // embedding — herda campos e métodos!
    Raca string
}

func (c Cachorro) Falar() string {
    return c.Nome + " late: Au au!"
}

func main() {
    d := Cachorro{
        Animal: Animal{Nome: "Rex"},
        Raca:   "Labrador",
    }
    fmt.Println(d.Nome)    // Rex (acesso direto ao campo embedded)
    fmt.Println(d.Falar()) // Rex late: Au au!
}
```

## Interfaces

Interfaces definem **comportamento** — qualquer tipo que implemente todos os métodos de uma interface a satisfaz automaticamente (duck typing):

```go
type Geometria interface {
    Area() float64
    Perimetro() float64
}

type Circulo struct {
    Raio float64
}

func (c Circulo) Area() float64 {
    return math.Pi * c.Raio * c.Raio
}

func (c Circulo) Perimetro() float64 {
    return 2 * math.Pi * c.Raio
}

// Retangulo já implementa Area() e Perimetro() definidos acima
// Ambos satisfazem a interface Geometria automaticamente!

func imprimirInfo(g Geometria) {
    fmt.Printf("Área: %.2f | Perímetro: %.2f\n", g.Area(), g.Perimetro())
}
```

!!! info "Interface implícita"
    Em Go, não há `implements` ou `extends`. Um tipo satisfaz uma interface simplesmente implementando seus métodos. Isso promove baixo acoplamento.

## Struct Tags

Tags são metadados adicionados aos campos de uma struct, muito usados para serialização JSON:

```go
type Usuario struct {
    ID    int    `json:"id"`
    Nome  string `json:"nome"`
    Email string `json:"email,omitempty"` // omite se vazio
    Senha string `json:"-"`               // nunca serializa
}

u := Usuario{ID: 1, Nome: "Alice", Email: "alice@go.dev"}
data, _ := json.Marshal(u)
fmt.Println(string(data))
// {"id":1,"nome":"Alice","email":"alice@go.dev"}
```
