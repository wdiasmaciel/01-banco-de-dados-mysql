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

## 3. Exemplos de SELECT entre tabelas

### 3.1 SELECT simples com WHERE e ORDER BY

Antes de combinar tabelas, revisamos o básico: filtrar e ordenar dados de uma única tabela.

```sql
SELECT nome, endereco
FROM Fornecedor
WHERE nome LIKE '%Ltda%'
ORDER BY nome ASC;
```

### 3.2 SELECT com subquery (subconsulta) entre tabelas

Aqui buscamos todos os produtos de um fornecedor específico, mas em vez de "sabermos de cor" o `cnpj`, usamos uma subquery que busca esse `cnpj` a partir do nome do fornecedor — uma forma de combinar informação de duas tabelas sem usar `JOIN` explicitamente.

```sql
SELECT nome
FROM Produto
WHERE cnpj_fornecedor = (
    SELECT cnpj FROM Fornecedor WHERE nome = 'Distribuidora Alfa Ltda'
);
```

### 3.3 SELECT combinando duas tabelas (junção "estilo antigo", com vírgula)

Antes de o `JOIN` explícito (que veremos na Seção 5) se tornar o padrão, era comum combinar tabelas listando-as no `FROM` separadas por vírgula e relacionando-as no `WHERE`. Funciona, mas é considerado estilo antigo — mostramos aqui só para contraste.

```sql
SELECT p.nome AS produto, f.nome AS fornecedor
FROM Produto p, Fornecedor f
WHERE p.cnpj_fornecedor = f.cnpj;
```

> **Ponto para explorar em aula:** essa consulta produz exatamente o mesmo resultado que um `INNER JOIN` (Seção 5.1) — é só uma forma antiga e menos legível de escrever a mesma coisa. Hoje em dia, prefira sempre a sintaxe explícita com `JOIN`.

### 3.4 SELECT combinando três tabelas

Um exemplo mais completo: nome do produto, nome da filial, preço e quantidade em estoque — juntando informação de `Produto`, `Filial` e `Estoque` em uma única consulta.

```sql
SELECT pr.nome AS produto, fl.nome AS filial, es.preco, es.quantidade
FROM Estoque es, Produto pr, Filial fl
WHERE es.id_produto = pr.id
  AND es.cnpj_filial = fl.cnpj
ORDER BY pr.nome, fl.nome;
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
