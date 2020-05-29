<!--
Projeto 3 de Linguagens de Programação I 2018/2019 (c) by Nuno Fachada

Projeto 3 de Linguagens de Programação I 2018/2019 is licensed under a
Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.

You should have received a copy of the license along with this
work. If not, see <http://creativecommons.org/licenses/by-nc-sa/4.0/>.
-->

# Projeto 3 de Linguagens de Programação I 2018/2019

## Descrição do problema

Os alunos devem implementar, em grupos de 2 a 3 elementos, um jogo _roguelike_
em C# (.NET Core console app) com níveis em grelha
[gerados procedimentalmente](#procedural). O
jogador começa no lado esquerdo da grelha (1ª coluna), e o seu objetivo é
encontrar a saída do nível, que se encontra do lado direito dessa mesma
grelha (última coluna). Pelo meio o jogador pode encontrar inimigos,
_power-ups_ e obstáculos. A dimensão da grelha deve ser especificada pelo
utilizador usando argumentos da linha de comando.

Os níveis vão ficando progressivamente mais difíceis, com mais inimigos e
menos _power-ups_. O [_score_](#score) final do jogador é igual ao nível
atingido. Deve existir, para cada dimensão da grelha, uma tabela dos _top_ 10
_high scores_, que deve persistir quando o programa termina e o PC é desligado.

O jogador e os inimigos só se podem deslocar na sua vizinhança de
[von Neumann], ou seja, para cima, para baixo e para os lados. O
jogador deve deslocar-se usando as setas de direção **e** as teclas WSAD.

## Modo de funcionamento

### Invocação do programa

O programa deve aceitar duas opções na linha de comando:

* `-r` - Número de linhas da grelha de jogo
* `-c` - Número de colunas da grelha de jogo.

Um exemplo de execução:

```
dotnet run -- -r 10 -c 7
```

A primeira opção, `--`, serve para separar entre as opções do comando `dotnet`
e as opções do programa a ser executado, neste caso o nosso jogo.

As opções indicadas são obrigatórias e podem ser dadas em qualquer ordem, desde
que o valor numérico suceda à opção propriamente dita. Se alguma das opções for
omitida o programa deve terminar com uma mensagem de erro indicando o modo de
uso.

Para quem preferir usar o Visual Studio Community 2019, as opções de linha de
comandos podem ser definidas diretamente da seguinte forma: 1) clicar com
o botão direito em cima do nome do projeto; 2) selecionar "Properties"; 3)
selecionar separador "Debug"; e, 4) na
caixa "Command line arguments" especificar os argumentos desejados.

