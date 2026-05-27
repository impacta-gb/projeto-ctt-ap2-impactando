# Documentação Go — Grupo Impactando

[![Documentation CI/CD](https://github.com/impacta-gb/projeto-ctt-ap2-impactando/actions/workflows/docs.yml/badge.svg)](https://github.com/impacta-gb/projeto-ctt-ap2-impactando/actions/workflows/docs.yml)

> Projeto desenvolvido para a disciplina **CTT — AP2** da Faculdade Impacta Tecnologia.

## 👥 Integrantes

| Nome              | GitHub                                                           | Contribuições                             |
|-------------------|------------------------------------------------------------------|-------------------------------------------|
| Luigi Santos      | [@luigisantt7](https://github.com/luigisantt7)                   | Introdução, Sintaxe, Estruturas de Controle |
| Gabriela Lujan    | [@gabilujan](https://github.com/gabilujan)                       | Arrays/Slices/Maps, Structs, Workflow CI/CD |
| Mariane Alves     | [@marialvzx](https://github.com/marialvzx)                       | Error Handling, Goroutines, Channels        |
| Nattalia Manga    | [@nattaliamanga-coder](https://github.com/nattaliamanga-coder)   | Go Modules, Testes, Navegação/README        |

## 📚 Conteúdo da Documentação

O site documenta a linguagem Go cobrindo os seguintes tópicos:

1. **Introdução e Instalação** — O que é Go, instalação e primeiro programa
2. **Sintaxe Básica e Variáveis** — Tipos, zero values, constantes, iota
3. **Estruturas de Controle** — if/else, for, switch, range
4. **Arrays, Slices e Maps** — Estruturas de coleção fundamentais
5. **Structs e Métodos** — Composição, interfaces, struct tags
6. **Tratamento de Erros** — Padrões idiomáticos, defer/panic/recover
7. **Concorrência I: Goroutines** — Goroutines, WaitGroup, Mutex
8. **Concorrência II: Channels** — Channels, select, padrões de concorrência
9. **Gerenciamento de Pacotes** — Go Modules, go.mod, dependências
10. **Testes Automatizados** — Table-driven tests, benchmarks, cobertura

## 🔄 Fluxo de Trabalho Git

Adotamos um fluxo colaborativo estrito:

```
feature branch → Pull Request → Code Review (1 aprovação) → merge na main
```

### Regras de proteção da branch `main`

- ✅ Push direto **bloqueado** — toda alteração exige PR
- ✅ **1 aprovação obrigatória** de outro membro antes do merge
- ✅ Discussões/comentários construtivos nos PRs
- ✅ Feature branches nomeadas com convenção: `feat/`, `fix/`, `docs/`

### Nomenclatura de branches utilizada

| Branch                       | Autor       | Conteúdo                        |
|------------------------------|-------------|---------------------------------|
| `feat/doc-intro-instalacao`  | luigisantt7 | Introdução e Instalação         |
| `feat/doc-sintaxe-variaveis` | luigisantt7 | Sintaxe e Variáveis             |
| `feat/doc-controle`          | luigisantt7 | Estruturas de Controle          |
| `feat/doc-arrays-slices-maps`| gabilujan   | Arrays, Slices e Maps           |
| `feat/doc-structs-metodos`   | gabilujan   | Structs e Métodos               |
| `feat/workflow-cicd`         | gabilujan   | GitHub Actions CI/CD            |
| `feat/doc-error-handling`    | marialvzx   | Tratamento de Erros             |
| `feat/doc-goroutines`        | marialvzx   | Goroutines                      |
| `feat/doc-channels`          | marialvzx   | Channels                        |
| `feat/doc-go-modules`        | nattaliamanga-coder | Go Modules              |
| `feat/doc-testes`            | nattaliamanga-coder | Testes Automatizados    |
| `feat/readme-e-nav`          | nattaliamanga-coder | README e Navegação      |

## ⚙️ Arquitetura do Workflow (GitHub Actions)

O pipeline CI/CD em `.github/workflows/docs.yml` foi construído com os seguintes requisitos:

### Triggers

```yaml
on:
  push:
    branches: [main]          # Deploy após merge
  pull_request:
    branches: [main]          # Validação antes do merge
  schedule:
    - cron: '0 0 * * 0'      # Toda domingo à meia-noite
```

### Jobs

```
┌─────────────────────────────────────────────────────┐
│                  build_site                          │
│  strategy: matrix: python-version: [3.10, 3.11]     │
│  ┌──────────────────┐  ┌──────────────────┐         │
│  │  Python 3.10     │  │  Python 3.11     │         │
│  │  ─ checkout      │  │  ─ checkout      │         │
│  │  ─ setup-python  │  │  ─ setup-python  │         │
│  │  ─ cache pip     │  │  ─ cache pip     │         │
│  │  ─ pip install   │  │  ─ pip install   │         │
│  │  ─ zensical build│  │  ─ zensical build│         │
│  │                  │  │  ─ upload artifact│         │
│  └──────────────────┘  └──────────────────┘         │
└─────────────────────────────────────────────────────┘
              │ needs: build_site
              ▼
┌─────────────────────────────────────────────────────┐
│  deploy_site                                         │
│  if: push || schedule  ← NUNCA em pull_request!     │
│  ─ download artifact                                 │
│  ─ configure-pages                                   │
│  ─ deploy-pages                                      │
└─────────────────────────────────────────────────────┘
```

### Funcionalidades do pipeline

| Recurso                  | Implementação                          |
|--------------------------|----------------------------------------|
| **Matrix Strategy**      | Build em Python 3.10 e 3.11            |
| **Cache de dependências**| `actions/cache` para `~/.cache/pip`    |
| **Desacoplamento**       | `build_site` → `deploy_site` via `needs:` |
| **Transferência**        | `upload-pages-artifact` / `download-artifact` |
| **Condicional segura**   | `if: github.event_name != 'pull_request'` |
| **Agendamento**          | Cron semanal todo domingo              |

## 🚀 Como executar localmente

```bash
# Clone o repositório
git clone https://github.com/impacta-gb/projeto-ctt-ap2-impactando.git
cd projeto-ctt-ap2-impactando

# Instale o Zensical
pip install zensical

# Inicie o servidor de desenvolvimento
zensical serve

# Build de produção
zensical build --clean
```

O site estará disponível em `http://127.0.0.1:8000`.
