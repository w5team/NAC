# Transporte de Dados

Este documento descreve a forma de acesso ao NAC (servidor) e também os parâmetros de transporte dos dados entre cliente e servidor.

>Nesta primeira versão da documentação será considerada a **conexão unilateral**. A conexão **bilateral** será tratada no capítulo [Tipo de conexão](https://github.com/w5team/NAC/blob/master/doc/tipoconexao.md).

## URL
O formato da url de acesso ao servidor é o "friendly url" (url amigável). Considere que o seu servidor esteja no domínio ```https://server.nac``` e você queria pegar a chave pública para o login. O formato da requisição seria:

Chamada|Método|Url
---|---|---
getkey|GET|https://server.nac/key

Até aqui tudo normal. 

Mas agora vamos considerar que já estamos logados e os dados devem trafegar encriptografado. Logicamente, não podemos fazer o transporte dos dados diretamente no método GET. Esporíamos parte da informação (o usuário acessou tal recurso).

### Então, como seria?

O produto da criptografia é um bloco de bits codificado em Base64 para facilitar o transporte pelo padrão HTTP. Este bloco contém todos os dados necessários para que o servidor consiga trabalhar a requisição e retornar (também um bloco criptografado) os dados relevantes.

Então, nossa requisição GET (por exemplo) precisaria passar esse bloco como um parâmetro comum. Nomeamos esse parametro como "d" e a URL ficaria assim:

Chamada|Método|Url
-|-|-
search|GET|https://server.nac/produtos/search?d=BLOCOBASE64...

Aí você deve estar pensando: como vou passar os parâmetros "reais" da requisição?

Vamos considerar esse pedido de "busca" por produtos. Temos que passar, pelo menos um parâmetro para a aplicação: a query, ou seja, o texto que define a busca. Fazemos isso em um JSON.

```
{
	"query": "texto da busca"
}
```
O processo que se segue são estes:

1. O JSON é convertido em texto *(JSON.stringify, em javascript)*;
2. O texto resultante é criptografado AES usando-se o *token* atual como chave;
3. O resultante é convertido em Base64 (hexadecimal em texto);
4. A chamada pelo método GET é feita e o parâmetro "d" conterá o bloco resultante do ítem anterior.

O transporte ocorrerá pela internet (ou rede privada) e chegará ao servidor.

No servidor o processo será:

1. O parâmetro "d" é convertido em byte;
2. Haverá a decriptação AES, usando como chave o *token* atual;
3. O resultante (texto) é recodificado para JSON e o campo "query" é entregue ao controlador que processará a busca.

Depois de processado, o resultado da busca é enviado ao cliente por um processo reverso ao que vimos até aqui.
.

.

.

---
--- coming soon ---
