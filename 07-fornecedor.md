# 07 - Fornecedor

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="06-fornecedor.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="08-produto.md">Próximo</a>
    </td>
  </tr>
</table>

--- 

# Tabela Fornecedor e o comando UPDATE

Explorar o comando `UPDATE` no MySQL.

---

## 1. Criando a tabela

Antes de popular a tabela, removermos uma versão anterior dela (caso exista), para evitar erros de "tabela já existe" ao reexecutar o script durante a aula:

```sql
DROP TABLE IF EXISTS Fornecedor;
```

Em seguida, criamos a tabela `Fornecedor` com os atributos: `cnpj` (chave primária), `nome`, `telefone` e `endereco`.

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

> **OBS:** por que `cnpj` foi definido como `VARCHAR` e não como `INT`? (Dica: CNPJ pode ter zeros à esquerda e não é usado em operações aritméticas, então não faz sentido tratá-lo como número.)

---

## 2. Inserindo os dados iniciais

Os `INSERT`s abaixo populam a tabela com 8 fornecedores fictícios. Note que alguns registros já nascem propositalmente com `telefone` ou `endereco` em `NULL`, para que possamos explorar isso nos exemplos de `UPDATE`.

```sql
INSERT INTO Fornecedor (cnpj, nome, telefone, endereco) VALUES
('11111111000101', 'Distribuidora Alfa Ltda',       '3132221111', 'Rua das Flores, 100 - Belo Horizonte/MG'),
('22222222000102', 'Comercial Beta S.A.',            '3132222222', 'Av. Brasil, 200 - Belo Horizonte/MG'),
('33333333000103', 'Gama Alimentos Ltda',            '3132223333', 'Rua da Bahia, 300 - Belo Horizonte/MG'),
('44444444000104', 'Delta Bebidas Ltda',             NULL,         'Av. Afonso Pena, 400 - Belo Horizonte/MG'),
('55555555000105', 'Epsilon Higiene e Limpeza Ltda', '3132225555', NULL),
('66666666000106', 'Zeta Papelaria ME',              '3132226666', 'Rua Rio de Janeiro, 600 - Belo Horizonte/MG'),
('77777777000107', 'Eta Eletrônicos Ltda',           '3132227777', 'Rua Curitiba, 700 - Belo Horizonte/MG'),
('88888888000108', 'Sigma Móveis e Decoração Ltda',  '3132228888', 'Av. do Contorno, 800 - Belo Horizonte/MG');
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

## 3. Explorando o comando UPDATE

A sintaxe básica do `UPDATE` é:

```sql
UPDATE nome_da_tabela
SET coluna1 = valor1, coluna2 = valor2, ...
WHERE condição;
```

A cláusula `WHERE` é **opcional**, mas fundamental: é ela quem decide *quais linhas* serão alteradas. 

Sem `WHERE`, o `UPDATE` afeta **todas as linhas da tabela**. 

Os exemplos a seguir exploram essa e outras variações, do caso mais simples ao mais avançado.

---

### 3.1 Atualizando um único atributo de um único registro

O uso mais comum do `UPDATE`: alterar **um** atributo de **uma** linha específica, identificada pela chave primária.

```sql
UPDATE Fornecedor
SET telefone = '3133331111'
WHERE cnpj = '11111111000101';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```
---

### 3.2 Atualizando vários atributos ao mesmo tempo

É possível alterar mais de uma coluna em um único comando, separando as atribuições por vírgula dentro do `SET`.

```sql
UPDATE Fornecedor
SET telefone = '3133332222',
    endereco = 'Av. Brasil, 250 - Belo Horizonte/MG'
WHERE cnpj = '22222222000102';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.3 Preenchendo um campo que estava NULL

O `UPDATE` também é usado para completar informações que faltavam. 

Aqui, o fornecedor Delta Bebidas foi cadastrado sem telefone (`NULL`) e agora recebe um valor.

