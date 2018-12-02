# Autenticação

Esse é o ponto mais vulnerável do NAC. A responsabilidae de autenticar um acesso ainda em ambiente aberto (não criptografado) e a fragilidade do tradicional par **login** & **senha** abre possibilidades de ataques quase inevitáveis!

Enquanto não descobrimos um método melhor, vamos esperar (pelo menos) que a sua conexão esteja sendo feita em SSL (https:// . . .). Vamos usar sobre essa camada uma criptografia (fraca) RSS com chave publica/privada.


Chamada | Método | Cliente | Servidor
--------|--------|---------|---
getkey|GET|solicita|envia a chave pública RSS 
login|POST|envia o login, senha e token criptografados com a chave pública|processa o login e retorna error/success (+ dados)

Vamos analizar o passo-a-passo do processo de login.

1. O Client solicita a chave pública (getkey);
2. Obtém o login e senha do usuário;
3. Gera uma chave aleatória para o próximo degrau de segurança (AES) com até 40 caracteres (token);
4. Empacota (RSS) esses três parâmetros, codificando em Base64, para transporte HTTP;
5. Envia usando método POST (login);
6. Recebe o resultado do processo de autenticação: **error** ou **success**.

O resultado é formatado em JSON e, além da flag (error/succes) também retorna dados do processo de login.

Error|Dados
---|---
True|o parametro "msg" contém a mensagem de erro
False|vários parâmetros com informações do usuário e o **token** de AES para o próximo acesso

Os parâmetros com as informações do usuário, por padrão, são os seguintes:

* User ID;
* User Name;
* User Level.

Outros parâmetros podem ser inseridos conforme as necessidades específicas do projeto desenvolvido ou solicitando diretamente a API do servidor, em chamada posterior.
