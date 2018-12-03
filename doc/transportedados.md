# Transporte de Dados

Este documento descreve a forma de acesso ao NAC (servidor) e também os parâmetros de transporte dos dados entre cliente e servidor.

>Nesta primeira versão da documentação será considerada a **conexão unilateral**. A conexão **bilateral** será tratada no capítulo [Tipo de conexão](https://github.com/w5team/NAC/blob/master/doc/tipoconexao.md).

## URL
O formato da url de acesso ao servidor é o "friendly url" (url amigável). Considere que o seu servidor esteja no domínio ```https://server.nac``` e você queria pegar a chave pública para o login. O formato da requisição seria:

Chamada|Método|Url
---|---|---
getkey|GET|https://server.nac/key

Até aqui tudo normal. 

Mas agora vamos considerar que já estamos logados e os dados devem trafegar encriptografado. Logicamente, não podemos fazer o transporte dos dados diretamente no método GET. Esporíamos parte da informação.

#### Então, como seria?

O produto da criptografia é um bloco de bits codificado em Base64 para facilitar o transporte pelo padrão HTTP. Este bloco contém todos os dados necessários para que o servidor consiga trabalhar a requisição e retornar (também um bloco criptografado) os dados relevantes.

Então, nossa requisição GET (por exemplo) precisaria passar esse bloco como um parâmetro comum. Nomeamos esse parametro como "d" e a URL ficaria assim:

Chamada|Método|Url
-|-|-
search|GET|https://server.nac/produtos/search?d=BLOCOBASE64...

Aí você deve estar pensando: como vou passar os parâmetros "reais" da requisição?