```sql
UPDATE Fornecedor
SET telefone = '3133334444'
WHERE cnpj = '44444444000104';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.4 Atribuindo NULL a um atributo

O oposto também é possível: "apagar" o valor de uma coluna específica atribuindo `NULL` a ela, sem excluir a linha inteira.

```sql
UPDATE Fornecedor
SET telefone = NULL
WHERE cnpj = '66666666000106';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> **OBS:** qual a diferença entre um campo `NULL` e um campo com string vazia (`''`)? Ambos representam "ausência de valor"? Não são a mesma coisa. `NULL` representa a ausência de informação. String vazia (`''`) é um valor concreto e conhecido: sabemos que o campo existe e, deliberadamente, não tem caracteres. É um dado, só que "vazio".

---

### 3.5 Condição com LIKE e uso de CONCAT

O operador `LIKE` permite comparar textos por padrão (usando `%` como curinga). 

Combinado com a função `CONCAT`, conseguimos atualizar uma coluna a partir do seu próprio valor atual. 

Aqui, marcamos o endereço de todos os fornecedores cujo nome contenha "Ltda".

```sql
UPDATE Fornecedor
SET endereco = CONCAT('[PJ] ', endereco)
WHERE nome LIKE '%Ltda%'
  AND endereco IS NOT NULL;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> **OBS:** a condição `AND endereco IS NOT NULL` é necessária porque `CONCAT` com um valor `NULL` resulta em `NULL`.

---

### 3.6 Condição com IN (lista de valores)

Quando queremos aplicar a mesma alteração a um conjunto específico de registros, o operador `IN` evita escrever vários `OR`.

```sql
UPDATE Fornecedor
SET telefone = '3130000000'
WHERE cnpj IN ('33333333000103', '77777777000107');
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.7 Usando funções de string no UPDATE

Funções como `UPPER()`, `LOWER()` ou `TRIM()` podem ser aplicadas diretamente no `SET` para transformar o valor de uma coluna.

```sql
UPDATE Fornecedor
SET nome = UPPER(nome)
WHERE cnpj = '55555555000105';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

**OBS:**

`UPPER()`: converte todos os caracteres de uma string para maiúsculas.

Exemplo:
```sql
SELECT UPPER('casa');
```

`LOWER()`: converte todos os caracteres de uma string para minúsculas.

Exemplo:
```sql
SELECT LOWER('CaSa');
```

`TRIM()`: remove espaços em branco (ou outro caractere especificado) do início e do fim de uma string.

Exemplo:
```sql
SELECT TRIM('   texto   '); -- resultado: 'texto'
```

```sql
SELECT TRIM(BOTH '*' FROM '***Produto***'); -- resultado: 'Produto'
```

```sql
SELECT TRIM(LEADING '0' FROM '000123'); -- resultado: '123'
```

```sql
SELECT TRIM(TRAILING '.' FROM 'Fornecedor...'); -- resultado: 'Fornecedor'
```

```sql
SELECT TRIM(TRAILING '.' FROM TRIM(LEADING '*' FROM '***Fornecedor...')); -- resultado: 'Fornecedor'
```

---

### 3.8 UPDATE com CASE WHEN (lógica condicional)

O `CASE WHEN` permite definir regras diferentes para cada linha dentro de um **único** comando `UPDATE`, em vez de escrever um `UPDATE` separado para cada condição.

```sql
UPDATE Fornecedor
SET telefone = CASE
                   WHEN cnpj = '11111111000101' THEN '3139990001'
                   WHEN cnpj = '22222222000102' THEN '3139990002'
                   ELSE telefone
               END
WHERE cnpj IN ('11111111000101', '22222222000102');
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> Note o uso de `ELSE telefone`: isso garante que, se nenhuma condição do `CASE` for satisfeita para uma linha, o valor original seja mantido (evita que o campo vire `NULL` acidentalmente).

