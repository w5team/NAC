# Autenticação

Esse é o ponto mais vulnerável do NAC. A responsabilidae de autenticar um acesso ainda em ambiente aberto (não criptografado) e a fragilidade do tradicional par **login** & **senha** abrindo possibilidade de ataques quase inevitáveis!

**Recomendamos** que sua conexão esteja sendo feita sob **SSL** (https). Usamos uma criptografia RSS com chave publica/privada de, pelo menos, 2048 bits.

Imaginando que a API seja acessada no endereço "http[s]://site.com/nac/", o ciclo inicial de conexão é estabelecido da seguinte forma:

Chamada | Método | Cliente | Servidor
--------|--------|---------|---
site.com/nac|GET|Solicita|Envia a chave pública RSS 
site.com/nac/login|POST|Envia o login, senha e SKey criptografados com a chave pública|processa o login (retorna error/success + dados)

Vamos analizar o passo-a-passo do processo de login.

1. O Client solicita a chave pública ([GET] ```http[s]://site.com/nac/key``` ou ```http[s]://site.com/nac```);
2. Obtém o login e senha do usuário (formulário html, etc);
3. Gera uma chave aleatória para o próximo degrau de segurança (AES) com até 40 caracteres (SKey);
4. Empacota (RSS) esses três parâmetros, codificando em Base64, para transporte HTTP;
5. Envia o bloco de dados para o servidor ([POST] ```http[s]://site.com/nac/login```);
6. Recebe o resultado do processo de autenticação: **error** ou **success** (+dados).

O resultado pode ser um ```Error 400``` (vazio) ou uma string com o seguinte formato:

**TOKEN**: uma sequência de caracteres alpha numéricos que pode ser dividida em duas sub partes:

1. **TM**: um número que representa um inteiro único (time stamp + tick), no formato CAN*;

2. **TRASH**: uma sequencia aleatória de caracteres terminada em um dos seguintes caracteres: "%$#&Ç".

**BODY**: dados criptografados em AES em formato Base64 contendo a informação.

 formatado em JSON e, além da flag (error/succes) também retorna dados do processo de Login.

Error|Dados
---|---
True|o parametro "msg" contém a mensagem de erro (opcional)
False|vários parâmetros com informações do usuário e o **token** de AES para o próximo acesso
Void|opcionalmente, em caso de erro, o retorno pode ser vazio sob a indicação HTTP 400 (pedido incorreto)

Os parâmetros esperados do servidor, por padrão, são os seguintes:

Parametro|Local|Opcional|Função
---|---|---|---
TOKEN | Cabeçalho da mensagem retornada | Não | Identifica o usuário conectado em ambos os lados
SKEY | Corpo (criptografado AES) | Sim | Indica uma troca de SKEY forçada pelo servidor
ACTION | Corpo | Não | Indica a função a ser usada para tratar a solicitação (server side)
DATA | Corpo | Sim | Dados da aplicação (resposta) ou parâmetros da ACTION (requisição)


## Ping & Pong
Para manter o usuário com status de conectado e, consequentemente o **token** válido, é necessário fazer uma chamada ao servidor NAC em um determinado intervalo de tempo. Isso é necessário caso não haja nenhuma outra atividade com o servidor dentro desse período de tempo.

Por default, o tempo configurado é de *30 segundos* (BD user.**life**). Excedido, um novo processo de login deverá ser iniciado.

Chamada | Método | Cliente | Servidor
--------|--------|---------|---
ping|GET|solicita|retorna o **token** (+dados extras)

Os *"dados extras"* são opcionais e, dependendo da aplicação, para a notificação de novas mensagens e avisos do sistema.

O **ping** só é relevante nas aplicações do tipo *unilaterais*, onde a ação sempre parte do cliente para o servidor (web tradicional). Para os casos os outros casos é preferível uma conexão permanente com SOCKETS. Veja mais detalhes em [Tipo de conexão](https://github.com/w5team/NAC/blob/master/doc/tipoconexao.md).


