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

??? Checkpoint

No exemplo das faixas, qual é a entrada mais correta para a Transformada de
Hough?

1. A fotografia original colorida.
2. O conjunto de pontos destacados depois do processamento de bordas.
3. A imagem com as retas desenhadas.

::: Gabarito
A opção mais correta é a **2**.

A aplicação completa pode começar com uma fotografia, mas a Transformada de
Hough, a rigor, recebe um **conjunto de pontos**. Em visão computacional, esses
pontos normalmente vêm de um mapa de bordas.
:::

???

Na figura acima, a ideia é:

1. a imagem original tem estruturas visuais;
2. o detector de bordas reduz a imagem a pontos destacados;
3. a Transformada de Hough encontra retas compatíveis com muitos desses pontos.

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

Antes de pensar em uma imagem inteira, vale olhar com cuidado o que um único
ponto nos diz sobre retas.

??? Checkpoint

Considere o ponto $(2,3)$ no plano.

1. Quantas retas distintas passam por esse único ponto?
2. E se acrescentarmos um segundo ponto, por exemplo $(5,7)$ — quantas retas
   passam pelos dois ao mesmo tempo?

::: Gabarito
1. **Infinitas.** Por um único ponto passam retas em todas as inclinações
   possíveis — horizontal, subindo devagar, subindo rápido, descendo, e assim
   por diante.

![](hough-ab-um-ponto.png)

2. **Exatamente uma.** Dois pontos distintos determinam uma única reta. Cada
   ponto adicional restringe drasticamente o conjunto de retas compatíveis.
:::

???

Essa assimetria — infinitas retas para um ponto, uma única para dois — é o
motor da Transformada de Hough. Para trabalhar com a coleção de retas que
passam por um ponto, precisamos descrevê-la algebricamente. Na forma
$y = ax + b$, exigir que a reta passe por $(x_0, y_0)$ significa $y_0 = ax_0 +
b$; isolando $b$, temos $b = y_0 - ax_0$. Para um ponto fixo, os parâmetros
$(a, b)$ não são quaisquer — eles obedecem a essa relação. Mas o que ela
realmente diz *sobre* $(a, b)$?

??? Checkpoint

Considere o ponto $(2,3)$. Complete a tabela para algumas inclinações possíveis,
e responda: o que os três pares $(a, b)$ têm em comum no plano dos parâmetros?

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

Todas essas retas passam pelo ponto $(2,3)$. No plano $(a,b)$, os três pares
estão sobre a mesma reta: $b = 3 - 2a$. Ou seja, o ponto $(2,3)$ da imagem
corresponde a uma reta inteira no plano dos parâmetros.
:::

???

Esse plano dos parâmetros é o que vamos chamar, por enquanto, de **espaço de
Hough**.

## Quando vários pontos concordam

Agora suponha três pontos:

$$(0,1), (1,3), (2,5).$$

Visualmente, eles parecem estar sobre uma mesma reta da imagem.

??? Checkpoint

Para cada um dos três pontos, escreva a reta correspondente no plano $(a,b)$
usando $b = y_0 - ax_0$.

::: Gabarito
* $(0,1)$ gera $b = 1$;
* $(1,3)$ gera $b = 3 - a$;
* $(2,5)$ gera $b = 5 - 2a$.
:::

???

![](hough-intersecao-exata.png)

??? Checkpoint

Resolva o sistema com duas dessas equações — por exemplo, $b = 1$ e $b = 3 - a$.
Onde elas se cruzam? E a terceira reta, $b = 5 - 2a$, também passa por esse
ponto?

::: Gabarito
Como $b = 1$, substituindo em $b = 3 - a$ vem $1 = 3 - a$, ou seja, $a = 2$. A
interseção é

$$(a,b) = (2,1).$$

Conferindo com a terceira reta: $b = 5 - 2 \cdot 2 = 1$. Confirma.

