# Autenticação

Esse é o ponto mais vulnerável do NAC. A responsabilidae de autenticar um acesso ainda em ambiente aberto (não criptografado) e a fragilidade do tradicional par **login** & **senha** abre possibilidades de ataques quase inevitáveis!

Enquanto não descobrimos um método melhor, vamos esperar (pelo menos) que a sua conexão esteja sendo feita em SSL (https:// . . .). Vamos usar sobre essa camada uma criptografia (fraca) RSS com chave publica/privada.

Pegamos a chave pública acessando a NAC: ```https://site.com/getkey``` e obtemos uma chave pública padrão para embalar o par *login* & *senha*.

Antes, porém, criamos uma chave aleatória de no máximo 40 caracteres para o processo posterior - encriptação AES.

Agora, empacotados em RSS com a chave pública, enviamos os dados para a NAC pelo link ```https://site.com/login``` usando "POST" para transportar os dados.

Chamada | Método | Transporte
--------|--------|-----------
getkey|GET|envia a chave pública RSS 
login|POST|recebe o login, senha e token criptografados RSS
