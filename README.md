```markdown
# Projeto Lógico de Banco de Dados para E-commerce

## Descrição do Projeto

Este projeto consiste na replicação da modelagem de um projeto lógico de banco de dados para um cenário de e-commerce. O objetivo é demonstrar a criação de um esquema de banco de dados relacional, incluindo a definição de tabelas, colunas, chaves primárias, chaves estrangeiras e constraints, além da implementação de relacionamentos presentes em modelos EER. Foram aplicados os refinamentos propostos no módulo de modelagem conceitual, como a distinção entre clientes Pessoa Física (PF) e Pessoa Jurídica (PJ), a possibilidade de um cliente ter múltiplas formas de pagamento e a inclusão de detalhes para a entrega (status e código de rastreio).

## Modelo Lógico

O modelo lógico do banco de dados para o e-commerce é composto pelas seguintes tabelas:

* **Cliente:** Contém informações gerais sobre os clientes (id, tipo - PF/PJ, endereço, telefone, email).
* **PessoaFisica:** Contém informações específicas para clientes PF (id do cliente, CPF, nome, data de nascimento), sendo uma especialização da tabela Cliente.
* **PessoaJuridica:** Contém informações específicas para clientes PJ (id do cliente, CNPJ, razão social, nome fantasia), sendo uma especialização da tabela Cliente.
* **FormaPagamento:** Armazena as formas de pagamento cadastradas por cada cliente (id, id do cliente, tipo de pagamento, detalhes).
* **Produto:** Contém informações sobre os produtos (id, nome, descrição, preço, estoque).
* **Categoria:** Armazena as categorias dos produtos (id, nome).
* **ProdutoCategoria:** Tabela de junção para o relacionamento muitos-para-muitos entre Produto e Categoria.
* **Pedido:** Registra os pedidos realizados pelos clientes (id, id do cliente, data do pedido, valor total).
* **ItemPedido:** Detalha os itens de cada pedido (id, id do pedido, id do produto, quantidade, preço unitário).
* **Entrega:** Contém informações sobre a entrega de cada pedido (id, id do pedido, status, código de rastreio).
* **Fornecedor:** Armazena informações sobre os fornecedores (id, nome, endereço, telefone, email).
* **FornecedorProduto:** Tabela de junção para o relacionamento muitos-para-muitos entre Fornecedor e Produto.

## Script SQL para Criação do Esquema

```sql
-- Criação da tabela Cliente
CREATE TABLE Cliente (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    tipo_cliente ENUM('PF', 'PJ') NOT NULL,
    endereco VARCHAR(255),
    telefone VARCHAR(20),
    email VARCHAR(100)
);

-- Criação da tabela PessoaFisica
CREATE TABLE PessoaFisica (
    id_cliente INT PRIMARY KEY,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    nome VARCHAR(100) NOT NULL,
    data_nascimento DATE,
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente)
);

-- Criação da tabela PessoaJuridica
CREATE TABLE PessoaJuridica (
    id_cliente INT PRIMARY KEY,
    cnpj VARCHAR(18) UNIQUE NOT NULL,
    razao_social VARCHAR(150) NOT NULL,
    nome_fantasia VARCHAR(100),
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente)
);

-- Criação da tabela FormaPagamento
CREATE TABLE FormaPagamento (
    id_forma_pagamento INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    tipo_pagamento VARCHAR(50) NOT NULL,
    detalhes_pagamento VARCHAR(255),
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente)
);

-- Criação da tabela Produto
CREATE TABLE Produto (
    id_produto INT PRIMARY KEY AUTO_INCREMENT,
    nome_produto VARCHAR(100) NOT NULL,
    descricao TEXT,
    preco DECIMAL(10, 2) NOT NULL,
    estoque INT NOT NULL DEFAULT 0
);

-- Criação da tabela Categoria
CREATE TABLE Categoria (
    id_categoria INT PRIMARY KEY AUTO_INCREMENT,
    nome_categoria VARCHAR(50) NOT NULL
);