Esse ponto $(2,1)$ codifica a reta $y = 2x + 1$ na imagem original. A regra
geral é: **pontos colineares na imagem viram retas concorrentes no espaço
$(a,b)$**, e o ponto de interseção carrega os parâmetros da reta original.
:::

???

Até aqui, talvez pareça que a Transformada de Hough é só calcular interseções.
Mas essa é a versão idealizada do problema. Em imagens reais, as coisas raramente
são exatas.

## O problema do quase

Considere agora pontos que estão quase alinhados, mas não perfeitamente:

$$(0,1.1), (1,2.8), (2,5.2), (3,6.9).$$

Visualmente, eles ainda sugerem uma reta.

??? Checkpoint

Calcule a interseção das retas correspondentes aos pares $\{(0,1.1), (1,2.8)\}$
e $\{(2,5.2), (3,6.9)\}$ no espaço $(a,b)$. As duas interseções coincidem?

::: Gabarito
Para o par $\{(0,1.1), (1,2.8)\}$, as retas são $b = 1.1$ e $b = 2.8 - a$, então

$$1.1 = 2.8 - a \Rightarrow a = 1.7, \quad b = 1.1.$$

Para o par $\{(2,5.2), (3,6.9)\}$, as retas são $b = 5.2 - 2a$ e $b = 6.9 - 3a$,
então

$$5.2 - 2a = 6.9 - 3a \Rightarrow a = 1.7, \quad b = 5.2 - 2 \cdot 1.7 = 1.8.$$

As interseções $(1.7,\,1.1)$ e $(1.7,\,1.8)$ não coincidem. Ficam próximas, mas
em pontos diferentes do espaço de parâmetros.
:::

???

![](hough-intersecao-ruido.png)

??? Checkpoint

Já que as interseções não coincidem mais, como você decidiria qual é a reta
"verdadeira" que melhor explica os pontos da imagem?

::: Gabarito
A ideia natural é procurar a reta que *chega mais perto* de explicar todos os
pontos — aquela cujos parâmetros $(a, b)$ ficam no meio da nuvem de interseções
quase-coincidentes, em vez de exigir uma interseção exata.

E como fazemos isso na prática? Com um **sistema de votos**: dividimos o
espaço de parâmetros em **células** e cada ponto da imagem vota nas células
que representam retas compatíveis com ele. No final, células com muitos votos
indicam retas que explicam muitos pontos — exatamente a tolerância que
faltava, transformando "quase passa pela mesma reta" em "recebe votos na mesma
região".
:::

???

## O acumulador

Agora precisamos transformar a ideia de "votos numa região" em algo concreto.

??? Checkpoint

Pense numa estrutura de dados para contar votos das células no plano $(a,b)$.
Que escolhas você precisa fazer *antes* de começar a contar?

::: Gabarito
Uma matriz de votos $A[a][b]$ resolve: uma dimensão para valores de $a$, outra
para valores de $b$. Essa matriz é o **acumulador**.

Antes de contar, é preciso decidir *quais* valores de $a$ e de $b$ testar — não
dá para testar infinitos. A escolha de uma **discretização** (por exemplo, $a$
variando de $0.1$ em $0.1$) é parte do projeto do algoritmo.
:::

???

![](hough-acumulador.png)

Com essas escolhas feitas, um algoritmo simples seria:

```txt
crie uma matriz de votos A[a][b] inicialmente zerada

para cada ponto (x, y) da entrada:
    para cada inclinação a que queremos testar:
        calcule b = y - a*x
        descubra a célula correspondente a (a, b)
        some 1 voto nessa célula

retorne as células com votos acima de um limiar
```

Quanto mais fina a discretização, mais preciso pode ser o resultado, mas maior
fica o custo.

:hough-votos

??? Checkpoint

