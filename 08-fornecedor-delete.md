# 08 - Fornecedor - Delete

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="07-fornecedor-update.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

--- 

# Tabela Fornecedor e o comando Delete

Explorar o comando `Delete` no MySQL.

---

# Banco de Dados — Tabela Fornecedor e o comando DELETE

O `DELETE` remove linhas **permanentemente**.

---

## 1. Criando a tabela

Como anteriormente, removemos qualquer versão anterior da tabela antes de recriá-la, para evitar erros ao reexecutar o script:

```sql
DROP TABLE IF EXISTS Fornecedor;
```

Recriamos a tabela `Fornecedor`, com os mesmos atributos do projeto: `cnpj` (chave primária), `nome`, `telefone` e `endereco`.

```sql
CREATE TABLE Fornecedor (
    cnpj      VARCHAR(14)  NOT NULL,
    nome      VARCHAR(100) NOT NULL,
    telefone  VARCHAR(15),
    endereco  VARCHAR(200),
    PRIMARY KEY (cnpj)
);
```

Observe a estrutura da tabela Fornecedor usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Fornecedor;
```

ou

```sql
DESC Fornecedor;
```

---

## 2. Inserindo os dados iniciais

Usamos o mesmo conjunto de fornecedores da aula anterior:

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG'),
('22222222000102', 'Comercial Beta S.A.',            '3132222222', 'Av. Brasil, 200 - Belo Horizonte/MG'),
('33333333000103', 'Gama Alimentos Ltda',            '3132223333', 'Rua da Bahia, 300 - Belo Horizonte/MG'),
('44444444000104', 'Delta Bebidas Ltda',             NULL,         'Av. Afonso Pena, 400 - Belo Horizonte/MG'),
('55555555000105', 'Epsilon Higiene e Limpeza Ltda', '3132225555', NULL),
('66666666000106', 'Zeta Papelaria ME',              '3132226666', 'Rua Rio de Janeiro, 600 - Belo Horizonte/MG'),
('77777777000107', 'Eta Eletrônicos Ltda',           '3132227777', 'Rua Curitiba, 700 - Belo Horizonte/MG'),
('88888888000108', 'Theta Móveis e Decoração Ltda',  '3132228888', 'Av. do Contorno, 800 - Belo Horizonte/MG');
```

Conferimos os dados antes de começar:

```sql
SELECT * FROM Fornecedor;
```

---

## 3. Explorando o comando DELETE

A sintaxe básica é:

```sql
DELETE FROM nome_da_tabela
WHERE condição;
```

Assim como no `UPDATE`, a cláusula `WHERE` é **opcional**, mas define quais linhas serão removidas. 

Sem `WHERE`, **todas as linhas da tabela são apagadas**. 

Diferentemente do `UPDATE` (em que um valor errado ainda pode ser corrigido), um `DELETE` malfeito **destrói os dados**, o que torna esse comando ainda mais sensível.

> **OBS:** antes de qualquer `DELETE`, hábitue-se, **por segurança**, a rodar primeiro um `SELECT` com a mesma condição do `WHERE`, para conferir exatamente quais linhas seriam afetadas antes de efetivamente apagá-las. Por exemplo, antes de `DELETE FROM Fornecedor WHERE cnpj = '...'`, rode `SELECT * FROM Fornecedor WHERE cnpj = '...'` primeiro.

---

### 3.1 Excluindo um único registro pela chave primária

O uso mais comum do `DELETE`: remover **uma** linha específica, identificada de forma inequívoca pela chave primária.

```sql
DELETE FROM Fornecedor
WHERE cnpj = '88888888000108';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.2 Excluindo com múltiplas condições (AND)

É possível combinar mais de uma condição no `WHERE`, tornando a exclusão mais restritiva.

Só será removido o registro que satisfizer **todas** as condições ao mesmo tempo.

```sql
DELETE FROM Fornecedor
WHERE nome LIKE '%Papelaria%'
  AND telefone IS NOT NULL;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.3 Excluindo com IN (lista de valores)

