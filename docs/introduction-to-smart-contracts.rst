#####################################
Introdução aos Contratos Inteligentes
#####################################

.. _simple-smart-contract:

*******************************
Um Contrato Inteligente Simples
*******************************

Vamos começar com um exemplo básico que define o valor de uma variável e a expõe
para que outros contratos possam acessá-la. Tudo bem se você não compreender
tudo agora, entraremos em mais detalhes depois.

Exemplo de Armazenamento
========================

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    }

A primeira linha informa que o código-fonte está licenciado sob a
GPL versão 3.0. Especificadores de licença que podem ser lidos e entendidos automaticamente por sistemas são importantes
em um cenário onde a publicação do código-fonte é o padrão.

A próxima linha especifica que o código-fonte foi escrito para
a versão 0.4.16 do Solidity, até, mas não incluindo, a versão 0.9.0.
Isso garante que o contrato não seja compilado com uma nova versão do compilador (incompatível), onde ele poderia se comportar de maneira diferente.
:ref:`Pragmas<pragma>` são instruções comuns para compiladores sobre como tratar o
código-fonte (por exemplo, `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_).

Um contrato no contexto do Solidity, é uma coleção de código (suas *funções*) e
dados (seu *estado*) que reside em um endereço específico na blockchain do Ethereum.
A linha ``uint storedData;`` declara uma variável de estado chamada ``storedData`` do
tipo ``uint`` (um número inteiro sem sinal de 256 bits). Você pode pensar nela como um único espaço
em um banco de dados, que pode ser consultado e alterado ao chamar funções do
código que gerencia o banco de dados. Neste exemplo, o contrato define as
funções ``set`` e ``get`` que podem ser usadas para modificar
ou recuperar o valor da variável.

Para acessar um elemento (como uma variável de estado) do contrato atual, você normalmente não precisa adicionar o prefixo ``this.``,
basta acessá-lo diretamente pelo seu nome.
Ao contrário de outras linguagens, omitir o ``this.`` não é apenas uma questão de estilo,
mas sim uma maneira completamente diferente de acessar a propriedade, embora isso seja explicado em mais detalhes posteriormente.

Este contrato ainda não faz muito, além de (devido à infraestrutura
construída pelo Ethereum) permitir que qualquer pessoa armazene um número único acessível por
qualquer pessoa no mundo, sem uma maneira (viável) de impedir que você publique
esse número. Qualquer um poderia chamar a função ``set`` novamente com um valor diferente
e substituir o seu número, mas o número ainda estará armazenado no histórico
da blockchain. Mais adiante, você verá como você pode impor restrições de acesso
para que apenas você possa alterar o número.

.. warning::
    Tenha cuidado ao usar texto Unicode, com caracteres que parecem semelhantes (ou até mesmo idênticos) podem
    ter pontos de código diferentes e, portanto, são codificados como arrays de bytes distintos.

.. note::
    Todos os identificadores (nomes de contrato, nomes de funções e nomes de variáveis) são restritos ao
    conjunto de caracteres ASCII. É possível armazenar dados codificados em UTF-8 em variáveis do tipo string.

.. index:: ! subcurrency

Exemplo de Submoeda
===================

O contrato a seguir implementa a forma mais simples de uma
criptomoeda. O contrato permite apenas seu criador emita novas moedas (diferentes esquemas são possíveis).
Qualquer pessoa pode enviar moedas para outra sem a necessidade de
registrar um usuário e senha, tudo o que você precisa é de um par de chaves Ethereum.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.26;

    // Isso será compilado apenas via IR
    contract Coin {
        // A palavra-chave "public" torna as variáveis
        // acessíveis para outros contratos
        address public minter;
        mapping(address => uint) public balances;

        // Eventos permitem que os clientes reajam a mudanças
        // específicas no contrato que você declarar
        event Sent(address from, address to, uint amount);

        // O código do construtor é executado apenas quando o contrato
        // é criado
        constructor() {
            minter = msg.sender;
        }

        // Envia uma quantidade de moedas recém-criadas para um endereço
        // Só pode ser chamado pelo criador do contrato
        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            balances[receiver] += amount;
        }

        // Erros permitem você forneça informações sobre
        // por que uma operação falhou. Eles são retornados
        // ao chamador da função.
        error InsufficientBalance(uint requested, uint available);

        // Envia uma quantidade de moedas existentes
        // de qualquer chamador para um endereço
        function send(address receiver, uint amount) public {
            require(amount <= balances[msg.sender], InsufficientBalance(amount, balances[msg.sender]));
            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

Este contrato introduz alguns conceitos novos, vamos abordá-los um a um.

A linha ``address public minter;`` uma variável de estado do tipo :ref:`address<address>`.
O tipo ``address`` é um valor de 160 bits que não permite operações aritméticas.
Ele é adequado para armazenar endereços de contratos, ou um hash da metade pública
de um par de chaves pertencente a :ref:`contas externas<accounts>`.

A palavra-chave ``public`` gera automaticamente uma função que permite acessar o valor atual da variável de estado
a partir de fora do contrato. Sem essa palavra-chave, outros contratos não tem como acessar a variável.
O código da função gerado pelo compilador é equivalente
ao seguinte (ignore ``external`` e ``view`` por enquanto):

.. code-block:: solidity

    function minter() external view returns (address) { return minter; }

Você poderia adicionar a função como mencionado acima por conta própria, mas teria uma função e uma variável de estado com o mesmo nome.
Não é preciso fazer isso, o compilador resolve isso por você.

.. index:: mapping

A próxima linha, ``mapping(address => uint) public balances;`` também
cria uma variável de estado pública, mas é um tipo de dado mais complexo.
O tipo :ref:`mapping <mapping-types>` mapeia endereços para :ref:`inteiros sem sinal <integers>`.

Mappings podem ser visto como `tabelas hash <https://en.wikipedia.org/wiki/Hash_table>`_ que são
virtualmente inicializadas de forma que todas as chaves possíveis existes desde início e são mapeadas para um
valor cuja representação de bytes é composta apenas de zeros. No entanto, não é possível obter uma lista de todas as chaves de
um mapping, nem uma lista de todos os valores. Registre o que você adicionou para o mapping ou use-o em um contexto onde isso não seja necessário. Ou,
ainda melhor, mantenha uma lista ou use um tipo de dado mais adequado.

A :ref:`função getter<getter-functions>` criada pela palavra-chave ``public``
é mais complexa no caso de um mapping. Ela se parece com o seguinte:

.. code-block:: solidity

    function balances(address account) external view returns (uint) {
        return balances[account];
    }

Voc6e pode usar essa função para consultar o saldo de uma única conta.

.. index:: event

A linha ``event Sent(address from, address to, uint amount);`` declara
um :ref:`"evento" <events>`, que é emitido na última linha da função
``send``. Clientes Ethereum, como aplicações web, podem
ouvir esses eventos emitidos na blockchain sem muito
custo. Assim que o evento é emitido, o ouvinte recebe os
argumentos ``from``, ``to`` e ``amount``, o que torna possível rastrear
transações.

Para ouvir esse evento, você poderia usar o seguinte
código JavaScript, que usa `web3.js <https://github.com/web3/web3.js/>`_ para criar o objeto do contrato ``Coin``,
e qualquer interface de usuário chama a função ``balances`` gerada automaticamente, mencionada anteriormente:

.. code-block:: javascript

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Transferência de moeda: " + result.args.amount +
                " moedas foram enviadas de " + result.args.from +
                " para " + result.args.to + ".");
            console.log("Saldos atualizados:\n" +
                "Remetente: " + Coin.balances.call(result.args.from) +
                "Destinatário: " + Coin.balances.call(result.args.to));
        }
    })