**OBS:** `WHERE` e `CASE` resolvem problemas diferentes. O `WHERE` define quais linhas o `UPDATE` vai atualizar, e o `CASE` define o que fazer com o valor de cada linha. Usar só o `CASE` sem `WHERE` "funciona" nesse caso simples do ponto de vista lógico, mas é uma prática arriscada que pode gerar problemas de desempenho e comportamento inesperado em cenários mais complexos (com triggers, tabelas grandes, etc.).

---

### 3.9 Atualização em massa com base em uma condição lógica

Diferente do `IN` (que lista valores específicos), aqui a condição é uma regra: "todo fornecedor sem telefone cadastrado". 

Esse `UPDATE` pode afetar **várias linhas de uma vez**, dependendo de quantas satisfizerem a condição.

```sql
UPDATE Fornecedor
SET telefone = '3130001000'
WHERE telefone IS NULL;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

---

### 3.10 UPDATE com ORDER BY e LIMIT (recurso específico do MySQL)

O MySQL permite combinar `ORDER BY` e `LIMIT` dentro de um `UPDATE`, algo que **não é padrão SQL** e não funciona em todos os SGBDs (Oracle e PostgreSQL, por exemplo, não suportam essa sintaxe diretamente). 

Isso é útil para atualizar apenas "o primeiro" registro segundo algum critério de ordenação.

```sql
UPDATE Fornecedor
SET nome = CONCAT(nome, ' (Fornecedor Destaque)')
ORDER BY nome ASC
LIMIT 1;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> **OBS:** nem sempre há portabilidade de código SQL entre diferentes bancos de dados. Cada Sistema Gerenciador de Banco de Dados (SGBD) tem seu "dialeto" específico.

**OBS:**  teste o exemplo anterior com `LIMIT 2;`.

---

### 3.11 UPDATE com subquery no WHERE

É possível basear a condição do `UPDATE` no resultado de uma subconsulta (subquery). 

Aqui, atualizamos o endereço apenas dos fornecedores cujo nome tem mais de 25 caracteres.

```sql
UPDATE Fornecedor
SET endereco = CONCAT(endereco, ' [nome longo]')
WHERE cnpj IN (
    SELECT cnpj FROM (
        SELECT cnpj FROM Fornecedor WHERE LENGTH(nome) > 25
    ) AS sub_tabela
)
AND endereco IS NOT NULL;
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> **OBS:** o MySQL não permite referenciar diretamente, dentro do `WHERE` de um `UPDATE`, um subselect que consulta a **mesma tabela** que está sendo atualizada. Por isso a subquery foi "embrulhada" em uma tabela derivada (`AS sub_tabela`).

**OBS:** teste o exemplo anterior sem o primeiro SELECT (o do `AS sub_tabela`) e verifique o erro que o MySQL retorna.

---

### 3.12 UPDATE sem WHERE: exemplo de alerta

`CUIDADO`: por fim, o exemplo mais importante do ponto de vista de boas práticas. Um `UPDATE` **sem** cláusula `WHERE` afeta **todas** as linhas da tabela, sem exceção.

```sql
UPDATE Fornecedor
SET endereco = 'Endereço atualizado em 2026';
```

Conferimos o resultado com um `SELECT`:

```sql
SELECT * FROM Fornecedor;
```

> **OBS:** analise o impacto de esquecer o `WHERE`, um erro comum e perigoso em ambientes de produção.

---

## Exercícios

> **Antes de começar:** execute o `DROP TABLE` + `CREATE TABLE` + `INSERT`s de cada exercício no MySQL para ter os dados disponíveis antes de criar os `UPDATE`s.

---

## Exercício 1 — Continuação do projeto: Produto

Este exercício usa a tabela `Produto` do projeto da disciplina (Fornecedor / Produto / Identificação / Filial / Estoque).

```sql
DROP TABLE IF EXISTS Produto;

CREATE TABLE Produto (
    id                INT AUTO_INCREMENT,
    cnpj_fornecedor   VARCHAR(14) NOT NULL,
    nome              VARCHAR(100) NOT NULL,
    PRIMARY KEY (id)
);

