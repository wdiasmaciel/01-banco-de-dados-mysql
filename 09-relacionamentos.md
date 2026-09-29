<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="08-fornecedor-delete.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="10-select-entre-tabelas.md">Próximo</a>
    </td>
  </tr>
</table>

# 9 - Projeto Empresa — Criação das Tabelas, Relacionamentos e Selects

Criação completa do banco de dados do projeto (Fornecedor, Produto, Identificação, Filial e Estoque), seguida de exemplos de `SELECT` entre tabelas.

---

## Criação das tabelas

### Removendo tabelas existentes

A ordem do `DROP TABLE` importa: 

1. Como `Estoque` e `Identificação` possuem chaves estrangeiras que dependem de `Produto`, e `Produto` depende de `Fornecedor`, precisamos remover primeiro as tabelas "filhas" (que têm Chave Estrangeira, Foreign Key, FK) antes das tabelas "mães" (referenciadas). 

2. Se tentássemos remover `Fornecedor` antes de `Produto`, o MySQL acusaria erro de dependência.

```sql
DROP TABLE IF EXISTS Estoque;
DROP TABLE IF EXISTS Identificacao;
DROP TABLE IF EXISTS Produto;
DROP TABLE IF EXISTS Filial;
DROP TABLE IF EXISTS Fornecedor;
```

### Criando as tabelas

A mesma lógica de dependência vale para a criação: 

1. Criamos primeiro as tabelas sem FK (`Fornecedor`, `Filial`).

2. Depois, `Produto` (que depende de `Fornecedor`).

3. Depois, `Identificacao` (que depende de `Produto`).

4. E, por fim, `Estoque` (que depende de `Produto` e `Filial`).


**Fornecedor**: 

1. Os campos (atributos, colunas) `nome` e `telefone` são `UNIQUE`, ou seja, não pode haver dois fornecedores com o mesmo nome ou o mesmo telefone cadastrados na tabela.

```sql
CREATE TABLE Fornecedor (
    cnpj      VARCHAR(14)  NOT NULL,
    nome      VARCHAR(100) NOT NULL UNIQUE,
    telefone  VARCHAR(15)  NOT NULL UNIQUE,
    endereco  VARCHAR(200) NOT NULL,
    PRIMARY KEY (cnpj)
);
```

Observe a estrutura da tabela, usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Fornecedor;
```

```sql
DESC Fornecedor;
```

**Filial**:

1. Segue a mesma estrutura de `Fornecedor`, ambas representam "entidades de endereço/contato" no projeto. 

2. Entretanto, `Filial` assume valores padrão (`default`) para os campos, caso algum não seja informado pelo usuário:

> a. `cnpj`: '10101010000110'. <br/>
> b. `nome`: 'Filial Centro'. <br/>
> c. `telefone`: '3140001010'. <br/>
> d. `endereco`: 'Rua Tupis, 50 - Belo Horizonte/MG'. <br/>

```sql
CREATE TABLE Filial (
    cnpj      VARCHAR(14)  NOT NULL        DEFAULT '10101010000110',
    nome      VARCHAR(100) NOT NULL UNIQUE DEFAULT 'Filial Centro',
    telefone  VARCHAR(15)  NOT NULL UNIQUE DEFAULT '3140001010',
    endereco  VARCHAR(200) NOT NULL        DEFAULT 'Rua Tupis, 50 - Belo Horizonte/MG',
    PRIMARY KEY (cnpj)
);
```

Observe a estrutura da tabela, usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Filial;
```

```sql
DESC Filial;
```

**Produto**:

1. O campo `id` é a chave primária (usamos `AUTO_INCREMENT` para gerar o valor automaticamente).

2. O campo `cnpj_fornecedor` é `NOT NULL`, porque, pela regra do projeto, todo produto precisa ter um fornecedor, não faria sentido um produto "órfão". 

3. A cláusula `FOREIGN KEY` garante que só é possível cadastrar um produto apontando para um `cnpj` que já exista em `Fornecedor`.

