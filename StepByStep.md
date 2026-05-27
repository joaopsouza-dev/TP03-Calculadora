# 🗺️ Guia Lógico de Desenvolvimento — Avaliador de Expressões

> Este documento descreve a **ordem lógica de raciocínio e desenvolvimento** do projeto,
> indicando o que construir primeiro, qual estrutura usar em cada etapa e por quê.

---

## 📋 Índice

- [Visão Geral da Estratégia](#-visão-geral-da-estratégia)
- [Passo 1 — Entender a Notação Pós-fixa](#-passo-1--entender-a-notação-pós-fixa)
- [Passo 2 — Projetar a Pilha](#-passo-2--projetar-a-pilha)
- [Passo 3 — Fazer o Tokenizador](#-passo-3--fazer-o-tokenizador)
- [Passo 4 — Implementar `getValor()`](#-passo-4--implementar-getvalor)
- [Passo 5 — Implementar `getInFixa()`](#-passo-5--implementar-getinfixa)
- [Passo 6 — Adicionar Funções Matemáticas](#-passo-6--adicionar-funções-matemáticas)
- [Passo 7 — Tratamento de Erros](#-passo-7--tratamento-de-erros)
- [Passo 8 — Testar Progressivamente](#-passo-8--testar-progressivamente)
- [Passo 9 — Versão Final (Prefixada + Radianos)](#-passo-9--versão-final-prefixada--radianos)
- [Resumo da Ordem de Desenvolvimento](#-resumo-da-ordem-de-desenvolvimento)

---

## 🧭 Visão Geral da Estratégia

O projeto tem **duas funções públicas** a implementar: `getValor()` e `getInFixa()`. O segredo é perceber que ambas usam **a mesma lógica central** — percorrer a expressão pós-fixa token por token, usando uma pilha. A diferença é apenas o que a pilha guarda:

| Função        | O que a pilha guarda | O que a função retorna      |
|---------------|----------------------|-----------------------------|
| `getValor()`  | Números (`float`)    | O resultado numérico        |
| `getInFixa()` | Strings (pedaços de expressão infixa) | A expressão montada |

Por isso, a estratégia é:

1. Construir a **pilha** e o **tokenizador** como base reutilizável
2. Implementar e validar `getValor()` primeiro (mais simples)
3. Depois implementar `getInFixa()` reaproveitando a mesma lógica

---

## 📌 Passo 1 — Entender a Notação Pós-fixa

Antes de escrever qualquer linha de código, internalize a regra de ouro da notação pós-fixa:

> **Ao encontrar um número → empilha. Ao encontrar um operador → desempilha os operandos, opera e empilha o resultado.**

### Exemplo mental — `3 4 + 5 *`

```
Token "3"  → pilha: [3]
Token "4"  → pilha: [3, 4]
Token "+"  → desempilha 4 e 3, calcula 3+4=7, empilha → pilha: [7]
Token "5"  → pilha: [7, 5]
Token "*"  → desempilha 5 e 7, calcula 7*5=35, empilha → pilha: [35]
Fim        → resultado = topo da pilha = 35 ✅
```

### Regra para funções unárias (sen, cos, log...)

Funções unárias como `sen`, `cos`, `log` consomem **apenas um operando** do topo:

```
Token "45" → pilha: [45]
Token "sen"→ desempilha 45, calcula sen(45°), empilha → pilha: [0.707]
```

Entendendo isso, os passos seguintes ficam diretos.

---

## 📌 Passo 2 — Projetar a Pilha

A pilha é a **estrutura central** do projeto. Como você precisará de duas pilhas diferentes (uma de `float` para `getValor`, outra de `strings` para `getInFixa`), pense em como organizar isso.

### Decisão de design

Você tem duas opções:

**Opção A — Duas pilhas separadas** (mais simples e recomendada para iniciantes)
- Uma `struct PilhaFloat` com array de `float` e índice do topo
- Uma `struct PilhaStr` com array de `char[]` e índice do topo

**Opção B — Pilha genérica com `void*`** (mais elegante, porém mais complexa)

> 💡 **Recomendação:** Use a Opção A. É mais clara, mais segura e suficiente para o escopo do projeto.

### O que a pilha precisa ter

Para cada tipo de pilha, implemente as operações:
- `push` — insere no topo
- `pop` — remove e retorna o topo
- `peek` — consulta o topo sem remover
- `isEmpty` — verifica se está vazia
- `isFull` — verifica se está cheia (se usar array estático)

> ⚠️ Defina um tamanho máximo razoável para a pilha. Como a expressão tem até 512 caracteres, uma pilha de 64 posições é mais que suficiente.

---

## 📌 Passo 3 — Fazer o Tokenizador

O **tokenizador** é a função que lê a string da expressão pós-fixa e a divide em tokens (pedaços separados por espaço). Ele deve ser implementado antes das funções principais, pois ambas dependem dele.

### O que um token pode ser

Ao ler a expressão, cada token será uma das seguintes categorias:

```
1. Número        → ex: "3", "0.5", "3.14"
2. Operador      → ex: "+", "-", "*", "/", "%", "^"
3. Função unária → ex: "sen", "cos", "tg", "log", "raiz"
4. Inválido      → qualquer outra coisa
```

### Como categorizar

Monte funções auxiliares de classificação, por exemplo:

- `isOperator(token)` → retorna 1 se o token é `+`, `-`, `*`, `/`, `%` ou `^`
- `isUnaryFunc(token)` → retorna 1 se o token é `sen`, `cos`, `tg`, `log` ou `raiz`
- `isNumber(token)` → tenta converter com `strtof()` e verifica se teve sucesso

> 💡 Use `strtok()` da biblioteca `<string.h>` para dividir a string por espaços. **Atenção:** `strtok()` modifica a string original, então trabalhe sempre com uma **cópia** da entrada usando `strcpy()`.

---

## 📌 Passo 4 — Implementar `getValor()`

Com a pilha de `float` e o tokenizador prontos, `getValor()` segue um fluxo direto:

### Fluxo lógico

```
Para cada token na expressão:
│
├── É número?
│     └── Converte para float com strtof() e empilha
│
├── É operador binário? (+, -, *, /, %, ^)
│     ├── Desempilha o operando B (topo)
│     ├── Desempilha o operando A
│     ├── Calcula A op B
│     └── Empilha o resultado
│
├── É função unária? (sen, cos, tg, log, raiz)
│     ├── Desempilha o operando A (topo)
│     ├── Aplica a função em A
│     └── Empilha o resultado
│
└── Não reconhecido?
      └── Sinaliza erro e retorna
```

### Ordem dos operandos — ponto crítico

Ao desempilhar para um operador binário, lembre-se que o **primeiro a sair é o operando da direita**:

```
Expressão: 8 2 /
Pilha antes de "/": [8, 2]   (2 está no topo)

Pop → B = 2   (operando direito)
Pop → A = 8   (operando esquerdo)
Resultado = A / B = 8 / 2 = 4 ✅

Se você fizesse B / A, obteria 0.25 ❌
```

### Funções matemáticas

- Use `<math.h>` para `sqrt()`, `sin()`, `cos()`, `tan()`, `log10()`, `pow()`
- Para seno, cosseno e tangente, **converta graus para radianos** antes de chamar as funções da `math.h`, usando a fórmula: `radianos = graus * (M_PI / 180.0)`

---

## 📌 Passo 5 — Implementar `getInFixa()`

`getInFixa()` segue **exatamente a mesma lógica** de `getValor()`, porém a pilha guarda **strings** (pedaços de expressão infixa) em vez de números.

### Fluxo lógico

```
Para cada token na expressão:
│
├── É número?
│     └── Empilha o token como string (ex: "3", "0.5")
│
├── É operador binário?
│     ├── Desempilha a string B (topo)
│     ├── Desempilha a string A
│     ├── Monta nova string: "A op B" ou "(A op B)"
│     └── Empilha a nova string
│
└── É função unária?
      ├── Desempilha a string A (topo)
      ├── Monta nova string: "func(A)"
      └── Empilha a nova string
```

### A parte mais difícil: quando colocar parênteses

O enunciado exige **parênteses mínimos necessários**. A regra geral é:

> Coloque parênteses ao redor de uma subexpressão **somente se ela for operando de um operador de maior precedência**.

A ordem de precedência (da menor para a maior):

```
1. + e -          (menor precedência)
2. * e / e %
3. ^
4. funções unárias (maior precedência — nunca precisam de parênteses no argumento simples)
```

#### Estratégia simplificada (recomendada)

Para não errar, guarde junto com cada string na pilha também a **precedência do operador raiz** daquela subexpressão. Na hora de montar uma nova subexpressão:

- Se o operador atual tiver **precedência maior** que o operador da subexpressão da pilha → adiciona parênteses
- Caso contrário → não adiciona

> 💡 Exemplos: `3+4` empilhado com precedência `1`. Se o próximo operador for `*`, como `*` tem precedência maior que `+`, envolve com parênteses → `(3+4)*5`.

---

## 📌 Passo 6 — Adicionar Funções Matemáticas

As funções unárias seguem um padrão uniforme, o que facilita muito a implementação. Crie uma função auxiliar como `aplicarFuncao(char *nome, float valor)` que recebe o nome do token e o operando, e retorna o resultado. Isso centraliza toda a lógica de funções em um único lugar.

### Mapeamento

```
"raiz" → sqrt(valor)       [validar: valor >= 0]
"sen"  → sin(graus2rad(valor))
"cos"  → cos(graus2rad(valor))
"tg"   → tan(graus2rad(valor))
"log"  → log10(valor)      [validar: valor > 0]
```

> Centralizar em uma única função torna o tratamento de erros e a adição de novas funções muito mais fáceis.

---

## 📌 Passo 7 — Tratamento de Erros

O tratamento de erros deve ser pensado **junto com cada operação**, não como uma etapa separada. Para cada operação, existe uma condição de invalidade:

| Operação      | Condição de erro                         | O que fazer                    |
|---------------|------------------------------------------|--------------------------------|
| Qualquer op   | Pilha vazia ao tentar desempilhar        | Operandos insuficientes → erro |
| `/` e `%`     | Divisor (B) igual a zero                 | Divisão por zero → erro        |
| `raiz`        | Operando negativo                        | Raiz de negativo → erro        |
| `log`         | Operando ≤ 0                             | Log inválido → erro            |
| Token lido    | Não é número, operador nem função válida | Operador desconhecido → erro   |
| Final         | Pilha com mais de 1 elemento             | Expressão mal formada → erro   |

### Como sinalizar o erro

Use uma variável de controle `int erro = 0` dentro das funções. Ao detectar uma condição inválida, marque `erro = 1` e interrompa o processamento. No final:
- `getValor()` pode retornar um valor sentinela (como `0.0f`) ou um valor específico de erro
- `getInFixa()` deve retornar `NULL` conforme especificado no enunciado

---

## 📌 Passo 8 — Testar Progressivamente

Não espere ter tudo pronto para testar. Siga esta ordem de validação:

### Fase 1 — Testes básicos de `getValor()`
Comece pelos casos mais simples e aumente a complexidade:

```
"3 4 +"          → 7
"7 2 *"          → 14
"3 4 + 5 *"      → 35
"6 2 /"          → 3
"2 3 ^"          → 8
```

### Fase 2 — Testes com funções unárias
```
"9 raiz"         → 3
"10 log"         → 1
"30 sen"         → 0.5
"60 cos"         → 0.5
```

### Fase 3 — Testes da tabela oficial
Valide os 9 casos do enunciado um por um.

### Fase 4 — Testes de `getInFixa()`
Valide as conversões paralelamente aos testes de valor.

### Fase 5 — Testes de erro
Force cada condição inválida e verifique que o programa não trava nem produz lixo:
```
"5 0 /"          → erro (divisão por zero)
"0 log"          → erro (log de zero)
"-4 raiz"        → erro (raiz de negativo)
"3 + 4"          → erro (operandos insuficientes)
"3 4 abc +"      → erro (operador desconhecido)
```

---

## 📌 Passo 9 — Versão Final (Prefixada + Radianos)

Após a versão básica funcionar completamente, implemente as duas exigências da versão final:

### Conversão para notação prefixada

A lógica é análoga à conversão para infixa, mas mais simples — não há necessidade de parênteses na notação prefixada. O operador sempre vem antes dos operandos:

```
Pós-fixa: "3 4 +"
Infixa:   "3+4"
Prefixada: "+3 4"
```

### Ângulos em radianos

Basta remover a conversão graus → radianos que foi feita no Passo 6. Os valores já chegam em radianos, então são passados diretamente para as funções `sin()`, `cos()` e `tan()` da `math.h`.

---

## ✅ Resumo da Ordem de Desenvolvimento

```
1. [ ] Entender e simular manualmente 2 ou 3 exemplos da tabela
2. [ ] Implementar a estrutura de Pilha de float
3. [ ] Implementar a estrutura de Pilha de strings
4. [ ] Implementar o tokenizador (dividir string em tokens e classificá-los)
5. [ ] Implementar getValor() para operadores básicos (+, -, *, /, %, ^)
6. [ ] Validar getValor() com os testes da Fase 1
7. [ ] Adicionar funções unárias (raiz, sen, cos, tg, log) em getValor()
8. [ ] Validar getValor() com os 9 casos da tabela oficial
9. [ ] Implementar getInFixa() reaproveitando a lógica de tokens
10. [ ] Implementar a lógica de parênteses mínimos em getInFixa()
11. [ ] Validar getInFixa() com os 9 casos da tabela oficial
12. [ ] Adicionar tratamento de todos os erros previstos
13. [ ] Testar casos de erro intencionalmente
14. [ ] (Versão final) Adicionar conversão para notação prefixada
15. [ ] (Versão final) Ajustar trigonometria para radianos
```

---

> 💬 **Dica final:** Implemente e teste um passo de cada vez. Avançar para o próximo passo com o anterior funcionando corretamente economiza horas de depuração.