Assim como no `UPDATE`, o `IN` evita escrever vários `OR` quando queremos excluir um conjunto específico de registros de uma vez.

```sql
DELETE FROM Fornecedor
WHERE cnpj IN ('33333333000103', '55555555000105');
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.4 Excluindo com LIKE (padrão de texto)

Aqui removemos todo fornecedor cujo nome termine com "S.A.".

```sql
DELETE FROM Fornecedor
WHERE nome LIKE '%S.A.';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.5 Excluindo registros com campo NULL

Uma condição comum em rotinas de "limpeza de dados": remover cadastros incompletos, como fornecedores sem telefone cadastrado.

```sql
DELETE FROM Fornecedor
WHERE telefone IS NULL;
```

> **OBS:** `WHERE telefone = NULL` **não funciona** (retorna sempre falso). É obrigatório usar `IS NULL`.

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.6 Excluindo com função aplicada à coluna

Assim como no `UPDATE`, podemos usar funções dentro da condição do `DELETE`. 

Aqui, removemos fornecedores cujo nome tenha mais de 30 caracteres.

```sql
DELETE FROM Fornecedor
WHERE LENGTH(nome) > 30;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```
---

### 3.7 Excluindo com subquery no WHERE

Da mesma forma que no `UPDATE`, podemos basear a exclusão no resultado de uma subconsulta. 

Aqui, removemos os fornecedores cujo `cnpj` aparece em uma lista obtida de outra consulta (uma simulação de "fornecedores da região X", por exemplo, filtrando pelo endereço).

```sql
DELETE FROM Fornecedor
WHERE cnpj IN (
    SELECT cnpj FROM (
        SELECT cnpj FROM Fornecedor WHERE endereco LIKE '%Belo Horizonte%'
    ) AS sub_tabela
    WHERE sub_tabela.cnpj = '66666666000106'
);
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> Assim como no `UPDATE`, o MySQL não permite referenciar diretamente a mesma tabela que está sendo alterada dentro da subquery do `WHERE` — por isso a subquery precisa ser "embrulhada / empacotada" em uma tabela derivada (`AS sub_tabela`).

---

### 3.8 Excluindo com ORDER BY e LIMIT (recurso específico do MySQL)

Assim como no `UPDATE`, o MySQL permite combinar `ORDER BY` e `LIMIT` em um `DELETE`.

Útil para remover "o primeiro" registro segundo algum critério.

Exemplo: o fornecedor cujo nome vem primeiro em ordem alfabética.

```sql
DELETE FROM Fornecedor
ORDER BY nome ASC
LIMIT 1;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> Esse recurso **não é padrão SQL**. Não funciona da mesma forma em todos os Sistemas Gerenciadores de Bancos de Dados (SGBDs). Cada SGBD tem seu dialeto (dificulta portabilidade entre bancos).

---

### 3.9 DELETE sem WHERE x TRUNCATE TABLE

Um `DELETE` sem `WHERE` remove todas as linhas, mas ainda é gravado no log de transações linha a linha (o que permite `ROLLBACK` dentro de uma transação, como veremos a seguir) e reinicia contadores de auto incremento de forma diferente do `TRUNCATE`.

```sql
-- Remove todas as linhas, mas ainda é uma operação DML "linha a linha"
DELETE FROM Fornecedor;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

Inserir:

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG');
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```


