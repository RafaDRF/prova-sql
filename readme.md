# Consultas Básicas

## Lista de funcionários ordenando pelo salário decrescente.

### Resposta: 

```sql
SELECT nome, salario 
    FROM vendedores 
    ORDER BY salario DESC;
```

## Lista de pedidos de vendas ordenado por data de emissão.

### Resposta: 

```sql
SELECT * FROM pedido 
    ORDER BY data_emissao;
```

## Valor de faturamento por cliente.

### Resposta: 

``` sql
SELECT clientes.id_cliente, razao_social, sum(valor_total) as faturamento 
    FROM clientes, pedido 
    WHERE clientes.id_cliente = pedido.id_cliente 
    GROUP BY clientes.id_cliente;
```

## Valor de faturamento por empresa.

### Resposta: 

```sql
SELECT empresa.id_empresa, razao_social, sum(valor_total) as faturamento 
    FROM empresa, pedido 
    WHERE empresa.id_empresa = pedido.id_empresa 
    GROUP BY empresa.id_empresa;
```

## Valor de faturamento por vendedor.

### Resposta: 

```sql
SELECT clientes.id_vendedor, sum(valor_total) as faturamento 
    FROM clientes, pedido 
    WHERE clientes.id_cliente = pedido.id_cliente 
    GROUP BY clientes.id_vendedor;
```

OBS.:  A tabela `PEDIDO` deveria conter uma coluna de `id_vendedor` ao invés da tabela `CLIENTES`, pois: 

1- Uma troca de vendedor (Ex: um vendedor foi desligado/está inativo) não deveria exigir qualquer alteração na tabela `CLIENTES`;

2- Queries simples ganham uma complexidade maior (Ex: faturamento por vendedor exige um JOIN) algo que não parece ser desejavel;

# Consultas de Junção

Escreva consultas SQL para obter as seguintes informações usando junções:
Utilizando o último pedido de cada cliente, formule o preço base do produto dentro da coluna chamada preco_base no seu select, conforme a seguinte regra:
- O preço base do produto deve ser o preço de venda do último pedido do cliente, porém deve obedecer a configuração de preço da tabela CONFIG_PRECO_PRODUTO.

Nesta mesma consulta, os seguintes campos deverão estar contidos: 
- Id do produto em questão;
- Descrição do produto;
- Id do cliente do pedido;
- Razão social do cliente;
- Id da empresa do pedido;
- Razão social da empresa;
- Id do vendedor do pedido;
- Nome do vendedor;
- Preço mínimo e máximo da configuração de preço;
- Preço base do produto conforme a regra.

### Resposta: 

```sql
SELECT distinct ON (pedido.id_cliente) produtos.id_produto, produtos.descricao, pedido.id_cliente, clientes.razao_social, pedido.id_empresa, empresa.razao_social, clientes.id_vendedor, vendedores.nome , config_preco_produto.preco_minimo, config_preco_produto.preco_maximo, valor_total as preco_base 
	FROM pedido
		INNER JOIN itens_pedido ON pedido.id_pedido = itens_pedido.id_pedido
		INNER JOIN produtos ON itens_pedido.id_produto = produtos.id_produto
		INNER JOIN clientes ON pedido.id_cliente = clientes.id_cliente
		INNER JOIN empresa ON pedido.id_empresa = empresa.id_empresa
		INNER JOIN vendedores ON clientes.id_vendedor = vendedores.id_vendedor
		INNER JOIN config_preco_produto ON itens_pedido.id_produto = config_preco_produto.id_produto
	WHERE pedido.id_pedido IN
		( -- pedidos que respeitam a config de produto 
			SELECT id_pedido FROM itens_pedido 
				INNER JOIN config_preco_produto ON itens_pedido.id_produto = config_preco_produto.id_produto
				WHERE itens_pedido.preco_praticado BETWEEN config_preco_produto.preco_minimo AND config_preco_produto.preco_maximo
		) 
ORDER BY pedido.id_cliente, pedido.data_emissao DESC;
```