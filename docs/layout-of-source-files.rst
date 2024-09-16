********************************
Layout de um Arquivo-Fonte Solidity
********************************

Arquivos-fontes podem conter um número arbitrário de
:ref:`contract definitions<contract_structure>`, import_ ,
:ref:`pragma<pragma>` e diretivas :ref:`using for<using-for>` e
:ref:`struct<structs>`, :ref:`enum<enums>`, :ref:`functions<functions>`, :ref:`error<errors>`
e declarações de :ref:`constant variable<constants>`.

.. index:: ! license, spdx

Identificador de Licença SPDX
=======================

A confiança nos smart contracts pode ser melhor estabelecida se o seu código-fonte
estiver disponível. Uma vez que a disponibilização do código-fonte levanta a possibilidade de problemas jurídicos
no que tange aos direitos de autoria, o compilador Solidity incentiva a utilização
de `identificadores de licença SPDX <https://spdx.org>`_ legíveis por máquina.
Todo arquivo-fonte deve começar com um comentário indicando sua licença:

``// SPDX-License-Identifier: MIT``

O compilador não valida se a licença faz parte da
`lista permitida pelo SPDX <https://spdx.org/licenses/>`_, mas
ele inclui a string fornecida no :ref:`bytecode metadata <metadata>`.

Se não quiser especificar uma licença ou se o código-fonte não for
open-source, por favor utilize o valor especial ``UNLICENSED``.
Note que ``UNLICENSED`` (não é permitido o uso, não está presente na lista de licenças SPDX)
é diferente de ``UNLICENSE`` (concede todos os direitos a todos).
O Solidity segue `a recomendação do npm <https://docs.npmjs.com/cli/v7/configuring-npm/package-json#license>`_.

É claro que o fato de fornecer esta observação não o isenta de outras
obrigações relacionadas com o licenciamento, como ter de mencionar
um cabeçalho de licença específico em cada arquivo-fonte ou o
titular original dos direitos de autoria.

O comentário é reconhecido pelo compilador em qualquer parte do arquivo ao nível do próprio,
mas recomenda-se que seja colocado no topo do arquivo.

Mais informações sobre como usar identificadores de licença SPDX
podem ser encontradas no site oficial do padrão `SPDX <https://spdx.dev/learn/handling-license-info/#how>`_.


.. index:: ! pragma

.. _pragma:

Pragmas
=======

A palavra-chave ``pragma`` é usada para habilitar certos recursos do compilador
ou verificações. Uma diretiva pragma é sempre local para um arquivo-fonte, então
você tem que adicionar o pragma a todos os seus arquivos se você quiser habilitá-lo
em todo o seu projeto. Se você :ref:`importar<import>` outro arquivo, o pragma
desse arquivo *não* se aplica automaticamente ao arquivo de importação.

.. index:: ! pragma;version

.. _version_pragma:

Pragma de Versão
--------------

Os arquivos-fonte podem (e devem) ser anotados com um pragma de versão para rejeitar a
compilação com futuras versões do compilador que possam introduzir alterações incompatíveis. 
Tentamos manter estas alterações num mínimo absoluto e
introduzi-las de forma a que as alterações na semântica também exijam alterações
na sintaxe, mas isso nem sempre é possível. Por isso, é sempre
uma boa ideia ler o changelog pelo menos para as versões que contêm
alterações de rutura. Esses lançamentos sempre têm versões do tipo
``0.x.0`` ou ``x.0.0``.

O pragma de versão é utilizado da seguinte forma: ``pragma solidity ^0.5.2;``

Um arquivo-fonte com a linha acima não compila com um compilador anterior à versão 0.5.2,
e também não funciona num compilador a partir da versão 0.6.0 (esta
segunda condição é adicionada pelo uso de ``^``). Porque
não haverá mudanças significativas até a versão ``0.6.0``, você pode
ter certeza de que seu código compila da maneira que você pretendia. A versão exata do compilador
compilador não é fixada, de modo que lançamentos de correção de bugs ainda são possíveis.

É possível especificar regras mais complexas para a versão do compilador,
estas seguem a mesma sintaxe usada por `npm <https://docs.npmjs.com/cli/v6/using-npm/semver>`_.

.. note::
  Usar o pragma de versão *não* altera a versão do compilador.
  Ele também *não* habilita ou desabilita recursos do compilador. Ele apenas
  instrui o compilador a verificar se sua versão corresponde àquela
  exigida pelo pragma. Se não corresponder, o compilador emite
  um erro.

