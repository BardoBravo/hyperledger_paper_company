# Paper Company HandsOn

Nesse HandsOn nós iremos:
* Criar uma network Fabric
* Ver a estrutura de um Smart Contract
* Trabalhar com uma **organização**, MagnetoCorp, para instalar e instanciar smart contracts
* Configurar **wallets e identidades** 
* Executar uma aplicação da MagnetoCorp **criando um paper comercial**
* Ver como uma segunda organização, **Digibank**, usa o Smart Contract em suas aplicações
* Atuando como **Digibank**, executar aplicação que **compra** e **recolhe** papéis comerciais

Para realizar esse HandsOn, é possível que vários **terminais** sejam abertos simultaneamente, para simular diferentes usuários

Vamos iniciar com uma network básica:
```
cd fabric-samples/basic-network
./start.sh
docker ps
```

## Trabalhando como MagnetoCorp

Abra um novo **terminal** para monitorar os processos da MagnetoCorp:
```
(magnetocorp admin)$ cd commercial-paper/organization/magnetocorp/configuration/cli/
(magnetocorp admin)$ ./monitordocker.sh net_basic
```
Esta janela exibirá o output dos containeres Docker, então vamos agora abrir um novo **terminal** para interagirmos com a Network como sendo MagnetoCorp.

Para fazê-lo, o administrador da MagnetoCorp usa comandos do Hyperledger Fabric **peer**
```
(magnetocorp admin)$ cd commercial-paper/organization/magnetocorp/configuration/cli/
(magnetocorp admin)$ docker-compose -f docker-compose.yml up -d cliMagnetoCorp
```
## Smart Contract
**issue**, **buy** e **redeem** são as funções-chave do smart contract da PaperNet. São usadas pelas aplicações para submeter transações sobre os papéis. Vamos examinar o Smart contract:
```
(magnetocorp developer)$ cd commercial-paper/organization/magnetocorp/contract
(magnetocorp developer)$ code .
```
Vamos analiar os arquivos **papercontract.js**, nosso principal smart contract!!!

## Instalar contrato
Antes de serem executados pelas aplicações, o smart contract precisa ser instalado nos peer nodes da PaperNet. Administradores da MagnetoCorp e Digibank podem instalar o **paper contract** em seus peers onde tem autorização:

```
(magnetocorp admin)$ cd contract

(magnetocorp admin)$ docker exec cliMagnetoCorp peer chaincode install -n papercontract -v 0 -p /opt/gopath/src/github.com/contract -l node
```

## Instanciar contrato
Uma vez instalado, o **papercontract** pode ser disponibilizado em diferentes **channels**. Nesse caso, dado que estamos usando uma network básica, vamos instanciar em somente uma channer: **mychannel**:
```
(magnetocorp admin)$ docker exec cliMagnetoCorp peer chaincode instantiate -n papercontract -v 0 -l node -c '{"Args":["org.papernet.commercialpaper:instantiate"]}' -C mychannel -P "AND ('Org1MSP.member')"
```
Veja o container de **papercontract** iniciado usando o comando **docker ps**:
```
(magnetocorp admin)$ docker ps
```

## Estrutura da aplicação
O Smart contract contido em **papercontract** is chamado pela aplicação **issue.js** da MagnetoCorp. **Isabela** usará essa aplicação para submeter trasação na ledger que cria o paper comercial **0001**.

Vamos olhar o arquivo **issue.js** em **commercial-paper/organization/magnetocorp/application**.

Entre na pasta e execute o comando **npm install**.

## Isabela - Criando um paper
Isabela está próxima de executar o contracto em **issue.js**, mas antes precisamos identificá-la. Portanto, abra um **novo terminal** e execute:
```
cd hl/fabric-samples/commercial-paper/organization/magnetocorp/application

node addToWallet.js

ls ../identity/user/isabella/wallet/User1@org1.example.com
```
Um novo usuário para Isabela foi criado!

## Aplicação de criação (Issue)
Isabela pode agora usar **issue.js** para criar o paper comercial **0001**:
```
node issue.js
```
A partir do log, pode-se ver que um paper **0001** no valor de **5M USD** foi criado - Uhu!

Vamos agora seguir o fluxo alterando para o **DigiBank**, que irá comprar nosso paper.

## Trabalhando como Digibank
Agora que o paper **0001** foi lançado pelo MagnetoCorp, vamos alterar o contexto para interagir com a PaperNet como empregados da DigiBank.

Primeiro, atuaremos como admnistrador que criará a configuração. **Balaji**, o usuário final, usará a aplicação **buy** da Digibank para comprar o paper **0001**, enviado-o para o próximo estágio.

Como referência, é importante denotar que usuários finais do DigiBank, apesar de terem sua própria lógica de aplicações, seguem as mesmas regras de negócio definidas no smart contract, o qual age sobre os dados da blokchain.

Então, vamos abrir um **novo terminal** representando o Digibank:
```
(digibank admin)$ cd commercial-paper/organization/digibank/configuration/cli/

(digibank admin)$ docker-compose -f docker-compose.yml up -d cliDigiBank

(digibank admin)$ docker ps
```
Verifique o container **Digibank** criado.

## Aplicações do Digibank
**Balaji** usa a aplicação **buy** para submeter uma transação que transfere a propriedade do paper **0001** da MagnetoCorp para o **Digibank**.

Vamos analisar o código do objeto **buy.js** encontrado em **/commercial-paper/organization/digibank/application/**.

## Executar aplicações como Digibank
Primeiro, vamos instalar as dependências para execução do **buy** e **redeem**:
```
(digibank admin)$ cd commercial-paper/organization/digibank/application/
(digibank admin)$ npm install
```

Em um **novo terminal**, vamos simular as operações do Balaji:
```
(balaji)$ cd commercial-paper/organization/digibank/application/
(balaji)$ node addToWallet.js
```

## Aplicação de Compra
**Balaji** pode agora submeter transações de compra de papéis. No terminal do **Balaji**, execute:
```
(balaji)$ node buy.js
```

## Aplicação de Resgate
A transação final no ciclo de vida do paper **0001** é o resgate do valor por parte do Digibank junto ao **MagnetoCorp**. Para isso, usamos a aplicação **redeem.js**:
```
(balaji)$ node redeem.js
```

