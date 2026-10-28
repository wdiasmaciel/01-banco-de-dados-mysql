<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="12-grup-by-having.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

# Select 


## 1. Operadores aritméticos e precedência

```sql
SELECT id_produto, id_filial, preco, 12 * preco + 100
FROM   Estoque;
```

```sql
SELECT id_produto, id_filial, preco, 12 * (preco + 100)
FROM   Estoque;
```

```sql
SELECT (SELECT nome FROM Produto WHERE id = e.id_produto) AS nome,
       e.preco AS "preço",
       12 * e.preco + 100 AS "novo preço"
FROM   Estoque e;
```

```sql
SELECT (SELECT nome FROM Produto WHERE id = e.id_produto) AS nome,
       e.preco As "preço",
       12 * (e.preco + 100) AS "novo preço"
FROM   Estoque e;
```

**Ponto para explorar em aula:** peça aos alunos para calcularem manualmente os dois resultados para o mesmo `preco` e explicarem por que são diferentes — é o exemplo clássico de precedência de operadores (`*` antes de `+`, a menos que parênteses mudem a ordem).

---

## 2. Concatenação de strings

No T-SQL, `+` concatena strings. No MySQL, `+` é **somente aritmético** — usar `+` entre strings tenta convertê-las para número e provavelmente retorna `0` ou erro. A função correta é `CONCAT()`.

```sql
SELECT CONCAT(nome, ' está localizado em ', endereco) AS "Detalhes do Fornecedor"
FROM   Fornecedor;
```

---

## 3. Operações com datas

No T-SQL, `data + 5` soma 5 dias a uma data. No MySQL, isso precisa da função `DATE_ADD()`.

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL 5 DAY) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

---

## 4. Operador LIKE e curingas

A explicação dos curingas continua igual no MySQL: `%` representa zero ou vários caracteres, `_` representa exatamente um caractere.

```sql
SELECT cnpj, nome
FROM   Fornecedor
WHERE  nome LIKE 'D%';
-- Retorna: Distribuidora Alfa Ltda, Delta Bebidas Ltda
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE 'S%L';
-- Retorna: Suco de Laranja 1L (começa com S, termina com L)
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '_o%';
-- Retorna: Fone de Ouvido Bluetooth (segunda letra = 'o')
```

---

## 5. ESCAPE no LIKE

Usado para procurar literalmente pelo caractere `%` ou `_` dentro de um texto (em vez de tratá-los como curinga). A sintaxe `ESCAPE` é idêntica no MySQL. Como nenhum dado do projeto contém `%` ou `_` no nome, inserimos um registro de teste para demonstrar:

```sql
INSERT INTO Produto (id, cnpj_fornecedor, nome)
VALUES (7, '55555555000105', 'Kit_Degustacao');

SELECT nome
FROM   Produto
WHERE  nome LIKE '%\_%' ESCAPE '\\';
-- Retorna: Kit_Degustacao (o '\' antes do '_' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```

