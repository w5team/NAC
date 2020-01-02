# Transporte de Dados

Este documento descreve a forma de acesso ao NAC (servidor) e também os parâmetros de transporte dos dados entre cliente e servidor.

>Nesta primeira versão da documentação será considerada a **conexão unilateral**. A conexão **bilateral** será tratada no capítulo [Tipo de conexão](https://github.com/w5team/NAC/blob/master/doc/tipoconexao.md).

## URL
O formato da url de acesso ao servidor é o *"friendly url"*. Considere que o seu servidor esteja no domínio ```https://server.nac``` e você queria pegar a chave pública para o login. O formato da requisição seria:

Chamada|Método|Url
---|---|---
getkey|GET|https://server.nac/key

Até aqui tudo normal. 

Mas, agora vamos considerar que já estamos logados e os dados devam trafegar encriptografados. Logicamente, não podemos fazer o transporte dos dados diretamente no método GET. Esporíamos parte da informação.

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
	query: "texto da busca"
}
```
O processo que se segue são estes:

1. O JSON é convertido em texto *(JSON.stringify, em javascript)*;
2. O texto resultante é criptografado com **AES**, usando o **token** atual como chave;
3. O resultante é convertido em **Base64** (hexadecimal em texto);
4. A chamada pelo método GET é feita e o parâmetro **"d"** conterá o bloco resultante do ítem anterior.

O transporte ocorrerá pela internet (ou rede privada) e chegará ao servidor.

No servidor o processo será:

1. O parâmetro "d" é convertido em byte;
2. Haverá a decriptação **AES**, usando como chave o **token** atual;
3. O resultante (texto) é recodificado para **JSON** e o campo "query" é entregue ao **controlador** que processará a busca.

Depois de processado, o resultado da busca é enviado ao cliente por um processo reverso do que vimos até aqui.

### Podemos passar mais parâmetros?

Claro que sim!

Para um resultado paginado, passamos:

```
{
	query: "texto da busca",
	page: 3,
	rpp: 10
}
```
Com isso obtemos a página (page) 3 de um conjunto de 10 resultados por página (rpp = row per page).

O servidor respnderá com o seguinte JSON (depois de todo o processo de transporte visto anteriormente):

```
{
	query: "texto da busca",
	page: 3,
	pages: 20,
	rpp: 10,
	total: 200,
	title: {
		desc: "Descrição",
		qtd: "Quantidade",
		uni: "Preço Unitário",
		tot: "Total"
	},
	row: [
		{
			desc: "Caneta esferográfica azul",
			qtd: 4,
			uni: 1.50,
			tot: 6
		},{
			desc: "Lápis HB2 padrão",
			qtd: 10,
			uni: .50,
			tot: 5
		},{		
		... etc ...
		}
	]
}
```

**Conseguiu entender esse resultado?**

A primeira parte (linhas 2 à 6) trazem os parametros da pesquisa e o total de registros conseguidos. Podemos montar uma string para o usuário, assim:

```
Mostrando 10 registros de 200.
```
E podemos montar uma linha de links para as outras páginas da pesquisa, com o número da primeira página (sempre é 1), a página atual (page: 3) e o total de páginas (pages: 10). Até mesmo a string (o texto) da busca temos na linha 1 e podemos enviar para as próximas buscas de páginas.

```
<prev> 1 - 2 - [3] - 4 ... 9 - 10 <next>
```
Em seguida, na linha 7, temos o campo **"title"** que traz um objeto contendo os títulos das colunas referentes a cada campo a ser mostrado. Esses títulos podem vir na linguagem default ou podem ser definidos na requisição, usando-se o campo **"lang"** (se a sua aplicação suportar multilinguagem). 

```
{
	query: "texto da busca",
	page: 3,
	rpp: 10,
	lang: "pr-BR"
}
```
O campo que se segue, **"row"**, é um array de objetos. Um para cada linha do resultado (10 no caso) com os valores referenciados.

Você, agora, pode montar facilmente uma tabela para a exibição dos dados.

Considere que usará o javascript que, baseado no JSON recebido (acima), gerará o HTML para exibição em uma página de WEB (navegador de internet). Teríamos o seguinte:

```
<table>
	<thead>
		<tr>
			<th>Descrição</th>
			<th>Quantidade</th>
			<th>Preço Unitário</th>
			<th>Total</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Caneta esferográfica azul</td>
			<td>4</td>
			<td>1.50</td>
			<td>6</td>
		</tr>
			
			.... demais linhas ...
			
	</tbody>
	<tfoot>
		<tr>
			<td colspan="4">Página 3 de 10 <i>(mostrando 10 registros de 200)</i></td>
		</tr>
	</tfoot>
</table>
```
**Bem fácil, né?**

Agora é só criar a navegação (links) para acessar as outras páginas e pronto!

### Tem mais?

Você pode querer **ordenar** o resultado por um determinado campo ou lista de campos e em ordem crescente ou decrescente. Pode ainda querer buscar em um campo específico ou fazer **buscas concorrentes** (diferentes argumentos para diferentes campos da tabela).

Basta passar os parâmetros assim:
```
{
	query: 	
	{
		"descricao": "texto da busca",
		"total": {
			type: ">",
			value: 10
		}
	},
	page: 3,
	rpp: 10,
	lang: "pr-BR",
	order: 
	{
		dec: false,
		fields: "quantidade, total"
	}
}
```

.

.

---
--- coming soon ---
