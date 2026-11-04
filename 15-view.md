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

```sql
SELECT *
FROM   vw_produtos_ativos;
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
FROM   vw_produtos_ativos;
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
FROM   vw_resumo_fornecedor;
```

Usar crases para nomes de colunas com caracteres especiais:

```sql
DROP VIEW vw_resumo_fornecedor;
```

```sql
CREATE VIEW vw_resumo_fornecedor
            (`codigo`, `nome do fornecedor`, `quantidade de produtos`) AS
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
FROM   vw_resumo_fornecedor;
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
FROM   vw_valor_estoque;
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

```sql
SELECT *
FROM   vw_contatos_fornecedor;
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
JOIN   Estoque e 
ON     e.id_produto = p.id;
```

```sql
SELECT *
FROM   vw_produto_estoque;
```

### Visão baseada em agregação (visão agregada)

Visões baseadas em agregações podem usar `SUM`, `COUNT`, `AVG`, `MIN` ou `MAX`. 

Elas são úteis para relatórios:

```sql
CREATE VIEW vw_total_por_fornecedor AS
SELECT cnpj_fornecedor,
       COUNT(*) AS quantidade_produtos
FROM   Produto
GROUP BY cnpj_fornecedor
ORDER BY quantidade_produtos DESC;
```

```sql
SELECT *
FROM   vw_total_por_fornecedor;
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
JOIN   Estoque e 
ON     e.id_produto = p.id
WHERE  p.ativo = TRUE;
```

```sql
SELECT *
FROM   vw_produtos_em_destaque;
```

```sql
UPDATE vw_produtos_em_destaque
SET    preco = 20.00
WHERE  id = 1
AND    cnpj_filial = '10101010000110';
```

```sql
SELECT *
FROM   vw_produtos_em_destaque;
```

No caso de um visão não poder ser atualizada, a alteração deve ser feita diretamente na tabela base (no exemplo: `Estoque`, que é onde a coluna `preco` está armazenada):

```sql
UPDATE Estoque
SET    preco = 25.00
WHERE  id_produto = 1
AND    cnpj_filial = '10101010000110';
```

```sql
SELECT *
FROM   vw_produtos_em_destaque;
```

### WITH CHECK OPTION

`WITH CHECK OPTION` impede que uma alteração faça a linha deixar de atender à condição da visão.

```sql
CREATE VIEW vw_produtos_baratos AS
SELECT id_produto, cnpj_filial, preco
FROM   Estoque
WHERE  preco < 10
WITH CHECK OPTION;
```

```sql
SELECT *
FROM   vw_produtos_baratos;
```

Tentar alterar `preco` para um valor maior ou igual a 10 será rejeitado:

```sql
UPDATE vw_produtos_baratos
SET    preco = 60
WHERE  id_produto = 1
AND    cnpj_filial = '30303030000130';
```

Em visões encadeadas, `LOCAL` verifica somente a condição da visão atual; `CASCADED`, que é o padrão, verifica também as condições das visões das quais ela depende:

```sql
CREATE VIEW vw_produtos_baratos_ativos AS
SELECT e.id_produto, e.cnpj_filial, p.nome, e.preco
FROM   vw_produtos_baratos e
JOIN   Produto p 
ON     p.id = e.id_produto
WHERE  p.ativo = TRUE
WITH LOCAL CHECK OPTION;
```

```sql
SELECT *
FROM   vw_produtos_baratos_ativos;
```

---

<!--
## Definidor e usuário da visão

Uma visão pode ser executada com dois conjuntos de privilégios diferentes:

- `SQL SECURITY DEFINER`: a consulta usa os privilégios do usuário definido como criador da visão.
- `SQL SECURITY INVOKER`: a consulta usa os privilégios de quem executa o `SELECT` na visão.

Criador (*definer*) é o usuário responsável por criar o objeto. 

Invocador (*invoker*) é a conta que consulta a visão depois que ela já foi criada.

### SQL SECURITY DEFINER

No MySQL, `DEFINER` informa qual usuário representa o criador da visão. 

Em uma sessão comum, `CURRENT_USER` corresponde ao usuário conectado que executa o `CREATE VIEW`.

Por exemplo, suponha que o administrador tenha criado esta visão:

```sql
CREATE DEFINER = CURRENT_USER
    SQL SECURITY DEFINER
