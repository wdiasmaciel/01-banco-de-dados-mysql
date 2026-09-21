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

> **Detalhe técnico importante:** o MySQL não permite referenciar diretamente, dentro do `WHERE` de um `UPDATE`, um subselect que consulta a **mesma tabela** que está sendo atualizada. Por isso a subquery foi "embrulhada" em uma tabela derivada (`AS sub_tabela`).

**OBS:** teste o exemplo anterior sem o primeiro SELECT (o do `AS sub_tabela`) e verifique o erro que o MySQL retorna.

---

### 3.12 UPDATE sem WHERE — o exemplo de alerta

Por fim, o exemplo mais importante da aula do ponto de vista de boas práticas: um `UPDATE` **sem** cláusula `WHERE` afeta **todas** as linhas da tabela, sem exceção.

```sql
UPDATE Fornecedor
SET endereco = CONCAT(endereco, ' - Atualizado em 2026');
```

> **Recomendação para a aula:** demonstre esse comando ao vivo (com a turma avisada) para que os alunos vejam o impacto real de esquecer o `WHERE` — um dos erros mais comuns e mais perigosos em ambientes de produção. Depois, mostre como restaurar os dados originais executando novamente o `DROP TABLE` + `CREATE TABLE` + os `INSERT`s do início do script.

---

## 4. Verificando o resultado final

Ao final de todas as alterações, um novo `SELECT` mostra o estado atual da tabela, permitindo comparar com o `SELECT` do início da aula:

```sql
SELECT * FROM Fornecedor;
```

---

## 5. Resumo dos conceitos praticados

| Exemplo | Conceito |
|---|---|
| 3.1 | UPDATE simples por chave primária |
| 3.2 | Atualização de múltiplas colunas |
| 3.3 | Preenchimento de campo NULL |
| 3.4 | Atribuição de NULL |
| 3.5 | LIKE + CONCAT |
| 3.6 | Operador IN |
| 3.7 | Funções de string (UPPER) |
| 3.8 | CASE WHEN dentro do SET |
| 3.9 | Atualização em massa (IS NULL) |
| 3.10 | ORDER BY + LIMIT (específico do MySQL) |
| 3.11 | Subquery no WHERE |
| 3.12 | UPDATE sem WHERE (alerta de boas práticas) |

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
