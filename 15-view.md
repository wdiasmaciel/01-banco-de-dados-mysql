# 15 - Visões com VIEW

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="14-alter.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="16-grant.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Comando CREATE VIEW

Uma visão (`VIEW`) é uma tabela virtual definida por uma consulta `SELECT`. Ela não duplica os dados da tabela base: os resultados são calculados quando a visão é consultada.

Uma visão pode simplificar consultas, esconder colunas sensíveis e padronizar a forma como determinadas informações são apresentadas.

Os exemplos desta aula usam as tabelas `Produto` e `Estoque` do projeto. A coluna `ativo` corresponde um campo adicional criado na aula prática anterior sobre `ALTER TABLE`.

```sql
CREATE VIEW vw_produtos_ativos AS
SELECT p.id,
       p.nome,
       e.cnpj_filial,
       e.preco
FROM   Produto p
JOIN   Estoque e 
ON     e.id_produto = p.id
WHERE  p.ativo = TRUE;
```

A visão pode ser usada em uma consulta como uma tabela:

```sql
SELECT nome, preco
FROM   vw_produtos_ativos
ORDER BY nome;
```

Para substituir uma visão já existente, mantendo o mesmo nome, usamos `CREATE OR REPLACE VIEW`:

```sql
CREATE OR REPLACE VIEW vw_produtos_ativos AS
SELECT p.id,
       p.nome,
       e.cnpj_filial,
       e.preco,
       p.cnpj_fornecedor
FROM   Produto p
JOIN   Estoque e 
ON     e.id_produto = p.id
WHERE  p.ativo = TRUE;
```

Se o nome já estiver sendo usado por outro tipo de objeto, o comando não será executado.

```sql
SELECT *
FROM vw_produtos_ativos
```

---

## Nomes de colunas da visão

O MySQL pode usar os nomes definidos pela consulta. Também é possível definir nomes explícitos na criação da visão:

```sql
CREATE VIEW vw_resumo_fornecedor
            (codigo, nome_fornecedor, quantidade_produtos) AS
SELECT    f.cnpj,
          f.nome,
          COUNT(p.id)
FROM      Fornecedor f
LEFT JOIN Produto p 
ON        p.cnpj_fornecedor = f.cnpj
GROUP BY  f.cnpj, f.nome;
```

```sql
SELECT *
FROM vw_resumo_fornecedor
```

Usar a crase para carcteres espeiciais:

```sql
DROP VIEW vw_resumo_fornecedor;
```

```sql
CREATE VIEW vw_resumo_fornecedor
            (`còdigo`, `nome do fornecedor`, `quantidade de produtos`) AS
SELECT    f.cnpj,
          f.nome,
          COUNT(p.id)
FROM      Fornecedor f
LEFT JOIN Produto p 
ON        p.cnpj_fornecedor = f.cnpj
GROUP BY  f.cnpj, f.nome;
```

```sql
SELECT *
FROM vw_resumo_fornecedor
```

Todas as expressões devem possuir nomes ou aliases. Por exemplo, uma função de agregação precisa ser identificada:

```sql
CREATE VIEW vw_valor_estoque AS
SELECT id_produto,
       SUM(quantidade * preco) AS valor_total
FROM   Estoque
GROUP BY id_produto
ORDER BY valor_total DESC;
```

```sql
SELECT *
FROM vw_valor_estoque
```

---

## Tipos de visão

### Visão simples

Uma visão simples consulta diretamente uma única tabela e apresenta colunas existentes:

```sql
CREATE VIEW vw_contatos_fornecedor AS
SELECT cnpj, nome, telefone
FROM   Fornecedor;
```

### Visão por junção

Uma visão pode reunir dados de várias tabelas com `JOIN`:

```sql
CREATE VIEW vw_produto_estoque AS
SELECT p.id,
       p.nome,
       e.cnpj_filial,
       e.quantidade,
       e.preco
FROM   Produto p
JOIN   Estoque e ON e.id_produto = p.id;
```

### Visão agregada

Visões agregadas podem usar `SUM`, `COUNT`, `AVG`, `MIN` ou `MAX`. Elas são úteis para relatórios:

```sql
CREATE VIEW vw_total_por_fornecedor AS
SELECT cnpj_fornecedor,
       COUNT(*) AS quantidade_produtos
FROM   Produto
GROUP BY cnpj_fornecedor;
```

---

## Visões atualizáveis

Uma visão atualizável permite comandos como `INSERT`, `UPDATE` e `DELETE`, mas existem restrições.

Normalmente são atualizáveis visões simples que não usam:

- funções de agregação;
- `GROUP BY`, `HAVING` ou `DISTINCT`;
- união de conjuntos;
- determinadas subconsultas;
- atributos não atualizáveis.

Exemplo:

```sql
CREATE VIEW vw_produtos_em_destaque AS
SELECT p.id,
       p.nome,
       e.cnpj_filial,
       e.preco
FROM   Produto p
JOIN   Estoque e ON e.id_produto = p.id
WHERE  p.ativo = TRUE;
```

```sql
UPDATE Estoque
SET    preco = 20.00
WHERE  id_produto = 1
AND    cnpj_filial = '10101010000110';
```

