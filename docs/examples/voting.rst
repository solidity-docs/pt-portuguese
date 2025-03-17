.. index:: voting, ballot

.. _voting:

******
Votação
******

O contrato a seguir é bastante complexo, mas mostra muitos recursos do Solidity. 
Ele implementa um contrato de votação. 
É claro que, o principal problema da votação eletrónica é como atribuir os direitos de voto às pessoas corretas
e como evitar a manipulação. Não resolveremos todos os problemas aqui, mas pelo menos mostraremos
como a votação delegada pode ser feita para que a contagem de votos
seja **automática e completamente transparente** ao mesmo tempo.

A ideia é criar um contrato por boletim de voto,
fornecendo um nome curto para cada opção.
Em seguida, o criador do contrato, que atua como
presidente, dará o direito de voto a cada
endereço individualmente.

As pessoas por trás dos endereços podem então escolher
votar elas próprias ou delegar o seu voto a uma pessoa da sua confiança.

No final do tempo de votação, a função ``winningProposal()``
retornará a proposta com o maior número
de votos.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    /// @title Votação com delegação
    contract Ballot {
        // Isto declara um novo tipo complexo
        // que será utilizado para variáveis mais tarde.
        // Ele representará um único votante.
        struct Voter {
            uint weight; // peso é acumulado por delegação
            bool voted;  // se for verdadeiro, essa pessoa já votou
            address delegate; // pessoa delegada
            uint vote;   // índice da proposta votada
        }

        // Trata-se de um tipo para uma proposta única.
        struct Proposal {
            bytes32 name;   // nome curto (up to 32 bytes)
            uint voteCount; // número de votos acumulados
        }

        address public chairperson;

        // Isto declara uma variável de estado que
        // armazena uma struct `Voter` para cada endereço possível.
        mapping(address => Voter) public voters;

        // Um array de tamanho dinâmico para a struct `Proposal`.
        Proposal[] public proposals;

        /// Criando um novo boletim de voto para escolher um dos `proposalNames`.
        constructor(bytes32[] memory proposalNames) {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;

            // Para cada um dos nomes de proposta fornecidos,
            // cria um novo objeto proposta
            // e adiciona-o ao final do array
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` cria um objeto temporário
                // e `proposals.push(...)`
                // adiciona-o ao final de `proposals`.
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // Dá ao `eleitor` o direito de votar neste boletim de voto.
        // Só pode ser chamado pelo `presidente`.
        function giveRightToVote(address voter) external {
            // Se o primeiro argumento de `require` for avaliado
            // como `false`, a execução termina e todas as
            // mudanças no estado e nos saldos de Ether
            // são revertidas.
            // Isto costumava consumir todo o gás em versões antigas do EVM, mas
            // não mais.
            // Muitas vezes é uma boa idéia usar `require` para verificar se
            // funções são chamadas corretamente.
            // Como um segundo argumento, você também pode fornecer uma
            // explicação sobre o que deu errado.
            require(
                msg.sender == chairperson,
                "Only chairperson can give right to vote."
            );
            require(
                !voters[voter].voted,
                "The voter already voted."
            );
            require(voters[voter].weight == 0);
            voters[voter].weight = 1;
        }

        /// Delega o seu voto ao eleitor `to`.
        function delegate(address to) external {
            // atribui uma referência
            Voter storage sender = voters[msg.sender];

            // faz verificações
            require(sender.weight != 0, "You have no right to vote");
            require(!sender.voted, "You already voted.");
            require(to != msg.sender, "Self-delegation is disallowed.");

            // Encaminha a delegação desde que
            // `to` também tenha sido delegado..
            // Em geral, tais loops são muito perigosos,
            // porque se eles rodarem por muito tempo, eles podem
            // precisar de mais gás do que o disponível num bloco.
            // Neste caso, a delegação não será executada,
            // mas noutras situações, estes loops podem
            // fazer com que um contrato fique completamente “preso”.
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // Encontramos um loop na delegação, o que não é permitido.
                require(to != msg.sender, "Found loop in delegation.");
            }

            Voter storage delegate_ = voters[to];

            // Os eleitores não podem delegar em contas que não podem votar.
            require(delegate_.weight >= 1);

            // Como `sender` é uma referência, isto
            // modifica `voters[msg.sender]`.
            sender.voted = true;
            sender.delegate = to;

            if (delegate_.voted) {
                // Se o delegado já votou,
                // adiciona diretamente ao número de votos
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // Se o delegado ainda não votou,
                // adicionar ao seu peso.
                delegate_.weight += sender.weight;
            }
        }

        /// Dar o seu voto (incluindo os votos que lhe foram delegados)
        /// à proposta `proposals[proposal].name`.
        function vote(uint proposal) external {
            Voter storage sender = voters[msg.sender];
            require(sender.weight != 0, "Has no right to vote");
            require(!sender.voted, "Already voted.");
            sender.voted = true;
            sender.vote = proposal;

            // Se `proposal` estiver fora do intervalo do array,
            // isto irá lançar um erro automaticamente e reverterá todas as
            // alterações.
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev Calcula a proposta vencedora tendo em conta todos os
        /// votos anteriores em consideração.
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // Chama a função winningProposal() para obter o índice
        // do vencedor contido no conjunto de propostas e depois
        // devolve o nome do vencedor.
        function winnerName() external view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


Possíveis melhorias
=====================

Atualmente, são necessárias muitas transacções para
atribuir os direitos de voto a todos os participantes.
Além disso, se duas ou mais propostas tiverem o mesmo
número de votos, ``winningProposal()`` não é capaz de registar um empate. 
Consegue pensar numa forma de resolver estes problemas?