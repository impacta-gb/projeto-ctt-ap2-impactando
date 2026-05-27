---
icon: lucide/package
---

# Gerenciamento de Pacotes (Go Modules)

Go Modules é o sistema oficial de gerenciamento de dependências introduzido no Go 1.11 e padrão desde o Go 1.16. Ele resolve o problema de reprodutibilidade de builds e elimina a dependência do `GOPATH`.

## Conceitos fundamentais

| Arquivo    | Função                                                       |
|------------|--------------------------------------------------------------|
| `go.mod`   | Declara o módulo, versão do Go e dependências diretas        |
| `go.sum`   | Checksums criptográficos de todas as dependências            |
| `vendor/`  | Cópia local das dependências (opcional)                      |

## Criando um módulo

```bash
# Inicializa um novo módulo
mkdir meu-app && cd meu-app
go mod init github.com/usuario/meu-app
```

Isso cria o arquivo `go.mod`:

```go title="go.mod"
module github.com/usuario/meu-app

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
    golang.org/x/crypto v0.17.0
)
```

## Comandos essenciais

### Adicionando dependências

```bash
# Adiciona e instala uma dependência
go get github.com/gin-gonic/gin@latest

# Versão específica
go get github.com/gin-gonic/gin@v1.9.1

# Downgrade para versão anterior
go get github.com/gin-gonic/gin@v1.8.0
```

### Organizando dependências

```bash
# Remove dependências não utilizadas e adiciona as faltantes
go mod tidy

# Baixa todas as dependências para o cache local
go mod download

# Copia dependências para a pasta vendor/
go mod vendor

# Verifica integridade das dependências
go mod verify
```

!!! tip "Execute go mod tidy regularmente"
    Após adicionar/remover imports no código, execute `go mod tidy` para manter o `go.mod` e `go.sum` sincronizados.

### Visualizando dependências

```bash
# Lista todas as dependências
go list -m all

# Mostra o grafo de dependências
go mod graph

# Por que uma dependência está incluída?
go mod why github.com/gin-gonic/gin
```

## Versionamento semântico

Go Modules segue **SemVer** (Semantic Versioning):

```
v MAJOR . MINOR . PATCH
  └─────   └─────   └── Bug fixes
           └─── Novas funcionalidades (retrocompatível)
  └─── Quebra de compatibilidade
```

!!! warning "Módulos v2+"
    Módulos com versão major ≥ 2 devem ter o sufixo `/v2` no caminho:
    ```go
    import "github.com/usuario/meu-pacote/v2"
    ```
    Isso garante que v1 e v2 possam coexistir no mesmo projeto.

## Pseudo-versões

Para commits sem tag de versão:

```
v0.0.0-20240115123456-abcdef012345
       └───────────  └─────────── Hash do commit (12 chars)
       Timestamp do commit
```

```bash
# Instalar versão de um commit específico
go get github.com/usuario/repo@abc1234
```

## Criando pacotes internos

```
meu-app/
├── go.mod
├── main.go
├── internal/           # Só acessível dentro do módulo
│   └── config/
│       └── config.go
└── pkg/                # Exportável para outros módulos
    └── mathutils/
        └── mathutils.go
```

```go title="pkg/mathutils/mathutils.go"
package mathutils  // Nome do pacote = nome da pasta

// Funções exportadas começam com letra MAIÚSCULA
func Soma(a, b int) int {
    return a + b
}

// Funções privadas começam com letra minúscula
func validar(n int) bool {
    return n >= 0
}
```

```go title="main.go"
package main

import (
    "fmt"
    "github.com/usuario/meu-app/pkg/mathutils"
)

func main() {
    fmt.Println(mathutils.Soma(3, 4)) // 7
}
```

## Workspace (Go 1.18+)

Para trabalhar em múltiplos módulos locais simultaneamente:

```bash
# Inicializa um workspace
go work init ./modulo-a ./modulo-b

# Adiciona um módulo ao workspace existente
go work use ./novo-modulo
```

```go title="go.work"
go 1.22

use (
    ./modulo-a
    ./modulo-b
)
```

!!! info "go.work não deve ir para produção"
    O arquivo `go.work` é para desenvolvimento local. Adicione-o ao `.gitignore` em projetos públicos para não afetar outros desenvolvedores.

## GOPROXY e GONOSUMCHECK

```bash
# Configurar proxy (padrão)
go env -w GOPROXY=https://proxy.golang.org,direct

# Usar proxy privado para pacotes internos
go env -w GOPRIVATE=github.com/minha-empresa/*
go env -w GONOSUMCHECK=github.com/minha-empresa/*
```
