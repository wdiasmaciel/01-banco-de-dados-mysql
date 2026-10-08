<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="10-select-entre-tabelas.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="12-grup-by-having.md">Próximo</a>
    </td>
  </tr>
</table>

---

## JOIN (Junção)

Para simplificar, todos os exemplos desta seção usam apenas duas tabelas: **Fornecedor** e **Produto**. 

### INNER JOIN

Retorna **apenas** as linhas em que há correspondência nas duas tabelas.

Ou seja: só aparecem fornecedores que **têm** pelo menos um produto, e só aparecem produtos que **têm** um fornecedor correspondente.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
INNER JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor 
ORDER BY f.nome;
```

ou

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor 
ORDER BY f.nome;
```

**OBS**:

*Fornecedores sem produto (Epsilon, Zeta, Theta) **não aparecem** no resultado.*


```sql
SELECT
    f.nome        AS fornecedor,
    p.nome        AS produto,
    i.descricao   AS descricao_produto,
    fl.nome       AS filial,
    e.preco,
    e.quantidade,
    e.validade
FROM       Produto p
INNER JOIN Fornecedor    f  ON p.cnpj_fornecedor = f.cnpj
INNER JOIN Identificacao i  ON p.id              = i.id
INNER JOIN Estoque       e  ON p.id              = e.id_produto
INNER JOIN Filial        fl ON e.cnpj_filial     = fl.cnpj
ORDER BY f.nome, p.nome, fl.nome;
```

ou

```sql
SELECT
    f.nome        AS fornecedor,
    p.nome        AS produto,
    i.descricao   AS descricao_produto,
    fl.nome       AS filial,
    e.preco,
    e.quantidade,
    e.validade
FROM       Produto p
JOIN Fornecedor    f  ON p.cnpj_fornecedor = f.cnpj
JOIN Identificacao i  ON p.id              = i.id
JOIN Estoque       e  ON p.id              = e.id_produto
JOIN Filial        fl ON e.cnpj_filial     = fl.cnpj
ORDER BY f.nome, p.nome, fl.nome;
```


### LEFT JOIN (ou LEFT OUTER JOIN)

Retorna **todas** as linhas da tabela à **esquerda** (`Fornecedor`), mesmo que não haja correspondência na tabela da direita, que é a tabela `Produto`. Nesse caso, as colunas vindas de `Produto` aparecem como `NULL`.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
LEFT JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor
ORDER BY f.nome;
```

ou

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
LEFT OUTER JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor
ORDER BY f.nome;
```

**OBS**:

*Agora Epsilon, Zeta e Theta aparecem no resultado, com `produto = NULL`.*

### RIGHT JOIN (ou RIGHT OUTER JOIN)

O espelho do `LEFT JOIN`: retorna **todas** as linhas da tabela à **direita** (`Produto`), mesmo sem correspondência na tabela da esquerda, que é a tabela `Fornecedor`. 

**OBS**:

Como, no nosso projeto, `cnpj_fornecedor` é `NOT NULL` (todo produto obrigatoriamente tem um fornecedor), esse exemplo específico produz o **mesmo resultado** de um `INNER JOIN`, mas o comando serve para ilustrar a sintaxe e o conceito.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
RIGHT JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor
ORDER BY p.nome;
```

ou

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
RIGHT OUTER JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor
ORDER BY p.nome;
```

> **OBS:** explique por que, nesse caso específico, `RIGHT JOIN` e `INNER JOIN` dão o mesmo resultado. Em que situação (se `cnpj_fornecedor` pudesse ser `NULL`) o resultado seria diferente.


### CROSS JOIN

Retorna o **produto cartesiano** entre as duas tabelas. Cada linha de uma tabela combinada com **todas** as linhas da outra, sem nenhuma condição de correspondência. 