```sql
-- Remove todas as linhas de forma mais rápida, mas é uma operação DDL:
-- não pode ser desfeita com ROLLBACK e reinicia o AUTO_INCREMENT
TRUNCATE TABLE Fornecedor;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> **OBS:** comandos **DML** (como `DELETE`), em geral, passam pelo log de transações e **DDL** (como `TRUNCATE`, que reconstrói a tabela), em geral, não passam pelo log de transações. Se executarmos um `TRUNCATE` sem querer, não é possível desfazer com `ROLLBACK`, na maioria dos casos. Por isso, é ainda mais perigoso que o `DELETE` sem `WHERE`.

---

### 3.10 Usando transações para testar DELETE com segurança

Antes de rodar exclusões "arriscadas", é uma ótima prática envolver o comando em uma transação, permitindo desfazer a operação com `ROLLBACK` caso algo saia errado.

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG');

-- Confira o resultado:
SELECT * FROM Fornecedor;

START TRANSACTION;

DELETE FROM Fornecedor
WHERE cnpj = '11111111000101';

-- Confira o resultado:
SELECT * FROM Fornecedor;

-- Se quiser desfazer, use:
-- ROLLBACK;

-- Confira o resultado:
SELECT * FROM Fornecedor;

-- Se estiver tudo certo, confirme a exclusão:
COMMIT;

-- Confira o resultado:
SELECT * FROM Fornecedor;
```

> **OBS:** esse exemplo demonstra o `ROLLBACK`. Excluindo um fornecedor "por engano" dentro da transação e depois desfazendo a operação, veremos o registro "voltar" à tabela. Isso ajuda exemplificar o conceito de transação.

---

### 3.11 DELETE e integridade referencial (chaves estrangeiras)

Se a tabela `Produto` for criada e tiver registros que referenciam um `cnpj` de `Fornecedor` através de uma chave estrangeira, tentar excluir esse fornecedor gera um **erro de violação de integridade referencial**, e não uma exclusão silenciosa:

```sql
-- Supondo que exista a FK Produto.cnpj_fornecedor -> Fornecedor.cnpj
DELETE FROM Fornecedor
WHERE cnpj = '77777777000107';
-- Erro esperado: Cannot delete or update a parent row:
-- a foreign key constraint fails
```

> **OBS:** as opções:
> 1. `ON DELETE RESTRICT`: padrão, impede a exclusão.
> 2. `ON DELETE CASCADE`: exclui também os produtos daquele fornecedor.
> 3. `ON DELETE SET NULL`: mantém o produto, mas "anula/null" a referência ao fornecedor. <br/>
> _É necessário avaliar qual é a opção mais adequada para cada projeto_.

---

### 3.12 DELETE "perigoso": sem WHERE e sem transação

Por fim, um `DELETE` sem `WHERE` remove **todas** as linhas da tabela, de forma irreversível se não estiver dentro de uma transação.

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG'),
('22222222000102', 'Comercial Beta S.A.',            '3132222222', 'Av. Brasil, 200 - Belo Horizonte/MG'),
('33333333000103', 'Gama Alimentos Ltda',            '3132223333', 'Rua da Bahia, 300 - Belo Horizonte/MG'),
('44444444000104', 'Delta Bebidas Ltda',             NULL,         'Av. Afonso Pena, 400 - Belo Horizonte/MG'),
('55555555000105', 'Epsilon Higiene e Limpeza Ltda', '3132225555', NULL),
('66666666000106', 'Zeta Papelaria ME',              '3132226666', 'Rua Rio de Janeiro, 600 - Belo Horizonte/MG'),
('77777777000107', 'Eta Eletrônicos Ltda',           '3132227777', 'Rua Curitiba, 700 - Belo Horizonte/MG'),
('88888888000108', 'Theta Móveis e Decoração Ltda',  '3132228888', 'Av. do Contorno, 800 - Belo Horizonte/MG');
```

Conferimos os dados antes de começar:

```sql
SELECT * FROM Fornecedor;
```

```sql
DELETE FROM Fornecedor;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

# Exercícios Práticos — Comando DELETE em MySQL

Sempre que possível, teste seus comandos `DELETE` dentro de uma transação (`START TRANSACTION` / `ROLLBACK`), para poder repetir o exercício sem precisar recriar a tabela do zero a cada tentativa.

