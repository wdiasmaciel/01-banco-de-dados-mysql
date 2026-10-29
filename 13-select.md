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


## Operadores aritméticos e precedência

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

---

## Concatenação de strings

No MySQL, `+` é **somente aritmético**. A função correta é `CONCAT()`.

```sql
SELECT CONCAT(nome, ' está localizado em ', endereco) AS "Detalhes do Fornecedor"
FROM   Fornecedor;
```

---

## Operações com datas

No MySQL, a função `DATE_ADD()` soma dias a uma data. 

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL 5 DAY) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

---

## Operador LIKE e curingas

Caracteres curingas no MySQL: 
1. `%` representa zero ou vários caracteres.
2. `_` representa exatamente um caractere.

```sql
SELECT cnpj, nome
FROM   Fornecedor
WHERE  nome LIKE 'D%'; -- nome tem que iniciar com a letra maiúscula D seguida de qualquer coisa.
-- Retorna: Distribuidora Alfa Ltda, Delta Bebidas Ltda.
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE 'S%L'; -- nome tem que iniciar com a letra maiúscula S seguida de qualquer coisa e terminar com a letra maiúscula L.
-- Retorna: Suco de Laranja 1L (começa com S, termina com L)
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '_o%'; -- nome tem que iniciar com uma única letra qualquer antes da letra minúscula "o" seguida de qualquer coisa. 
-- Retorna: Fone de Ouvido Bluetooth (segunda letra = 'o')
```

---

## ESCAPE no LIKE

Usado para procurar literalmente pelo caractere `%` ou `_` dentro de um texto (em vez de tratá-los como curinga). 

No MySQL, a sintaxe `ESCAPE` é `\`. 

Como nenhum dado do projeto contém `%` (`\%`) ou `_` (`\_`) no nome, inserimos registros de teste para demonstrar:

```sql
INSERT INTO Produto (id, cnpj_fornecedor, nome) VALUES 
(7, '55555555000105', 'Kit_Degustação'),
(8, '55555555000105', 'Iogurte 0% Lactose');
``` 

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '%\_%' ESCAPE '\\';
-- Retorna: Kit_Degustacao (o '\' antes do '_' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '%\%%' ESCAPE '\\';
-- Retorna: Iogurte 0% Lactose (o '\' antes do '%' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```

