

# Como Contribuir para a Documentação em Português de Solidity

Obrigado por se interessar em contribuir para a tradução da documentação oficial de Solidity para o português! Este guia vai te ajudar a começar corretamente.

## Passos para Contribuir

1. **Fork o Repositório:**
   - Primeiro, faça um fork do repositório oficial para o seu próprio GitHub.

   ```bash
   git fork https://github.com/solidity-docs/pt-portuguese.git
   ```

2. **Clone o Repositório Forkado:**
   - Em seguida, clone o repositório forkado para o seu ambiente local.

   ```bash
   git clone <o repo que forkaste>
   ```

3. **Instale as Dependências:**
   - Navegue até o diretório do projeto clonado e instale as dependências usando `pnpm`.

   ```bash
   cd pt-portuguese
   pnpm install
   ```

4. **Crie uma Branch:**
   - Crie uma nova branch para o seu trabalho de tradução. Nomeie a branch usando as iniciais do seu primeiro e último nome, por exemplo, `fb-docs-translation`.

   ```bash
   git checkout -b fb-docs-translation
   ```

5. **Escolha uma Página e Traduza:**
   - Consulte a lista de páginas disponíveis para tradução. Escolha apenas uma página por vez. Traduza o conteúdo da página seguindo as diretrizes de tradução.

6. **Traduza Comentários:**
   - Se houver comentários no código ou no conteúdo da documentação que também precisam ser traduzidos, faça a tradução desses comentários para garantir que tudo esteja acessível e compreensível para os falantes de português.

7. **Teste e Verifique o Projeto:**
   - Após fazer suas alterações, é importante testar e verificar se tudo está funcionando corretamente. Aqui está a seção "Começando" traduzida para o português:

   ```md
   ## Começando

   Primeiro, inicie o servidor de desenvolvimento:

   ```bash
   npm run dev
   # ou
   yarn dev
   # ou
   pnpm dev
   # ou
   bun dev
   ```

   Abra [http://localhost:3000](http://localhost:3000) no seu navegador para ver o resultado.



8. **Submeta um Pull Request (PR):**
   - Após concluir a tradução e testar o projeto, faça commit das suas mudanças, envie para a branch que você criou e submeta um PR para revisão.

   ```bash
   git add .
   git commit -m "Tradução da página XYZ para o português"
   git push origin fb-docs-translation
   ```

9. **Envie o PR:**
   - No GitHub, envie um PR da sua branch para o repositório original, descrevendo brevemente as mudanças que você fez.

## Diretrizes de Tradução

- **Consulte o Glossário e o Guia de Estilo:**
  - Antes de iniciar, familiarize-se com o glossário e o guia de estilo. Eles contêm termos técnicos específicos de Solidity/Ethereum e instruções sobre o tom e estilo da tradução.

- **Rapidez nas Traduções:**
  - Se você não puder continuar a tradução, avise os responsáveis para que a página possa ser atribuída a outra pessoa.
  
  - Traduzir comentarios.
 ``` 
                // SPDX-License-Identifier: GPL-3.0
                pragma solidity ^0.8.26;

                //Isso só será compilado via IR
                contract Coin {
                  ....
                }
   ```
## Recursos Úteis
- [Nextra](https://nextra.vercel.app/) - Ferramenta utilizada para gerar e gerenciar a documentação.
- [Glossário Técnico de Solidity (se aplicável)](link-do-glossário)

```

Agora o guia inclui um link para o Nextra, que pode ajudar os colaboradores a entender como a documentação é gerada e gerenciada. Se precisar de mais ajustes, é só avisar!