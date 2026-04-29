# Transformada de Hough

Detectar uma reta parece fácil quando estamos olhando para uma imagem. Se alguns
pontos aparecem mais ou menos alinhados, nosso olho completa as falhas, ignora
ruídos pequenos e enxerga uma estrutura.

Para um programa, porém, a imagem não vem com a frase "existe uma reta aqui".
Depois de algum pré-processamento, como um detector de bordas, o que sobra é
apenas uma coleção de pontos destacados. A Transformada de Hough é uma maneira
de transformar esses pontos em votos para retas possíveis.

Neste handout veremos como a transformada de Hough pode ser usada para encontrar retas em imagens.

## Objetivo

Ao final, você deve conseguir explicar:

1. qual é a entrada e qual é a saída da Transformada de Hough;
2. por que um ponto da imagem vira uma reta no espaço de parâmetros;
3. por que uma reta da imagem vira uma interseção no espaço de parâmetros;
4. por que a implementação real usa votação em células;
5. por que a forma polar $(\rho, \theta)$ é usada para lidar com retas verticais.

## Um exemplo antes da matemática

Imagine um sistema simples de assistência de direção. A câmera observa a rua,
algum algoritmo destaca bordas e a Transformada de Hough procura retas que
representam faixas.

![](hough-aplicacao.png)

Na figura acima, a ideia é:

1. a imagem original tem estruturas visuais;
2. o detector de bordas reduz a imagem a pontos destacados;
3. a Transformada de Hough encontra retas compatíveis com muitos desses pontos.


??? Checkpoint

No exemplo das faixas, qual é a entrada mais correta para a Transformada de
Hough?

1. A fotografia original colorida.
2. O conjunto de pontos destacados depois do processamento de bordas.
3. A resposta final com as retas desenhadas.

::: Gabarito
A opção mais correta é a **2**.

A aplicação completa pode começar com uma fotografia, mas a Transformada de
Hough, a rigor, recebe um **conjunto de pontos**. Em visão computacional, esses
pontos normalmente vêm de um mapa de bordas.
:::

???

## Entrada e saída

Para detecção de retas, vamos tratar a entrada como um conjunto

$$P = \{(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)\}.$$

Cada ponto é um candidato a pertencer a alguma reta. Alguns pontos podem ser
ruído. Outros podem fazer parte de retas diferentes.

![](hough-pontos.png)

A saída desejada é uma lista de retas. Por enquanto, vamos representar uma reta
pela forma clássica:

$$y = ax + b.$$

Nessa forma, $a$ é a inclinação da reta e $b$ é o ponto em que ela corta o eixo
$y$. Se a saída for $(a,b) = (2,1)$, por exemplo, a reta detectada é:

$$y = 2x + 1.$$

## De um ponto para várias retas

Pegue um único ponto da imagem, por exemplo $(2,3)$. Quantas retas passam por
ele?

A resposta é: infinitas. Uma delas é horizontal, outra sobe devagar, outra sobe
rápido, outra desce, e assim por diante.

![](hough-ab-um-ponto.png)

Na forma $y = ax + b$, dizer que a reta passa pelo ponto $(x_0,y_0)$ significa
que:

$$y_0 = ax_0 + b.$$

Reorganizando:

$$b = y_0 - ax_0.$$

Perceba a troca:

* no plano da imagem, $(x_0,y_0)$ é um ponto fixo;
* no plano dos parâmetros, $(a,b)$, esse mesmo ponto vira a reta
  $b = y_0 - ax_0$.

Esse plano dos parâmetros é o que vamos chamar, por enquanto, de **espaço de
Hough**.

??? Checkpoint

Considere o ponto $(2,3)$. Complete a tabela para algumas inclinações possíveis.

| $a$ | $b = 3 - 2a$ | reta no plano da imagem |
|-----|--------------|--------------------------|
| $0$ |              |                          |
| $1$ |              |                          |
| $2$ |              |                          |

::: Gabarito

| $a$ | $b = 3 - 2a$ | reta no plano da imagem |
|-----|--------------|--------------------------|
| $0$ | $3$          | $y = 3$                 |
| $1$ | $1$          | $y = x + 1$             |
| $2$ | $-1$         | $y = 2x - 1$            |

Todas essas retas passam pelo ponto $(2,3)$. No espaço $(a,b)$, elas são apenas
pontos diferentes sobre a reta $b = 3 - 2a$.
:::

???

## Quando vários pontos concordam

Agora suponha que temos três pontos exatamente colineares:

$$(0,1), (1,3), (2,5).$$

No plano da imagem, eles pertencem à reta:

$$y = 2x + 1.$$

No espaço $(a,b)$, cada ponto gera uma reta diferente:

