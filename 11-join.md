<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="10-select-entre-tabelas.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="12-select.md">Próximo</a>
    </td>
  </tr>
</table>

---

## 5. JOIN (Junção)

Para simplificar, todos os exemplos desta seção usam apenas duas tabelas: **Fornecedor** e **Produto**. 

### 5.1 INNER JOIN

Retorna **apenas** as linhas em que há correspondência nas duas tabelas.

Ou seja: só aparecem fornecedores que **têm** pelo menos um produto, e só aparecem produtos que **têm** um fornecedor correspondente.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
INNER JOIN Produto p 
ON f.cnpj = p.cnpj_fornecedor 
ORDER BY f.nome;
```

*Fornecedores sem produto (Epsilon, Zeta, Theta) **não aparecem** no resultado.*

### 5.2 LEFT JOIN (ou LEFT OUTER JOIN)

Retorna **todas** as linhas da tabela à esquerda (`Fornecedor`), mesmo que não haja correspondência em `Produto` — nesse caso, as colunas vindas de `Produto` aparecem como `NULL`.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
LEFT JOIN Produto p ON p.cnpj_fornecedor = f.cnpj
ORDER BY f.nome;
```

*Agora Epsilon, Zeta e Theta aparecem no resultado, com `produto = NULL`.*

### 5.3 RIGHT JOIN (ou RIGHT OUTER JOIN)

O espelho do `LEFT JOIN`: retorna **todas** as linhas da tabela à direita (`Produto`), mesmo sem correspondência em `Fornecedor`. Como, no nosso projeto, `cnpj_fornecedor` é `NOT NULL` (todo produto obrigatoriamente tem um fornecedor), esse exemplo específico produz o **mesmo resultado** de um `INNER JOIN` — mas o comando serve para ilustrar a sintaxe e o conceito.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
RIGHT JOIN Produto p ON p.cnpj_fornecedor = f.cnpj
ORDER BY p.nome;
```

> **Ponto para explorar em aula:** peça aos alunos para explicarem por que, nesse caso específico, `RIGHT JOIN` e `INNER JOIN` dão o mesmo resultado — e em que situação (se `cnpj_fornecedor` pudesse ser `NULL`) o resultado seria diferente.

### 5.4 CROSS JOIN

Retorna o **produto cartesiano** entre as duas tabelas — cada linha de uma tabela combinada com **todas** as linhas da outra, sem nenhuma condição de correspondência. É raramente usado no dia a dia, mas importante entender o conceito.

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
CROSS JOIN Produto p
ORDER BY f.nome, p.nome;
```

> Com 8 fornecedores e 6 produtos, esse `CROSS JOIN` retorna `8 x 6 = 48` linhas — a maioria delas sem sentido no contexto do projeto (um produto "combinado" com um fornecedor que não é o dele). Bom exemplo para mostrar por que **sempre** usamos uma condição `ON` nos outros tipos de `JOIN`.

### 5.5 "FULL OUTER JOIN" no MySQL

O MySQL **não possui** o comando `FULL OUTER JOIN` nativamente (diferente de PostgreSQL e SQL Server). Para simular esse comportamento — trazer tanto os fornecedores sem produto quanto (hipotéticos) produtos sem fornecedor — combinamos um `LEFT JOIN` e um `RIGHT JOIN` com `UNION`:

```sql
SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
LEFT JOIN Produto p ON p.cnpj_fornecedor = f.cnpj

UNION

SELECT f.nome AS fornecedor, p.nome AS produto
FROM Fornecedor f
RIGHT JOIN Produto p ON p.cnpj_fornecedor = f.cnpj;
```

> `UNION` (sem `ALL`) também elimina automaticamente as linhas duplicadas entre os dois resultados — outro conceito que vale reforçar aqui.

---

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="10-select-entre-tabelas.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="12-select.md">Próximo</a>
    </td>
  </tr>
</table>