.. index:: coin

O :ref:`constructor<constructor>` é uma função especial que é executada durante a criação do contrato e
não pode ser chamada posteriormente. Neste caso, ele armazena permanentemente o endereço da pessoa que está criando o contrato.
A variável ``msg`` (junto com ``tx`` e ``block``) é uma
:ref:`variável global especial <special-variables-functions>` que
contém propriedades que permitem o acesso à blockchain. ``msg.sender`` é
sempre o endereço de onde a chamada de função atual (externa) se originou.

As funções que compõem o contrato, e que usuários e contratos podem chamar, são ``mint`` e ``send``.

A função ``mint`` envia uma quantidade de moedas recém-criadas para outro endereço. A chamada da função :ref:`require
<assert-and-require>` define condições que revertem todas as alterações se não forem atendidas. Neste
exemplo, ``require(msg.sender == minter);`` garante que apenas o criador do contrato possa chamar
``mint``. Em geral, o criador pode minerar quantos tokens quiser, mas em algum momento, isto levará
a um fenômeno chamado "overflow". Observe que, devido à :ref:`Aritmética verifica<unchecked>` por padrão,
a transação seria revertida se a expressão ``balances[receiver] += amount;``
transbordar (overflows), ou seja, quando ``balances[receiver] + amount`` em aritmética de precisão arbitrária for maior
que o valor máximo de ``uint`` (``2**256 - 1``). Isso também é válido para a instrução
``balances[receiver] += amount;`` na função ``send``.