---

## Exercício 1 — Continuação do projeto: Produto

```sql
DROP TABLE IF EXISTS Fornecedor;
```

```sql
CREATE TABLE Fornecedor (
    cnpj      VARCHAR(14)  NOT NULL,
    nome      VARCHAR(100) NOT NULL,
    telefone  VARCHAR(15),
    endereco  VARCHAR(200),
    PRIMARY KEY (cnpj)
);
```

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG'),
('22222222000102', 'Comercial Beta S.A.',            '3132222222', 'Av. Brasil, 200 - Belo Horizonte/MG'),
('33333333000103', 'Gama Alimentos Ltda',            '3132223333', 'Rua da Bahia, 300 - Belo Horizonte/MG'),
('44444444000104', 'Delta Bebidas Ltda',             NULL,         'Av. Afonso Pena, 400 - Belo Horizonte/MG'),
('55555555000105', 'Epsilon Higiene e Limpeza Ltda', '3132225555', NULL),
('66666666000106', 'Zeta Papelaria ME',              '3132226666', 'Rua Rio de Janeiro, 600 - Belo Horizonte/MG'),
('77777777000107', 'Eta Eletrônicos Ltda',           '3132227777', 'Rua Curitiba, 700 - Belo Horizonte/MG'),
('88888888000108', 'Theta Móveis e Decoração Ltda',  '3132228888', 'Av. do Contorno, 800 - Belo Horizonte/MG');
```

```sql
DROP TABLE IF EXISTS Produto;

CREATE TABLE Produto (
    id                INT AUTO_INCREMENT,
    cnpj_fornecedor   VARCHAR(14) NOT NULL,
    nome              VARCHAR(100) NOT NULL,
    PRIMARY KEY (id)
);

INSERT INTO Produto (id, cnpj_fornecedor, nome) VALUES
(1, '11111111000101', 'Arroz Tipo 1 5kg'),
(2, '11111111000101', 'Feijao carioca 1kg'),
(3, '22222222000102', 'Refrigerante cola 2l'),
(4, '33333333000103', 'Macarrao espaguete'),
(5, '44444444000104', 'Suco de laranja 1l'),
(6, '77777777000107', 'Fone de ouvido bluetooth'),
(7, '77777777000107', 'Carregador usb-c');
```

**Tarefas:**

a) Exclua o produto de `id = 5`.

b) Usando `IN`, exclua os produtos de `id` `6` e `7` em um único comando.

c) Usando `LIKE`, exclua todos os produtos cujo nome contenha a palavra `"cola"`.

d) Tente excluir todos os produtos do fornecedor de `cnpj` `'11111111000101'` usando `WHERE cnpj_fornecedor = '11111111000101'`. Quantas linhas foram afetadas? Isso seria arriscado em um sistema real? Por quê?

e) Envolva um `DELETE` de sua escolha em uma transação (`START TRANSACTION` ... `ROLLBACK`) e explique, com suas palavras, o que aconteceria com os dados se o `ROLLBACK` fosse trocado por `COMMIT`.

---

## Exercício 2 — Padaria: tabela Pedido

```sql
DROP TABLE IF EXISTS Pedido;

CREATE TABLE Pedido (
    id           INT AUTO_INCREMENT,
    cliente      VARCHAR(200) NOT NULL,
    item         VARCHAR(100) NOT NULL,
    quantidade   INT NOT NULL,
    status       VARCHAR(20) NOT NULL,  -- 'pendente', 'preparando', 'pronto', 'entregue'
    observacao   VARCHAR(200),
    PRIMARY KEY (id)
);

