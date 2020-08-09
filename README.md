<!--
Projeto de Época Especial de Linguagens de Programação I 2019/2020 (c) by Nuno Fachada

Projeto de Época Especial de Linguagens de Programação I 2019/2020 is licensed under a
Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.

You should have received a copy of the license along with this
work. If not, see <http://creativecommons.org/licenses/by-nc-sa/4.0/>.
-->

# Projeto de Época Especial de Linguagens de Programação I 2019/2020

## Introdução

Os grupos devem implementar uma versão 2D do jogo [Pedra, Papel e Tesoura]
na forma de uma aplicação de consola .NET Core na linguagem C#.

Este modelo explora um ecossistema de três espécies, pedras (cor azul), papéis
(cor verde) e tesouras (cor vermelha), que competem por espaço no mundo de
simulação. As interações entre as espécies são baseadas no jogo
[Pedra, Papel e Tesoura], ou seja, tesoura vence papel, papel vence pedra, e
pedra vence tesoura. Os organismos competem com os seus vizinhos, movem-se no
mundo e reproduzem-se. Estas interações resultam em padrões de espiral cujo
tamanho e estabilidade dependem dos parâmetros da simulação.

O modelo está implementado como [exemplo][RPSNetLogo] no software [NetLogo]
(aberto e gratuito, também corre diretamente no _browser_), implementação esta
que pode e deve ser usada como termo de comparação ao projeto desenvolvido.

## Descrição

### O modelo de simulação

A simulação corre numa grelha com dimensões (`x`, `y`) toroidal com vizinhança
de [Von Neumann] de raio 1. Toroidal significa que a grelha "dá a volta" na
vertical e na horizontal, ou seja, na prática não tem paredes.

Cada célula da grelha pode estar ocupada por uma das três espécies ou estar
vazia (neste caso assume a cor de fundo do terminal). A simulação funciona por
turnos e em cada turno podem acontecer os seguintes eventos:

1. Evento de **Troca/Movimento**:
   * Duas células vizinhas são aleatoriamente escolhidas.
   * O estado de cada uma das células passa a ser o estado da célula vizinha
     (ou seja, as células trocam de estado).
2. Evento de **Reprodução**:
   * Duas células vizinhas são aleatoriamente escolhidas.
   * Se uma das células for vazia passa a ser ocupada por um elemento da
     espécie na célula vizinha.
   * Nada acontece se nenhuma das células ou ambas as células forem vazias.
3. Evento de **Seleção**:
   * Duas células vizinhas são aleatoriamente escolhidas.
   * Os seus ocupantes competem um com o outro segundo as regras do
     [Pedra, Papel e Tesoura].
   * A célula do vizinho perdedor torna-se vazia.
   * Caso alguma das células escolhidas seja vazia, nada acontece.

A simulação tem cinco parâmetros:

* `xdim` - Número inteiro que representa a dimensão horizontal da grelha
  de simulação.
* `ydim` - Número inteiro que representa a dimensão vertical da grelha
  de simulação.
* `swap_rate_exp`, número real entre -1.0 e 1.0, que representa a taxa dos
  eventos de troca/movimento.
* `repr_rate_exp`, número real entre -1.0 e 1.0, que representa a taxa dos
  eventos de reprodução.
* `selc_rate_exp`, número real entre -1.0 e 1.0, que representa a taxa dos
  eventos de seleção.

