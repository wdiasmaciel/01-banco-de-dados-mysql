<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="08-fornecedor-delete.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

# 9 - Projeto Empresa — Criação das Tabelas, Relacionamentos e Selects

Criação completa do banco de dados do projeto (Fornecedor, Produto, Identificação, Filial e Estoque), seguida de exemplos de `SELECT` entre tabelas.

---

## Criação das tabelas

### Removendo tabelas existentes

A ordem do `DROP TABLE` importa: 

* Como `Estoque` e `Identificação` possuem chaves estrangeiras que dependem de `Produto`, e `Produto` depende de `Fornecedor`, precisamos remover primeiro as tabelas "filhas" (que têm Chave Estrangeira, Foreign Key, FK) antes das tabelas "mães" (referenciadas). 

* Se tentássemos remover `Fornecedor` antes de `Produto`, o MySQL acusaria erro de dependência.

```sql
DROP TABLE IF EXISTS Estoque;
DROP TABLE IF EXISTS Identificacao;
DROP TABLE IF EXISTS Produto;
DROP TABLE IF EXISTS Filial;
DROP TABLE IF EXISTS Fornecedor;
```

### Criando as tabelas

A mesma lógica de dependência vale para a criação: 

* Criamos primeiro as tabelas sem FK (`Fornecedor`, `Filial`).

* Depois, `Produto` (que depende de `Fornecedor`).

* Depois, `Identificacao` (que depende de `Produto`).

* E, por fim, `Estoque` (que depende de `Produto` e `Filial`).


**Fornecedor**: 

Os campos (atributos, colunas) `nome` e `telefone` são `UNIQUE`, ou seja, não pode haver dois fornecedores com o mesmo nome ou o mesmo telefone cadastrados na tabela.

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

Segue a mesma estrutura de `Fornecedor`, ambas representam "entidades de endereço/contato" no projeto. 

Entretanto, `Filial` assume valores padrão (`default`), caso não sejam informados pelo usuário:

1. `cnpj`: '10101010000110'. 
2. `nome`: 'Filial Centro'.
3. `telefone`: '3140001010'.
4. `endereco`: 'Rua Tupis, 50 - Belo Horizonte/MG'.

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
DESCRIBE Fornecedor;
```

```sql
DESC Fornecedor;
```

**Produto**:

O campo `id` é a chave primária (usamos `AUTO_INCREMENT` para gerar o valor automaticamente).

O campo `cnpj_fornecedor` é `NOT NULL`, porque, pela regra do projeto, todo produto precisa ter um fornecedor, não faria sentido um produto "órfão". 

A cláusula `FOREIGN KEY` garante que só é possível cadastrar um produto apontando para um `cnpj` que já exista em `Fornecedor`.

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
DESCRIBE Fornecedor;
```

```sql
DESC Fornecedor;
```

**Identificacao** — aqui `id` é, ao mesmo tempo, chave primária **e** chave estrangeira para `Produto.id`. Isso implementa o relacionamento **1:1** descrito no projeto: cada produto tem exatamente uma identificação, e cada identificação pertence a exatamente um produto. Como `id` não é gerado automaticamente aqui (ele precisa ser igual ao `id` do produto correspondente), **não** usamos `AUTO_INCREMENT` nessa tabela.

```sql
CREATE TABLE Identificacao (
    id           INT NOT NULL,
    descricao    VARCHAR(200) NOT NULL,
    observacao   VARCHAR(200) NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (id) REFERENCES Produto(id)
);
```

**Estoque** — é a entidade-relacionamento entre `Produto` e `Filial` (resolve o relacionamento **N:N** entre elas: um produto pode estar em várias filiais, e uma filial vende vários produtos). Por isso, sua chave primária é **composta** por `id_produto` + `cnpj_filial`: juntos, eles identificam de forma única "o estoque de um produto específico em uma filial específica".