INSERT INTO Produto (id, cnpj_fornecedor, nome) VALUES
(1, '11111111000101', 'arroz tipo 1 5kg'),
(2, '11111111000101', 'feijao carioca 1kg'),
(3, '22222222000102', 'refrigerante cola 2l'),
(4, '33333333000103', 'macarrao espaguete'),
(5, '44444444000104', 'suco de laranja 1l'),
(6, '77777777000107', 'fone de ouvido bluetooth'),
(7, '77777777000107', 'carregador usb-c');
```

**Tarefas:**

a) Corrija o nome do produto de id `1` para `'Arroz Tipo 1 5kg'` (com a formatação correta de maiúsculas/minúsculas).

b) Usando `LIKE`, atualize o nome de todos os produtos que contenham a palavra `"usb-c"`, adicionando o prefixo `"[Eletrônico] "` ao nome atual.

c) Usando `IN`, transfira os produtos de `id` `6` e `7` para o fornecedor de `cnpj` `'11111111000101'` (ou seja, atualize o campo `cnpj_fornecedor`).

d) Usando `CASE WHEN`, concatene [Produto 2], [Produto 3] e [Produto 4] ao nome dos produtos de `id` `2`, `3` e `4`, respectivamente.

e) Explique por que seria arriscado escrever um `UPDATE` que trocasse `cnpj_fornecedor` **sem** cláusula `WHERE` nessa tabela. O que aconteceria com a integridade dos dados?

---

## Exercício 2 — Padaria: tabela Pedido

Uma padaria controla seus pedidos em uma tabela simples:

```sql
DROP TABLE IF EXISTS Pedido;

CREATE TABLE Pedido (
    id           INT AUTO_INCREMENT,
    cliente      VARCHAR(100) NOT NULL,
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

a) O pedido de `id = 3` acabou de ficar pronto. Atualize seu `status` para `'pronto'`.

b) Todos os pedidos da cliente `'Ana Souza'` que ainda estão `'pendente'` precisam de uma observação: `'cliente vai retirar às 18h'`. Escreva o `UPDATE` correspondente (atenção: são dois pedidos diferentes, use uma condição que capture ambos sem citar os `id`s diretamente).

c) Usando `CASE WHEN`, escreva um único `UPDATE` que avance o status de todos os pedidos `'preparando'` para `'pronto'` e de todos os pedidos `'pronto'` para `'entregue'`, em um só comando.

d) Usando subquery, atualize a observação dos pedidos cuja `quantidade` seja maior que 10, adicionando o texto `' - pedido grande'` ao final da observação atual (trate o caso de observação `NULL`).

---

## Exercício 3 — Agência de viagens: tabela Pacote

Uma agência de viagens organiza seus pacotes turísticos:

```sql
DROP TABLE IF EXISTS Pacote;

CREATE TABLE Pacote (
    id          INT AUTO_INCREMENT,
    destino     VARCHAR(100) NOT NULL,
    preco       DECIMAL(10,2) NOT NULL,
    vagas       INT NOT NULL,
    ativo       BOOLEAN NOT NULL DEFAULT 1,  -- 1/TRUE = ativo, 0/FALSE = inativo
    PRIMARY KEY (id)
);

INSERT INTO Pacote (id, destino, preco, vagas, ativo) VALUES
(1, 'Rio de Janeiro - RJ',  1200.00, 10, 1),
(2, 'Fernando de Noronha - PE', 4500.00,  5, 1),
(3, 'Gramado - RS',          980.00, 15, 1),
(4, 'Foz do Iguacu - PR',   1350.00,  8, 1),
(5, 'Bonito - MS',          2100.00,  0, 1),
(6, 'Salvador - BA',        1100.00, 12, TRUE);
```

**Tarefas:**

a) O pacote para `'Bonito - MS'` está com `vagas = 0`. Atualize o campo `ativo` para `0`, indicando que o pacote não está mais disponível para venda.

b) Aplique um reajuste de 10% no `preco` de todos os pacotes que têm mais de 10 `vagas` disponíveis (dica: use uma expressão aritmética diretamente no `SET`, como `preco = preco * 1.10`).

