Vou visualizar o arquivo para traduzir seu conteúdo.

Aqui está a tradução do conteúdo para português:

---

<h1>Bend</h1>
<p>Uma linguagem de programação de alto nível e massivamente paralela</p>

## Índice
1. [Introdução](#introdução)
2. [Notas Importantes](#notas-importantes)
3. [Instalação](#instalação)
4. [Primeiros Passos](#primeiros-passos)
5. [Exemplos de Aceleração](#exemplos-de-aceleração)
6. [Recursos Adicionais](#recursos-adicionais)

## Introdução

Bend oferece a sensação e recursos de linguagens expressivas como Python e Haskell. Isso inclui alocações rápidas de objetos, suporte completo para funções de ordem superior com closures, recursão irrestrita e até continuações.

Bend escala como CUDA, executando em hardware massivamente paralelo como GPUs, com aceleração quase linear baseada na contagem de núcleos, e sem anotações explícitas de paralelismo: sem criação de threads, locks, mutexes ou atômicos.

Bend é alimentado pelo runtime [HVM2](https://github.com/higherorderco/hvm).

## Notas Importantes

* Bend é projetado para se destacar em escalar performance com núcleos, suportando mais de 10000 threads simultâneas.
* A versão atual pode ter performance de núcleo único mais baixa.
* Você pode esperar melhorias substanciais na performance conforme avançamos em nossas técnicas de geração de código e otimização.
* Ainda estamos trabalhando para suportar Windows. Use [WSL2](https://learn.microsoft.com/pt-br/windows/wsl/install) como solução alternativa.
* [Atualmente suportamos apenas GPUs NVIDIA](https://github.com/HigherOrderCO/Bend/issues/341).

## Instalação

### Instalar dependências

#### No Linux
```sh
# Instale Rust se ainda não tiver.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Para a versão C do Bend, use GCC. Recomendamos uma versão até 12.x.
sudo apt install gcc
```
Para o runtime CUDA [instale o CUDA toolkit para Linux](https://developer.nvidia.com/cuda-downloads?target_os=Linux) versão 12.x.

#### No Mac
```sh
# Instale Rust se ainda não tiver.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Para a versão C do Bend, use GCC. Recomendamos uma versão até 12.x.
brew install gcc
```

### Instalar Bend

1. Instale o HVM2 executando:
```sh
# HVM2 é o avaliador de Combinadores de Interação massivamente paralelo da HOC.
cargo install hvm

# Isso garante que o HVM está corretamente instalado e acessível.
hvm --version
```

2. Instale o Bend executando:
```sh
# Este comando irá instalar o Bend
cargo install bend-lang

# Isso garante que o Bend está corretamente instalado e acessível.
bend --version
```

### Primeiros Passos

#### Executando Programas Bend
```sh
bend run    <arquivo.bend> # usa o interpretador C por padrão (paralelo)
bend run-rs <arquivo.bend> # usa o interpretador Rust (sequencial)
bend run-c  <arquivo.bend> # usa o interpretador C (paralelo)
bend run-cu <arquivo.bend> # usa o interpretador CUDA (massivamente paralelo)

# Notas
# Você também pode compilar Bend para arquivos C/CUDA standalone usando gen-c e gen-cu para máxima performance.
# O gerador de código ainda está em seus estágios iniciais e não é tão maduro quanto compiladores como GCC e GHC.
# Você pode usar a flag -s para ter mais informações sobre
  # Reduções
  # Tempo que o código levou para executar
  # Interações por segundo (Em milhões)
```

#### Testando Programas Bend
O exemplo abaixo soma todos os números no intervalo de `start` até `target`. Pode ser escrito em dois métodos diferentes: um que é inerentemente sequencial (e portanto não pode ser paralelizado), e outro que é facilmente paralelizável. (Usaremos a flag `-s` na maioria dos exemplos, para fins de visibilidade)

#### Versão Sequencial:
Primeiro, crie um arquivo chamado `sequential_sum.bend`
```sh
# Escreva este comando no seu terminal
touch sequential_sum.bend
```
Então com seu editor de texto, abra o arquivo `sequential_sum.bend`, copie o código abaixo e cole no arquivo.

```py
# Define a função Sum com dois parâmetros: start e target
def Sum(start, target):
  if start == target:
    # Se o valor de start é o mesmo que target, retorna start.
    return start
  else:
    # Se start não é igual a target, chama recursivamente Sum com
    # start incrementado em 1, e adiciona o resultado a start.
    return start + Sum(start + 1, target)  

def main():
  # Isso se traduz em (1 + (2 + (3 + (....... + (999999 + 1000000)))))
  # Note que isso vai estourar o valor máximo de um número em Bend
  return Sum(1, 1_000_000)
```

##### Executando o arquivo
Você pode executá-lo usando o interpretador Rust (Sequencial)
```sh
bend run-rs sequential_sum.bend -s
```

Ou pode executá-lo usando o interpretador C (Sequencial)
```sh
bend run-c sequential_sum.bend -s
```

Se você tem uma GPU NVIDIA, também pode executar em CUDA (Sequencial)
```sh
bend run-cu sequential_sum.bend -s
```

Nesta versão, o próximo valor a ser calculado depende da soma anterior, significando que não pode prosseguir até que a computação atual esteja completa. Agora, vejamos a versão facilmente paralelizável.

#### Versão Paralelizável:
Primeiro feche o arquivo antigo e então vá para o seu terminal para criar `parallel_sum.bend`
```sh
# Escreva este comando no seu terminal
touch parallel_sum.bend
```
Então com seu editor de texto, abra o arquivo `parallel_sum.bend`, copie o código abaixo e cole no arquivo.

```py
# Define a função Sum com dois parâmetros: start e target
def Sum(start, target):
  if start == target:
    # Se o valor de start é o mesmo que target, retorna start.
    return start
  else:
    # Se start não é igual a target, calcula o ponto médio (half),
    # então chama recursivamente Sum em ambas as metades.
    half = (start + target) / 2
    left = Sum(start, half)  # (Start -> Half)
    right = Sum(half + 1, target)
    return left + right

# Uma soma paralelizável de números de 1 a 1000000
def main():
  # Isso se traduz em (((1 + 2) + (3 + 4)) + ... (999999 + 1000000)...)
  return Sum(1, 1_000_000)
```

Neste exemplo, a soma (3 + 4) não depende da (1 + 2), significando que pode executar em paralelo porque ambas computações podem acontecer ao mesmo tempo.

##### Executando o arquivo
Você pode executá-lo usando o interpretador Rust (Sequencial)
```sh
bend run-rs parallel_sum.bend -s
```

Ou pode executá-lo usando o interpretador C (Paralelo)
```sh
bend run-c parallel_sum.bend -s
```

Se você tem uma GPU NVIDIA, também pode executar em CUDA (Massivamente paralelo)
```sh
bend run-cu parallel_sum.bend -s
```

Em Bend, pode ser paralelizado apenas mudando o comando de execução. Se seu código **pode** executar em paralelo ele **vai** executar em paralelo.

### Exemplos de Aceleração
O trecho de código abaixo implementa um [ordenador bitônico](https://en.wikipedia.org/wiki/Bitonic_sorter) com *rotações de árvore imutáveis*. Não é o tipo de algoritmo que você esperaria executar rápido em GPUs. No entanto, como usa uma abordagem de dividir e conquistar, que é inerentemente paralela, Bend vai executá-lo em múltiplas threads, sem criação de threads, sem gerenciamento explícito de locks.

#### Benchmark do Ordenador Bitônico

- `bend run-rs`: CPU, Apple M3 Max: 12.15 segundos
- `bend run-c`: CPU, Apple M3 Max: 0.96 segundos
- `bend run-cu`: GPU, NVIDIA RTX 4090: 0.21 segundos

<details>
 <summary><b>Clique aqui para o código do Ordenador Bitônico</b></summary>

```py
# Rede de Ordenação = apenas gire as árvores!
def sort(d, s, tree):
  switch d:
    case 0:
      return tree
    case _:
      (x,y) = tree
      lft   = sort(d-1, 0, x)
      rgt   = sort(d-1, 1, y)
      return rots(d, s, (lft, rgt))

# Rotaciona sub-árvores (Caixa Azul/Verde)
def rots(d, s, tree):
  switch d:
    case 0:
      return tree
    case _:
      (x,y) = tree
      return down(d, s, warp(d-1, s, x, y))

# Troca valores distantes (Caixa Vermelha)
def warp(d, s, a, b):
  switch d:
    case 0:
      return swap(s ^ (a > b), a, b)
    case _:
      (a.a, a.b) = a
      (b.a, b.b) = b
      (A.a, A.b) = warp(d-1, s, a.a, b.a)
      (B.a, B.b) = warp(d-1, s, a.b, b.b)
      return ((A.a,B.a),(A.b,B.b))

# Propaga para baixo
def down(d,s,t):
  switch d:
    case 0:
      return t
    case _:
      (t.a, t.b) = t
      return (rots(d-1, s, t.a), rots(d-1, s, t.b))

# Troca um único par
def swap(s, a, b):
  switch s:
    case 0:
      return (a,b)
    case _:
      return (b,a)

# Testes
# -------

# Gera uma árvore grande
def gen(d, x):
  switch d:
    case 0:
      return x
    case _:
      return (gen(d-1, x * 2 + 1), gen(d-1, x * 2))

# Soma uma árvore grande
def sum(d, t):
  switch d:
    case 0:
      return t
    case _:
      (t.a, t.b) = t
      return sum(d-1, t.a) + sum(d-1, t.b)

# Ordena uma árvore grande
def main:
  return sum(20, sort(20, 0, gen(20, 0)))

```

</details>

Se você está interessado em outros algoritmos, pode conferir nossa [pasta de exemplos](https://github.com/HigherOrderCO/Bend/tree/main/examples)

### Recursos Adicionais
- Para entender a tecnologia por trás do Bend, confira o [artigo](https://paper.higherorderco.com/) do HVM2.
- Estamos trabalhando em uma documentação oficial, enquanto isso para uma explicação mais aprofundada confira [GUIDE.md](https://github.com/HigherOrderCO/Bend/blob/main/GUIDE.md)
- Leia sobre nossos recursos em [FEATURES.md](https://github.com/HigherOrderCO/Bend/blob/main/FEATURES.md)
- Bend é desenvolvido pela [HigherOrderCO](https://higherorderco.com/) - junte-se ao nosso [Discord](https://discord.higherorderco.com)!
