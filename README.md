# 🚀 Rover Espacial — Simulador com Linguagem Própria

> Interpretador completo com pipeline **Lexer → Parser → AST → Simulador** para uma linguagem de controle de rover em grid 2D. Projeto acadêmico demonstrando fundamentos de compiladores em Python puro.

---

## 📋 Sumário

- [Visão Geral](#-visão-geral)
- [Arquitetura](#-arquitetura)
- [Estrutura de Arquivos](#-estrutura-de-arquivos)
- [Requisitos](#-requisitos)
- [Como Usar](#-como-usar)
- [Linguagem do Rover](#-linguagem-do-rover)
- [Ambiente de Simulação](#-ambiente-de-simulação)
- [Tratamento de Erros](#-tratamento-de-erros)

---

## 🌌 Visão Geral

O **Rover Espacial** é um simulador que interpreta uma linguagem de domínio específico (DSL) para controlar um rover em um grid 2D. Toda a cadeia de processamento — da tokenização à execução — é implementada do zero em Python, sem dependências externas.

**Destaques:**
- 🧩 Linguagem própria com comandos de movimento, detecção e controle de fluxo
- 🔍 Análise léxica, sintática e geração de AST
- 🗺️ Visualização do grid com posição e direção do rover em tempo real
- 💬 REPL interativo para exploração incremental
- ❌ Mensagens de erro precisas com linha e coluna

---

## 🏗️ Arquitetura

O código percorre o seguinte pipeline para executar um script `.rover`:

```
Código-fonte (.rover)
       │
       ▼
   [ LEXER ]      → Tokenização: texto bruto → lista de tokens
       │
       ▼
   [ PARSER ]     → Análise sintática: tokens → Árvore Sintática Abstrata (AST)
       │
       ▼
   [ AST ]        → Representação estruturada dos comandos
       │
       ▼
  [ SIMULATOR ]   → Execução passo a passo no grid 2D
       │
       ▼
  Log de execução + Mapa visual
```

| Módulo | Responsabilidade |
|---|---|
| `lexer.py` | Converte texto em tokens (`FORWARD`, `NUMBER`, `IF`, etc.) |
| `ast_nodes.py` | Define os nós da AST (dataclasses) |
| `parser.py` | Constrói a AST a partir dos tokens |
| `compiler.py` | Fachada que orquestra Lexer + Parser |
| `simulator.py` | Executa a AST no grid, gerencia estado e colisões |
| `main.py` | Ponto de entrada: REPL, modo arquivo e modo demo |

---

## 📁 Estrutura de Arquivos

```
rover/
├── lexer.py                   # Analisador Léxico
├── ast_nodes.py               # Nós da AST
├── parser.py                  # Analisador Sintático
├── compiler.py                # Fachada Lexer + Parser
├── simulator.py               # Motor de simulação 2D
├── main.py                    # Ponto de entrada (CLI)
├── autopilot.py               # Modo autopiloto automático
├── autopiloto_interativo.py   # Autopiloto interativo
└── exemplos/
    ├── exemplo_basico.rover   # Comandos básicos de movimento
    ├── exemplo_avancado.rover # IF OBSTACLE + REPEAT
    └── exemplo_erros.rover    # Erros propositais para teste
```

---

## ⚙️ Requisitos

- **Python 3.10+**
- Nenhuma biblioteca externa — apenas a stdlib!

---

## 🚀 Como Usar

### Modo Demo

A forma mais rápida de ver o simulador em ação:

```bash
python main.py --demo
```

Executa um script de demonstração completo com visualização da AST e mapa final.

---

### Modo Arquivo

Execute qualquer script `.rover`:

```bash
python main.py --file exemplos/exemplo_basico.rover
```

Para exibir também a AST compilada:

```bash
python main.py --file exemplos/exemplo_avancado.rover --ast
```

---

### Modo Interativo (REPL)

```bash
python main.py
```

Digite comandos linha a linha. **Linha em branco** executa o buffer acumulado.

| Comando especial | Ação |
|---|---|
| *(linha vazia)* | Executa os comandos digitados |
| `novo` | Reseta o rover para a posição inicial |
| `mapa` | Exibe o grid atual |
| `ast` | Mostra a AST do buffer atual |
| `ajuda` | Exibe referência completa da linguagem |
| `sair` | Encerra o REPL |

---

## 📖 Linguagem do Rover

### Comandos Básicos

| Comando | Descrição |
|---|---|
| `FORWARD <n>` | Avança **n** posições na direção atual |
| `BACKWARD <n>` | Recua **n** posições |
| `LEFT` | Gira 90° à esquerda |
| `RIGHT` | Gira 90° à direita |
| `DETECT` | Detecta se há obstáculo imediatamente à frente |

### Controle de Fluxo

| Comando | Descrição |
|---|---|
| `IF OBSTACLE THEN <cmd>` | Executa `<cmd>` somente se houver obstáculo à frente |
| `REPEAT <n> ( <cmds> )` | Repete o bloco de comandos **n** vezes |

### Comentários

```
# Isto é um comentário — linha inteira ignorada
FORWARD 3  # Comentário no fim de linha também funciona
```

### Exemplo Completo

```rover
# Missão de exploração
FORWARD 3
RIGHT
DETECT
IF OBSTACLE THEN LEFT

REPEAT 2 (
    FORWARD 2
    DETECT
)

BACKWARD 1
```

---

## 🗺️ Ambiente de Simulação

- **Grid:** 10 × 10 (coordenadas de 0–9 em X e Y)
- **Posição inicial:** `(0, 0)` — canto inferior esquerdo
- **Direção inicial:** Norte `↑`
- **Obstáculos padrão:** `(3,3)`, `(3,4)`, `(6,7)`, `(5,2)`, `(8,5)`

### Legenda do Mapa

| Símbolo | Significado |
|---|---|
| `↑ ↓ ← →` | Rover com sua direção atual |
| `▓` | Obstáculo |
| `·` | Célula livre |

O rover **não sai dos limites** do grid e **para na célula anterior** ao colidir com um obstáculo — sem crash.

---

## ❌ Tratamento de Erros

### Erros de Compilação

Detectados antes da execução, com linha e coluna exatos:

| Tipo | Exemplo |
|---|---|
| Comando inválido | `GIRAR_ESQUERDA` |
| Parâmetro inválido | `FORWARD -5` |
| Sintaxe incorreta | `IF THEN RIGHT` (sem `OBSTACLE`) |
| Bloco mal fechado | `REPEAT 3 (` sem `)` |

### Erros de Simulação

| Situação | Comportamento |
|---|---|
| Rover atinge a borda do grid | Bloqueado; movimento ignorado |
| Rover colide com obstáculo | Para na célula anterior |

---

## 📄 Licença

Projeto desenvolvido para fins acadêmicos