c) Uma promoção reduz em 15% o preço de todos os pacotes cujo `destino` contenha a palavra `'RS'` **ou** `'PR'` (use `LIKE` combinado com `OR`, ou `REGEXP`).

d) Escreva um `UPDATE` com `ORDER BY` e `LIMIT` que reduza em 1 o número de `vagas` do pacote mais barato (`preco` mais baixo) da tabela — simulando a venda de uma vaga.

e) Por que, nesse contexto de agência de viagens, seria perigoso um funcionário rodar um `UPDATE` no campo `preco` **sem** `WHERE` durante uma promoção? Descreva um cenário real de erro.

---

## Exercício 4 — Banco: tabela Conta

Um banco mantém o controle simplificado de contas correntes:

```sql
DROP TABLE IF EXISTS Conta;

CREATE TABLE Conta (
    numero        INT,
    titular       VARCHAR(100) NOT NULL,
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

a) A conta `1004` recebeu um depósito de `R$ 500,00`. Atualize o `saldo` somando esse valor ao saldo atual (não sobrescreva o valor, calcule a partir do saldo existente).

b) Preencha o `telefone` das contas que estão com esse campo `NULL`, atribuindo o valor `'0800-000-0000'` (telefone padrão de contato do banco).

c) Todas as contas do tipo `'poupanca'` com saldo acima de `R$ 5.000,00` recebem 0,5% de rendimento mensal. Escreva o `UPDATE` que aplica esse rendimento ao `saldo`.

d) Usando `CASE WHEN`, escreva um único `UPDATE` que classifique — em um novo cenário — contas `'corrente'` com saldo negativo como um caso especial: se o saldo de alguma conta corrente estivesse abaixo de zero, o `saldo` deveria ser zerado e o `telefone` marcado com `'CONTATO URGENTE'`. (Não há conta negativa nos dados atuais — insira você mesmo uma linha de teste antes de rodar esse `UPDATE`, ou adapte a condição para testar a lógica.)

e) Reflita: por que, em um sistema bancário real, um `UPDATE` direto de `saldo` (como nos itens acima) dificilmente seria feito assim, "na mão"? Que outros mecanismos (transações, procedures, trilhas de auditoria) você imagina que um banco de verdade usaria para alterar saldos com segurança?

---

## Exercício 5 — Desafio combinando conceitos (projeto Fornecedor/Filial/Estoque)

Considere que, além de `Produto`, também existam as tabelas `Filial` e `Estoque` do projeto original:

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
(4, '10101010000110', 12.50,  40, '2026-12-20'),
(5, '20202020000120',  9.90,  25, '2026-10-05');
```

**Tarefas:**

a) O produto de `id = 5` está com validade próxima (`'2026-10-05'`). Atualize seu `preco` no estoque da filial `'20202020000120'` aplicando um desconto de 30% (simulando uma queima de estoque por proximidade da validade).

b) Escreva um `UPDATE` que zere a `quantidade` de todos os itens de estoque cuja `validade` seja anterior a `'2026-11-01'` (produto vencido — ainda não vamos excluir a linha, apenas zerar o estoque disponível).

c) Some `10` unidades à `quantidade` de todos os itens de estoque do produto `id = 1`, em ambas as filiais, em um único comando `UPDATE`.

d) Usando subquery, aumente em 5% o `preco` de todos os itens de estoque cujo produto (`id_produto`) pertença ao fornecedor de `cnpj` `'11111111000101'` (você vai precisar relacionar `Estoque` com `Produto` através de uma subconsulta, já que `Estoque` não tem diretamente o `cnpj_fornecedor`).

e) Justifique por que a tabela `Estoque` tem **chave primária composta** (`id_produto` + `cnpj_filial`) e o que isso implica na cláusula `WHERE` de um `UPDATE` que precise alterar **um único** registro de estoque.


---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="06-fornecedor.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="08-produto.md">Próximo</a>
    </td>
  </tr>
</table>