* $(0,1)$ gera $b = 1$;
* $(1,3)$ gera $b = 3 - a$;
* $(2,5)$ gera $b = 5 - 2a$.

![](hough-intersecao-exata.png)

Essas três retas se cruzam no mesmo ponto do espaço de parâmetros:

$$(a,b) = (2,1).$$

Esse ponto representa a reta $y = 2x + 1$ na imagem original.

??? Checkpoint

Use apenas os pontos $(0,1)$ e $(1,3)$. Encontre a interseção das retas no
espaço de parâmetros.

As retas são:

$$b = 1$$

e

$$b = 3 - a.$$

Qual é o par $(a,b)$?

::: Gabarito
Como $b = 1$, podemos substituir na segunda equação:

$$1 = 3 - a.$$

Logo,

$$a = 2.$$

Portanto, a interseção é:

$$(a,b) = (2,1).$$

Esse ponto representa a reta $y = 2x + 1$.
:::

???

Até aqui, talvez pareça que a Transformada de Hough é só calcular interseções.
Mas essa é a versão idealizada do problema. Em imagens reais, as coisas raramente
são exatas.

## O problema do quase

Considere agora pontos que estão quase alinhados, mas não perfeitamente:

$$(0,1.1), (1,2.8), (2,5.2), (3,6.9).$$

Visualmente, eles ainda sugerem uma reta. Só que, no espaço de parâmetros, as
retas geradas por esses pontos não passam todas por uma única interseção exata.
Elas passam perto umas das outras.

![](hough-intersecao-ruido.png)

Esse "perto" é justamente onde a votação entra.

Em vez de exigir uma interseção perfeita, dividimos o espaço de parâmetros em
células. Cada ponto da imagem vota nas células que representam retas compatíveis
com ele. No final, células com muitos votos indicam retas que explicam muitos
pontos da imagem.

??? Checkpoint

Por que não basta procurar uma interseção exata no espaço de parâmetros?

::: Gabarito
Porque os pontos de entrada podem ter ruído, falhas e pequenas imprecisões.

Se três pontos estivessem exatamente em uma mesma reta, as curvas no espaço de
parâmetros teriam uma interseção perfeita. Mas, em uma imagem real, bordas podem
estar deslocadas por alguns pixels. Então o algoritmo precisa de tolerância.

A votação em células é uma forma de transformar "quase passa pela mesma reta" em
"recebe votos na mesma região".
:::

???

## O acumulador

O **acumulador** é uma matriz de votos. No caso da forma $y = ax + b$, uma
dimensão representa valores possíveis de $a$ e a outra representa valores
possíveis de $b$.

![](hough-acumulador.png)

Um algoritmo simples seria:

```
crie uma matriz de votos A[a][b] inicialmente zerada

para cada ponto (x, y) da entrada:
    para cada inclinação a que queremos testar:
        calcule b = y - a*x
        descubra a célula correspondente a (a, b)
        some 1 voto nessa célula

retorne as células com votos acima de um limiar
```

O detalhe importante é que não testamos infinitos valores de $a$ e $b$. Escolhemos
uma discretização: por exemplo, testar $a$ de $0.5$ em $0.5$, ou de $0.1$ em
$0.1$. Quanto mais fina a discretização, mais preciso pode ser o resultado, mas
maior fica o custo.

:hough-votos

??? Checkpoint

Suponha os pontos abaixo, quase sobre a reta $y = 2x + 1$:

$$(1,3.1), (2,5.0), (3,7.2), (4,8.9).$$

Vamos testar apenas três inclinações: $a = 1$, $a = 2$ e $a = 3$. Para cada uma,
calcule valores aproximados de $b = y - ax$.

Qual inclinação deve concentrar os valores de $b$ mais perto de $1$?

::: Gabarito
Para $a = 2$:

* $(1,3.1)$ dá $b = 3.1 - 2 \cdot 1 = 1.1$;
* $(2,5.0)$ dá $b = 5.0 - 2 \cdot 2 = 1.0$;
* $(3,7.2)$ dá $b = 7.2 - 2 \cdot 3 = 1.2$;
* $(4,8.9)$ dá $b = 8.9 - 2 \cdot 4 = 0.9$.

Todos ficam perto de $b = 1$. Então a célula perto de $(a,b) = (2,1)$ deve
receber muitos votos.
:::

???

## O que sai do acumulador

Depois da votação, procuramos picos no acumulador. Um pico é uma célula com
muitos votos em comparação com as vizinhas.

Se a célula mais votada está perto de $(a,b) = (2,1)$, a reta detectada é:

$$y = 2x + 1.$$

