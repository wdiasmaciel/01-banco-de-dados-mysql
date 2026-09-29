<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="09-relacionamentos.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="11-select.md">Próximo</a>
    </td>
  </tr>
</table>

---

## Funções de Agregação e SELECT entre tabelas


### Funções de agregação

Funções de agregação em SQL são comandos que processam várias linhas de dados para retornar apenas um único valor resumido.

Principais Funções de Agregação:

• `COUNT()`: conta o número de linhas ou de valores não nulos em uma coluna.

• `SUM()`: soma todos os valores numéricos de uma coluna.

• `AVG()`: calcula a média aritmética dos valores de uma coluna numérica.

• `MIN()`: retorna o menor valor de um conjunto de dados.

• `MAX()`: retorna o maior valor de um conjunto de dados.

Exemplos:

1. `COUNT()`: apresentar quantos produtos não estão disponíveis no estoque, estão com quantidade igual a zero. 

```sql
SELECT COUNT(*)
FROM Estoque
WHERE quantidade = 0;
```

2. `SUM()`: somar a quantidade de produtos no estoque com preço igual a zero.

```sql
SELECT SUM(quantidade)
FROM Estoque
WHERE preco = 0;
```

3. `AVG()`: calcular a média aritmética dos preços dos itens do estoque.

```sql
SELECT AVG(preco)
FROM Estoque;
```

Calcular a média aritmética dos preços dos itens do estoque cujo preço é maior que zero.

```sql
SELECT AVG(preco)
FROM Estoque
WHERE preco > 0;
```

4. `MIN()`: apresentar o menor preço do estoque.

```sql
SELECT MIN(preco)
FROM Estoque;
```

Apresentar o menor preço do estoque, mas maior que zero.

```sql
SELECT MIN(preco)
FROM Estoque
WHERE preco > 0;
```


5. `MAX()`: apresentar o maior preço do estoque.

```sql
SELECT MAX(preco)
FROM Estoque;
```

### SELECT simples com WHERE e ORDER BY

Antes de combinar tabelas, revisamos o básico: filtrar e ordenar dados de uma única tabela.

```sql
SELECT nome, endereco
FROM Fornecedor
WHERE nome LIKE '%Ltda%'
ORDER BY nome ASC;
```

### SELECT com subquery (subconsulta) entre tabelas

1. Aqui buscamos todos os produtos de um fornecedor específico.

2. Mas, em vez de "sabermos de cor" o `cnpj`, usamos uma subquery que busca esse `cnpj` a partir do nome do fornecedor.

3. Uma forma de combinar informação de duas tabelas sem usar `JOIN` explicitamente.

```sql
SELECT nome
FROM Produto
WHERE cnpj_fornecedor = (
    SELECT cnpj 
    FROM Fornecedor 
    WHERE nome = 'Distribuidora Alfa Ltda'
);
```

### Quais produtos possuem preço maior que o do produto 3?  
    
```sql
SELECT id_produto, preco
FROM   Estoque
WHERE  preco >= (
                  SELECT preco
                  FROM   Estoque
                  WHERE  id_produto > 3
                );
```

Se, nos dados de exemplo, mais de um produto tiver `id_produto > 3` (o que é o caso, pois temos os produtos 4, 5 e 6), esse comando falha com o erro:

```text
ERROR 1242 (21000): Subquery returns more than 1 row
```

O operador `>=` espera comparar `preco` com um único valor, mas a subquery retorna vários preços (um para cada produto com `id_produto > 3`). O MySQL não sabe com qual desses valores comparar.

### ANY

Com `ANY`, aceita a linha se `preco` for maior ou igual a pelo menos um dos valores retornados pela subquery:

```sql
SELECT id_produto, preco
FROM   Estoque
WHERE  preco >= ANY (
                       SELECT preco
                       FROM   Estoque
                       WHERE  id_produto > 3
                     );
```

### ALL

Com ALL, aceita a linha só se `preco` for maior ou igual a todos os valores retornados pela subquery (ou seja, maior ou igual ao maior preço do grupo):

```sql
SELECT id_produto, preco
FROM   Estoque
WHERE  preco >= ALL (
                       SELECT preco
                       FROM   Estoque
                       WHERE  id_produto > 3
                     );            
```