.. index:: ! ABI coder, ! pragma; abicoder, pragma; ABIEncoderV2
.. _abi_coder:

Pragma do Codificador ABI
----------------

Ao utilizar ``pragma abicoder v1`` ou ``pragma abicoder v2`` você pode
selecionar entre as duas implementações do codificador e descodificador ABI.

O novo codificador ABI (v2) é capaz de codificar e descodificar
arrays e structs. Para além de suportar mais tipos, implica verificações de validação e segurança mais extensas
validação e verificações de segurança mais extensas, o que pode resultar em custos de gás mais elevados, mas também numa maior
segurança. É considerado
não-experimental a partir do Solidity 0.6.0 e é ativado por padrão a partir do Solidity 0.8.0.
O antigo codificador ABI ainda pode ser selecionado usando ``pragma abicoder v1;``.

O conjunto de tipos suportados pelo novo codificador é um superconjunto estrito dos
suportados pelo antigo. Os contratos que o utilizam podem interagir com os
que não o utilizam sem limitações. O inverso só é possível desde que o contrato
que não seja o ``abicoder v2`` não tente efetuar chamadas que exijam
tipos de descodificação apenas suportados pelo novo codificador. O compilador pode detectar isto
e emitirá um erro. Simplesmente habilitar o ``abicoder v2`` para o seu contrato é
suficiente para que o erro desapareça.

.. note::
  Este pragma aplica-se a todo o código definido no arquivo onde é ativado,
  independentemente de onde esse código acabe por ficar. Isto significa que um contrato
  cujo arquivo-fonte está selecionado para compilar com o codificador ABI v1
  pode ainda conter código que utiliza o novo codificador
  herdando-o de outro contrato. Isto é permitido se os novos tipos forem apenas
  utilizados internamente e não em assinaturas de funções externas.

.. note::
  Até a versão 0.7.4 do Solidity, era possível selecionar o codificador ABI v2
  usando ``pragma experimental ABIEncoderV2``, mas não era possível
  selecionar explicitamente o codificador v1 porque ele era o padrão.

.. index:: ! pragma; experimental
.. _experimental_pragma:

Pragma Experimental
-------------------

O segundo pragma é o pragma experimental. Ele pode ser usado para habilitar
recursos do compilador ou da linguagem que ainda não estão habilitados por padrão.
Os seguintes pragmas experimentais são atualmente suportados:

.. index:: ! pragma; ABIEncoderV2

ABIEncoderV2
~~~~~~~~~~~~

Porque o codificador ABI v2 não é mais considerado experimental,
ele pode ser selecionado via ``pragma abicoder v2`` (veja acima)
desde Solidity 0.7.4.

.. index:: ! pragma; SMTChecker
.. _smt_checker:

SMTChecker
~~~~~~~~~~

Este componente tem de ser ativado quando o compilador Solidity é construído
e por isso não está disponível em todos os binários do Solidity.
As :ref:`instruções de compilação<smt_solvers_build>` explicam como ativar esta opção.
Ela é ativada para os lançamentos do Ubuntu PPA na maioria das versões,
mas não para as imagens Docker, binários Windows ou binários
binários Linux construídos estaticamente. Ela pode ser ativada para o solc-js através do comando
`smtCallback <https://github.com/ethereum/solc-js#example-usage-with-smtsolver-callback>`_ se você tiver um solver SMT
instalado localmente e executar o solc-js via node (não via navegador).

Se utilizar ``pragma experimental SMTChecker;``, então obtém-se
adicionais :ref:`advertências de segurança<formal_verification>` que são obtidas consultando um
SMT solver.
O componente ainda não suporta todas as funcionalidades da linguagem Solidity e
provavelmente emite muitos avisos. No caso de reportar caraterísticas não suportadas, a
análise pode não ser totalmente correta.

.. index:: source file, ! import, module, source unit

.. _import:

Importação de Outros Arquivos-Fontes
============================

Sintaxe e Semântica
--------------------

O Solidity suporta instruções de importação para ajudar a modularizar o seu código que
são semelhantes às disponíveis em JavaScript
(a partir do ES6). No entanto, Solidity não suporta o conceito de
uma `exportação padrão <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#description>`_.

A nível global, é possível utilizar instruções de importação com o seguinte formato:

.. code-block:: solidity

    import "filename";