:ref:`Erros <errors>` permitem que você forneça mais informações ao chamador sobre
por que uma condição ou operação falhou. Erros são usados juntamente com a
:ref:`instrução revert <revert-statement>`. A instrução ``revert`` aborta incondicionalmente
e reverte todas as alterações, de maneira semelhante à :ref:`função require <assert-and-require-statements>`.
Ambas as abordagens permitem que você forneça o nome de um erro e dados adicionais, que serão enviados ao chamador
(e eventualmente para a aplicação front-end ou explorador de blocos), facilitando
o processo de depuração ou reação a uma falha.

A função ``send`` pode ser usada por qualquer pessoa (que já
possua algumas dessas moedas) para enviar moedas para qualquer outra pessoa. Se o remetente não tiver
moedas suficiente para enviar, a condição ``if`` será avaliada como verdadeira. Como resultado, a instrução ``revert`` fará com que a operação falhe
fornecendo ao remetente detalhes do erro utilizando o erro ``InsufficientBalance.

.. note::
    Se você usar
    este contrato para enviar moedas para um endereço, não verá nada quando você
    olhar para esse endereço em um exporador de blockchain, por que o registro que você enviou
    moedas e os saldos alterados são apenas armazenados no armazenamento de dados deste
    contrato de moeda específico. Usando eventos, você pode criar
    um "explorador de blockchain" que rastreia transações e saldos da sua nova moeda,
    mas é necessário inspecionar o endereço do contrato da moeda e não os endereços dos
    proprietários das moedas.

.. _blockchain-basics:

*******************************
Noções Básicas sobre Blockchain
*******************************

Blockchains como conceito não são muito difíceis de entender para programadores. A razão é que
muitas das complicações (mineração, `hashing <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_,
`criptografia de curva elíptica <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_,
`redes peer-to-peer <https://en.wikipedia.org/wiki/Peer-to-peer>`_, etc.)
estão presentes apenas para fornecer um certo conjuto de recuros e garantias para a plataforma. Uma vez que você aceita esses
recursos como garantidos, não precisa se preocupar com a tecnologia subjacente - ou você precisa
saber como o AWS da Aamazon funciona internamente para utilizá-lo?

.. index:: transaction

Transações
==========

Uma blockchain é um banco de dados transacional compartilhado globalmente.
Isso significa que todos podem ler entradas no banco de dados apenas participando da rede.
Se você quiser alterar algo no banco de dados, precisa criar uma chamada transação,
que deve ser aceita por todos os outros.
A palavra transação implica que a alteração que você deseja fazer (suponha que você deseja alterar
dois valores ao mesmo tempo) ou não está pronta ao todo ou não foi completamente aplicada. Além disso,
enquanto sua transação está sendo aplicada ao banco de dados, nenhuma outra transação pode modificá-la.

Como exemplo, imagine uma tabela que lista os saldos de todas as contas em uma
moeda eletrônica. Se uma transferência de uma conta para outra for solicitada,
a natureza transacional do banco de dados garante que, se o valor for
subtraído de uma conta, ele será sempre adicionada à outra conta. Se, por
qualquer motivo, não for possível adicionar o valor à conta de destino, a
conta de origem também não será modificada.