VIEW vw_produto_estoque_publico AS
SELECT p.id, p.nome, e.quantidade
FROM   Produto p
JOIN   Estoque e 
ON     e.id_produto = p.id;
```

Nesse exemplo:

- O criador da visão é o usuário atual da sessão;
- A visão guarda o usuário criador como seu `DEFINER`;
- Ao consultar a visão, o MySQL executa internamente o `JOIN` com os privilégios desse criador;
- O usuário que consulta a visão precisa possuir pelo menos `SELECT` sobre a própria visão, mas normalmente não precisa possuir acesso direto às tabelas `Produto` e `Estoque`.

Na identificação `'consulta_produto'@'localhost'`, `consulta_produto` é o nome da conta de usuário criada no MySQL, e `localhost` indica que ela pode se conectar a partir do próprio servidor, ou seja, a máquina local.

Por exemplo, o nome pode ser escolhido de acordo com a finalidade da conta:

```sql
CREATE USER 'consulta_produto'@'localhost'
IDENTIFIED BY 'SenhaForte123!';
```

Os nomes das contas podem variar. Em vez de `consulta_produto`, poderia ser usado, por exemplo, `'joao'@'localhost'`. O nome `consulta_produto` apenas indica, neste exemplo, que essa conta será utilizada para consultar produtos. Ele não concede automaticamente nenhuma permissão.

Primeiro, o usuário administrador concede acesso à visão:

```sql
GRANT SELECT
ON empresa.vw_produto_estoque_publico
TO 'consulta_produto'@'localhost';
```

Depois, a conta de consulta pode verificar os dados:

```sql
SELECT *
FROM   vw_produto_estoque_publico;
```

Para criar e consultar a visão com sucesso, o criador normalmente precisa possuir `SELECT` nas tabelas que a compõem. 

Caso o criador não tenha esses privilégios, o MySQL impede a criação ou o uso do objeto.

### SQL SECURITY INVOKER

Com `SQL SECURITY INVOKER`, a conta que executa o `SELECT` sobre a visão precisa de autorização tanto sobre a visão quanto sobre todas as tabelas usadas pela consulta interna.

```sql
CREATE SQL SECURITY INVOKER VIEW vw_fornecedores_ativos AS
SELECT cnpj, nome, telefone
FROM   Fornecedor
WHERE  ativo = TRUE;
```

O proprietário da conta `consulta_fornecedor` precisa receber os dois tipos de privilégio:

```sql
GRANT SELECT
ON empresa.vw_fornecedores_ativos
TO 'consulta_fornecedor'@'localhost';
```

```sql
GRANT SELECT (cnpj, nome, telefone)
ON empresa.Fornecedor
TO 'consulta_fornecedor'@'localhost';
```

A consulta pode então ser executada pela conta autorizada:

```sql
SELECT *
FROM   vw_fornecedores_ativos;
```

Se `consulta_fornecedor` tiver acesso à visão, mas não tiver acesso às colunas `cnpj`, `nome` ou `telefone` da tabela `Fornecedor`, o `SELECT` deverá ser negado. 

Em geral, o acesso por uma visão não permite contornar os privilégios das tabelas quando `SQL SECURITY INVOKER` está ativo.

### Resumo

| Segurança da visão | Quem executa a consulta interna? | Privilégio mínimo do usuário da visão | Tabelas base são necessárias ao invocador? |
|---|---|---|---|
| `DEFINER` | Usuário definido como criador | `SELECT` sobre a visão | Normalmente não |
| `INVOKER` | Usuário que executa o `SELECT` | `SELECT` sobre a visão | Sim, para os objetos usados pela consulta |

O nome `DEFINER` não concede automaticamente os privilégios da tabela base. Ele indica quais privilégios serão usados; esses privilégios ainda precisam existir. Da mesma forma, criar uma visão não concede acesso a quem a utiliza: é preciso executar `GRANT SELECT` sobre a própria visão.

### Qual modo de segurança é usado por padrão?

Uma visão pode ser criada com um dos seguintes modos de segurança:

- `SQL SECURITY DEFINER`: a consulta interna usa os privilégios do usuário definido como criador da visão.
- `SQL SECURITY INVOKER`: a consulta interna usa os privilégios de quem executa o `SELECT` na visão.

Ao criar uma visão sem informar o modo, o MySQL usa `SQL SECURITY DEFINER` por padrão. O criador da visão também pode ser indicado explicitamente com `DEFINER`, por exemplo:

```sql
CREATE DEFINER = CURRENT_USER
    SQL SECURITY DEFINER