Como esta visão utiliza `JOIN`, ela pode não ser atualizável. Neste caso, a alteração deve ser feita diretamente na tabela base `Estoque`, que é onde a coluna `preco` está armazenada.

### WITH CHECK OPTION

`WITH CHECK OPTION` impede que uma alteração faça a linha deixar de atender à condição da visão.

```sql
CREATE VIEW vw_produtos_baratos AS
SELECT id_produto, cnpj_filial, preco
FROM   Estoque
WHERE  preco < 50
WITH CHECK OPTION;
```

Tentar alterar `preco` para um valor maior ou igual a 50 será rejeitado:

```sql
UPDATE vw_produtos_baratos
SET    preco = 60
WHERE  id_produto = 1
AND    cnpj_filial = '10101010000110';
```

Em visões encadeadas, `LOCAL` verifica somente a condição da visão atual; `CASCADED`, que é o padrão, verifica também as condições das visões das quais ela depende:

```sql
CREATE VIEW vw_produtos_baratos_ativos AS
SELECT e.id_produto, e.cnpj_filial, p.nome, e.preco
FROM   vw_produtos_baratos e
JOIN   Produto p ON p.id = e.id_produto
WHERE  p.ativo = TRUE
WITH LOCAL CHECK OPTION;
```

---

## Definidor e usuário da visão

Por padrão, o MySQL usa `SQL SECURITY DEFINER`, ou seja, a visão é executada com os privilégios do usuário que a criou.

É possível escolher o usuário que executa a consulta:

```sql
CREATE DEFINER = CURRENT_USER
    SQL SECURITY DEFINER
VIEW vw_produto_estoque_publico AS
SELECT p.id, p.nome, e.quantidade
FROM   Produto p
JOIN   Estoque e ON e.id_produto = p.id;
```

A opção `SQL SECURITY INVOKER` faz a consulta ser executada com os privilégios de quem usa a visão:

```sql
CREATE SQL SECURITY INVOKER VIEW vw_fornecedores_ativos AS
SELECT cnpj, nome, telefone
FROM   Fornecedor
WHERE  ativo = TRUE;
```

---

## Consultando e removendo visões

`SHOW TABLES` lista também as visões. A consulta a `information_schema.views` fornece a definição:

```sql
SHOW FULL TABLES;

SELECT table_name, view_definition
FROM   information_schema.views
WHERE  table_schema = DATABASE();
```

Para consultar a definição completa de uma visão criada no MySQL:

```sql
SHOW CREATE VIEW vw_produtos_ativos;
```

O comando `DROP VIEW` remove a visão, mas não remove os dados nem a tabela base:

```sql
DROP VIEW vw_produtos_ativos;
```

Quando não se sabe se a visão existe, pode-se usar `IF EXISTS`:

```sql
DROP VIEW IF EXISTS vw_produtos_ativos;
```

---

## Boas práticas

1. Use nomes claros, como `vw_cliente_resumo`, para diferenciar visões de tabelas.
2. Evite visões aninhadas desnecessárias.
3. Não espere que toda visão seja atualizável.
4. Revise permissões e testes de segurança ao usar `SQL SECURITY DEFINER`.
5. Confira a documentação da versão do MySQL utilizada, pois recursos e comportamentos podem variar.

---

# Exercícios

1. Crie uma visão chamada `vw_pao_disponivel` que liste o nome e o preço dos pães cadastrados como disponíveis.
2. Consulte a visão criada por você e ordene os pães pelo nome.
3. Crie uma visão que mostre apenas o código, o nome e o telefone dos clientes de uma padaria.
4. O dono da padaria precisa ver o nome dos ingredientes e suas quantidades em estoque. Crie uma visão para esse relatório.
5. Crie uma visão agregada com o valor vendido por dia em uma padaria.
6. Crie uma visão que calcule o preço médio dos produtos da categoria “Bebidas”.
7. Crie uma visão que mostre medicamentos com validade superior à data atual em uma farmácia.
8. Crie uma visão que junte venda e medicamento, exibindo o nome do remédio e a quantidade vendida.
9. Crie uma visão com nomes explícitos para um relatório de passagens aéreas.
10. Crie uma visão atualizável que permita corrigir telefones dos clientes de uma agência de viagens.
11. Crie uma visão que permita atualizar somente pacotes turísticos com vagas disponíveis.
12. Restrinja essa visão de pacotes com `WITH CHECK OPTION` para impedir alterações que removam pacotes da visualização.
13. Crie uma visão com `SQL SECURITY INVOKER` para a tabela de clientes de um supermercado.
14. Explique por que uma visão baseada em `SUM(valor)` normalmente não deve receber `UPDATE`.
15. Liste todas as visões do banco de dados atual usando metadados.
16. Obtenha o texto completo da visão que mostra os fornecedores e seus produtos.
17. Tente substituir uma visão existente com `CREATE OR REPLACE VIEW` para incluir uma nova coluna de relatório.
18. Remova uma visão de produtos de baixo estoque sem apagar a tabela de estoque.
19. Crie uma visão de auditoria que nunca permita atualizações, pois contém `GROUP BY` e `COUNT`.
20. Crie uma visão para um relatório gerencial de vendas e decida quais privilégios ela deve ter.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="14-alter.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="16-grant.md">Próximo</a>
    </td>
  </tr>
</table>