Além disso, uma transação é sempre assinada criptograficamente pelo remetente (criador).
Isso torna simples proteger o acesso a modificações específicas do
banco de dados. No exemplo da moeda eletrônica, uma verificação simples garante que
somente a pessoa que possui as chaves da conta pode transferir uma quantida, como Ether, a partir dela.

.. index:: ! block

Blocos
======

Um grande obstáculo à ser superado é o que (em termos de Bitcoin) é chamado de "ataque de gasto duplo":
O que acontece se duas transações existirem na rede, ambas tentando esvaziar uma conta?
Somente uma das transações pode ser válida, tipicamente a que for aceita primeiro.
O problema é que "primeiro" não é um termo objetivo em uma rede peer-to-peer.

A resposta abstrata para isso é que você não precisa se preocupar. Uma ordem globalmente aceita de transaçòes
será selecionada por você, resolvendo o conflito. As transações serão agrupadas em um "bloco" e,
em seguida, serão executadas e distribuídas entre todos os nós participantes.
Se duas transações se contradizerem, aquela que terminar em segundo será
rejeitada e não fará parte do bloco.

Esses blocos formam uma seguência linear no tempo, e é daí que vem a palavra "blockchain" (cadeia de blocos).
Blocos são adicionados à cadeia em intervalos regulares, embora esses intervalos possam ser sujeitos a mudanças no futuro.
Para obter informações mais atualizadas, é recomendado monitorar a rede, por example, no `Etherscan <https://etherscan.io/chart/blocktime>`_.

Como parte do "mecanismo de seleção de ordem", o qual é chamado de `atestação <https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/attestations/>`_, pode acontecer que
blocos sejam revertidos de tempos em tempos, mas apenas na "ponta" da cadeia. Quanto mais
blocos são adicionados sobre um bloco em específico, menor é a probabilidade que esse bloco seja revertido. Portanto, é possível que suas transações sejam revertidas e até removidas da blockchain, mas quanto mais você esperar, menos provável isso será.

.. note::
    Transações não são garantidas de serem incluídas no próximo block ou qualquer bloco futuro específico,
    uma vez que não cabe ao remetente da transação, mas sim aos mineradores, determinar em qual bloco a transação será incluída.

    Se você quiser agendar chamadas futuras do seu contrato, pode usar
    uma ferramenta de contratos inteligentes ou um serviço de oráculo.

.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

********************************
A Máquina Virtual Ethereum (EVM)
********************************

Visão Geral
===========

A Máquina Virtual Ethereum ou EVM é o ambiente de execução
para contratos inteligentes na rede Ethereum. Não é apenas isolada, mas
completamente isolada, o que significa que o código executando
dentro da EVM não tem acesso à rede, ao sistema de arquivos ou a outros processos.
Os contratos inteligentes tem acesso limitado à outros contratos inteligentes.

.. index:: ! account, address, storage, balance

.. _accounts:

Contas
======

Existem dois tipos de contas no Ethereum que compartilham o mesmo
espaço de endereço: **contas externas** que são controladas por
pares de chaves públicas e privadas (ou seja, pessoas) e **contas de contrato** que são
controladas pelo código armazenado junto com a conta.

O endereço de uma conta externa é determinado a partir
da chave pública, enquanto o endereço de um contrato é
determinado no momento em que o contrato é criado
(ele é derivado do endereço do criador e do número
de transações enviadas a partir dasse endereço, o chamado "nonce").

Independente de a conta armazenar ou não código, ambos os tipos são
tratados de forma igual pela EVM.

Cada conta possui um armazenamento persistente de chave-valor que mapeia palavras de 256 bits para palavras de 256 bits
chamado de **armazenamento**.

Além disso, cada conta possui um **saldo** em
Ether (em "Wei" para ser exata, ``1 ether`` é ``10**18 wei``) que pode ser modificado ao enviar transações que
incluem Ether.

.. index:: ! transaction

Transações
==========

Uma transação é uma mensagem que é enviada de uma conta para outra
conta (que poderia ser a mesma ou vazia, veja abaixo).
Ela pode incluir dados binários (que é chamado de "payload") e Ether.