INSERT INTO Pedido (id, cliente, item, quantidade, status, observacao) VALUES
(1, 'Marcos Silva',     'pao frances',        20, 'pendente',   NULL),
(2, 'Ana Souza',        'bolo de chocolate',   1, 'pendente',   NULL),
(3, 'Carlos Pereira',   'croissant',           6, 'preparando', NULL),
(4, 'Juliana Lima',     'pao de queijo',      12, 'pronto',     NULL),
(5, 'Ana Souza',        'torta de morango',    1, 'pendente',   NULL),
(6, 'Roberto Alves',    'baguete',             3, 'entregue',   NULL);
```

**Tarefas:**

a) Exclua o pedido de `id = 6`.

b) Usando `IN`, exclua os pedidos `1` e `5` em um único comando.

c) Escreva um `DELETE` que remova todos os pedidos com `status = 'entregue'`.

d) Usando `ORDER BY` e `LIMIT`, exclua apenas o pedido com a **maior** `quantidade` da tabela.

e) Por que, no contexto de uma padaria, pode ser melhor manter um campo como `status = 'cancelado'` em vez de simplesmente fazer um `DELETE` do pedido cancelado? (Pesquise ou reflita sobre o conceito de "exclusão lógica" ou *soft delete*.)

---

## Exercício 3 — Agência de viagens: tabela Pacote

```sql
DROP TABLE IF EXISTS Pacote;

CREATE TABLE Pacote (
    id          INT AUTO_INCREMENT,
    destino     VARCHAR(100) NOT NULL,
    preco       DECIMAL(10,2) NOT NULL,
    vagas       INT NOT NULL,
    ativo       BOOLEAN NOT NULL DEFAULT 1,
    PRIMARY KEY (id)
);

INSERT INTO Pacote (id, destino, preco, vagas, ativo) VALUES
(1, 'Rio de Janeiro - RJ',  1200.00, 10, 1),
(2, 'Fernando de Noronha - PE', 4500.00,  5, 1),
(3, 'Gramado - RS',          980.00, 15, 1),
(4, 'Foz do Iguacu - PR',   1350.00,  8, 1),
(5, 'Bonito - MS',          2100.00,  0, 0),
(6, 'Salvador - BA',        1100.00, 12, 1);
```

**Tarefas:**

a) Exclua o pacote de `id = 5` (`'Bonito - MS'`).

b) Usando uma condição composta (`AND`), exclua todos os pacotes que estejam com `ativo = 0` **e** `vagas = 0` simultaneamente.

c) Usando subquery, exclua os pacotes cujo `preco` seja maior que R$ 2.000,00.

d) Envolva a exclusão do item (c) em uma transação, confira o resultado com `SELECT` antes de decidir, e só então execute `COMMIT` ou `ROLLBACK`.

e) Por que excluir fisicamente um pacote do banco pode ser um problema caso já existam reservas antigas de clientes associadas a esse pacote? Que tipo de restrição de chave estrangeira (`ON DELETE ...`) poderia impedir isso?

---

## Exercício 4 — Banco: tabela Conta

```sql
DROP TABLE IF EXISTS Conta;

CREATE TABLE Conta (
    numero        INT,
    titular       VARCHAR(200) NOT NULL,
    saldo         DECIMAL(12,2) NOT NULL,
    tipo          VARCHAR(20) NOT NULL,  -- 'corrente', 'poupanca'
    telefone      VARCHAR(15),
    PRIMARY KEY (numero)
);

