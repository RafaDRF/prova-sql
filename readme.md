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

2- Queries simples ganham uma complexidade maior, faturamento por vendedor exige um JOIN, algo que não parece ser desejavel;

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

# Estrutura de Tabelas

## Criação das Tabelas
```sql
CREATE TABLE EMPRESA
(
    id_empresa   SERIAL PRIMARY KEY,
    razao_social VARCHAR(255) NOT NULL,
    inativo      BOOLEAN      NOT NULL
);

CREATE TABLE PRODUTOS
(
    id_produto SERIAL PRIMARY KEY,
    descricao  VARCHAR(255) NOT NULL,
    inativo    BOOLEAN      NOT NULL
);

CREATE TABLE VENDEDORES
(
    id_vendedor   SERIAL PRIMARY KEY,
    nome          VARCHAR(255)   NOT NULL,
    cargo         VARCHAR(100)   NOT NULL,
    salario       DECIMAL(10, 2) NOT NULL,
    data_admissao DATE           NOT NULL,
    inativo       BOOLEAN        NOT NULL
);

CREATE TABLE CONFIG_PRECO_PRODUTO
(
    id_config_preco_produto SERIAL PRIMARY KEY,
    id_vendedor             INTEGER REFERENCES VENDEDORES (id_vendedor),
    id_empresa              INTEGER REFERENCES EMPRESA (id_empresa),
    id_produto              INTEGER REFERENCES PRODUTOS (id_produto),
    preco_minimo            DECIMAL(10, 2) NOT NULL,
    preco_maximo            DECIMAL(10, 2) NOT NULL
);

CREATE TABLE CLIENTES
(
    id_cliente    SERIAL PRIMARY KEY,
    razao_social  VARCHAR(255) NOT NULL,
    data_cadastro DATE         NOT NULL,
    id_vendedor   INTEGER REFERENCES VENDEDORES (id_vendedor),
    id_empresa    INTEGER REFERENCES EMPRESA (id_empresa),
    inativo       BOOLEAN      NOT NULL
);

CREATE TABLE PEDIDO
(
    id_pedido    SERIAL PRIMARY KEY,
    id_empresa   INTEGER REFERENCES EMPRESA (id_empresa),
    id_cliente   INTEGER REFERENCES CLIENTES (id_cliente),
    valor_total  DECIMAL(10, 2) NOT NULL,
    data_emissao DATE           NOT NULL,
    situacao     VARCHAR(50)    NOT NULL
);

CREATE TABLE ITENS_PEDIDO
(
    id_item_pedido  SERIAL PRIMARY KEY,
    id_pedido       INTEGER REFERENCES PEDIDO (id_pedido),
    id_produto      INTEGER REFERENCES PRODUTOS (id_produto),
    preco_praticado DECIMAL(10, 2) NOT NULL,
    quantidade      INTEGER        NOT NULL
);
```

## Inserção de Dados Fictícios