```sql
CREATE TABLE Produto (
    id                INT AUTO_INCREMENT,
    cnpj_fornecedor   VARCHAR(14)  NOT NULL,
    nome              VARCHAR(100) NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (cnpj_fornecedor) REFERENCES Fornecedor(cnpj)
);
```

Observe a estrutura da tabela, usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Produto;
```

```sql
DESC Produto;
```

**Identificacao**:

1. O campo `id` é, ao mesmo tempo, chave primária **e** chave estrangeira para `Produto.id`. 

2. Isso implementa o relacionamento **1:1** descrito no projeto: cada produto tem exatamente uma identificação e cada identificação pertence a exatamente um produto. 

3. Como `id` não é gerado automaticamente aqui (ele precisa ser igual ao `id` do produto correspondente), **não** usamos `AUTO_INCREMENT` nessa tabela.

```sql
CREATE TABLE Identificacao (
    id           INT NOT NULL,
    descricao    VARCHAR(200) NOT NULL,
    observacao   VARCHAR(200) NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (id) REFERENCES Produto(id)
);
```

Observe a estrutura da tabela, usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Identificacao;
```

```sql
DESC Identificacao;
```

**Estoque**:

1. É a **entidade-relacionamento** entre `Produto` e `Filial` (resolve o relacionamento **N:N** entre elas: um produto pode estar em várias filiais e uma filial vende vários produtos). 

2. Por isso, sua chave primária é **composta** por `id_produto` e `cnpj_filial`: juntos, eles identificam de forma única "o estoque de um produto específico em uma filial específica".

3. Os campos `preco` e `quantidade` assumem o valor padrão (`default`) zero.

4. A restrição (`constraint`) `CHECK` é usada com as instruções `INSERT` e `UPDATE`, para verificação de valores de um campo (atributo, coluna). Quando for de coluna: não pode fazer referência a outras colunas na mesma tabela. Quando for de tabela: pode fazer referência a outras colunas da mesma tabela. Não pode conter subconsultas, independentemente de ser de coluna ou de tabela.

```sql
CREATE TABLE Estoque (
    id_produto   INT           NOT NULL,
    cnpj_filial  VARCHAR(14)   NOT NULL,
    preco        DECIMAL(10,2) NOT NULL DEFAULT 0 CHECK (preco >= 0), -- Restrição (constraint) de coluna: não pode referenciar outras colunas da tabela (constraint de coluna: só pode referenciar a própria coluna, neste caso: preco).
    quantidade   INT           NOT NULL DEFAULT 0,
    validade     DATE          NOT NULL,
    PRIMARY KEY (id_produto, cnpj_filial),
    FOREIGN KEY (id_produto) REFERENCES Produto(id),
    FOREIGN KEY (cnpj_filial) REFERENCES Filial(cnpj),
    CONSTRAINT check_validade_minima CHECK (validade > '1900-01-01') -- Restrição (constraint) de tabela: poderia referenciar outras colunas da tabela (constraint de tabela: poderia referenciar várias colunas, aqui usa só validade).
);
```

Observe a estrutura da tabela, usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Estoque;
```

```sql
DESC Estoque;
```

---

## Inserindo dados de exemplo

> **OBS:** a ordem dos `INSERT`s também respeita as dependências de chave estrangeira: não é possível inserir um `Produto` antes de seu `Fornecedor` existir, por exemplo.

### Fornecedor

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG'),
('22222222000102', 'Comercial Beta S.A.',            '3132222222', 'Av. Brasil, 200 - Belo Horizonte/MG'),
('33333333000103', 'Gama Alimentos Ltda',            '3132223333', 'Rua da Bahia, 300 - Belo Horizonte/MG'),
('44444444000104', 'Delta Bebidas Ltda',             '3132224444', 'Av. Afonso Pena, 400 - Belo Horizonte/MG'),
('55555555000105', 'Epsilon Higiene e Limpeza Ltda', '3132225555', 'Rua Curitiba, 500 - Belo Horizonte/MG'),
('66666666000106', 'Zeta Papelaria ME',              '3132226666', 'Rua Rio de Janeiro, 600 - Belo Horizonte/MG'),
('77777777000107', 'Eta Eletrônicos Ltda',           '3132227777', 'Rua Curitiba, 700 - Belo Horizonte/MG'),
('88888888000108', 'Theta Móveis e Decoração Ltda',  '3132228888', 'Av. do Contorno, 800 - Belo Horizonte/MG');
```

