# 🧮 Avaliador de Expressões Numéricas

> Trabalho Prático — Estrutura de Dados | Universidade Católica de Brasília (UCB)  
> Professor: Marcelo Eustáquio | 1° semestre de 2026

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Funcionalidades](#-funcionalidades)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Como Compilar e Executar](#-como-compilar-e-executar)
- [Interface da API](#-interface-da-api)
- [Operadores e Funções Suportados](#-operadores-e-funções-suportados)
- [Tabela de Testes](#-tabela-de-testes)
- [Tratamento de Erros](#-tratamento-de-erros)
- [Observações Técnicas](#-observações-técnicas)

---

## 📖 Sobre o Projeto

Este projeto implementa um **avaliador de expressões matemáticas em notação pós-fixa (RPN — Reverse Polish Notation)** desenvolvido em linguagem C, como parte da disciplina de **Estrutura de Dados**.

O sistema é capaz de:
- **Avaliar** o resultado numérico de expressões na notação pós-fixa;
- **Converter** expressões da notação pós-fixa para a notação infixa (e prefixada na versão final);
- Suportar operações aritméticas básicas e funções matemáticas especiais;
- Detectar e tratar entradas inválidas de forma robusta.

A estrutura de dados central utilizada é a **pilha (stack)**, fundamental para o processamento das notações.

---

## ⚙️ Funcionalidades

- [x] Conversão de notação **pós-fixa → infixa**
- [ ] Conversão de notação **pós-fixa → prefixada** *(versão final)*
- [x] Operações aritméticas: `+`, `-`, `*`, `/`, `%`, `^`
- [x] Funções matemáticas: `raiz`, `sen`, `cos`, `tg`, `log`
- [x] Ângulos em graus para funções trigonométricas
- [ ] Ângulos em radianos *(versão final)*
- [x] Validação e tratamento de erros nas expressões

---

## 📁 Estrutura do Repositório

```
.
├── calculadora.h      # Cabeçalho principal (não deve ser alterado)
├── XXXXXXXXXX.c       # Implementação das funções (arquivo principal do aluno)
├── main.c             # Arquivo de testes fornecido pelo professor
└── README.md          # Este arquivo
```

> **Nota:** Substitua `XXXXXXXXXX` pelos dígitos do seu número de matrícula (ex: `26209999.c`).

---

## 🚀 Como Compilar e Executar

### Pré-requisitos

- Compilador GCC (disponível no Linux, macOS ou via MinGW/Dev-C++ no Windows)
- Compatível com **Windows** e **Linux/macOS**

### Compilação

```bash
gcc 26209999.c main.c -o calculadora.exe -lm
```

> A flag `-lm` pode ser necessária em sistemas Linux/macOS para vincular a biblioteca matemática.

### Execução

```bash
./calculadora.exe
```

---

## 📐 Interface da API

O cabeçalho `calculadora.h` define a seguinte interface pública:

```c
#ifndef EXPRESSAO_H
#define EXPRESSAO_H

typedef struct {
    char posFixa[512]; // Expressão na forma pós-fixa, ex: "3 12 4 + *"
    char inFixa[512];  // Expressão na forma infixa, ex: "3*(12+4)"
    float Valor;       // Valor numérico da expressão
} Expressao;

char *getInFixa(char *Str); // Retorna a forma infixa de Str (pós-fixa)
float getValor(char *Str);  // Calcula o valor de Str (na forma pós-fixa)

#endif
```

### `getInFixa(char *Str)`

Recebe uma string em notação pós-fixa e retorna a expressão convertida para a notação infixa, **sem espaços** e com **parênteses mínimos necessários**. Retorna `NULL` em caso de erro.

### `getValor(char *Str)`

Recebe uma string em notação pós-fixa e retorna o valor numérico `float` da expressão avaliada.

---

## 🔢 Operadores e Funções Suportados

### Operadores Binários

| Símbolo | Operação         | Exemplo pós-fixo | Resultado |
|---------|------------------|------------------|-----------|
| `+`     | Adição           | `3 4 +`          | `7`       |
| `-`     | Subtração        | `9 3 -`          | `6`       |
| `*`     | Multiplicação    | `3 4 *`          | `12`      |
| `/`     | Divisão          | `8 2 /`          | `4`       |
| `%`     | Módulo           | `9 4 %`          | `1`       |
| `^`     | Potenciação      | `2 3 ^`          | `8`       |

### Funções Unárias

| Símbolo | Operação              | Exemplo pós-fixo | Resultado      |
|---------|-----------------------|------------------|----------------|
| `raiz`  | Raiz quadrada         | `9 raiz`         | `3`            |
| `sen`   | Seno (em graus)       | `30 sen`         | `0.5`          |
| `cos`   | Cosseno (em graus)    | `60 cos`         | `0.5`          |
| `tg`    | Tangente (em graus)   | `45 tg`          | `1`            |
| `log`   | Logaritmo decimal     | `100 log`        | `2`            |

---

## 🧪 Tabela de Testes

| # | Notação Pós-fixa          | Notação Infixa               | Valor Esperado |
|---|---------------------------|------------------------------|----------------|
| 1 | `3 4 + 5 *`               | `(3+4)*5`                    | `35`           |
| 2 | `7 2 * 4 +`               | `7*2+4`                      | `18`           |
| 3 | `8 5 2 4 + * +`           | `8+(5*(2+4))`                | `38`           |
| 4 | `6 2 / 3 + 4 *`           | `(6/2+3)*4`                  | `24`           |
| 5 | `9 5 2 8 * 4 + * +`       | `9+(5*(4+8*2))`              | `109`          |
| 6 | `2 3 + log 5 /`           | `log(2+3)/5`                 | `≈ 0.14`       |
| 7 | `10 log 3 ^ 2 +`          | `log(10)^3+2`                | `3`            |
| 8 | `45 60 + 30 cos *`        | `(45+60)*cos(30)`            | `≈ 90.93`      |
| 9 | `0.5 45 sen 2 ^ +`        | `0.5+sen(45)^2`              | `1`            |

> Para resultados aproximados, é aceita uma diferença absoluta máxima de **0.001**.

---

## ⚠️ Tratamento de Erros

As seguintes situações são consideradas **expressões inválidas** e devem ser devidamente tratadas:

| Situação                             | Comportamento esperado       |
|--------------------------------------|------------------------------|
| Operador desconhecido                | Retornar erro / `NULL`       |
| Operandos insuficientes na pilha     | Retornar erro / `NULL`       |
| Divisão por zero (`/` ou `%`)        | Retornar erro / `NULL`       |
| Logaritmo de número ≤ 0              | Retornar erro / `NULL`       |
| Raiz quadrada de número negativo     | Retornar erro / `NULL`       |

---

## 📝 Observações Técnicas

- Funções auxiliares podem ser criadas em `calculadora.c`, mas seus protótipos **não devem** ser incluídos em `calculadora.h`.
- A string retornada por `getInFixa()` não deve conter espaços nem parênteses desnecessários.
- O código deve ser compatível com o padrão C e compiladores como **Dev-C++** e **VSCode**.
- Em caso de erro em `getInFixa()`, a função deve retornar ponteiro `NULL`.

---

## 📚 Referências

- CORMEN, T. H. et al. *Introdução a Algoritmos*. 3. ed. MIT Press, 2009.
- TENENBAUM, A. M.; LANGSAM, Y.; AUGENSTEIN, M. J. *Data Structures Using C*. Prentice Hall, 1990.
- Documentação da biblioteca `math.h` — [cppreference.com](https://en.cppreference.com/w/c/numeric/math)