O número exato de cada tipo de evento em cada turno é determinado
aleatoriamente através de uma [distribuição de Poisson], que expressa a
probabilidade de uma série de eventos ocorrer num certo período de tempo. Uma
vez que este é um projeto de programação, não é exigido aos alunos que
compreendam a fundo esta distribuição, apenas que consigam implementar um
método chamado `Poisson()` que aceite um valor real `λ` (média da
distribuição) e devolva um número inteiro aleatório obtido através desta
distribuição. A secção
[Gerar inteiros aleatórios a partir da distribuição de Poisson](#gerar-inteiros-aleatórios-a-partir-da-distribuição-de-poisson) discute
possíveis abordagens. Uma possível declaração deste método é a seguinte:

```cs
private int Poisson(double lambda);
```

O número exato de eventos em cada turno é então obtido através do método
`Poisson()`, em que `λ` é dado por:

```cs
double lambda = (xdim * ydim / 3.0) * Math.Pow(10, rate_exp);
```

A variável `rate_exp` pode ser `swap_rate_exp`, `repr_rate_exp` ou
`selc_rate_exp`, dependendo do tipo de evento em questão. Por exemplo, o
número de trocas/movimentos num dado turno pode ser dado por:

```cs
// Obter o lambda λ, valor médio das trocas/movimentos
// Notar que este valor é constante ao longo da simulação
double lambdaSwap = (xdim * ydim / 3.0) * Math.Pow(10, swap_rate_exp);

// Obter o número de trocas/movimentos a efetuar no turno atual
int numSwaps = Poisson(lambdaSwap);
```

Uma vez obtido o número de cada tipo de eventos para o turno atual, os mesmos
devem ser individualmente colocados numa lista. Essa lista deve ser então
embaralhada (ver secção
[Embaralhar uma lista ou _array_](#embaralhar-uma-lista-ou-array)), e
finalmente percorrida, de modo a que cada evento seja executado. A ideia
é que os eventos sejam executados numa ordem aleatória. Por exemplo, se
num dado turno serão executadas 3 trocas, 4 reproduções e 2 seleções
(valores obtidos aleatoriamente a partir da distribuição de Poisson), a
lista com estes eventos deverá ter inicialmente o seguinte conteúdo:

```
Swap
Swap
Swap
Reproduction
Reproduction
Reproduction
Reproduction
Selection
Selection
```

Após o embaralhamento a ordem dos conteúdos é randomizada:

```
Reproduction
Swap
Reproduction
Selection
Reproduction
Selection
Swap
Swap
Reproduction
```

É nesta fase, após o embaralhamento, que a lista deve ser percorrida, e cada
um dos eventos executado para o turno atual.

### Visualização

A visualização deve ser atualizada após todos os eventos de dado turno terem
sido executados. A visualização pode ser um pouco lenta, pelo que a secção
[Eficiência do código](#eficiência-do-código) apresenta algumas sugestões
para melhorar o _frame rate_.

A imagem/vídeo em baixo mostra uma implementação de consola em C# com
dimensões 300 x 70 e todos os `rates_exp` colocados a zero.

[![Rock Paper Scissors](https://img.youtube.com/vi/uYiQg-rkG6A/hqdefault.jpg)](https://youtu.be/uYiQg-rkG6A)

### Funcionamento do programa

O programa deve aceitar apenas uma opção na linha de comando, nomeadamente o
nome do ficheiro que especifica os parâmetros da simulação. Um exemplo de
execução:

```
dotnet run -- config.txt
```

A primeira opção, `--`, serve para separar entre as opções do comando `dotnet`
e as opções do programa a ser executado, neste caso a nossa simulação.

O ficheiro que especifica os parâmetros da simulação deve ter os seguintes
conteúdos:

* Opção `xdim`, seguida de um inteiro, indica a dimensão horizontal da grelha
  de simulação.
* Opção `ydim`, seguida de um inteiro, indica a dimensão vertical da grelha de
  simulação.
* Opção `swap-rate-exp`, seguida de um número real (`double`) entre -1.0 e 1.0.
* Opção `repr-rate-exp`, seguida de um número real (`double`) entre -1.0 e 1.0.
* Opção `selc-rate-exp`, seguida de um número real (`double`) entre -1.0 e 1.0.
* Entre cada opção e o seu valor deve existir um espaço.
* Espaços no início e fim das linhas devem ser ignorados.
* Linhas em branco ou começadas por `#` ou `//` devem ser ignoradas, sendo
  estas últimas consideradas comentários.

As opções indicadas podem ser dadas em qualquer ordem. Se alguma delas for
omitida o programa deve terminar com uma mensagem de erro indicando a opção em
falta. Opções desconhecidas ou com valores inválidos também fazem com que o
programa termine embora indicando exatamente qual a linha inválida encontrada.
Exemplo de um ficheiro válido:

```
# Taxas
selc-rate-exp -0.02
swap-rate-exp 0.75
repr-rate-exp 0.00

# Dimensões
xdim 100
ydim 40
```

Notar que podem existir problemas no _parsing_ dos valores reais devido ao
separador decimal ser uma vírgula e não um ponto na língua portuguesa (ver
secção [_Parsing_ de números reais](#parsing-de-números-reais)).
Se as opções forem corretas, a simulação começa imediatamente, não existindo
quaisquer paragens ou demoras entre turnos. A simulação deve correr o mais
rapidamente possível (ver secção [Eficiência do código](#eficiência-do-código)).

A simulação termina quando o utilizador pressiona a tecla `Escape`. É possível
verificar se alguma tecla for pressionada através da propriedade
[`Console.KeyAvailable`](https://docs.microsoft.com/dotnet/api/system.console.keyavailable), evitando deste modo que o programa fique preso à espera de
uma tecla.

### Resumo

Resumindo, a simulação é executada de acordo com os seguintes passos:

1. Criar o mundo de simulação, cada célula inicializada aleatoriamente (pedra,
   papel, tesoura, vazia).
2. Determinar número de eventos de troca/movimento, reprodução e seleção,
   a partir da distribuição de Poisson tal como explicado nas secções
   anteriores.
3. Colocar esses eventos numa lista, um a um.
4. Embaralhar a lista.
5. Percorrer a lista e executar esses eventos, um a um.
6. Limpar a lista.
7. Atualizar visualização.
8. Se utilizador pressionou a tecla `Escape` terminar a simulação, caso
   contrário voltar para o ponto 2.

### Dicas e sugestões

#### Gerar inteiros aleatórios a partir da distribuição de Poisson

Um gerador de números aleatórios obtidos a partir da distribuição de Poisson
recebe um valor real `λ` (média dos números aleatórios a devolver) e devolve
um número inteiro que corresponde ao número (aleatório) de eventos.

A linguagem C# apenas oferece a classe [Random], que produz números aleatórios
a partir da [distribuição uniforme]. O Wikipédia sugere [alguns
algoritmos](genPoisson) para obter valores a partir da distribuição de
Poisson tendo como base a distribuição uniforme. No entanto, apenas o segundo,
"**algorithm** _poisson random number (Junhao, based on Knuth)_",
funciona bem com valores elevados de `λ`, necessários para este projeto.
Na parte do algoritmo que diz _Generate uniform random number u  in (0,1)_
pode ser usado o método [`NextDouble()`] da classe [Random] para obter
valores aleatórios uniformes entre 0 e 1. Todas as variáveis internas deste algoritmo devem ser `double`, exceto a variável `k`, que deve ser
um `int`. O parâmetro `STEP` deve ser uma constante com o valor 500.

Embora seja preferível os alunos implementarem esta função a partir
do pseudo-código disponível no Wikipédia, também se aceita o uso de código
encontrado na Internet com esta funcionalidade. Nesse caso, deve ser feita
referência à fonte.

#### Embaralhar uma lista ou _array_

O algoritmo
[Fisher–Yates](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)
é um método de embaralhamento (_shuffling_) tipicamente utilizado para
embaralhar listas ou _arrays_.

Tal como no caso do gerador de números aleatórios de Poisson, é preferível
serem os alunos a implementar este algoritmo a partir do pseudo-código
disponível no Wikipédia. No entanto, também se aceita o uso código
encontrado na Internet, com a devida referência à fonte.

#### _Parsing_ de números reais

De modo a converter uma _string_ num número real (neste caso, um `double`),
usa-se tipicamente uma das seguintes abordagens:

```cs
// s é uma string, x é um double
x = Convert.ToDouble(s);   // Abordagem 1
x = double.Parse(s);       // Abordagem 2
double.TryParse(s, out x); // Abordagem 3 (preferida)
```

A última forma é a preferida, pois permite-nos verificar se a conversão foi
inválida. No entanto pode ocorrer um problema caso o PC esteja configurado com
a língua portuguesa, na qual o separador decimal é uma vírgula e não um ponto.
Para evitar este problema, podemos indicar ao C# que pretendemos uma conversão
independente da língua configurada no computador, assumindo o ponto como
separador decimal:

```cs
// Requer using extra no início da classe
using System.Globalization;
//...
// s é uma string, x é um double
x = Convert.ToDouble(s, CultureInfo.InvariantCulture);               // Abordagem 1
x = double.Parse(s, NumberStyles.Any, CultureInfo.InvariantCulture); // Abordagem 2
double.TryParse(s, NumberStyles.Any, CultureInfo.InvariantCulture, out x); // Abordagem 3 (preferida)
```

Notar que a abordagem com `TryParse()` é normalmente usada com um `if`:

```cs
if (double.TryParse(...))
{
    // Conversão feita com sucesso
}
else
{
    // Conversão falhou
}
```

## Requisitos do código

### Organização do código e estrutura de classes

O projeto deve estar devidamente organizado, fazendo uso de classes, _structs_
e enumerações. Cada classe, _struct_ ou enumeração deve ser colocada num
ficheiro com o mesmo nome. Por exemplo, uma classe chamada `Simulation` deve
ser colocada no ficheiro `Simulation.cs`. A estrutura de classes deve ser bem
pensada e organizada de uma forma lógica, e [cada classe deve ter uma
responsabilidade específica e bem definida][SRP].

Serão bonificadas soluções em que seja muito simples alterar ou estender o
modelo, como por exemplo usar outro tipo de vizinhança (por exemplo, para uma
vizinhança de Moore e/ou uma vizinhança com raio maior) ou ter mais do que
três espécies. No entanto tal não é de todo obrigatório para obter uma boa
nota.

### Eficiência do código

Existem várias formas de implementar esta simulação corretamente.
Soluções mais eficientes (que executem a simulação mais rapidamente) serão
bonificadas na nota. Todas as otimizações implementadas devem ser mencionadas
no relatório. Duas sugestões mutuamente exclusivas (i.e., não podem ser
usadas em conjunto):

* Atualizar apenas as células que foram modificadas num turno e não todas.
* Atualizar o mundo de uma só vez (com um único `Console.Write()`), pré-criando
  uma _string_ com todos os seus conteúdos (isto requer o uso de [sequências de
  _escape_ ANSI](https://stackoverflow.com/questions/4842424/list-of-ansi-color-escape-sequences), indo além da formatação de cores
  disponível na classe `Console`). A forma mais eficiente de "ir construíndo"
  uma _string_ é através de um
  [`StringBuilder`](https://docs.microsoft.com/dotnet/api/system.text.stringbuilder),
  e não com concatenação.

Estas otimizações só devem ser efetuadas após a simulação estar completamente
funcional, e é perfeitamente possível ter nota máxima, ou perto disso, sem a
implementação das mesmas.

### Requisitos de multi-plataforma

A aplicação deve funcionar em Windows, macOS e Linux. A melhor estratégia para
garantir que assim seja é testar o jogo em Linux (e.g., numa máquina virtual).
Algumas instruções incompatíveis com macOS e Linux são, por exemplo:

* [Console.Beep()](https://docs.microsoft.com/dotnet/api/system.console.beep)
* [Console.SetBufferSize()](https://docs.microsoft.com/dotnet/api/system.console.setbuffersize)
* [Console.SetWindowPosition()](https://docs.microsoft.com/dotnet/api/system.console.setwindowposition)
* [Console.SetWindowSize()](https://docs.microsoft.com/dotnet/api/system.console.setwindowsize)
* Entre outras.

As instruções que só funcionam em Windows têm a seguinte indicação na sua
documentação:

![The current operating system is not Windows.](img/notsupported.png "The current operating system is not Windows.")

## Objetivos e critério de avaliação

Este projeto tem os seguintes objetivos:

* **O1** - Programa deve funcionar como especificado. Atenção aos detalhes,
  pois é fácil desviarem-se das especificações caso não leiam o enunciado com
  atenção.
* **O2** - Projeto e código bem organizados, nomeadamente:
  * Estrutura de classes bem pensada e código eficiente.
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
  * Observações e resultados:
    * Indicar o que acontece ao colocar `swap-rate-exp` a 1.0 deixando as
      restantes `rate-exp` a zero.
    * Indicar o que acontece ao colocar `swap-rate-exp` a -1.0 deixando as
      restantes `rate-exp` a zero.
    * É possível encontrar algum conjunto de parâmetros que resulte na extinção
      de uma das espécies? Quando uma espécie se extingue, o que acontece às
      outras duas?
  * Referências, incluindo trocas de ideias com colegas, código aberto
    reutilizado (e.g., do StackOverflow) e bibliotecas de terceiros
    utilizadas. Devem ser o mais detalhados possível.
  * **Nota:** o relatório deve ser simples e breve, com informação mínima e
    suficiente para que seja possível ter uma boa ideia do que foi feito.
    Atenção aos erros ortográficos e à correta formatação [Markdown], pois
    ambos serão tidos em conta na nota final.

O projeto tem um peso de 10 valores na nota final da disciplina e será avaliado
de forma qualitativa. Isto significa que todos os objetivos têm de ser
parcialmente ou totalmente cumpridos. A cada objetivo, O1 a O5, será atribuída
uma nota entre 0 e 1. A nota do projeto será dada pela seguinte fórmula:

*N = 10 x O1 x O2 x O3 x O4 x O5 x D*

Em que *D* corresponde à nota da discussão e percentagem equitativa de
realização do projeto, também entre 0 e 1. Isto significa que se os alunos
ignorarem completamente um dos objetivos, não tenham feito nada no projeto ou
não comparecerem na discussão, a nota final será zero.

## Entrega

O projeto deve ser entregue por **grupos de 2 a 3 alunos** via Moodle até às
23h de 4 de setembro de 2020. Um (e apenas um) dos elementos do grupo deve ser
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
aluno deve ser claramente indicada, com devida referência ao respectivo
autor. O não cumprimento desta regra constitui plágio.

O plágio inclui a utilização de ideias, código ou conjuntos de soluções
de outros alunos ou indivíduos, ou quaisquer outras fontes para além
dos textos de apoio à disciplina, sem dar o respectivo crédito a essas
fontes. Os alunos são encorajados a discutir os problemas com outros
alunos e devem mencionar essa discussão quando submetem os projetos.
Essa menção **não** influenciará a nota. Os alunos não deverão, no
entanto, copiar códigos, documentação e relatórios de outros alunos, ou dar os
seus próprios códigos, documentação e relatórios a outros em qualquer
circunstância. De facto, não devem sequer deixar códigos, documentação e
relatórios em computadores de uso partilhado, e muito menos usar
repositórios Git públicos (embora os mesmos possam ser tornados públicos
12h após a data limite de submissão).

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

* Head, B., Grider, R. and Wilensky, U. (2017). **NetLogo Rock Paper Scissors
  model**. http://ccl.northwestern.edu/netlogo/models/RockPaperScissors. Center
  for Connected Learning and Computer-Based Modeling, Northwestern University,
  Evanston, IL.
* Whitaker, R. B. (2016). **The C# Player's Guide** (3rd Edition).
  Starbound Software.
* Albahari, J. (2017). **C# 7.0 in a Nutshell**. O’Reilly Media.
* Dorsey, T. (2017). **Doing Visual Studio and .NET Code Documentation
  Right**. Visual Studio Magazine. Retrieved from
  <https://visualstudiomagazine.com/articles/2017/02/21/vs-dotnet-code-documentation-tools-roundup.aspx>.

## Licenças

* Este enunciado é disponibilizado através da licença [CC BY-NC-SA 4.0].

## Metadados

* Autor: [Nuno Fachada]
* Curso:  [Licenciatura em Videojogos][lamv]
* Instituição: [Universidade Lusófona de Humanidades e Tecnologias][ULHT]

[CC BY-NC-SA 4.0]:https://creativecommons.org/licenses/by-nc-sa/4.0/
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
[2º projeto de LP1 2018/19]:https://github.com/VideojogosLusofona/lp1_2018_p2_solucao
[Von Neumann]:https://en.wikipedia.org/wiki/Von_Neumann_neighborhood
[Pedra, Papel e Tesoura]:https://pt.wikipedia.org/wiki/Pedra,_papel_e_tesoura
[RPSNetLogo]:http://ccl.northwestern.edu/netlogo/models/RockPaperScissors
[NetLogo]:http://ccl.northwestern.edu/netlogo
[distribuição de Poisson]:https://en.wikipedia.org/wiki/Poisson_distribution
[distribuição uniforme]:https://en.wikipedia.org/wiki/Uniform_distribution_(continuous)
[Random]:https://docs.microsoft.com/dotnet/api/system.random
[`NextDouble()`]:https://docs.microsoft.com/dotnet/api/system.random.nextdouble
[genPoisson]:https://en.wikipedia.org/wiki/Poisson_distribution#Generating_Poisson-distributed_random_variables
