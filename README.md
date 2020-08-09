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
mundo e reproduzem-se. Estas interações resultam em padrões em espiral cujo
tamanho e estabilidade dependem dos parâmetros da simulação.

O modelo está implementado como [exemplo][RPSNetLogo] no software [NetLogo]
(aberto e gratuito, também corre diretamente no _browser_), implementação esta
que pode e deve ser usada como termo de comparação ao projeto desenvolvido.

## Descrição detalhada

A simulação corre numa grelha com dimensões (`x`, `y`) toroidal com vizinhança
de [Von Neumann]. Toroidal significa que a grelha "dá a volta" na vertical e
na horizontal, ou seja, na prática não tem paredes.

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
     [Pedra, Papel e Tesoura]
   * A célula do vizinho perdedor torna-se vazia.
   * Caso alguma das células escolhidas seja vazia, nada acontece.

O número exato de cada tipo de evento em cada turno é determinado
aleatoriamente através de uma [distribuição de Poisson], que expressa a
probabilidade de uma série de eventos ocorrer num certo período de tempo. Uma
vez que este é um projeto de programação, não é exigido aos alunos que
compreendam a fundo esta distribuição, apenas que consigam implementar um
gerador de números aleatórios de Poisson a partir da [distribuição uniforme]
(sendo esta última dada pelos métodos da classe [Random] do C#).

O Wikipédia sugere [alguns algoritmos](genPoisson) para o efeito, mas apenas
o segundo, "**algorithm** _poisson random number (Junhao, based on Knuth)_",
funcionará bem com este modelo . Este algoritmo recebe o valor `λ` (média) e
devolve um número inteiro que corresponde a um número aleatório de eventos.
O parâmetro `λ` representa a média dos números aleatórios devolvidos. Na
parte do algoritmo que diz _Generate uniform random number u  in (0,1)_
deve ser usado o método [`NextDouble()`] da classe [Random] para obter
valores aleatórios uniformes entre 0 e 1. Todas as variáveis internas deste algoritmo devem ser `double`, exceto a variável `k` que deve ser
um `int`. O parâmetro `STEP` deve ser uma constante com o valor 500.

_Em construção_

## Funcionamento da simulação

A simulação termina quando o utilizador pressiona a tecla "Escape".

### Opção da linha de comando e ficheiro de parâmetros

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
* Espaços no início das linhas devem ser ignorados.
* Linhas em branco ou começadas por `#` ou `//` devem ser ignoradas, pois são
  consideradas comentários.

As opções indicadas podem ser dadas em qualquer ordem. Se alguma delas for
omitida o programa deve terminar com uma mensagem de erro indicando a opção em
falta. Opções desconhecidas ou com valores inválidos também fazem com que o
programa termine, indicando exatamente qual a linha inválida do ficheiro.

### Visualização

_Em construção_

### Resumo

Resumindo, a simulação é executada de acordo com os seguintes passos:

_Em construção_

## Requisitos do código

### Organização do código e estrutura de classes

O projeto deve estar devidamente organizado, fazendo uso de classes, _structs_
e enumerações. Cada classe, _struct_ ou enumeração deve ser colocada num
ficheiro com o mesmo nome. Por exemplo, uma classe chamada `Simulation` deve
ser colocada no ficheiro `Simulation.cs`. A estrutura de classes deve ser bem
pensada e organizada de uma forma lógica, e [cada classe deve ter uma
responsabilidade específica e bem definida][SRP].

### Eficiência do código

Existem variadíssimas formas de implementar esta simulação corretamente.
Soluções mais eficientes (que executem a simulação mais rapidamente) serão
bonificadas na nota. Todas as otimizações implementadas devem ser mencionadas
no relatório.

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