-- Criação da tabela ProdutoCategoria (Relacionamento muitos-para-muitos)
CREATE TABLE ProdutoCategoria (
    id_produto INT,
    id_categoria INT,
    PRIMARY KEY (id_produto, id_categoria),
    FOREIGN KEY (id_produto) REFERENCES Produto(id_produto),
    FOREIGN KEY (id_categoria) REFERENCES Categoria(id_categoria)
);

-- Criação da tabela Pedido
CREATE TABLE Pedido (
    id_pedido INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valor_total DECIMAL(10, 2),
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente)
);

-- Criação da tabela ItemPedido
CREATE TABLE ItemPedido (
    id_item_pedido INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT NOT NULL,
    id_produto INT NOT NULL,
    quantidade INT NOT NULL DEFAULT 1,
    preco_unitario DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_pedido) REFERENCES Pedido(id_pedido),
    FOREIGN KEY (id_produto) REFERENCES Produto(id_produto)
);

-- Criação da tabela Entrega
CREATE TABLE Entrega (
    id_entrega INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT UNIQUE NOT NULL,
    status_entrega VARCHAR(50),
    codigo_rastreio VARCHAR(100),
    FOREIGN KEY (id_pedido) REFERENCES Pedido(id_pedido)
);

-- Criação da tabela Fornecedor
CREATE TABLE Fornecedor (
    id_fornecedor INT PRIMARY KEY AUTO_INCREMENT,
    nome_fornecedor VARCHAR(100) NOT NULL,
    endereco VARCHAR(255),
    telefone VARCHAR(20),
    email VARCHAR(100)
);

-- Criação da tabela FornecedorProduto (Relacionamento muitos-para-muitos)
CREATE TABLE FornecedorProduto (
    id_fornecedor INT,
    id_produto INT,
    PRIMARY KEY (id_fornecedor, id_produto),
    FOREIGN KEY (id_fornecedor) REFERENCES Fornecedor(id_fornecedor),
    FOREIGN KEY (id_produto) REFERENCES Produto(id_produto)
);
```

## Persistência de Dados para Testes (Exemplo)

```sql
-- Inserindo clientes PF
INSERT INTO Cliente (tipo_cliente, endereco, telefone, email) VALUES ('PF', 'Rua das Flores, 123', '11 99999-8888', 'maria@email.com');
INSERT INTO PessoaFisica (id_cliente, cpf, nome, data_nascimento) VALUES (LAST_INSERT_ID(), '123.456.789-00', 'Maria Silva', '1990-05-15');

INSERT INTO Cliente (tipo_cliente, endereco, telefone, email) VALUES ('PF', 'Avenida Brasil, 456', '21 88888-7777', 'joao@email.com');
INSERT INTO PessoaFisica (id_cliente, cpf, nome, data_nascimento) VALUES (LAST_INSERT_ID(), '987.654.321-11', 'João Pereira', '1985-12-20');

-- Inserindo clientes PJ
INSERT INTO Cliente (tipo_cliente, endereco, telefone, email) VALUES ('PJ', 'Rua da Empresa, 789', '31 77777-6666', 'empresa1@email.com');
INSERT INTO PessoaJuridica (id_cliente, cnpj, razao_social, nome_fantasia) VALUES (LAST_INSERT_ID(), '12.345.678/0001-90', 'Empresa Alfa Ltda.', 'Alfa');

-- Inserindo formas de pagamento
INSERT INTO FormaPagamento (id_cliente, tipo_pagamento, detalhes_pagamento) VALUES (1, 'Cartão de Crédito', 'Visa ****-****-****-1234');
INSERT INTO FormaPagamento (id_cliente, tipo_pagamento, detalhes_pagamento) VALUES (1, 'Boleto Bancário', null);
INSERT INTO FormaPagamento (id_cliente, tipo_pagamento, detalhes_pagamento) VALUES (2, 'Cartão de Débito', 'Mastercard ****-****-****-5678');
INSERT INTO FormaPagamento (id_cliente, tipo_pagamento, detalhes_pagamento) VALUES (3, 'Pix', 'chave@empresa1.com');