Se a conta de destino contém código, esse código é executado e
o payload é fornecido como dados de entrada.

Se a conta de destino não estiver definida (ou seja, a transação não tem
um destinatário ou o destinatário está definido como ``null``), a transação
cria um **novo contrato**.

Como já mencionado, o endereço desse contrato não é
o endereço zero, mas um endereço derivado de remetente e
do número de transações enviadas (o "nonce"). O payload
de uma transação de criação de contrato é considerado como
bytecode EVM e executado. Os dados de saída dessa execução são
armazenados permanentemente como código do contrato.
Isso significa que, para criar um contrato, você não
envia o código real do contrado, mas, na verdade, um código que
retorna este código quando executado.

.. note::
  Enquanto um contrato está sendo criado, seu código ainda está vazio.
  Por causa disso, você não deve chamar o contrato em
  construção até que seu construtor tenha
  terminado de execução.

.. index:: ! gas, ! gas price

Gas
===

Na criação de uma transação, é cobrada uma certa quantidade de **gas**
que deve ser paga pelo originador da transação (``tx.origin``).
À medida que a EVM executa a
transação, o gas é gradualmente consumido de acordo com regras específicas.
Se o gas se esgotar em algum momento (ou seja, se o valor ficar negativo),
uma execção de falta de gas é acionada, o que encerra a execução e reverte todas as modificações
feitas no estado no quadro de chamada atual.

Esse mecanismo incentiva o uso econônico do tempo de execução da EVM
e também compensam os executores da EVM (ou seja, mineradores / validadores) pelo seu trabalho.
Como cada bloco tem uma quantidade máxima de gas, isso também limita a quantidade
de trabalho necessário para validar o bloco.

O **preço do gas** é um valor definido pelo originador da transação, que
deve pagar ``gas_price * gas`` antecipadamente para o executor da EVM.
Se algum gas sobrar após a execução, este é reembolsado ao originador da transação.
Em caso de uma exceção que reverte mudanças, o gas já utilizado não é reembolsado.

Como executores da EVM pode escolher incluir uma transação ou não,
os remetentes de transações não podem abusar do sistema definindo um preço de gas muito baixo.

.. index:: ! storage, ! memory, ! stack

Armazenamento, Memória e a Pilha 
================================

A Máquina Virtual Ethereum (EVM) têm três áreas onde pode armazenar dados:
storage (armazenamento), memory (memória) e a stack (pilha).

Cada conta tem uma área de dados chamada **storage**, que é persistente entre chamadas de função
e transações.

O storage é uma estrutura de chave-valor que mapeia palavras de 256 bits para palavras de 256 bits.
Não é possível enumerar o storage de dentro de um contrato, e é
relativamente caro tanto para leitura quanto, especialmente, para inicialização e modificação. Por conta desse custo,
e recomendável minimizar o que é armazenado de forma persistente, limitando-se ao que o contrato realmente precisa para funcionar.
Dados como calculos derivados, caches e agregrados devem ser armazenados fora do contrato.
Um contrato não pode ler ou escrever em nenhum storage que não seja o seu próprio.

A segunda área de dados é chamada de **memory**, no qual o contrato recebe
uma instância recém-limpa a cada chamada de mensagem. A memory é linear e pode ser
endereçada em nível de byte, mas leituras são limitadas a uma largura de 256 bits, enquanto escritas
pode ser 8 bits ou 256 bits de largura. A memory é expandida por palavra (256 bits) quando
se acessa (leitura ou escrita) uma palavra de memória previamente intocada. No momento da expansão, o custo em gas deve ser pago. A memory torna-se mais
cara quanto maior for seu crescimento (seu custo escala quadraticamente).

A EVM não é uma máquina de registradores, mas uma máquina de pilha, então todos
os cálculos são realizados em uma área de dados chamada **stack** (pilha). Tem um tamanho máximo de
1024 elementos e contém palavras de 256 bits. O acesso a stack é
limitado à extremidade superior da seguinte maneira:
é possível copiar um dos
16 elementos mais altos para o topo da stack ou trocar o
elemento superior com um dos 16 elementos abaixo dele.
Todas as outras operações pegam os dois (ou um, ou mais, dependendo da
operação) elementos do topo da stack e colocam o resultado de volta no topo da stack.
Naturalmente, é possível mover os elementos da stack para o storage ou memory
para obter acesso mais profundo à stack,
mas não é possível acessar diretamente elementos mais profundos na stack sem primeiro
remover os elementos do topo.

