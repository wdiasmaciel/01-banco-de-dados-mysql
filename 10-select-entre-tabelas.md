<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="09-relacionamentos.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

## Exemplos de SELECT entre tabelas

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

### SELECT combinando duas tabelas (sem join)

1. Sem o `JOIN` explícito, é possível combinar tabelas listando-as no `FROM` separadas por vírgula e relacionando-as no `WHERE`. 

2. Funciona, mas é considerado um estilo antigo.

```sql
SELECT p.nome AS produto, f.nome AS fornecedor
FROM Produto p, Fornecedor f
WHERE p.cnpj_fornecedor = f.cnpj;
```

> **OBS:** essa consulta produz exatamente o mesmo resultado que um `INNER JOIN`.
> Mas é considerada uma forma antiga e menos legível de escrever a mesma coisa. 
> Hoje em dia, prefira sempre a sintaxe explícita com `JOIN`.

### SELECT combinando três tabelas

Um exemplo mais completo: nome do produto, nome da filial, preço e quantidade em estoque, relacionando informação de `Produto`, `Filial` e `Estoque` em uma única consulta.

```sql
SELECT p.nome AS Produto, f.nome AS Filial, e.preco AS Preço, e.quantidade As Quantidade
FROM Produto p, Estoque e, Filial f
WHERE p.id = e.id_produto 
  AND e.cnpj_filial = f.cnpj
ORDER BY p.nome, f.nome;
```

### 3.5 SELECT com DISTINCT entre tabelas

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
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
