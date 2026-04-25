<!DOCTYPE html>
<html lang="pt-br">

<head>
    <title>Transformada de Hough</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,700;1,400;1,700&family=Open+Sans+Condensed:ital,wght@0,300;1,300&family=Open+Sans:ital,wght@0,300;0,400;0,700;0,800;1,300;1,400;1,700;1,800&family=Oxygen+Mono&family=Josefin+Sans:ital,wght@0,200;0,600;1,200;1,600&display=swap">
    <link rel="stylesheet" href="assets/css/reset.css">
    <link rel="stylesheet" href="assets/css/highlight.css">
    <link rel="stylesheet" href="assets/css/style.css">
    <link rel="stylesheet" href="assets/css/color.css">
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <script async src="https://static.codepen.io/assets/embed/ei.js"></script>
    <script src="assets/js/highlight.js"></script>
    <script src="assets/js/script.js"></script>
</head>

<body>
    <div class="container">
        <header>
            <a href="">abrir tudo</a>
            <a href="">fechar tudo</a>
        </header>

        <main>
            <h1>Transformada de Hough</h1>
            <h2>Detecção de linhas e círculos</h2>

            <h2>1. Introdução</h2>

            <p>
                A Transformada de Hough é uma técnica de visão computacional usada para detectar formas
                geométricas em imagens. Neste projeto, o foco será a detecção de linhas e círculos.
            </p>

            <p>
                A ideia central é transformar os pontos de uma imagem em votos para possíveis formas.
                Quando muitos pontos votam na mesma forma, o algoritmo entende que essa forma provavelmente
                existe na imagem.
            </p>

            <h2>2. Problema</h2>

            <p>
                Em uma imagem, o computador não enxerga objetos como uma pessoa enxerga. Ele trabalha com
                pixels. Por isso, para identificar uma linha, um círculo ou outro padrão visual, é necessário
                usar algum método de análise.
            </p>

            <p>
                A Transformada de Hough resolve esse problema procurando padrões geométricos em pontos de
                borda da imagem.
            </p>

            <h2>3. Entrada do algoritmo</h2>

            <p>
                A entrada geralmente é uma imagem já processada para destacar bordas. Isso significa que,
                antes da Transformada de Hough, a imagem pode passar por etapas como conversão para tons de
                cinza e detecção de bordas.
            </p>

            <p>
                Na prática, a entrada pode ser entendida como um conjunto de pontos da imagem. Cada ponto
                pode fazer parte de uma linha ou de um círculo.
            </p>

            <h2>4. Ideia principal</h2>

            <p>
                A Transformada de Hough funciona como uma votação. Cada ponto de borda vota nas formas que
                poderiam passar por ele.
            </p>

            <p>Fluxo geral:</p>

            <ol>
                <li><p>pegar os pontos de borda da imagem;</p></li>
                <li><p>verificar quais formas poderiam passar por esses pontos;</p></li>
                <li><p>somar votos para essas formas;</p></li>
                <li><p>escolher as formas com mais votos.</p></li>
            </ol>

            <h2>5. Detecção de linhas</h2>

            <p>
                Para detectar linhas, o algoritmo procura retas que passam por vários pontos da imagem.
                Se muitos pontos votam na mesma reta, essa reta provavelmente existe.
            </p>

            <p>
                Uma linha pode ser descrita por sua posição e sua orientação. Em muitas implementações,
                esses valores são chamados de rho e theta.
            </p>

            <p>
                A saída, no caso das linhas, pode indicar onde a linha está e qual é sua inclinação ou
                direção.
            </p>

            <h2>6. Detecção de círculos</h2>

            <p>
                Para detectar círculos, o processo é parecido. A diferença é que um círculo precisa de mais
                informações para ser descrito.
            </p>

            <p>
                Um círculo pode ser representado pelo centro e pelo raio. Assim, o algoritmo procura quais
                centros e raios recebem mais votos dos pontos de borda.
            </p>

            <p>
                A saída, no caso dos círculos, pode indicar o centro do círculo e o tamanho do raio.
            </p>

            <h2>7. Comparação entre linhas e círculos</h2>

            <p>
                A detecção de linhas é mais simples porque uma linha pode ser descrita por apenas dois
                parâmetros <strong>(rho e theta)</strong>. Já a detecção de círculos é mais complexa porque um círculo
                precisa de três parâmetros <strong>(centro x, centro y e raio)</strong>.
            </p>

            <h2>8. Saída do algoritmo</h2>

            <p>
                A saída da Transformada de Hough são as formas encontradas na imagem. Para linhas, a saída
                pode ser uma lista de retas ou segmentos. Para círculos, a saída pode ser uma lista de centros
                e raios.
            </p>

            <p>
                Portanto, a saída responde perguntas como: existe uma linha? Onde ela está? Existe um
                círculo? Qual é seu centro? Qual é seu raio?
            </p>

            <h2>9. Próximos passos do projeto</h2>

            <p>
                Nas próximas sprints, este handout poderá ser melhorado com exemplos visuais, pseudocódigo,
                exercícios, imagens explicativas e exemplos de implementação em Python ou OpenCV.
            </p>

            <h2>10. Conclusão</h2>

            <p>
                A Transformada de Hough é uma forma de detectar figuras geométricas usando votação. Ela é
                útil porque permite encontrar linhas e círculos mesmo quando a imagem possui ruídos ou quando
                a forma não está perfeitamente contínua.
            </p>
        </main>

        <footer>
            <a href="http://creativecommons.org/licenses/by-sa/4.0/" target="_blank">
                <img alt="CC BY-SA 4.0" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" title="Creative Commons Attribution-ShareAlike 4.0 International License" />
            </a>
            © 2024 Marcelo Hashimoto
        </footer>
    </div>
</body>

</html>