.. index:: ! instruction

Conjunto de instruções
======================

O conjunto de instruções da EVM é mantido minimalista para evitar
implementações incorretas ou inconsistentes, o que poderia causar problemas de consenso.
Todas as instruções operam no tipo de dado básico, palavras de 256 bits ou em fatias de memória
(ou outros arrays de bytes).
As operações aritméticas usuais, de bits, lógicas e de comparação estão presentes.
Saltos condicionais e incondicionais são possíveis. Além disso,
contratos podem acessar propriedades relevantes do bloco atual,
como seu número e timestamp.

Para uma lista completa, por favor veja a :ref:`lista de opcodes <opcodes>` como parte da
documentação de assembly inline.

.. index:: ! message call, function;call

Chamadas de Mensagem
====================

Os contratos podem chamar outros contratos ou enviar Ether para contas
sem contratos através de chamadas de mensagem. As chamadas de mensagens são semelhantes
às transações, pois possuem uma origem, um destino, payload de dados,
Ether, gas e dados de retorno. Na verdade, toda transação consiste em
uma chamada de mensagem de nível superior que, por sua vez, pode criar outras chamadas de mensagem.

Um contrato pode decidir quanto de seu **gas** restante será enviado
com a chamada de mensagem interna e quanto deseja reter.
Se uma exceção de falta de gas ocorrer na chamada interna (ou qualquer
outra exceção), isso será assinado por um valor de erro colocado na stack.
Nesse caso, apenas o gas enviado junto com a chamada será utilizado.
No Solidity, o contrato que faz a chamada gera uma exceção manual por padrão em
tais situações, de modo que as exceções "subam" a pilha de chamadas.

Como já mencionado, o contrato chamado (que pode ser o mesmo que o chamador)
irá receber uma instância de memória limpa e terá acesso ao
payload da chamada - que será fornecida em uma área separada chamada **calldata**.
Após finalizar sua execução, ele pode retornar dados que serão armazenados em
um local da memória do chamador, pré-alocado por ele.
Todas essas chamadas são totalmente síncronas.

As chamadas são **limitadas** a uma profundidade de 1024, o que significa que, para operações
mais complexas, loops devem ser preferidos a chamadas recursivas. Além disso,
apenas 63/64 do gas pode ser encaminhado em uma chamada de mensagem, o que limita a
profundidade prática a um pouco menos de 1000.

.. index:: delegatecall, library

Delegatecall e Bibliotecas
==========================

Existe uma variante especial de uma chamada de mensagem, chamada **delegatecall**,
que é idêntica a uma chamada de mensagem, exceto pelo fato de que
o código no endereço de destino é executado no contexto (ou seja, no endereço) do contrato
que está fazendo a chamada, e ``msg.sender`` e ``msg.value`` não mudam seus valores.

Isso significa que um contrato pode carregar dinamicamente, código de um endereço
diferente em tempo de execução. O Storage, o endereço atual e o saldo ainda
se referem ao contrato que faz a chamada, apenas o código é obtido do endereço chamado.

Isso torna possível implementar o recurso de "biblioteca" no Solidity:
código de biblioteca reutilizável que pode ser aplicado ao storage de um contrato, por exemplo,
para implementar uma estrutura de dados complexa.

.. index:: log

Logs
====

É possível armazenar dados em uma estrutura de dados especialmente indexada
que mapeia até o nível de bloco. Esse recurso chamada **logs**
é utilizado pelo Solidity para implementar :ref:`eventos <events>`.
Os contratos não podem acessar os dados dos logs depois de serem criados, mas eles
podem ser acessados eficientemente fora da blockchain.
Como parte dos dados dos logs é armazenada em `filtros de bloom <https://en.wikipedia.org/wiki/Bloom_filter>`_, é
possível buscar esses dados de forma eficiente e criptograficamente
segura. Dessa forma, pares de rede que não baixam a blockchain inteira
(os chamados "light clients") ainda podem encontrar esses logs.