> **OBS:** 
1. No MySQL, a barra invertida (`\`) já é o caractere de escape padrão.
2. Então, `ESCAPE '\\'` costuma ser redundante.
3. O exemplo acima funcionaria mesmo sem a cláusula `ESCAPE`. 
4. Ainda assim, vale mostrar a sintaxe explícita, já que ela é exigida em outros bancos (como o SQL Server).

---

## NOT IN

Encontrar fornecedores que **não têm** algum produto associado:

```sql
SELECT nome
FROM   Fornecedor
WHERE  cnpj NOT IN (SELECT DISTINCT cnpj_fornecedor FROM Produto);
```

Encontrar o nome das filiais que não estejam entre ('Filial Centro', 'Filial Savassi'):

```sql
SELECT nome
FROM   Filial
WHERE  nome NOT IN ('Filial Centro', 'Filial Savassi');
-- Retorna: Filial Contagem
```

---

## Precedência de AND / OR

**OBS:** 
1. O `AND` tem precedência maior que o `OR`. 

2. Na primeira consulta, a condição é avaliada como `preco < 10 OR (quantidade > 50 AND cnpj_filial = '10101010000110')`. Ou seja: 
    * Qualquer item com `preco < 10` aparece, **mesmo que não seja da Filial Centro**. 

3. Na segunda consulta, os parênteses forçam que a filial seja sempre `'Filial Centro'`. 

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

---

## ORDER BY

1. **Por alias** (sem aspas duplas, que no MySQL têm outro significado por padrão):

```sql
SELECT id_produto, preco, preco * 1.5 AS 'novo preço'
FROM   Estoque
ORDER BY 'novo preço';
```

**OBS**:
No MySQL padrão, aspas duplas ("), por padrão, podem representar strings literais, da mesma forma que aspas simples ('), quando o modo `SQL ANSI_QUOTES` não está ativo.

Exemplo:
```sql
SELECT "Produto";  -- Tratado como string literal: Produto
```
```sql
SELECT "nome";     -- Em alguns casos, também pode funcionar como identificador
```

Por isso, para gerar aliases com espaços ou caracteres especiais, é mais seguro usar aspas simples ou, preferencialmente, crases para identificadores:

```sql
SELECT nome AS 'Nome do produto';  -- Aliases com string
```

```sql
SELECT nome AS `Nome do produto`;  -- Aliases como identificador MySQL
```

2. **Por posição da coluna no SELECT:**

```sql
SELECT id, descricao
FROM   Identificacao
ORDER BY 2;
```

3. **Por mais de uma coluna:**

```sql
SELECT nome, cnpj_fornecedor
FROM   Produto
ORDER BY cnpj_fornecedor, nome;
```

4. **Por uma coluna que não está no SELECT**:

```sql
SELECT nome, endereco
FROM   Filial
ORDER BY cnpj;
```

5. **Múltiplas colunas com direções diferentes:**

```sql
SELECT id_produto, cnpj_filial
FROM   Estoque
ORDER BY cnpj_filial, id_produto DESC;
```

---

## Funções de manipulação de caracteres

| Função | Observação |
|---|---|
| `LENGTH()` / `CHAR_LENGTH()` | `LENGTH()` conta bytes; `CHAR_LENGTH()` conta caracteres (importante com acentos) |
| `LOCATE(busca, texto, pos)` | Localiza a primeira ocorrência da substring `busca` em `texto`, começando pela posição `pos`; retorna 0 se não encontrar |
| `LEFT()` / `RIGHT()` | Retorna a quantidade de caracteres indicada à esquerda (`LEFT`) ou à direita (`RIGHT`) de um texto |
| `LTRIM()` / `RTRIM()` | Remove espaços em branco do início (`LTRIM`) ou do fim (`RTRIM`) de um texto |
| `REPLACE()` | Substitui todas as ocorrências de um trecho por outro dentro do texto informado |
| `SUBSTRING()` | Extrai uma parte do texto a partir de uma posição inicial e, opcionalmente, com um comprimento definido |

```sql
SELECT nome, LENGTH(nome) AS bytes, CHAR_LENGTH(nome) AS tamanho
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
SELECT LTRIM('   Arroz   ') AS esquerda, RTRIM('   Feijao   ') AS direita;
```

```sql
SELECT nome, REPLACE(nome, 'a', 'o') AS troca
FROM   Produto;
```

```sql
SELECT nome, SUBSTRING(nome, 1, 5) AS substring
FROM   Produto;
```

---

## Funções numéricas

| Função | Descrição |
|---|---|
| `ABS()` | Retorna o valor absoluto de um número, ou seja, o mesmo número sem sinal negativo. |
| `CEILING()` | Arredonda um número para cima, retornando o menor inteiro maior ou igual ao valor informado. |
| `FLOOR()` | Arredonda um número para baixo, retornando o maior inteiro menor ou igual ao valor informado. |
| `SIGN()` | Retorna `-1` para números negativos, `0` para zero e `1` para números positivos. |
| `ROUND()` | Arredonda um número para uma quantidade específica de casas decimais. |
| `TRUNCATE()` | Remove casas decimais de um número sem realizar arredondamento. |

```sql
SELECT ABS(-1.0) AS abs1, ABS(0.0) AS abs2, ABS(1.0) AS abs3;
```

```sql
SELECT preco, CEILING(preco) AS teto, FLOOR(preco) AS piso
FROM   Estoque;
```

```sql
SELECT SIGN(-10.0) AS negativo, SIGN(0.0) AS zero, SIGN(10.0) AS positivo;
```

```sql
SELECT ROUND(748.58, -1) AS round1, 
       ROUND(748.58, -2) AS round2,
       ROUND(748.58, 0)  AS round3, 
       ROUND(748.58, 1)  AS round4;
```

```sql
SELECT TRUNCATE(150.98, 0) AS truncate1, 
       TRUNCATE(150.98, 1) AS truncate2;
```

---

## CASE WHEN

Classificar o preço dos itens de estoque e traduzir o código da filial para o nome:

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

Classificando a validade por mês de vencimento:

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

## IFNULL 

O projeto não tem, no momento, nenhuma coluna numérica que aceite `NULL`. 

Para demonstrar `IFNULL()` de forma prática, adicionamos temporariamente uma coluna opcional `desconto` a `Estoque`:

```sql
ALTER 
TABLE Estoque 
  ADD COLUMN desconto DECIMAL(10,2) NULL;
```

```sql
UPDATE Estoque 
   SET desconto = 2.00 
 WHERE id_produto = 1 
   AND cnpj_filial = '10101010000110';
-- Os demais registros permanecem com desconto = NULL
```

```sql
SELECT id_produto, cnpj_filial, preco, desconto,
       preco - IFNULL(desconto, 0) AS preco_final
FROM   Estoque;
```

> **OBS:** 
1. Sem o `IFNULL()`, `preco - desconto` resultaria em `NULL` para todas as linhas em que `desconto` não foi informado.
2. Porém, qualquer operação aritmética com `NULL` retorna `NULL`.

---

## CAST, CONVERT e formatação de datas

```sql
SELECT id_produto, 
       CONCAT('O preço é ', CAST(preco AS CHAR)) AS preco_formatado
FROM   Estoque;
```

Para formatar datas, o MySQL usa `DATE_FORMAT()` com máscaras de formato:

```sql
SELECT id_produto, 
       cnpj_filial, 
      DATE_FORMAT(validade, '%d/%m/%Y') AS data_formatada
FROM   Estoque;
```

Outros formatos úteis:

```sql
-- Ano-mês-dia, adequado para exibição internacional (padrão ISO):
SELECT id_produto, 
       cnpj_filial, 
       DATE_FORMAT(validade, '%Y-%m-%d') AS data_iso
FROM   Estoque;
```

```sql
-- Dia da semana, dia, mês por extenso e ano:
SELECT id_produto, 
       cnpj_filial, 
       DATE_FORMAT(validade, '%W, %d de %M de %Y') AS data_por_extenso
FROM   Estoque;
```

```sql
-- Dia, mês abreviado e ano:
SELECT id_produto, 
       cnpj_filial, 
       DATE_FORMAT(validade, '%d/%b/%Y') AS data_abreviada
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