> * Observe que todos os `nome` e `telefone` são diferentes entre si; condição obrigatória, pois essas colunas são `UNIQUE`. 
>
> * Se alguém tentar inserir um telefone repetido, o MySQL vai recusar com um erro de violação de restrição única (`Duplicate entry ... for key`).

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Fornecedor;
```

### Filial

1. O `insert` abaixo criará um registro com os valores padrões (`default`) dos campos (atributos, colunas) da tabela:

```sql
INSERT INTO Filial VALUES (cnpj, nome, telefone, endereco);
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Filial;
```

2. O `insert` abaixo criará registros cam valores diferentes dos valores padrões (`default`):

```sql
INSERT INTO Filial (cnpj, nome, telefone, endereco) VALUES
('20202020000120', 'Filial Savassi',   '3140002020', 'Rua Pernambuco, 400 - Belo Horizonte/MG'),
('30303030000130', 'Filial Contagem',  '3140003030', 'Av. João César de Oliveira, 1000 - Contagem/MG');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Filial;
```

### Produto

1. Apenas 5 dos 8 fornecedores possuem produtos cadastrados neste exemplo.

2. Isso é **proposital**, para explorarmos mais adiante a diferença entre `INNER JOIN` e `LEFT/RIGHT JOIN` (fornecedores sem produtos associados).

```sql
INSERT INTO Produto (id, cnpj_fornecedor, nome) VALUES
(1, '11111111000101', 'Arroz Tipo 1 5kg'),
(2, '11111111000101', 'Feijao Carioca 1kg'),
(3, '22222222000102', 'Refrigerante Cola 2L'),
(4, '33333333000103', 'Macarrao Espaguete'),
(5, '44444444000104', 'Suco de Laranja 1L'),
(6, '77777777000107', 'Fone de Ouvido Bluetooth');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Produto;
```

### Identificacao

1. Cada produto na tabela `Produto` recebe sua identificação correspondente (`relacionamento 1:1`).

```sql
INSERT INTO Identificacao (id, descricao, observacao) VALUES
(1, 'Arroz branco tipo 1, pacote de 5kg',        'Produto de giro rápido, alta demanda'),
(2, 'Feijao carioca tipo 1, pacote de 1kg',      'Sensível à umidade, armazenar em local seco'),
(3, 'Refrigerante sabor cola, garrafa de 2 litros', 'Manter refrigerado apos aberto'),
(4, 'Macarrao tipo espaguete, pacote 500g',      'Sem glúten disponível sob encomenda'),
(5, 'Suco de laranja integral, garrafa 1 litro', 'Sem conservantes, validade curta'),
(6, 'Fone de ouvido bluetooth intra-auricular',  'Garantia de 12 meses do fabricante');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Identificacao;
```

### Estoque

1. Alguns produtos são comercializados em apenas uma filial.

2. Outros, em mais de uma. 

3. Isso será explorado mais adiante nos exemplos de agregação.

```sql
INSERT INTO Estoque (id_produto, cnpj_filial, preco, quantidade, validade) VALUES
(1, '10101010000110',  22.90,  50, '2027-01-15'),
(1, '20202020000120',  23.50,  30, '2027-01-15'),
(2, '10101010000110',   8.90, 100, '2026-11-01'),
(3, '10101010000110',   6.50,  80, '2026-09-30'),
(3, '20202020000120',   6.90,  40, '2026-09-30'),
(4, '10101010000110',  12.50,  40, '2026-12-20'),
(5, '20202020000120',   9.90,  25, '2026-10-05'),
(6, '10101010000110', 199.90,  15, '2027-06-01'),
(6, '20202020000120', 209.90,  10, '2027-06-01');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Estoque;
```

4. Os produtos abaixo terão o preço padrão (`default`) zero.

```sql
INSERT INTO Estoque (id_produto, cnpj_filial, quantidade, validade) VALUES
(1, '30303030000130',  6, '2027-01-15'),
(3, '30303030000130',  7, '2026-09-30');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Estoque;
```

5. O produto abaixo terá a quantidade padrão (`default`) zero.

```sql
INSERT INTO Estoque (id_produto, cnpj_filial, preco, validade) VALUES
(5, '30303030000130',   9.90,  '2026-10-05');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Estoque;
```

6. O produto abaixo terá o preço e a quantidade padrão (`default`) zero.

```sql
INSERT INTO Estoque (id_produto, cnpj_filial, validade) VALUES
(6, '30303030000130', '2027-06-01');
```

Observe os dados inseridos na tabela:

```sql
SELECT * FROM Estoque;
```

---

## Exercício

1. Execute o comando abaixo:

```sql
INSERT INTO Filial (cnpj, nome, endereco) VALUES
('20202020000120', 'Filial Savassi', 'Rua Pernambuco, 400 - Belo Horizonte/MG');
```

&nbsp;&nbsp;&nbsp; O `insert` cria um registro com o mesmo telefone da filial central (valor padrão, `default`, do campo telefone)? 

&nbsp;&nbsp;&nbsp; Justifique sua resposta.


2. Apresente os itens do estoque cujo preço é zero.

3. Apresente os itens do estoque cuja quantidade é zero.

4. Apresente os itens do estoque cujo preço e quantidade são zero.

5. Apresente os dados da filial Savassi.

6. Apesente todos os produtos fornecidos pelo fornecedor com CNPJ '11111111000101'.

7. Apresente todos os produtos com data de validade '2026-09-30'.

8. Apresente os itens do estoque com validade posterior a '2026-09-30' e anterior a '2027-01-15'.

9. Apresente os itens do estoque com validade posterior a '2027-01-01'.

10. Apresente os itens do estoque com validade anterior a '2026-10-01'.

11. Quais identificações de produto possuem a expressão 'alta demanda'?

12. Quais identificações de produto são 'sob encomenda'?

13. Quais identificações de produto são 'sem glúten' ou 'sem conservante'.

14. Quais produtos são vendidos por quilograma (kg)?

15. Quais filiais não estão em Belo Horizonte (`NOT LIKE '%Belo Horizonte%'`)?

16. Atualize o preço de um dos itens no estoque de forma que o novo preço seja negativo. Comente a resposta retornada pelo MySQL.

17. Atualize a data de validade de um dos itens no estoque de forma que a nova validade seja anterior a '1900-01-01'. Comente a resposta retornada pelo MySQL.

18. Realize a inserção de um item no estoque com preço negativo. Comente a resposta retornada pelo MySQL.

19. Realize a inserção de um item no estoque com validade anterior a '1900-01-01'. Comente a resposta retornada pelo MySQL.

20. Insira um novo fornecedor no banco de dados. Insira os produtos desse novo fornecedor, com as respecitivas identificações. Insira os produtos desse novo fornecedor no estoque das filiais da empresa. Apresente os dados dos produtos do novo fornecedor.

21. Analise o código abaixo:

```sql
CREATE TABLE Promocao (
    id                INT AUTO_INCREMENT,
    data_inicio       DATE NOT NULL,
    data_fim          DATE NOT NULL,
    preco_normal      DECIMAL(10,2) NOT NULL,
    preco_promocional DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (id),

    -- Constraint de tabela referenciando duas colunas:
    CONSTRAINT check_periodo CHECK (data_fim >= data_inicio),
    CONSTRAINT check_preco   CHECK (preco_promo < preco_normal)
);
```

  - Estabeleça o relacionamento correto entre a tabela `Promocao`e a tabela `Estoque`.

  - Crie a tabela `Promocao` no banco de dados `empresa`.

  - Insira dados na tabela `Promocao`.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="08-fornecedor-delete.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="10-select-entre-tabelas.md">Próximo</a>
    </td>
  </tr>
</table>