Se os alunos implementarem a [Componente Opcional](#componente-opcional),
nomeadamente a opção de _Save Game_, o programa pode também aceitar a opção
`-l` para fazer _load_ de um jogo guardado anteriormente. Esta opção tem de
ser usada exclusivamente. Por exemplo:

```
dotnet run -- -l mysavegame.sav
```

### Menu principal

O jogo começa por apresentar o menu principal, que deve conter as seguintes
opções:

```
1. New game
2. High scores
3. Instructions
4. Credits
5. Quit
```

Caso o utilizador selecione as opções 2 a 4, é mostrada a informação
solicitada, após a qual o utilizador pressiona ENTER (ou qualquer tecla) para
voltar ao menu principal. A opção 5 termina o programa. Se for selecionada a
opção 1, começa um novo jogo.

### O jogo

#### Posição inicial do jogador e saída do nível

A posição inicial do jogador é na primeira coluna (do lado esquerdo), sendo a
linha determinada aleatoriamente.

A saída no nível é colocada na última coluna (do lado direito), sendo a linha
determinada aleatoriamente. Quando o jogador entra na casa contendo a saída,
o nível termina, passando o jogador para o nível seguinte, que terá
probabilisticamente um nível de dificuldade mais elevado.

#### O jogador

Em cada turno, o jogador pode andar **duas** casas na sua vizinhança de
[von Neumann]. O jogador não se pode mover contra as paredes do nível,
nem para casas contendo obstáculos ou inimigos. O jogador é obrigado a
mover-se, ou seja, não pode ficar parado. O jogador é controlado
através das teclas de direção **e** pelas teclas WSAD.

O estado do jogador é composto apenas pelo seu `HP`, que no início do jogo é
igual a:

_HP = (r * c) / 4_

Em que _r_ e _c_ indicam o número de linhas e colunas da grelha,
respetivamente.

Em cada passo, o jogador consome automaticamente 1 HP. Logicamente,
por turno o jogador perde 2 HP devido ao movimento.

#### Inimigos

Em cada turno, cada inimigo pode andar **uma** casa na sua vizinhança de
[von Neumann], em direção ao jogador. Por outras palavras, em cada turno, cada
inimigo move-se na direção que minimiza a distância de [Manhattan /
táxi][Manhattan] entre ele próprio e o jogador. Se esta direção estiver
bloqueada por um obstáculo, o inimigo move-se aleatoriamente numa direção
sem obstáculos. Para 2D, e considerando dois vetores de posição
_**p** = (p<sub>c</sub>, p<sub>r</sub>)_ e
_**q** = (q<sub>c</sub>, q<sub>r</sub>)_, a distância de [Manhattan] entre eles
é dada por
*|p<sub>c</sub> - q<sub>c</sub>| + |p<sub>r</sub> - q<sub>r</sub>|*.

Se um inimigo estiver na vizinhança de [von Neumann] do jogador, retira _d_ HP
ao jogador, não se movendo. O valor _d_ depende do inimigo, existindo dois
tipos de inimigo:

1. _Minion_: inimigo mais comum, retira _d =_ 5 HP ao jogador.
2. _Boss_: inimigo mais raro, retira _d =_ 10 HP ao jogador.

A posição inicial dos inimigos no nível é aleatória, mas não pode coincidir
com obstáculos, com a saída do nível e com o jogador. A quantidade de
inimigos por nível está descrita na secção
[Geração procedimental](geracao-procedimental).

#### Power-ups

Quando o jogador se move para uma casa que contenha um _power-up_, esse
_power-up_ é automaticamente consumido e o HP do jogador aumenta
_u_ unidades, dependendo do _power-up_. Existem três tipos de _power-up_:

1. _Small_: _power-up_ mais comum, aumenta HP do jogador em _u =_ 4 unidades.
2. _Medium_: _power-up_ relativamente frequente, aumenta HP do jogador em _u =_
   8 unidades.
3. _Large_: _power-up_ mais raro, aumenta HP do jogador em _u =_ 16 unidades.

Os inimigos também se podem mover para casas contendo _power-ups_, mas nesse
caso nada acontece. O jogador simplesmente não se move move para essa casa
(pois está lá o inimigo).

A posição inicial dos _power-ups_ no nível é aleatória, mas não pode coincidir
com obstáculos, com a saída do nível e com a posição inicial do jogador. A
quantidade de _power-ups_ por nível está descrita na secção
[Geração procedimental](geracao-procedimental).

#### Obstáculos

Por nível existem entre 0 a _min(r, c) - 1_ obstáculos (valor determinado
aleatoriamente). A posição de cada obstáculo é aleatória mas não pode
coincidir com os outros elementos do nível (jogador, saída do nível,
_power-ups_ e inimigos).

#### Fim do jogo

O jogo termina quando o HP do jogador chega a zero. A pontuação do jogador é
igual ao nível atingido. Se a pontuação alcançada estiver entre as 10 melhores,
solicita-se ao jogador o seu nome para o mesmo figurar na tabela de _high
scores_ para a dimensão da grelha em questão. [Este código][ficheiros] mostra
como se pode guardar uma lista de _high scores_ num ficheiro.

O jogo também termina se o jogador pressionar a tecla _Escape_.

### Geração procedimental

A [geração procedimental][GP] é uma peça fundamental na história dos
Videojogos, tanto antigos como atuais. A [geração procedimental][GP] consiste
na criação algorítmica e automática de dados, por oposição à criação manual
dos mesmos. É usada nos Videojogos para criar grandes quantidades de conteúdo, promovendo a imprevisibilidade e a rejogabilidade dos jogos.

O C# oferece a classe [Random] para geração de números aleatórios, que por
sua vez tem vários métodos úteis. Para usarmos esta classe é primeiro
necessário criar uma instância da mesma:

```cs
// Criar uma instância de Random usando como semente a hora atual do sistema
// Para efeitos de debugging durante o desenvolvimento do jogo pode ser
// conveniente usar uma semente fixa
// Deve ser usada a mesma instância de Random para o jogo todo
Random rnd = new Random();
```

Deve existir apenas uma instância de [Random] por jogo. Se for necessário
partilhar a instância entre vários métodos de uma classe, devem usar uma
variável de instância:

```cs
// Ao nível da classe, fora dos métodos
private Random rnd;
```

A instanciação desta variável deve apenas ser feita uma vez e de preferência
no construtor da classe que a contém. Se for necessário partilhar esta
instância entre várias classes (o que provavelmente significa que estão com uma
fraca organização de código), devem fazê-lo através de uma propriedade só de
leitura.

A classe [Random] disponibiliza três versões (_overloads_) do método
[Next()], úteis para obter números inteiros aleatórios:

* `Next()` - Retorna um inteiro aleatório não-negativo.
* `Next(int b)` - Retorna um inteiro aleatório no intervalo `[0, b[`.
* `Next(int a, int b)` - Retorna um inteiro aleatório no intervalo `[a, b[`.

Estes métodos são apropriados para determinar a quantidade inicial de inimigos,
_power-ups_ e obstáculos, bem como a posição inicial do jogador, e a saída do
nível.

O seguinte código exemplifica uma possível forma de criar os inimigos para um
novo nível (atenção que o código é meramente exemplificativo, não sendo
muito prático ou reutilizável na forma apresentada):

```cs
// Quantidade inicial de minions no nível
int numberOfMinions = rnd.Next(maxMinionsForThisLevel);

// Criar cada um dos minions
for (int i = 0; i < numberOfMinions; i++)
{
    // Determinar posição do minion
    int row = rnd.Next(numRows); // numRows representa o número de linhas da grelha
    int col = rnd.Next(numCols); // numCols representa o número de colunas da grelha

    // Criar small enemy
    Enemy minion = new MinionEnemy();

    // Adicionar minion ao local escolhido aleatoriamente
    // (falta verificar se local está disponível)
    level[row, col].Add(minion);
}
```

O código anterior assume que a variável `maxMinionsForThisLevel` já
existe. Esta variável deve ir aumentando de valor à medida que o jogador vai
passando os níveis. A forma mais simples consiste em usar uma
[função][funções] para relacionar a variável desejada (*y*) com o nível atual
(*x*). Algumas funções apropriadas para o efeito são a
[função linear], a [função linear por troços], a [função logística] ou a
[função logarítmica]. No site disponibilizado através deste [link][funções],
é possível manipular os diferentes parâmetros das várias funções de modo a
visualizar como as mesmas podem relacionar o nível (*x*) com o valor de saída desejado (*y*). É também [disponibilizada uma classe
_static_](ProcGenFunctions.cs) com as várias funções sugeridas.

Existem diferentes valores de saída *y* a serem obtidos para os níveis deste
jogo, nomeadamente:

* Número de _minions_, aumenta à medida que os níveis avançam.
* Número de _bosses_, aumenta à medida que os níveis avançam.
* Número de _small power-ups_, diminui à medida que os níveis avançam.
* Número de _medium power-ups_, diminui à medida que os níveis avançam.
* Número de _large power-ups_, diminui à medida que os níveis avançam.

Para cada um destes valores, é necessário escolherem uma função e parâmetros
apropriados, garantindo que:

* A dificuldade aumenta probabilisticamente ao longo dos níveis, levando a que
  o jogador eventualmente perca o jogo.
* Que o número máximo de inimigos nunca ocupe mais de metade no nível.
* Que o número máximo de _power-ups_ nunca seja zero.

### Visualização do jogo

A visualização do jogo deve ser feita em modo de texto (consola) e deve
mostrar o seguinte:

* Mapa do jogo, distinguindo claramente o jogador, a saída do nível, os
  obstáculos, bem como os diferentes inimigos e _power-ups_.
* Uma legenda, explicando o que é cada elemento no mapa.
* O HP do jogador.
* O nível atual.
* As teclas com que se pode jogar ou sair do jogo.
* Mensagens descrevendo o resultado das ações mais recentes por parte do
  jogador e inimigos.

Podem causar um pequeno _delay_ entre os movimentos dos inimigos, que
facilitam a visualização e a compreensão do que se está a passar, usando a
instrução [`Thread.Sleep()`] que aceita um valor em milissegundos. O uso desta
instrução requer o _namespace_ `System.Threading`.

Uma vez que o C# suporta nativamente a representação [Unicode], os respetivos
caracteres podem e devem ser usados para melhorar a visualização do jogo. Para
o efeito deve ser incluída a instrução `Console.OutputEncoding = Encoding.UTF8;`
no método `Main()` (é necessário usar o _namespace_ `System.Text`).

### Componente opcional

Este projeto contém uma componente opcional que consiste na implementação de
um sistema de _save games_. Caso esta componente seja implementada, o jogo
pode ser salvo entre níveis, devendo ser apresentada a opção de salvar o jogo
após o jogador ter saído de um nível. Se o jogador responder afirmativamente,
deve-lhe ser perguntado qual o nome do ficheiro para o qual gravar o jogo.

Havendo a opção de salvar o jogo, deve existir também a opção de carregar o
mesmo, usando para o efeito a opção `-l`, seguida do nome do ficheiro de
_save game_. O jogo deve então recomeçar no nível seguinte.

A implementação completa desta fase permite compensar eventuais problemas
noutras partes do código e/ou do projeto, facilitando a obtenção da nota
máxima de 3 valores.

## Avaliação e entrega

### Organização do projeto e estrutura de classes

O projeto deve estar devidamente organizado, fazendo uso de classes, _structs_
e enumerações. Cada classe, _struct_ ou enumeração deve ser colocada num
ficheiro com o mesmo nome. Por exemplo, uma classe chamada `Enemy` deve ser
colocada no ficheiro `Enemy.cs`. A estrutura de classes deve ser bem pensada e
organizada de uma forma lógica, e [cada classe deve ter uma responsabilidade
específica e bem definida][SRP].

### Objetivos e critério de avaliação

Este projeto tem os seguintes objetivos:

* **O1** - Programa deve funcionar como especificado e deve ter em conta as
  regras básicas do _game design_.
* **O2** - Projeto e código bem organizados, nomeadamente:
  * Estrutura de classes bem pensada (ver secção [Organização do código e
    estrutura de classes](#organizacao-do-codigo-e-estrutura-de-classes)).
  * Código devidamente comentado e indentado.
  * Inexistência de código "morto", que não faz nada, como por exemplo
    variáveis, propriedades ou métodos nunca usados.
  * Projeto compila e executa sem erros e/ou *warnings*.
* **O3** - Projeto adequadamente documentado com [comentários de documentação
  XML][XML]. A documentação gerada em formato HTML em [Doxygen] ou [DocFX],
  deve estar incluída no `zip` do projeto, mas **não** integrada no repositório
  Git.
* **O4** - Repositório Git deve refletir boa utilização do mesmo, nomeadamente:
  * Devem existir *commits* de todos os elementos do grupo, _commits_ esses
    com mensagens que sigam as melhores práticas para o efeito (como indicado
    [aqui](https://chris.beams.io/posts/git-commit/),
    [aqui](https://gist.github.com/robertpainsi/b632364184e70900af4ab688decf6f53),
    [aqui](https://github.com/erlang/otp/wiki/writing-good-commit-messages) e
    [aqui](https://stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)).
  * Ficheiros binários não necessários, como por exemplo todos os que são
    criados nas pastas `bin` e `obj`, bem como os ficheiros de configuração
    do Visual Studio (na pasta `.vs` ou `.vscode`), não devem estar no
    repositório. Ou seja, devem ser ignorados ao nível do ficheiro
    `.gitignore`.
  * *Assets* binários necessários, como é o caso da imagem do diagrama UML,
    devem ser integrados no repositório em modo Git LFS.
* **O5** - Relatório em formato [Markdown] (ficheiro `README.md`),
  organizado da seguinte forma:
  * Título do projeto.
  * Autoria:
    * Nome dos autores (primeiro e último) e respetivos números de aluno.
    * Informação de quem fez o quê no projeto. Esta informação é
      **obrigatória** e deve refletir os *commits* feitos no Git.
    * Indicação do repositório Git utilizado. Esta indicação é
      opcional, pois podem preferir manter o repositório privado após a
      entrega.
  * Arquitetura da solução:
    * Descrição da solução, com breve explicação de como o código foi
      organizado, bem como dos algoritmos não triviais que tenham sido
      implementados.
    * Um diagrama UML de classes simples (i.e., sem indicação dos
      membros da classe) descrevendo a estrutura de classes.
  * Referências, incluindo trocas de ideias com colegas, código aberto
    reutilizado (e.g., do StackOverflow) e bibliotecas de terceiros
    utilizadas. Devem ser o mais detalhados possível.
  * **Nota:** o relatório deve ser simples e breve, com informação mínima e
    suficiente para que seja possível ter uma boa ideia do que foi feito.
    Atenção aos erros ortográficos e à correta formatação [Markdown], pois
    ambos serão tidos em conta na nota final.

O projeto tem um peso de 3 valores na nota final da disciplina e será avaliado
de forma qualitativa. Isto significa que todos os objetivos têm de ser
parcialmente ou totalmente cumpridos. A cada objetivo, O1 a O5, será atribuída
uma nota entre 0 e 1. A nota do projeto será dada pela seguinte fórmula:

*N = 3 x O1 x O2 x O3 x O4 x O5 x D*

Em que *D* corresponde à nota da discussão e percentagem equitativa de
realização do projeto, também entre 0 e 1. Isto significa que se os alunos
ignorarem completamente um dos objetivos, não tenham feito nada no projeto ou
não comparerecem na discussão, a nota final será zero.

### Entrega

O projeto deve ser entregue por **grupos de 2 a 3 alunos** via Moodle até às
23h de 12 de junho de 2020. Um (e apenas um) dos elementos do grupo deve ser
submeter um ficheiro `zip` com a solução completa, nomeadamente:

* Pasta escondida `.git` com o repositório Git local do projeto.
* Documentação HTML gerada com [Doxygen] ou [DocFX]. Esta documentação
  **não** deve ser incluída no repositório Git, pelo que a respetiva pasta deve
  estar explicitamente ignorada a nível do ficheiro `.gitignore`.
* Ficheiro da solução (`.sln`).
* Pasta do projeto, contendo os ficheiros `.cs` e o ficheiro do projeto
  (`.csproj`).
* Ficheiro `README.md` contendo o relatório do projeto em formato [Markdown].
* Ficheiro de imagem contendo o diagrama UML. Este ficheiro deve ser incluído
  no repositório em modo Git LFS.
* Outros ficheiros de configuração, como por exemplo `.gitignore`,
  `.gitattributes`, `Doxyfile` ou `docfx.json`.

Não serão avaliados projetos sem estes elementos e que não sejam entregues
através do Moodle.

## Honestidade académica

Nesta disciplina, espera-se que cada aluno siga os mais altos padrões de
honestidade académica. Isto significa que cada ideia que não seja do
aluno deve ser claramente indicada, com devida referência ao respetivo
autor. O não cumprimento desta regra constitui plágio.

O plágio inclui a utilização de ideias, código ou conjuntos de soluções
de outros alunos ou indivíduos, ou quaisquer outras fontes para além
dos textos de apoio à disciplina, sem dar o respetivo crédito a essas
fontes. Os alunos são encorajados a discutir os problemas com outros
alunos e devem mencionar essa discussão quando submetem os projetos.
Essa menção **não** influenciará a nota. Os alunos não deverão, no
entanto, copiar códigos, documentação e relatórios de outros alunos, ou dar os
seus próprios códigos, documentação e relatórios a outros em qualquer
circunstância. De facto, não devem sequer deixar códigos, documentação e
relatórios em computadores de uso partilhado.

Nesta disciplina, a desonestidade académica é considerada fraude, com
todas as consequências legais que daí advêm. Qualquer fraude terá como
consequência imediata a anulação dos projetos de todos os alunos envolvidos
(incluindo os que possibilitaram a ocorrência). Qualquer suspeita de
desonestidade académica será relatada aos órgãos superiores da escola
para possível instauração de um processo disciplinar. Este poderá
resultar em reprovação à disciplina, reprovação de ano ou mesmo suspensão
temporária ou definitiva da ULHT.

*Texto adaptado da disciplina de [Algoritmos e
Estruturas de Dados][aed] do [Instituto Superior Técnico][ist]*

## Referências

* \[1\] Whitaker, R. B. (2016). **The C# Player's Guide**
  (3rd Edition). Starbound Software.
* \[2\] Albahari, J. (2017). **C# 7.0 in a Nutshell**.
  O’Reilly Media.
* \[3\] Dorsey, T. (2017). **Doing Visual Studio and .NET
  Code Documentation Right**. Visual Studio Magazine. Retrieved from
  <https://visualstudiomagazine.com/articles/2017/02/21/vs-dotnet-code-documentation-tools-roundup.aspx>.
* \[4\] Procedural generation. (2018). Retrived May 25, 2020
  from https://en.wikipedia.org/wiki/Procedural_generation.

## Licenças

Este enunciado é disponibilizado através da licença [CC BY-NC-SA 4.0].

## Metadados

* Autor: [Nuno Fachada]
* Curso:  [Licenciatura em Videojogos][lamv]
* Instituição: [Universidade Lusófona de Humanidades e Tecnologias][ULHT]

[ref1]:#ref1
[ref2]:#ref2
[ref3]:#ref3
[ref4]:#ref4
[CC BY-NC-SA 4.0]:https://creativecommons.org/licenses/by-nc-sa/4.0/
[GPLv3]:https://www.gnu.org/licenses/gpl-3.0.en.html
[lamv]:https://www.ulusofona.pt/licenciatura/videojogos
[Nuno Fachada]:https://github.com/fakenmc
[ULHT]:https://www.ulusofona.pt/
[aed]:https://fenix.tecnico.ulisboa.pt/disciplinas/AED-2/2009-2010/2-semestre/honestidade-academica
[ist]:https://tecnico.ulisboa.pt/pt/
[Markdown]:https://guides.github.com/features/mastering-markdown/
[Doxygen]:https://www.stack.nl/~dimitri/doxygen/
[DocFX]:https://dotnet.github.io/docfx/
[KISS]:https://en.wikipedia.org/wiki/KISS_principle
[XML]:https://docs.microsoft.com/dotnet/csharp/codedoc
[SRP]:https://en.wikipedia.org/wiki/Single_responsibility_principle
[KISS]:https://en.wikipedia.org/wiki/KISS_principle
[GP]:https://en.wikipedia.org/wiki/Procedural_generation
[`Console`]:https://docs.microsoft.com/dotnet/api/system.console
[Unicode]:https://unicode-table.com/
[von Neumann]:https://en.wikipedia.org/wiki/Von_Neumann_neighborhood
[`Thread.Sleep()`]:https://docs.microsoft.com/dotnet/api/system.threading.thread.sleep
[UTF-8]:https://en.wikipedia.org/wiki/UTF-8
[Unicode]:https://en.wikipedia.org/wiki/Unicode
[Random]:https://docs.microsoft.com/dotnet/api/system.random
[NextDouble()]:https://docs.microsoft.com/dotnet/api/system.random.nextdouble
[Next()]:https://docs.microsoft.com/dotnet/api/system.random.next
[função logística]:https://en.wikipedia.org/wiki/Logistic_function
[função linear por troços]:https://en.wikipedia.org/wiki/Piecewise_linear_function
[função logarítmica]:https://en.wikipedia.org/wiki/Logarithm#Logarithmic_function
[função linear]:https://en.wikipedia.org/wiki/Linear_function_(calculus)
[funções]:https://www.desmos.com/calculator/pb2w7iinza
[Manhattan]:https://en.wikipedia.org/wiki/Taxicab_geometry
[ficheiros]:https://gist.github.com/fakenmc/f70b38814ac6552e790dc0a86c3c67d0