```sql
CREATE TABLE Estoque (
    id_produto   INT          NOT NULL,
    cnpj_filial  VARCHAR(14)  NOT NULL,
    preco        DECIMAL(10,2) NOT NULL,
    quantidade   INT          NOT NULL,
    validade     DATE         NOT NULL,
    PRIMARY KEY (id_produto, cnpj_filial),
    FOREIGN KEY (id_produto) REFERENCES Produto(id),
    FOREIGN KEY (cnpj_filial) REFERENCES Filial(cnpj)
);
```

---

## 2. Inserindo dados de exemplo

> **Observação:** a ordem dos `INSERT`s também respeita as dependências de chave estrangeira — não é possível inserir um `Produto` antes de seu `Fornecedor` existir, por exemplo.

### 2.1 Fornecedor

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

> Repare que todos os `nome` e `telefone` são diferentes entre si — condição obrigatória agora que essas colunas são `UNIQUE`. Se, em aula, alguém tentar inserir um telefone repetido, o MySQL vai recusar com um erro de violação de restrição única (`Duplicate entry ... for key`).

### 2.2 Filial

```sql
INSERT INTO Filial (cnpj, nome, telefone, endereco) VALUES
('10101010000110', 'Filial Centro',  '3140001010', 'Rua Tupis, 50 - Belo Horizonte/MG'),
('20202020000120', 'Filial Savassi', '3140002020', 'Rua Pernambuco, 400 - Belo Horizonte/MG');
```

### 2.3 Produto

Apenas 5 dos 8 fornecedores possuem produtos cadastrados neste exemplo — isso é **proposital**, para explorarmos mais adiante a diferença entre `INNER JOIN` e `LEFT/RIGHT JOIN` (fornecedores sem produtos associados).

```sql
INSERT INTO Produto (id, cnpj_fornecedor, nome) VALUES
(1, '11111111000101', 'Arroz Tipo 1 5kg'),
(2, '11111111000101', 'Feijao Carioca 1kg'),
(3, '22222222000102', 'Refrigerante Cola 2L'),
(4, '33333333000103', 'Macarrao Espaguete'),
(5, '44444444000104', 'Suco de Laranja 1L'),
(6, '77777777000107', 'Fone de Ouvido Bluetooth');
```

### 2.4 Identificacao

Cada produto acima recebe sua identificação correspondente (relação 1:1).

```sql
INSERT INTO Identificacao (id, descricao, observacao) VALUES
(1, 'Arroz branco tipo 1, pacote de 5kg',        'Produto de giro rápido, alta demanda'),
(2, 'Feijao carioca tipo 1, pacote de 1kg',      'Sensível à umidade, armazenar em local seco'),
(3, 'Refrigerante sabor cola, garrafa de 2 litros', 'Manter refrigerado apos aberto'),
(4, 'Macarrao tipo espaguete, pacote 500g',      'Sem glúten disponível sob encomenda'),
(5, 'Suco de laranja integral, garrafa 1 litro', 'Sem conservantes, validade curta'),
(6, 'Fone de ouvido bluetooth intra-auricular',  'Garantia de 12 meses do fabricante');
```

### 2.5 Estoque

Alguns produtos são vendidos em apenas uma filial; outros, em ambas — o que vai gerar dados interessantes para os exemplos de agregação da Seção 4.

```sql
INSERT INTO Estoque (id_produto, cnpj_filial, preco, quantidade, validade) VALUES
(1, '10101010000110', 22.90,  50, '2027-01-15'),
(1, '20202020000120', 23.50,  30, '2027-01-15'),
(2, '10101010000110',  8.90, 100, '2026-11-01'),
(3, '10101010000110',  6.50,  80, '2026-09-30'),
(3, '20202020000120',  6.90,  40, '2026-09-30'),
(4, '10101010000110', 12.50,  40, '2026-12-20'),
(5, '20202020000120',  9.90,  25, '2026-10-05'),
(6, '10101010000110', 199.90, 15, '2027-06-01'),
(6, '20202020000120', 209.90, 10, '2027-06-01');
```

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="08-fornecedor-delete.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