.. index:: contract creation

Criar
=====

Os contratos até podem criar outros contratos usando um opcode especial (ou seja,
eles não simplesmente chamam o endereço zero como uma trasação faria). A única diferença entre
essas **chamadas de criação** e as chamadas de mensagem normais é que os dados do payload são
executados e o resultado é armazenado como código, e o chamador / criador
recebe o endereço do novo contrato na stack.

.. index:: ! selfdestruct, deactivate

Desativação e Autodestruição
============================

A única maneira de remover código da blockchain é quando um contrato nesse
endereço executa a operação ``selfdestruct``. O Ether restante armazenado
nesse endereço é enviado para um destino designado, e então o armazenamento e o código
são removidos do estado. Remover o contrato em teoria parece ser uma boa
idéia, mas isso é potencialmente perigoso, pois se alguém enviar Ether para contratos
removidos, esse Ether será perdido para sempre.

.. warning::
    A partir do ``EVM >= Cancun``, o ``selfdestruct`` irá **apenas** enviar todo Ether da conta para o destinatário indicado e não destruirá o contrato.
    No entando, quando o ``selfdestruct`` é chamado na mesma transação que cria o contrato,
    o comportamento de ``selfdestruct`` antes ao hardfork Cancun (ou seja, ``EVM <= Shangai``) é preservado, destruindo o contrato atual e
    excluindo qualquer dado, incluindo chaves de armazenamento, código e a própria conta.
    Consulte o `EIP-6780 <https://eips.ethereum.org/EIPS/eip-6780>`_ para mais detalhes.

    Esse novo comportamento resulta de uma mudança na rede que afeta todos os contratos presentes na
    mainnet e testnets do Ethereum.
    Vale destacar que essa mudança depende da versão do EVM da rede onde
    o contrato é implantado.
    A configuração ``--evm-version`` usado ao compilar o contrato não interfere nesse comportamento.

    Além disso, a opcode ``selfdestruct`` foi depreciada na versão 0.8.18 do Solidity,
    como recomendado pelo `EIP-6049 <https://eips.ethereum.org/EIPS/eip-6049>`_.
    A depreciação continua em vigor, e o compilador ainda emitirá avisos ao seu uso.
    O uso em novos contratos é fortemente desencorajado, mesmo levando em consideração o novo comportamento.
    Mudanças futuras no EVM podem reduzir ainda mais a funcionalidade dessa opcode.

.. warning::
    Mesmo que um contrato seja removido pelo ``selfdestruct``, ele ainda faz parte do
    histórico da blockchain e provavelmente é retido por muitos dos nós do Ethereum.
    Portanto, usar ``selfdestruct`` não é o mesmo que excluir dados de um disco rígido.

.. note::
    Mesmo que o código de um contrato não contenha uma chamada para ``selfdestruct``,
    ele ainda pode realizar essa operação utilizando ``delegatecall`` ou ``callcode``.

Se você quiser desativar seus contratos, deve **desativá-los**
alterando algum estado interno que faça com que todas as funções revertam. Isso
torna impossível o uso do contrato, já que ele retornaria Ether imediatamente.


.. index:: ! precompiled contracts, ! precompiles, ! contract;precompiled

.. _precompiledContracts:

Contratos pré-compilados
========================

Existe um pequeno conjunto de endereços de contratos que são especiais:
O intervalo de endereços entre ``1`` e (incluindo) ``0x0a`` contém
"contratos pré-compilados" que podem ser chamados como qualquer outro contrato,
mas seu comportamento (e seu consumo de gas) não é definido
pelo código EVM armazenado nesse endereço (eles não contêm código),
e sim implementado diretamente no ambiente de execução do EVM.

Diferentes cadeias compatíveis com o EVM podem utilizar um conjunto diferente de
contratos pré-compilados. Também pode ser possível que novos
contratos pré-compilados sejam adicionados à cadeia principal do Ethereum no futuro,
mas é razoável esperar que eles estejam sempre no intervalo entre
``1`` e ``0xffff`` (inclusive).