A parte ``filename`` é chamada de *importpath*.
Essa declaração importa todos os símbolos globais de “filename” (e símbolos importados para lá) para o escopo global atual
(diferente do ES6, mas compatível com as versões anteriores do Solidity).
Esta forma não é recomendada para uso, porque polui imprevisivelmente o espaço de nomes.
Se você adicionar novos itens de nível superior dentro de “filename”, eles automaticamente
aparecem em todos os ficheiros que importam assim de “filename”. É melhor importar símbolos
específicos explicitamente.

O exemplo a seguir cria um novo símbolo global ``symbolName`` cujos membros são todos os
os símbolos globais de ``“filename”``:

.. code-block:: solidity

    import * as symbolName from "filename";

o que faz com que todos os símbolos globais estejam disponíveis no formato ``symbolName.symbol``.

Uma variante desta sintaxe que não faz parte do ES6, mas que pode ser útil, é:

.. code-block:: solidity

  import "filename" as symbolName;

que é equivalente a ``importar * as symbolName from “filename”;``.

Se houver um conflito de nomes, pode mudar o nome dos símbolos durante a importação. Por exemplo,
o código abaixo cria novos símbolos globais ``alias`` e ``symbol2`` que referenciam
``symbol1`` e ``symbol2`` de dentro de ``“filename”``, respectivamente.

.. code-block:: solidity

    import {symbol1 as alias, symbol2} from "filename";

.. index:: virtual filesystem, source unit name, import; path, filesystem path, import callback, Remix IDE

Caminhos de Importação
------------

Para poder suportar compilações reproduzíveis em todas as plataformas, o compilador Solidity tem de
abstrair os detalhes do sistema de arquivos onde os arquivos de origem estão armazenados.
Por este motivo, os caminhos de importação não se referem diretamente a arquivos no sistema de arquivos do anfitrião.
Em vez disso, o compilador mantém uma base de dados interna (*virtual filesystem* ou *VFS*) onde
a cada unidade-fonte é atribuído um *source unit name* único, que é um identificador opaco e não estruturado.
O caminho de importação especificado numa instrução de importação é traduzido para um nome de unidade de origem e utilizado para
encontrar a unidade de origem correspondente nesta base de dados.

Utilizando a API :ref:`Standard JSON <compiler-api>` é possível fornecer diretamente os nomes e
conteúdo de todos os arquivos-fonte como parte da entrada do compilador.
Neste caso, os nomes das unidades de origem são verdadeiramente arbitrários.
Se, no entanto, você quiser que o compilador encontre e carregue automaticamente o código-fonte no VFS, seus
nomes de unidades de código-fonte precisam ser estruturados de forma a possibilitar uma :ref:`import callback
<import-callback>` para localizá-las.
Ao usar o compilador de linha de comando, a callback de importação padrão suporta apenas o carregamento do código fonte
do sistema de arquivos do host, o que significa que os nomes das unidades de código-fonte devem ser caminhos.
Alguns ambientes fornecem callbacks personalizados que são mais versáteis.
Por exemplo, o `Remix IDE <https://remix.ethereum.org/>`_ fornece uma que
permite `importar arquivos de URLs HTTP, IPFS e Swarm ou referir-se diretamente a pacotes no registro NPM
<https://remix-ide.readthedocs.io/en/latest/import.html>`_.

Para uma descrição completa do sistema de virtual filesystem e da lógica de resolução de caminhos utilizada pelo
compilador veja :ref:`Resolução de caminhos <path-resolution>`.

.. index:: ! comment, natspec

Comentários
========

São possíveis comentários de uma linha (``//``) e comentários de várias linhas (``/*...*/``).

.. code-block:: solidity

    // Este é um comentário de linha única.

    /*
    Este é um comentário
    de múltiplas linhas.
    */

.. note::
  Um comentário de linha única é terminado por qualquer terminador de linha unicode
  (LF, VF, FF, CR, NEL, LS ou PS) na codificação UTF-8. O terminador ainda é parte do
  do código-fonte após o comentário, portanto, se não for um símbolo ASCII
  (estes são NEL, LS e PS), isso levará a um erro do parser.

Adicionalmente, existe um outro tipo de comentário chamado comentário NatSpec,
que é detalhado no :ref:`guia de estilo<style_guide_natspec>`. Eles são escritos com uma
barra tripla (``///``) ou um bloco de asterisco duplo (``/** ... */``) e
devem ser usados diretamente acima de declarações de funções ou declarações.