-- Inserindo produtos
INSERT INTO Produto (nome_produto, descricao, preco, estoque) VALUES ('Camiseta Algodão', 'Camiseta básica de algodão', 25.99, 100);
INSERT INTO Produto (nome_produto, descricao, preco, estoque) VALUES ('Calça Jeans', 'Calça jeans tradicional', 79.90, 50);
INSERT INTO Produto (nome_produto, descricao, preco, estoque) VALUES ('Tênis Esportivo', 'Tênis para corrida', 129.99, 75);

-- Inserindo categorias
INSERT INTO Categoria (nome_categoria) VALUES ('Vestuário');
INSERT INTO Categoria (nome_categoria) VALUES ('Calçados');

-- Inserindo relacionamento ProdutoCategoria
INSERT INTO ProdutoCategoria (id_produto, id_categoria) VALUES (1, 1);
INSERT INTO ProdutoCategoria (id_produto, id_categoria) VALUES (2, 1);
INSERT INTO ProdutoCategoria (id_produto, id_categoria) VALUES (3, 2);

-- Inserindo pedidos
INSERT INTO Pedido (id_cliente, valor_total) VALUES (1, 155.88);
INSERT INTO Pedido (id_cliente, valor_total) VALUES (2, 79.90);
INSERT INTO Pedido (id_cliente, valor_total) VALUES (3, 129.99);

-- Inserindo itens de pedido
INSERT INTO ItemPedido (id_pedido, id_produto, quantidade, preco_unitario) VALUES (1, 1, 2, 25.99);
INSERT INTO ItemPedido (id_pedido, id_produto, quantidade, preco_unitario) VALUES (1, 2, 1, 79.90);
INSERT INTO ItemPedido (id_pedido, id_produto, quantidade, preco_unitario) VALUES (2, 2, 1, 79.90);
INSERT INTO ItemPedido (id_pedido, id_produto, quantidade, preco_unitario) VALUES (3, 3, 1, 129.99);

-- Inserindo entregas
INSERT INTO Entrega (id_pedido, status_entrega, codigo_rastreio) VALUES (1, 'Em trânsito', 'BR123456789XX');
INSERT INTO Entrega (id_pedido, status_entrega, codigo_rastreio) VALUES (2, 'Entregue', 'BR987654321YY');
INSERT INTO Entrega (id_pedido, status_entrega, codigo_rastreio) VALUES (3, 'Processando', null);

-- Inserindo fornecedores
INSERT INTO Fornecedor (nome_fornecedor, endereco, telefone, email) VALUES ('Fornecedor A', 'Rua X, 10', '41 55555-4444', 'fornecedorA@email.com');
INSERT INTO Fornecedor (nome_fornecedor, endereco, telefone, email) VALUES ('Fornecedor B', 'Avenida Y, 20', '42 66666-5555', 'fornecedorB@email.com');

-- Inserindo relacionamento FornecedorProduto
INSERT INTO FornecedorProduto (id_fornecedor, id_produto) VALUES (1, 1);
INSERT INTO FornecedorProduto (id_fornecedor, id_produto) VALUES (1, 2);
INSERT INTO FornecedorProduto (id_fornecedor, id_produto) VALUES (2, 3);
```

## Queries SQL Mais Complexas

A seguir, são apresentadas algumas queries SQL mais complexas que demonstram a utilização de diferentes cláusulas para fornecer uma perspectiva mais aprofundada dos dados.

**Pergunta 1: Quantos pedidos foram feitos por cada cliente (mostrando nome do cliente)?**

```sql
SELECT
    c.nome,
    COUNT(p.id_pedido) AS total_pedidos
FROM
    PessoaFisica c
JOIN
    Cliente cl ON c.id_cliente = cl.id_cliente