### Várias Subqueries

Neste exemplo, buscamos itens de estoque cujo `preco` seja igual ao preço do `'Suco de Laranja 1L'` vendido na `Filial Savassi` e cujo `cnpj_filial` seja o da `'Filial Centro'`.

```sql
SELECT id_produto, preco
FROM   Estoque e1
WHERE  e1.preco = (
                    SELECT preco
                    FROM   Estoque e2, Produto p
                    WHERE  p.id = e2.id_produto
                      AND  p.nome = 'Suco de Laranja 1L'
                      AND  e2.cnpj_filial = (
                                              SELECT cnpj
                                              FROM   Filial
                                              WHERE  nome = 'Filial Savassi' 
                                            )
                  )
  AND  e1.cnpj_filial = (
                         SELECT cnpj
                         FROM   Filial
                         WHERE  nome = 'Filial Centro'
                        );
```

**OBS:** com os dados atuais inseridos no banco, neum registro é retornado. Na última subquery, substitua 'Filial Centro' por 'Filial Savassi'. Explique a diferença de resultados.


### Subquery Correlacionando Consigo Mesma

A versão abaixo não exclui o próprio produto referência "Arroz Tipo 1 5kg": ele aparece no resultado, porque satisfaz a própria condição.

```sql
SELECT id, nome, cnpj_fornecedor
FROM   Produto
WHERE  cnpj_fornecedor = (
                           SELECT cnpj_fornecedor
                           FROM   Produto
                           WHERE  nome = 'Arroz Tipo 1 5kg'
                         );
```
Se for necessário ver só os outros produtos do mesmo fornecedor, sem incluir o produto de referência, basta adicionar uma segunda condição:

```sql
SELECT id, nome, cnpj_fornecedor
FROM   Produto
WHERE  cnpj_fornecedor = (
                           SELECT cnpj_fornecedor
                           FROM   Produto
                           WHERE  nome = 'Arroz Tipo 1 5kg'
                         )
  AND  nome <> 'Arroz Tipo 1 5kg'; -- <> e != são os operadores 'diferente'.
```

### Subqueries e Funções de Agregação

Apresentar os dados do produto mais caro em estoque,   

```sql   
   SELECT *
   FROM   Produto
   WHERE  id = (
                SELECT id_produto
                FROM   Estoque 
                WHERE  preco = (
                                SELECT MAX(preco)
                                FROM   Estoque
                               )
               );	
```

### SELECT Combinando Duas Tabelas (Sem JOIN)

1. Sem o `JOIN` explícito, é possível combinar tabelas listando-as no `FROM` separadas por vírgula e relacionando-as no `WHERE`. 

2. Funciona, mas é considerado um estilo antigo.

```sql
SELECT p.nome AS produto, f.nome AS fornecedor
FROM Produto p, Fornecedor f
WHERE p.cnpj_fornecedor = f.cnpj;
```

> **OBS:** essa consulta produz exatamente o mesmo resultado que um `INNER JOIN`.
> Mas é considerada uma forma antiga e menos legível de escrever a mesma coisa. 
> Atualmente, prefira sempre a sintaxe explícita com `JOIN`.

### SELECT Combinando Três Tabelas

Um exemplo mais completo: nome do produto, nome da filial, preço e quantidade em estoque, relacionando informação de `Produto`, `Filial` e `Estoque` em uma única consulta.

```sql
SELECT p.nome AS Produto, 
       f.nome AS Filial, 
       e.preco AS Preço, 
       e.quantidade As Quantidade
FROM Produto p, 
     Estoque e, 
     Filial f
WHERE p.id = e.id_produto 
  AND e.cnpj_filial = f.cnpj
ORDER BY p.nome, f.nome;
```

### SELECT com DISTINCT entre Tabelas

Para saber quais fornecedores **realmente têm** produtos cadastrados (sem repetir o mesmo fornecedor várias vezes, caso ele tenha mais de um produto):

```sql
SELECT DISTINCT f.nome
FROM Fornecedor f, Produto p
WHERE f.cnpj = p.cnpj_fornecedor;
```

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="09-relacionamentos.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="11-select.md">Próximo</a>
    </td>
  </tr>
</table>
