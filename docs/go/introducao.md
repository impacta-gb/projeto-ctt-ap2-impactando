---
icon: lucide/book-open
---

# Introdução e Instalação do Go

Go (também chamado de **Golang**) é uma linguagem de programação de código aberto criada pelo Google em 2009 por **Robert Griesemer**, **Rob Pike** e **Ken Thompson**. Foi projetada para ser simples, eficiente e com suporte nativo à concorrência.

## Por que aprender Go?

| Característica       | Descrição                                                     |
|----------------------|---------------------------------------------------------------|
| **Performance**      | Compilada para código nativo, velocidade próxima ao C         |
| **Simplicidade**     | Sintaxe enxuta, fácil de aprender                             |
| **Concorrência**     | Goroutines e Channels nativos                                 |
| **Tooling**          | Ferramentas integradas: `fmt`, `test`, `vet`, `build`         |
| **Tipagem estática** | Erros detectados em tempo de compilação                       |

## Instalação

### Windows

1. Acesse [https://go.dev/dl/](https://go.dev/dl/)
2. Baixe o instalador `.msi` mais recente
3. Execute o instalador e siga os passos
4. Verifique a instalação:

```bash
go version
# go version go1.22.0 windows/amd64
```

### Linux / macOS

```bash
# Baixar e instalar (substitua pela versão mais recente)
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Adicionar ao PATH (bash/zsh)
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# Verificar
go version
```

### Verificando o ambiente

```bash
go env
```

Deve mostrar variáveis como `GOPATH`, `GOROOT`, `GOOS`, `GOARCH`.

## Seu primeiro programa Go

Crie um arquivo `hello.go`:

```go title="hello.go"
package main

import "fmt"

func main() {
    fmt.Println("Olá, Mundo!") // (1)!
}
```

1. `fmt.Println` imprime uma linha no terminal. O pacote `fmt` é parte da biblioteca padrão.

Execute:

```bash
go run hello.go
# Olá, Mundo!
```

Ou compile e execute:

```bash
go build hello.go
./hello        # Linux/macOS
hello.exe      # Windows
```

!!! tip "Dica"
    Use `go run` durante o desenvolvimento para executar rapidamente sem compilar um binário permanente.

!!! info "GOPATH vs Módulos"
    Desde o Go 1.11, a gestão de dependências é feita com **Go Modules** (`go.mod`). O `GOPATH` ainda existe mas não é mais obrigatório para organizar projetos.

## Estrutura básica de um projeto Go

```
meu-projeto/
├── go.mod          # Declaração do módulo e dependências
├── go.sum          # Hash das dependências (gerado automaticamente)
├── main.go         # Ponto de entrada da aplicação
└── internal/       # Pacotes internos do projeto
    └── utils/
        └── utils.go
```

Inicializando um novo módulo:

```bash
mkdir meu-projeto && cd meu-projeto
go mod init github.com/usuario/meu-projeto
```

## Ferramentas essenciais

```bash
go fmt ./...      # Formata todo o código
go vet ./...      # Analisa o código em busca de erros
go test ./...     # Executa todos os testes
go build ./...    # Compila todos os pacotes
```

!!! warning "Atenção"
    Go tem formatação **obrigatória**. Sempre execute `go fmt` antes de fazer commit. Editores como VS Code com a extensão Go fazem isso automaticamente ao salvar.