INSERT INTO Conta (numero, titular, saldo, tipo, telefone) VALUES
(1001, 'Fernanda Costa',  2500.75, 'corrente', '3199990001'),
(1002, 'Bruno Martins',    120.00, 'poupanca', NULL),
(1003, 'Camila Rocha',   15300.40, 'corrente', '3199990003'),
(1004, 'Diego Fernandes',    0.00, 'corrente', NULL),
(1005, 'Eduarda Nunes',   8700.00, 'poupanca', '3199990005');
```

**Tarefas:**

a) Exclua a conta `1004`, que está com saldo zerado e sem movimentação (conta encerrada pelo cliente).

b) Usando `IS NULL`, exclua todas as contas que não têm telefone de contato cadastrado. Antes de rodar, escreva o `SELECT` equivalente para conferir quantas linhas seriam afetadas.

c) Escreva o comando `DELETE FROM Conta;` sem `WHERE`. Em seguida, explique por escrito por que esse comando jamais deveria ser executado em um banco de dados de produção de uma instituição financeira real.

d) Envolva o `DELETE` do item (a) em uma transação e pratique o `ROLLBACK`, restaurando a conta em seguida.

e) Pesquise/reflita: instituições financeiras reais realmente excluem fisicamente uma conta encerrada do banco de dados, ou apenas marcam algum campo (como `status = 'encerrada'`) mantendo o histórico? Por que isso é importante do ponto de vista legal e de auditoria?

---

## Exercício 5 — Desafio combinando conceitos (projeto Fornecedor/Filial/Estoque)

```sql
DROP TABLE IF EXISTS Estoque;
DROP TABLE IF EXISTS Filial;

CREATE TABLE Filial (
    cnpj      VARCHAR(14) NOT NULL,
    nome      VARCHAR(100) NOT NULL,
    telefone  VARCHAR(15),
    endereco  VARCHAR(200),
    PRIMARY KEY (cnpj)
);

CREATE TABLE Estoque (
    id_produto   INT NOT NULL,
    cnpj_filial  VARCHAR(14) NOT NULL,
    preco        DECIMAL(10,2) NOT NULL,
    quantidade   INT NOT NULL,
    validade     DATE,
    PRIMARY KEY (id_produto, cnpj_filial)
);

INSERT INTO Filial (cnpj, nome, telefone, endereco) VALUES
('10101010000110', 'Filial Centro',    '3140001010', 'Rua Tupis, 50 - Belo Horizonte/MG'),
('20202020000120', 'Filial Savassi',   '3140002020', 'Rua Pernambuco, 400 - Belo Horizonte/MG');

INSERT INTO Estoque (id_produto, cnpj_filial, preco, quantidade, validade) VALUES
(1, '10101010000110', 22.90,  50, '2027-01-15'),
(1, '20202020000120', 23.50,  30, '2027-01-15'),
(2, '10101010000110',  8.90, 100, '2026-11-01'),
(4, '10101010000110', 12.50,   0, '2026-12-20'),
(5, '20202020000120',  9.90,   0, '2026-10-05');
```

**Tarefas:**

a) Como a chave primária de `Estoque` é composta (`id_produto` + `cnpj_filial`), exclua **apenas** o registro do produto `id = 1` referente à filial `'20202020000120'`, sem afetar o mesmo produto na outra filial. Escreva a condição `WHERE` necessária.

b) Exclua todos os registros de `Estoque` com `quantidade = 0` (itens esgotados), em um único comando.

c) Usando subquery, exclua os registros de `Estoque` cujo produto (`id_produto`) pertence a um fornecedor específico — por exemplo, todos os produtos do fornecedor de `cnpj` `'11111111000101'` (você vai precisar relacionar `Estoque` com `Produto`, criada no Exercício 1, através de uma subconsulta).

d) Tente excluir a filial `'10101010000110'` da tabela `Filial` diretamente, sabendo que ela ainda possui registros associados em `Estoque` (supondo que exista uma chave estrangeira `Estoque.cnpj_filial -> Filial.cnpj`). O que deveria acontecer? Que opção de `ON DELETE` você configuraria para esse relacionamento, e por quê (`RESTRICT`, `CASCADE` ou `SET NULL`)?

e) Escreva, em uma única transação, uma sequência seguindo esta ordem: exclua primeiro os registros de `Estoque` relacionados à filial `'10101010000110'`, depois exclua a própria filial, confira o resultado com `SELECT` e só então decida entre `COMMIT` ou `ROLLBACK`. Explique por que a **ordem das exclusões** importa nesse cenário.


---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="07-fornecedor-update.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