VIEW vw_exemplo AS
SELECT codigo, nome
FROM   Produto;
```

### Definer é mais seguro ou mais restritivo que Invoker?

Não necessariamente. Os modos atendem a objetivos diferentes:

- Em uma visão com `DEFINER`, o usuário precisa possuir normalmente apenas acesso à própria visão, pois a consulta interna utiliza os privilégios do criador.
- Em uma visão com `INVOKER`, o usuário precisa possuir privilégios sobre a visão e sobre os objetos usados pela consulta interna.

A escolha depende da política de acesso. `DEFINER` permite que uma conta administrativa controle os dados disponibilizados por meio da visão, mas pode ser arriscado se o criador possuir privilégios excessivos. `INVOKER` tende a respeitar melhor os privilégios individuais, mas exige um gerenciamento mais detalhado das tabelas base.

Esse assunto é apresentado junto das visões porque criar uma view não concede automaticamente acesso a ela nem às tabelas base. A view reúne a definição de objetos (`CREATE VIEW`), consultas (`SELECT`) e controle de privilégios (`GRANT` e `REVOKE`): a definição decide quais dados serão calculados, enquanto o criador ou invocador determina quais privilégios serão usados nessa consulta.



Qual segurança é usada por padrão ao criar uma view?
No MySQL, o padrão é SQL SECURITY DEFINER.

Na prática, se apenas CREATE VIEW ... for utilizado, a view é criada de forma equivalente a:

CREATE DEFINER = <usuário_atual>
SQL SECURITY DEFINER
VIEW vw_exemplo AS
SELECT ...;

DEFINER é mais seguro que INVOKER?
Não necessariamente.

DEFINER centraliza os privilégios no usuário que criou a view.
INVOKER exige que cada usuário consulte a view com seus próprios privilégios também nas tabelas base.
Cada modo pode ser adequado em situações diferentes:

Use DEFINER quando quiser fornecer acesso controlado aos dados de uma view sem conceder acesso direto às tabelas.
Use INVOKER quando quiser que o MySQL respeite integralmente os privilégios individuais de quem consulta.
É importante não interpretar DEFINER como sinônimo de “mais seguro”. Um criador com privilégios excessivos pode permitir a um usuário consultar dados além do necessário.

Por exemplo, conceder SELECT sobre uma view não concede automaticamente acesso às tabelas base, mas a view pode expor dados que não deveriam ser visíveis se houver DEFINER com privilégios elevados.

DEFINER é mais restritivo que INVOKER?
Também não necessariamente.

DEFINER é mais restritivo para quem consulta quanto à consulta às tabelas base, mas não exige que essa pessoa tenha privilégios diretamente nelas. Em vez disso, o criador da view precisa possuir os privilégios necessários.

Já INVOKER geralmente é mais restritivo para quem consulta: ele precisa possuir privilégio tanto sobre a view quanto sobre os objetos usados pela consulta interna.

Portanto, a diferença pode ser resumida assim:

Comando	Consulta interna executada como	Usuário da view precisa acessar tabelas base?
DEFINER	Criador da view	Normalmente não
INVOKER	Quem consulta a view	Sim
Em ambos os casos, o usuário normalmente precisa possuir pelo menos SELECT sobre a própria view.

Por que apresentar esse assunto nesta aula?
O assunto introduz uma compreensão importante sobre segurança de banco de dados e vem logo após o conteúdo sobre criação e uso de views.

Os alunos podem aprender que uma view não é apenas uma consulta reutilizável. Ela também pode representar um limite entre dados internos e usuários externos.

Neste momento, eles já conhecem:

criação de tabelas;
consultas com várias tabelas;
restrições e privilégios;
o funcionamento das views;
formas de impedir alterações com WITH CHECK OPTION.
Assim, podem relacionar views com autenticação e autorização, entendendo que:

uma view pode ocultar colunas;
a segurança pode depender de privilégios;
o criador e o usuário não precisam possuir as mesmas permissões;
uma view mal configurada pode produzir um risco de acesso indevido.
O conteúdo também prepara os próximos assuntos de administração de banco de dados, como GRANT, REVOKE e DENY, permitindo que os alunos compreendam mais do que apenas conceder e remover privilégios.
---
-->

## Consultando e removendo visões

`SHOW TABLES` lista também as visões. 

```sql
SHOW FULL TABLES;
```

A consulta a `information_schema.views` fornece a definição:

```sql
SELECT table_name, view_definition
FROM   information_schema.views
WHERE  table_schema = DATABASE();
```

**OBS**:
> DATABASE() retorna o banco de dados corrente:

```sql
SELECT DATABASE();
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