Se existirem vários picos fortes, isso pode indicar várias retas na mesma
entrada. Essa é uma das vantagens da Transformada de Hough: uma única votação
pode revelar mais de uma estrutura.

!!! Observação
O limiar de votos é uma escolha da implementação. Um limiar baixo encontra mais
retas, mas também aceita mais falsos positivos. Um limiar alto é mais exigente,
mas pode perder retas curtas ou incompletas.
!!!

??? Checkpoint

Se duas células diferentes do acumulador recebem muitos votos, o que isso pode
significar no plano da imagem?

::: Gabarito
Pode significar que existem duas retas diferentes apoiadas por pontos da entrada.

Por exemplo, em uma imagem de uma rua, uma célula pode representar a faixa da
esquerda e outra pode representar a faixa da direita.
:::

???

## Por que trocar para a forma polar?

A explicação com $y = ax + b$ é boa para entender a ideia principal, mas tem um
problema sério: retas verticais.

Uma reta vertical, como

$$x = 4,$$

não pode ser escrita como $y = ax + b$ com um valor finito de $a$. A inclinação
seria infinita.

Para evitar esse caso especial, implementações de Hough para retas normalmente
usam a forma polar:

$$\rho = x\cos(\theta) + y\sin(\theta).$$

Nessa forma:

* $\rho$ é a distância perpendicular da reta até a origem;
* $\theta$ é o ângulo da reta perpendicular em relação ao eixo $x$.

![](hough-polar.png)

Agora uma reta é representada pelo par $(\rho,\theta)$, não por $(a,b)$. Isso
resolve o problema das retas verticais porque nenhuma inclinação infinita precisa
ser representada.

??? Checkpoint

Por que a forma polar é mais conveniente para implementar a Transformada de
Hough em imagens?

::: Gabarito
Porque ela representa retas verticais sem precisar de inclinação infinita.

Na forma $y = ax + b$, retas verticais são um caso problemático. Na forma
$\rho = x\cos(\theta) + y\sin(\theta)$, toda reta 2D pode ser descrita por uma
distância $\rho$ e um ângulo $\theta$.
:::

???

## Complexidade

Na versão para retas, suponha:

* $N$: quantidade de pontos de entrada;
* $T$: quantidade de valores de ângulo ou inclinação testados;
* $R$: quantidade de valores possíveis para a outra dimensão do acumulador.

Para cada ponto, testamos $T$ possibilidades. Portanto, a votação custa:

$$O(NT).$$

Além disso, a matriz acumuladora tem tamanho:

$$O(TR).$$

Se a implementação inicializa explicitamente a matriz inteira, esse custo de
inicialização também aparece. Então uma forma mais completa de escrever o custo
é:

$$O(TR + NT).$$

Essa análise também explica por que detectar retas é relativamente prático: o
espaço de busca tem duas dimensões.

;;; Curiosidade/desafio: círculos

Um círculo precisa de três parâmetros:

$$(x_c, y_c, r).$$

Ou seja, além do centro, precisamos do raio. O acumulador deixa de ser uma matriz
2D e passa a ser um volume 3D.

Na versão ingênua, isso aumenta muito o custo:

* tempo: cada ponto vota em muitas combinações de centro e raio;
* memória: o acumulador precisa guardar votos para $(x_c, y_c, r)$.

Por isso, círculos são uma extensão natural da ideia, mas não são o melhor lugar
para aprender o algoritmo pela primeira vez. Primeiro entenda bem retas. Depois,
a pergunta para círculos fica mais clara: "como reduzir o espaço de busca?".

??? Desafio

Se uma reta precisa de dois parâmetros e um círculo precisa de três, qual deles
tende a exigir mais memória no acumulador? Por quê?

::: Gabarito
O círculo tende a exigir mais memória, porque seu acumulador tem três dimensões:
centro $x$, centro $y$ e raio $r$.

Para retas, o acumulador é 2D. Para círculos, ele é 3D. Esse aumento de dimensão
costuma pesar bastante.
:::

???

;;;

## Resumo

A Transformada de Hough troca uma pergunta difícil por uma votação.

Em vez de perguntar diretamente "quais pontos formam uma reta?", fazemos:

1. cada ponto vota nas retas que poderiam passar por ele;
2. cada reta candidata é uma célula no espaço de parâmetros;
3. células com muitos votos indicam retas apoiadas por muitos pontos;
4. a discretização dá tolerância para ruído e bordas imperfeitas;
5. a forma polar $(\rho,\theta)$ evita o problema das retas verticais.

Essa é a parte mais importante: o algoritmo não funciona por mágica. Ele funciona
porque pontos alinhados concordam sobre os parâmetros da mesma reta.