Considere os pontos $(1,3.1), (2,5.0), (3,7.2), (4,8.9)$, quase sobre a reta
$y = 2x + 1$. Antes de fazer contas, *aposte*: testando $a = 1$, $a = 2$ e
$a = 3$, qual inclinação deve concentrar os valores de $b$ na mesma região? Por
quê? Depois confirme calculando $b = y - ax$.

::: Gabarito
A aposta natural é $a = 2$, porque essa é a inclinação verdadeira que os pontos
sugerem. Conferindo:

* $(1,3.1)$ dá $b = 3.1 - 2 \cdot 1 = 1.1$;
* $(2,5.0)$ dá $b = 5.0 - 2 \cdot 2 = 1.0$;
* $(3,7.2)$ dá $b = 7.2 - 2 \cdot 3 = 1.2$;
* $(4,8.9)$ dá $b = 8.9 - 2 \cdot 4 = 0.9$.

Todos perto de $b = 1$. A célula próxima de $(a,b) = (2,1)$ recebe muitos votos.
Para $a = 1$ ou $a = 3$, os valores de $b$ ficam espalhados — nenhuma célula
concentra votos.
:::

???

## O que sai do acumulador

??? Checkpoint

Depois da votação, você tem uma matriz inteira de números. Como você usaria
essa matriz para *decidir* quais retas existem na imagem? E se duas células bem
distantes recebessem muitos votos cada uma, o que isso significaria?

::: Gabarito
Procuramos **picos** no acumulador: células com muitos votos em comparação com
as vizinhas. Cada pico indica uma reta apoiada por muitos pontos da imagem. Se
a célula mais votada está perto de $(a,b) = (2,1)$, a reta detectada é
$y = 2x + 1$.

Se *duas* células distantes recebem muitos votos, é sinal de que existem duas
retas diferentes — por exemplo, em uma imagem de uma rua, uma célula
representando a faixa da esquerda e outra a faixa da direita. Uma única votação
pode revelar mais de uma estrutura.
:::

???

!!! Observação
O limiar de votos é uma escolha da implementação. Um limiar baixo encontra mais
retas, mas também aceita mais falsos positivos. Um limiar alto é mais exigente,
mas pode perder retas curtas ou incompletas.
!!!

## Por que trocar para a forma polar?

A explicação com $y = ax + b$ é boa para entender a ideia principal, mas tem um
problema escondido.

??? Checkpoint

Tente representar a reta $x = 4$ na forma $y = ax + b$. O que dá errado?

::: Gabarito
Não dá. $x = 4$ é uma reta **vertical**: sua inclinação $a$ seria infinita e
$b$ ficaria indefinido. A forma $y = ax + b$ não consegue codificar retas
verticais — qualquer implementação que use essa parametrização tem um buraco.
:::

???

??? Checkpoint

Se "inclinação" não funciona para retas verticais, que outras grandezas
geométricas você poderia usar para descrever *qualquer* reta no plano, sem
exceção? Como a reta $x = 4$ ficaria nessa nova descrição?

::: Gabarito
Uma alternativa é descrever a reta pela **distância perpendicular** dela até a
origem e pelo **ângulo** dessa perpendicular em relação ao eixo $x$. Chamando a
distância de $\rho$ e o ângulo de $\theta$, toda reta 2D — vertical, horizontal
ou oblíqua — pode ser descrita por um par $(\rho, \theta)$.

Para $x = 4$: a perpendicular a partir da origem aponta na direção do eixo $x$
(ângulo $\theta = 0$) e a distância até a origem é $\rho = 4$. Nada de
inclinação infinita.
:::

???

Formalmente, essa forma polar se escreve:

$$\rho = x\cos(\theta) + y\sin(\theta).$$

Nessa forma:

* $\rho$ é a distância perpendicular da reta até a origem;
* $\theta$ é o ângulo da reta perpendicular em relação ao eixo $x$.

![](hough-polar.png)

Uma reta agora é representada pelo par $(\rho,\theta)$, não por $(a,b)$ —
nenhuma inclinação infinita precisa ser representada.

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