LEFT JOIN
    Pedido p ON cl.id_cliente = p.id_cliente
GROUP BY
    c.nome
UNION
SELECT
    cj.nome_fantasia,
    COUNT(p.id_pedido) AS total_pedidos
FROM
    PessoaJuridica cj
JOIN
    Cliente cl ON cj.id_cliente = cl.id_cliente
LEFT JOIN
    Pedido p ON cl.id_cliente = p.id_cliente
GROUP BY
    cj.nome_fantasia
ORDER BY
    total_pedidos DESC;
```

**Pergunta 2: Algum vendedor também é fornecedor? (Considerando que não modelamos vendedores, vamos adaptar para verificar se algum nome de cliente PF é igual ao nome de algum fornecedor)**

```sql
SELECT
    pf.nome AS nome_cliente,
    f.nome_fornecedor
FROM
    PessoaFisica pf
JOIN
    Cliente c ON pf.id_cliente = c.id_cliente
JOIN
    Fornecedor f ON pf.nome = f.nome_fornecedor;
```

**Pergunta 3: Relação de produtos fornecedores e estoques (mostrando nome do produto, nome do fornecedor e estoque total do produto)?**

```sql
SELECT
    p.nome_produto,
    f.nome_fornecedor,
    p.estoque
FROM
    Produto p
JOIN
    FornecedorProduto fp ON p.id_produto = fp.id_produto
JOIN
    Fornecedor f ON fp.id_fornecedor = f.id_fornecedor
ORDER BY
    p.nome_produto, f.nome_fornecedor;
```

**Pergunta 4: Relação de nomes dos fornecedores e nomes dos produtos que eles fornecem (ordenado por nome do fornecedor)?**

```sql
SELECT
    f.nome_fornecedor,
    GROUP_CONCAT(DISTINCT p.nome_produto ORDER BY p.nome_produto SEPARATOR ', ') AS produtos_fornecidos
FROM
    Fornecedor f
JOIN
    FornecedorProduto fp ON f.id_fornecedor = fp.id_fornecedor
JOIN
    Produto p ON fp.id_produto = p.id_produto
GROUP BY
    f.nome_fornecedor
ORDER BY
    f.nome_fornecedor;
```

**Pergunta 5: Quais clientes (nome completo para PF e nome fantasia para PJ) fizeram pedidos com valor total acima de um certo valor (ex: R$100,00)?**

```sql
SELECT
    pf.nome AS nome_cliente,
    p.valor_total
FROM
    PessoaFisica pf
JOIN
    Cliente c ON pf.id_cliente = c.id_cliente
JOIN
    Pedido p ON c.id_cliente = p.id_cliente
WHERE
    p.valor_total > 100
UNION
SELECT
    pj.nome_fantasia AS nome_cliente,
    p.valor_total
FROM
    PessoaJuridica pj
JOIN
    Cliente c ON pj.id_cliente = c.id_cliente
JOIN
    Pedido p ON c.id_cliente = p.id_cliente
WHERE
    p.valor_total > 100
ORDER BY
    valor_total DESC;
```

**Pergunta 6: Quais produtos foram comprados no pedido com o código de rastreio específico ('BR123456789XX')?**

```sql
SELECT
    p.nome_produto,
    ip.quantidade,
    ip.preco_unitario
FROM
    Produto p
JOIN
    ItemPedido ip ON p.id_produto = ip.id_produto
JOIN
    Pedido pe ON ip.id_pedido = pe.id_pedido
JOIN
    Entrega e ON pe.id_pedido = e.id_pedido
WHERE
    e.codigo_rastreio = 'BR123456789XX';
```

## Considerações Finais

Este projeto demonstra a criação de um modelo lógico de banco de dados para um cenário de e-commerce, abordando os requisitos específicos de clientes PF e PJ, múltiplas formas de pagamento e informações de entrega. As queries SQL apresentadas ilustram como extrair informações relevantes do banco de dados utilizando diversas cláusulas e junções entre tabelas.
```
