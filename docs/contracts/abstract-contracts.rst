.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
Contratos Abstratos
******************

Contratos devem ser marcados como abstratos quando pelo menos uma de suas funções não está implementada
ou quando eles não fornecem argumentos para todos os construtores de contrato base. Mesmo que este não
seja o caso, um contrato ainda pode ser marcado como abstrato, como quando você não pretende que o 
contrato seja criado diretamente. Contratos abstratos são semelhantes a :ref:interfaces, mas uma 
interface é mais limitada ao que pode ser declarado.

Um contrato abstrato é declarado usando a palavra-chave ``abstract``, como mostrado no exemplo a seguir.
Observe que este contrato precisa ser definido como abstrato, porque a função ``utterance()`` é declarada,
mas nenhuma implementação foi fornecida (nenhum bloco de código de implementação ``{}`` foi fornecido).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
        function utterance() public virtual returns (bytes32);
    }
    
Contratos abstratos não podem ser instanciados diretamente. Isso também é verdade se um contrato abstrato 
implementar todas as funções definidas. O uso de um contrato abstrato como classe base é mostrado no 
seguinte exemplo:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
        function utterance() public pure virtual returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public pure override returns (bytes32) { return "miaow"; }
    }

Se um contrato herda de um contrato abstrato e não implementa todas as funções definidas por
sobreposição, ele também precisa ser marcado como abstrato.

Observe que uma função sem implementação é diferente de uma :ref:`Function Type <function_types>` embora sua sintaxe seja muito semelhante.

Exemplo de função sem implementação (uma declaração de função):

.. code-block:: solidity

    function foo(address) external returns (address);

Exemplo de uma declaração de uma variável cujo tipo é um tipo de função:

.. code-block:: solidity

    function(address) external returns (address) foo;

Contratos abstratos desacoplam a definição de um contrato de sua
implementação, fornecendo melhor extensibilidade e auto-documentação e
facilitando padrões como o `Método Template <https://en.wikipedia.org/wiki/Template_method_pattern>`_ e removendo a duplicação de código.
Contratos abstratos são úteis da mesma forma que definir métodos
em uma interface é útil. É uma maneira para o designer do
contrato abstrato dizer "qualquer filho meu deve implementar este método".


.. note::

  Contratos abstratos não podem substituir uma função virtual implementada por uma não implementada.
