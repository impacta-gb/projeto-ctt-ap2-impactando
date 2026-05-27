---
icon: lucide/layers
---

# Arrays, Slices e Maps

Go oferece três estruturas de dados fundamentais para coleções: **arrays** (tamanho fixo), **slices** (tamanho dinâmico) e **maps** (chave-valor).

## Arrays

Arrays têm tamanho fixo, definido em tempo de compilação:

```go title="arrays.go"
// Declaração
var nums [5]int                    // [0 0 0 0 0]
primos := [5]int{2, 3, 5, 7, 11}
cores := [...]string{"red", "green", "blue"} // tamanho inferido

fmt.Println(len(primos)) // 5
fmt.Println(primos[0])   // 2
```

!!! warning "Arrays são valores, não referências"
    Em Go, ao atribuir um array a outra variável, é feita uma **cópia completa**:
    ```go
    a := [3]int{1, 2, 3}
    b := a       // cópia!
    b[0] = 99
    fmt.Println(a[0]) // 1 — a não mudou
    fmt.Println(b[0]) // 99
    ```

## Slices

Slices são a estrutura mais usada em Go — um view dinâmico sobre um array:

```go
// Criação
s := []int{1, 2, 3, 4, 5}
vazio := make([]int, 5)        // [0 0 0 0 0], len=5, cap=5
comCap := make([]int, 3, 10)   // len=3, cap=10

fmt.Println(len(s), cap(s))    // 5 5
```

### Fatiamento (slicing)

```go
nums := []int{10, 20, 30, 40, 50}

a := nums[1:3]   // [20 30]    (índice 1 até 2)
b := nums[:3]    // [10 20 30] (do início até 2)
c := nums[2:]    // [30 40 50] (do índice 2 até o fim)
d := nums[:]     // cópia da referência completa
```

!!! info "Slices compartilham memória"
    Um slice é uma referência ao array subjacente. Modificar um slice modifica o array original:
    ```go
    original := []int{1, 2, 3}
    sub := original[0:2]
    sub[0] = 99
    fmt.Println(original) // [99 2 3]
    ```
    Para evitar isso, use `copy()`.

### append e copy

```go
s := []int{1, 2, 3}

// append pode mudar o array subjacente!
s = append(s, 4, 5)          // [1 2 3 4 5]
s = append(s, []int{6, 7}...) // spread operator

// copy — cópia segura
dst := make([]int, len(s))
n := copy(dst, s)
fmt.Println(n, dst) // 7 [1 2 3 4 5 6 7]
```

### Iteração

```go
frutas := []string{"maçã", "banana", "uva"}
for i, f := range frutas {
    fmt.Printf("%d: %s\n", i, f)
}
```

## Maps

Maps são dicionários chave-valor com lookup O(1):

```go title="maps.go"
// Criação
capitais := map[string]string{
    "BR": "Brasília",
    "US": "Washington",
    "JP": "Tóquio",
}

// Com make
scores := make(map[string]int)
scores["Alice"] = 95
scores["Bob"] = 87
```

### Operações básicas

```go
// Leitura
capital := capitais["BR"]          // "Brasília"
inexistente := capitais["XX"]      // "" (zero value, não erro!)

// Verificar existência
if capital, ok := capitais["BR"]; ok {
    fmt.Println("Capital:", capital)
} else {
    fmt.Println("País não encontrado")
}

// Deletar
delete(capitais, "JP")

// Tamanho
fmt.Println(len(capitais)) // 2
```

!!! tip "Sempre use o padrão ok"
    Acessar uma chave inexistente retorna o zero value do tipo, não um erro. Use sempre `valor, ok := mapa[chave]` quando a existência da chave for incerta.

### Iteração sobre maps

```go
// ATENÇÃO: a ordem de iteração é aleatória!
for chave, valor := range capitais {
    fmt.Printf("%s: %s\n", chave, valor)
}
```

!!! warning "Ordem aleatória em maps"
    Go **não garante** ordem de iteração em maps. Se você precisar de ordem, ordene as chaves em um slice antes de iterar.

## Comparação: Array vs Slice vs Map

| Característica     | Array           | Slice            | Map                    |
|--------------------|-----------------|------------------|------------------------|
| **Tamanho**        | Fixo            | Dinâmico         | Dinâmico               |
| **Indexação**      | `[i]`           | `[i]`            | `[chave]`              |
| **Chave**          | `int` (0..n-1)  | `int` (0..n-1)   | Qualquer tipo comparável |
| **Zero value**     | Array zerado    | `nil`            | `nil`                  |
| **Uso típico**     | Tamanho fixo    | Coleções gerais  | Lookup por chave       |
