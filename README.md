# 01-banco-de-dados-mysql
 
---

# DDL, DML, DQL e DCL

Os principais comandos de SQL (Structured Query Language, ou Linguagem de Consulta Estruturada)dividem-se em:

1. DDL: Data Definition Language (Linguagem de Definição de Dados).

2. DML: Data Manipulation Language (Linguagem de Manipulação de Dados).

3. DQL: Data Query Language (Linguagem de Consulta de Dados).

4. DCL: Data Control Language (Linguagem de Controle de Dados).

Essas categorias são empregadas de acordo com a estrutura e o objetivo de cada operação no banco de dados.

---

# DDL (Data Definition Language)

A DDL define e altera a estrutura do banco de dados e de seus objetos (como tabelas, índices e visões).

## Principais comandos (cláusula):

1. `CREATE`: cria um novo objeto no banco de dados (ex: CREATE TABLE).

2. `ALTER`: modifica a estrutura de um objeto existente (ex: adicionar uma coluna).

3. `DROP`: remove um objeto do banco de dados.

4. `TRUNCATE`: apaga todos os registros de uma tabela rapidamente, mantendo sua estrutura.

---

# DML (Data Manipulation Language)

A DML manipula os dados armazenados dentro das tabelas (inserindo, atualizando ou removendo registros).

## Principais comandos (cláusula):

1. `INSERT`: insere novas linhas ou registros em uma tabela.

2. `UPDATE`: atualiza dados que já existem em uma tabela.

3. `DELETE`: exclui registros específicos de uma tabela.

**OBS**: algumas literaturas antigas também incluem o `SELECT` na DML, mas a separação atual é mais comum.

---

# DQL (Data Query Language)

A DQL consulta e recupera os dados armazenados nas tabelas.

## Principais comandos (cláusula):

1. `SELECT`: o principal comando usado para buscar e filtrar dados no banco.

2. Funções de agregação: como COUNT(), SUM(), AVG(), MIN() e MAX().

3. `FROM`: indica a tabela onde estão os dados.

4. `WHERE`: filtra as linhas conforme uma regra.

5. `JOIN`: estabelece uma junção entre linhas de duas ou mais tabelas com base em uma coluna relacionada entre elas. Principais Tipos de JOIN:
 
 - `INNER JOIN`: retorna apenas os registros que possuem correspondência (match) em ambas as tabelas.
 
 - `LEFT JOIN`: retorna todos os registros da tabela da esquerda e apenas os correspondentes da tabela da direita. Se não houver correspondência, retorna valores nulos (NULL).
 
 - `RIGHT JOIN`: retorna todos os registros da tabela da direita e apenas os correspondentes da tabela da esquerda. Se não houver correspondência, retorna valores nulos (NULL).
 
 - `FULL OUTER JOIN`: retorna todos os registros quando houver correspondência em qualquer uma das tabelas. Se não houver correspondência, retorna valores nulos (NULL).

6. `GROUP BY`: agrupa linhas de dados com base em uma ou mais colunas. Quase sempre é utilizado em conjunto com funções de agregação, como COUNT(), SUM(), AVG(), MIN() ou MAX().

7. `HAVING`: filtra os resultados após o agrupamento. Funciona como uma cláusula `WHERE`, mas foi projetada especificamente para lidar com condições em dados agregados (agrupados).

8. `ORDER BY`: ordena a saída final da consulta em ordem crescente (ASC, padrão, default) ou decrescente (DESC).

**OBS**: as cláusulas `GROUP BY`, `HAVING` e `ORDER BY` devem ser escritas em uma sequência específica (hierarquia) para formar uma consulta válida. A ordem lógica e sintática correta é `GROUP BY` → `HAVING` → `ORDER BY`.

---

# DCL (Data Control Language)

A DCL é responsável pela definição de permissões para os usuários do banco de dados.

## Principais comandos (cláusula):

1. `GRANT`: concede permissões específicas a um usuário ou papel (role).

2. `REVOKE`: remove ou retira permissões que haviam sido concedidas anteriormente.

3. `DENY`: bloquear, restringir, o acesso a objetos do banco de dados.

---

# Link

https://learnsql.com.br/blog/o-que-sao-ddl-dml-dql-e-dcl-em-sql/

<div style="display: flex; justify-content: flex-end; width: 100%;">
  <a href="01_fornecedor.md" style="text-decoration: none;">Próximo →</a>
</div>