> **Observação:** no MySQL, a barra invertida (`\`) já é o caractere de escape padrão, então `ESCAPE '\\'` costuma ser redundante — o exemplo acima funcionaria mesmo sem a cláusula `ESCAPE`. Ainda assim, vale mostrar a sintaxe explícita, já que ela é exigida em outros bancos (como o SQL Server do material original).

---

## 6. IS NULL

No projeto atual, **não há colunas que aceitem `NULL`** (todas são `NOT NULL`), então o exemplo original não tem equivalente direto. A situação mais próxima, no nosso contexto, é usar uma subquery para encontrar fornecedores que **não têm** nenhum produto associado — o mesmo resultado que apareceria como `produto IS NULL` caso fizéssemos um `LEFT JOIN` entre `Fornecedor` e `Produto`:

```sql
SELECT nome
FROM   Fornecedor
WHERE  cnpj NOT IN (SELECT cnpj_fornecedor FROM Produto);
```

**Ponto para explorar em aula:** essa é uma boa ponte para comparar com o `LEFT JOIN ... WHERE produto IS NULL` que vocês viram na aula de `JOIN`s — duas formas diferentes de responder à mesma pergunta ("quem não tem correspondência?"), uma com subquery e outra com junção externa.

---

## 7. NOT IN

```sql
SELECT nome
FROM   Filial
WHERE  nome NOT IN ('Filial Centro', 'Filial Savassi');
-- Retorna: Filial Contagem
```

---

## 8. Precedência de AND / OR

```sql
SELECT id_produto, cnpj_filial, preco, quantidade
FROM   Estoque
WHERE  preco < 10
OR     quantidade > 50
AND    cnpj_filial = '10101010000110';
```

```sql
SELECT id_produto, cnpj_filial, preco, quantidade
FROM   Estoque
WHERE  (preco < 10
OR      quantidade > 50)
AND     cnpj_filial = '10101010000110';
```

**Ponto para explorar em aula:** assim como no original, o `AND` tem precedência maior que o `OR`. Na primeira consulta, a condição é avaliada como `preco < 10 OR (quantidade > 50 AND cnpj_filial = '...')` — ou seja, qualquer item com `preco < 10` aparece, **mesmo que não seja da Filial Centro**. Na segunda, os parênteses forçam que a filial seja sempre `'Filial Centro'`. Peça aos alunos para rodar as duas e comparar os resultados.

---

## 9. ORDER BY

**Por alias** (sem aspas duplas, que no MySQL têm outro significado por padrão):

```sql
SELECT id_produto, preco * 1.5 AS novo_preco
FROM   Estoque
ORDER BY novo_preco;
```

**Por posição da coluna no SELECT:**

```sql
SELECT id, descricao
FROM   Identificacao
ORDER BY 2;
```

**Por mais de uma coluna:**

```sql
SELECT nome, cnpj_fornecedor
FROM   Produto
ORDER BY cnpj_fornecedor, nome;
```

**Por uma coluna que não está no SELECT** (válido tanto em T-SQL quanto em MySQL):

```sql
SELECT nome, endereco
FROM   Filial
ORDER BY cnpj;
```

**Múltiplas colunas com direções diferentes:**

```sql
SELECT id_produto, cnpj_filial
FROM   Estoque
ORDER BY cnpj_filial, id_produto DESC;
```

---

## 10. Funções de manipulação de caracteres

| T-SQL | MySQL | Observação |
|---|---|---|
| `LEN()` | `LENGTH()` / `CHAR_LENGTH()` | `LENGTH()` conta bytes; `CHAR_LENGTH()` conta caracteres (importante com acentos) |
| `CHARINDEX(busca, texto, pos)` | `LOCATE(busca, texto, pos)` | mesma ordem de argumentos |
| `LEFT()` / `RIGHT()` | `LEFT()` / `RIGHT()` | sintaxe idêntica |
| `LTRIM()` / `RTRIM()` | `LTRIM()` / `RTRIM()` | sintaxe idêntica (só removem espaços em branco) |
| `REPLACE()` | `REPLACE()` | sintaxe idêntica |
| `SUBSTRING()` | `SUBSTRING()` | sintaxe idêntica |

```sql
SELECT nome, SUBSTRING(nome, 1, 5) AS sub
FROM   Produto;
```

```sql
SELECT nome, CHAR_LENGTH(nome) AS tamanho
FROM   Fornecedor;
```

```sql
SELECT nome, LOCATE('i', nome, 3) AS posicao
FROM   Produto;
```

```sql
SELECT nome, LEFT(nome, 3) AS esquerda, RIGHT(nome, 3) AS direita
FROM   Produto;
```

```sql
SELECT LTRIM('   Arroz   ') AS esq, RTRIM('   Feijao   ') AS dir;
```

```sql
SELECT nome, REPLACE(nome, 'a', 'o') AS troca
FROM   Produto;
```

---

## 11. Funções numéricas

`ABS()`, `CEILING()`, `FLOOR()` e `SIGN()` têm sintaxe idêntica no MySQL. `ROUND()` também existe, mas **não aceita um terceiro parâmetro para truncar** como no T-SQL — para truncar, o MySQL usa a função `TRUNCATE()`.

```sql
SELECT ROUND(748.58, -1) AS r1, ROUND(748.58, -2) AS r2,
       ROUND(748.58, 0)  AS r3, ROUND(748.58, 1)  AS r4;
```

```sql
-- Truncamento: no T-SQL seria ROUND(150.98, 0, 1) e ROUND(150.98, 1, 1)
SELECT TRUNCATE(150.98, 0) AS t1, TRUNCATE(150.98, 1) AS t2;
```

```sql
SELECT ABS(-1.0) AS a1, ABS(0.0) AS a2, ABS(1.0) AS a3;
```

```sql
SELECT preco, CEILING(preco) AS teto, FLOOR(preco) AS piso
FROM   Estoque;
```

```sql
SELECT SIGN(-10.0) AS neg, SIGN(0.0) AS zero, SIGN(10.0) AS pos;
```

---

## 12. CASE WHEN

A sintaxe de `CASE` é a mesma em T-SQL e MySQL. Adaptamos o conteúdo para classificar o preço dos itens de estoque e traduzir o código da filial para o nome:

```sql
SELECT id_produto,
       cnpj_filial,
       preco,
       CASE
           WHEN preco < 10 THEN 'Baixo'
           ELSE 'Alto'
       END AS "Faixa de Preço",
       CASE cnpj_filial
           WHEN '10101010000110' THEN 'Filial Centro'
           WHEN '20202020000120' THEN 'Filial Savassi'
           WHEN '30303030000130' THEN 'Filial Contagem'
       END AS "Nome da Filial"
FROM   Estoque;
```

Classificando a validade por mês de vencimento (equivalente ao exemplo original de `datadevolucao BETWEEN ...`):

```sql
SELECT id_produto, cnpj_filial, validade,
       CASE
           WHEN validade BETWEEN '2026-10-01' AND '2026-10-31' THEN 'Vence em outubro'
           WHEN validade BETWEEN '2026-11-01' AND '2026-11-30' THEN 'Vence em novembro'
           WHEN validade BETWEEN '2026-12-01' AND '2026-12-31' THEN 'Vence em dezembro'
           ELSE 'Vence em outro período'
       END AS "Período de vencimento"
FROM   Estoque;
```

---

## 13. IFNULL (equivalente ao ISNULL do T-SQL)

O projeto não tem, hoje, nenhuma coluna numérica que aceite `NULL`. Para demonstrar `IFNULL()` de forma prática, adicionamos temporariamente uma coluna opcional `desconto` a `Estoque`:

```sql
ALTER TABLE Estoque ADD COLUMN desconto DECIMAL(10,2) NULL;

UPDATE Estoque SET desconto = 2.00 WHERE id_produto = 1 AND cnpj_filial = '10101010000110';
-- Os demais registros permanecem com desconto = NULL

SELECT id_produto, cnpj_filial, preco, desconto,
       preco - IFNULL(desconto, 0) AS preco_final
FROM   Estoque;
```

> **Ponto para explorar em aula:** sem o `IFNULL()`, `preco - desconto` resultaria em `NULL` para todas as linhas em que `desconto` não foi informado (qualquer operação aritmética com `NULL` retorna `NULL`) — o mesmo comportamento discutido na aula sobre a diferença entre `NULL` e string vazia.

---

## 14. CAST, CONVERT e formatação de datas

```sql
SELECT id_produto, CONCAT('O preço é ', CAST(preco AS CHAR)) AS preco_formatado
FROM   Estoque;
```

> No T-SQL seria `CAST(valor AS varchar)`; no MySQL o tipo correto para converter número em texto é `CHAR`.

Para formatar datas (equivalente ao `CONVERT(VARCHAR, data, estilo)` do T-SQL, que usa códigos numéricos de estilo), o MySQL usa `DATE_FORMAT()` com máscaras de formato:

```sql
SELECT id_produto, cnpj_filial, DATE_FORMAT(validade, '%d/%m/%Y') AS data_formatada
FROM   Estoque;
```

---

## 15. INSERT INTO ... SELECT

No T-SQL do material original, o `SELECT` aparece entre parênteses depois do `INSERT INTO`. No MySQL, **não se usa parênteses** ao redor do `SELECT` nesse caso.

```sql
CREATE TABLE FornecedorBH (
    cnpj      VARCHAR(14)  NOT NULL,
    nome      VARCHAR(100) NOT NULL,
    endereco  VARCHAR(200) NOT NULL,
    PRIMARY KEY (cnpj)
);

INSERT INTO FornecedorBH (cnpj, nome, endereco)
SELECT cnpj, nome, endereco
FROM   Fornecedor
WHERE  endereco LIKE '%Belo Horizonte%';
```

---

# Exercício

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="12-grup-by-having.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>