```sql
INSERT INTO EMPRESA (razao_social, inativo)
VALUES ('Empresa A', FALSE),
       ('Empresa B', FALSE);

INSERT INTO PRODUTOS (descricao, inativo)
VALUES ('Produto 1', FALSE),
       ('Produto 2', FALSE);

INSERT INTO VENDEDORES (nome, cargo, salario, data_admissao, inativo)
VALUES ('Vendedor 1', 'Cargo 1', 5000.00, '2022-01-01', FALSE),
       ('Vendedor 2', 'Cargo 2', 6000.00, '2022-02-01', FALSE);

INSERT INTO CLIENTES (razao_social, data_cadastro, id_vendedor, id_empresa, inativo)
VALUES ('Cliente A', '2022-01-15', 1, 1, FALSE),
       ('Cliente B', '2022-02-01', 2, 2, FALSE);

INSERT INTO CONFIG_PRECO_PRODUTO (id_vendedor, id_empresa, id_produto, preco_minimo, preco_maximo)
VALUES (1, 1, 1, 50.00, 100.00),
       (2, 2, 2, 60.00, 120.00);

-- Inserção de 20 Pedidos Aleatórios
INSERT INTO PEDIDO (id_pedido, id_empresa, id_cliente, valor_total, data_emissao, situacao)
VALUES (1, 1, 1, 120.00, '2022-03-03', 'Fechado'),
       (2, 1, 2, 45.50, '2022-03-04', 'Aberto'),
       (3, 2, 1, 80.00, '2022-03-05', 'Fechado'),
       (4, 2, 2, 60.25, '2022-03-06', 'Fechado'),
       (5, 1, 1, 35.75, '2022-03-07', 'Aberto'),
       (6, 2, 2, 55.50, '2022-03-08', 'Aberto'),
       (7, 1, 1, 95.25, '2022-03-09', 'Aberto'),
       (8, 2, 2, 42.00, '2022-03-10', 'Fechado'),
       (9, 1, 1, 75.50, '2022-03-11', 'Fechado'),
       (10, 1, 2, 60.00, '2022-03-12', 'Aberto'),
       (11, 2, 1, 110.75, '2022-03-13', 'Fechado'),
       (12, 2, 2, 38.25, '2022-03-14', 'Fechado'),
       (13, 1, 1, 88.50, '2022-03-15', 'Aberto'),
       (14, 1, 2, 70.00, '2022-03-16', 'Aberto'),
       (15, 2, 1, 50.25, '2022-03-17', 'Aberto'),
       (16, 2, 2, 65.75, '2022-03-18', 'Fechado'),
       (17, 1, 1, 42.00, '2022-03-19', 'Fechado'),
       (18, 1, 2, 90.50, '2022-03-20', 'Aberto'),
       (19, 2, 1, 55.25, '2022-03-21', 'Aberto'),
       (20, 2, 2, 78.00, '2022-03-22', 'Fechado');


-- Inserção de Itens para os 20 Pedidos Aleatórios
INSERT INTO ITENS_PEDIDO (id_pedido, id_produto, preco_praticado, quantidade)
VALUES
    -- Itens para o Pedido 1
    (1, 1, 75.00, 2),

    -- Itens para o Pedido 2
    (2, 2, 90.00, 1),

    -- Itens para o Pedido 3
    (3, 1, 55.00, 3),
    (3, 2, 65.50, 2),

    -- Itens para o Pedido 4
    (4, 2, 30.25, 1),
    (4, 1, 50.75, 4),

    -- Itens para o Pedido 5
    (5, 1, 20.50, 2),

    -- Itens para o Pedido 6
    (6, 2, 45.25, 3),
    (6, 1, 10.00, 1),

    -- Itens para o Pedido 7
    (7, 1, 75.00, 5),

    -- Itens para o Pedido 8
    (8, 2, 20.50, 2),
    (8, 1, 21.50, 1),

    -- Itens para o Pedido 9
    (9, 1, 30.00, 2),
    (9, 2, 45.50, 3),

    -- Itens para o Pedido 10
    (10, 2, 30.25, 1),
    (10, 1, 30.75, 4),

    -- Itens para o Pedido 11
    (11, 1, 45.50, 3),

    -- Itens para o Pedido 12
    (12, 2, 25.25, 2),
    (12, 1, 40.00, 1),

    -- Itens para o Pedido 13
    (13, 1, 55.75, 4),

    -- Itens para o Pedido 14
    (14, 2, 30.00, 2),
    (14, 1, 40.50, 1),

    -- Itens para o Pedido 15
    (15, 1, 35.50, 3),

    -- Itens para o Pedido 16
    (16, 2, 15.25, 2),
    (16, 1, 35.00, 1),

    -- Itens para o Pedido 17
    (17, 1, 50.00, 5),

    -- Itens para o Pedido 18
    (18, 2, 25.50, 2),
    (18, 1, 40.25, 1),

    -- Itens para o Pedido 19
    (19, 1, 30.75, 3),
    (19, 2, 59.50, 2),

    -- Itens para o Pedido 20
    (20, 2, 40.00, 1),
    (20, 1, 38.50, 4);
```