É raramente usado no dia a dia, mas importante entender o conceito.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
CROSS JOIN Produto p
ORDER BY f.nome, p.nome;
```

> Com 8 fornecedores e 6 produtos, esse `CROSS JOIN` retorna `8 x 6 = 48` linhas, a maioria delas sem sentido no contexto do projeto (um produto "combinado" com um fornecedor que não é o dele). Exemplifica a importância de **sempre** usarmos uma condição `ON` nos outros tipos de `JOIN`.

```sql
SELECT count(*)
FROM Fornecedor f
CROSS JOIN Produto p
ORDER BY f.nome, p.nome;
```

O produto cartesiano também pode ser obtido da seguinte forma:

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f, Produto p
ORDER BY f.nome, p.nome;
```

```sql
SELECT count(*)
FROM Fornecedor f, Produto p
ORDER BY f.nome, p.nome;
```


### "FULL OUTER JOIN" no MySQL

O MySQL **não possui** o comando `FULL OUTER JOIN` nativamente (diferente de PostgreSQL e SQL Server). 

Para simular esse comportamento. trazer tanto os fornecedores sem produto quanto (hipotéticos) produtos sem fornecedor, combinamos um `LEFT JOIN` e um `RIGHT JOIN` com `UNION`:

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
LEFT JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor

UNION

SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
RIGHT JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor
```

Contanto o número de registros retornados:

```sql
SELECT COUNT(*) AS total_linhas
FROM (
    SELECT f.nome AS fornecedor, p.nome AS produto
    FROM Fornecedor f
    LEFT JOIN Produto p 
    ON f.cnpj = p.cnpj_fornecedor

    UNION

    SELECT f.nome AS fornecedor, p.nome AS produto
    FROM Fornecedor f
    RIGHT JOIN Produto p 
    ON f.cnpj = p.cnpj_fornecedor
) AS resultado_uniao;
```

**OBS**:

> `UNION` (sem `ALL`) também elimina automaticamente as linhas duplicadas entre os dois resultados.


```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
LEFT JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor

UNION ALL

SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
RIGHT JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor
```

Contanto o número de registros retornados:

```sql
SELECT COUNT(*) AS total_linhas
FROM (
    SELECT f.nome AS fornecedor, p.nome AS produto
    FROM Fornecedor f
    LEFT JOIN Produto p 
    ON f.cnpj = p.cnpj_fornecedor

    UNION ALL

    SELECT f.nome AS fornecedor, p.nome AS produto
    FROM Fornecedor f
    RIGHT JOIN Produto p 
    ON f.cnpj = p.cnpj_fornecedor
) AS resultado_uniao;
```

**OBS**:

Pontos importantes:

1. O alias da subquery é obrigatório. O MySQL exige que toda tabela derivada no FROM tenha um nome (`AS resultado_uniao`, nos exemplos). Sem ele, o comando falha com erro de sintaxe.

2. `COUNT(*)` conta todas as linhas, incluindo as que têm `NULL`. Como essa consulta usa `LEFT JOIN` e `RIGHT JOIN`, é provável que apareçam linhas com `produto = NULL` (fornecedores sem produto) ou `fornecedor = NULL` (produtos sem fornecedor, se existissem). 

3. COUNT(*) conta essas linhas normalmente, diferentemente de COUNT(coluna), que ignora NULLs daquela coluna específica.

4. Por que `UNION ALL` e não `UNION`? 
  - `UNION ALL` mantém linhas duplicadas entre os dois SELECTs (o que provavelmente infla a contagem, já que, como vimos antes, `RIGHT JOIN` e `LEFT JOIN` produzem resultados sobrepostos quando `cnpj_fornecedor` é `NOT NULL`. 
  - Se você quiser contar apenas combinações distintas, use `UNION` (sem `ALL`) dentro da subquery.

---
# Exercícios

Nos exercícios abaixo, empregue a instrução `JOIN`.

> Observe como os `NULL`s aparecem em `LEFT JOIN` e `RIGHT JOIN`.
>
> Gere os `creates` e os `inserts` necessários.

## 1) Joins básicos no banco empresa

1. Liste o nome do fornecedor e o nome do produto para todos os fornecedores que possuem produtos cadastrados.
2. Mostre todos os fornecedores e, se existirem, os produtos relacionados. Inclua também os fornecedores sem produto.
3. Exiba todos os produtos e, quando houver, o nome do fornecedor correspondente.
4. Verifique quais fornecedores não possuem nenhum produto cadastrado.
5. Apresente os fornecedores, seus produtos e a identificação dos produtos.

## 2) Joins com filtros e ordenação

6. Liste somente os produtos cujo preço seja maior que R$ 50,00 junto com o nome do fornecedor.
7. Mostre os fornecedores cujo nome começa com "A" e os produtos relacionados, ordenados por fornecedor e produto.
8. Exiba os fornecedores e produtos em ordem alfabética decrescente por fornecedor.
9. Liste os fornecedores com seus produtos, mas somente quando o produto tiver preço entre R$ 20,00 e R$ 100,00.
10. Escreva uma consulta que retorne os nomes dos fornecedores, os nomes dos produtos desses fornecedores, a quantidade desses produtos no estoque e as filiais em que esses produtos são vendidos. Inclua também os fornecedores sem produtos.

## 3) Banco de uma padaria

Considere as tabelas `Cliente`, `Pedido` e `Produto`:

- `Cliente(id_cliente, nome, telefone)`
- `Pedido(id_pedido, id_cliente, data_pedido, total)`
- `Produto(id_produto, nome, preco)`
- `ItemPedido(id_pedido, id_produto, quantidade)`

11. Liste todos os clientes e os pedidos realizados por eles, incluindo clientes que ainda não fizeram pedidos.
12. Mostre cada pedido com o nome do cliente e o valor total do pedido.
13. Liste todos os produtos vendidos em cada pedido, com o nome do produto, quantidade e valor total do item.
14. Descubra quais clientes nunca fizeram pedido.
15. Gere uma consulta que mostre o nome do cliente, o número do pedido e o total do pedido, considerando apenas pedidos com valor acima de R$ 80,00.

## 4) Banco de uma agência de viagens

Considere as tabelas `Cliente`, `Reserva` e `Destino`:

- `Cliente(id_cliente, nome, email)`
- `Reserva(id_reserva, id_cliente, id_destino, data_viagem, valor)`
- `Destino(id_destino, nome_destino, pais)`

16. Liste todos os clientes e suas reservas, incluindo clientes sem reservas.
17. Mostre cada reserva com o nome do cliente e o nome do destino.
18. Exiba os destinos que ainda não receberam reserva.
19. Exiba os clientes que ainda não fizeram reserva.
20. Exiba todos os clientes e destinos, com ou sem de reserva.

## 5) Banco de uma farmácia

Considere as tabelas `Medicamento`, `FornecedorFarmacia` e `Compra`:

- `Medicamento(id_medicamento, nome, preco, validade, laboratorio)`
- `Fornecedor(id_fornecedor, nome, cidade)`
- `Compra(id_compra, id_medicamento, id_fornecedor, quantidade, data_compra)`

21. Liste todos os medicamentos com o nome do fornecedor que os forneceu, incluindo medicamentos que ainda não tiveram compra registrada.

22. Liste todos os medicamentos sem forecedores e que foram comprados.

23. Liste todos os medicamentos com forecedores e que ainda não foram comprados.

24. Liste todos os medicamentos sem forecedores e que ainda não foram comprados.

25. Liste todos os dados das compras, dos medicamentos e dos fornecedores, com ou sem registro de fornecedor e de compra.

## 6) Banco de uma clínica

Considere as tabelas `Paciente`, `Consulta` e `Medico`:

- `Paciente(id_paciente, nome, telefone)`
- `Medico(id_medico, nome, especialidade)`
- `Consulta(id_consulta, id_paciente, id_medico, data_consulta, valor)`

26. Liste todos os pacientes e suas consultas, incluindo pacientes sem consultas.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="10-select-entre-tabelas.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="12-grup-by-having.md">Próximo</a>
    </td>
  </tr>
</table>
