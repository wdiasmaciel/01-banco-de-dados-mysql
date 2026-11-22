# 21 - Tabelas Temporárias no MySQL

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="20-deny.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="22-trigger.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Tabelas temporárias

1. Uma tabela temporária armazena dados de trabalho durante uma conexão. 

2. Ela é útil para preparar resultados intermediários sem manter uma tabela auxiliar permanente no banco.

3. No MySQL, a tabela é visível somente na sessão que a criou. 

4. Ao encerrar essa conexão, o servidor remove a tabela automaticamente. 

5. Duas sessões podem criar tabelas temporárias com o mesmo nome sem compartilhar os dados.

6. Para criar uma tabela temporária no MySQL, use o comando `CREATE TEMPORARY TABLE`. 

7. Uma tabela temporária existe apenas durante a sessão atual e é apagada automaticamente quando a conexão fecha (quando a conexão é encerrada).

8. Objetivo de se criar tabela temporária: agilizar consultas complexas, armazenar resultados intermediários, testar dados, etc.

---

## Sintaxe básica (definindo colunas)

```sql
USE empresa;
```

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_cliente;

CREATE TEMPORARY TABLE IF NOT EXISTS tmp_cliente (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    email VARCHAR(150)
);
```

```sql
SELECT *
FROM   tmp_cliente;
```

## Manipulando dados na tabela temporária

Após criar `tmp_cliente`, podemos inserir registros:

```sql
INSERT INTO tmp_cliente (nome, email)
VALUES ('Ana Souza', 'ana.souza@email.com');

INSERT INTO tmp_cliente (nome, email)
VALUES ('Bruno Lima', 'bruno.lima@email.com');

INSERT INTO tmp_cliente (nome, email)
VALUES ('Carla Mendes', 'carla.mendes@email.com');

INSERT INTO tmp_cliente (nome, email)
VALUES ('Daniel Rocha', 'daniel.rocha@email.com');

INSERT INTO tmp_cliente (nome, email)
VALUES ('Elisa Costa', 'elisa.costa@email.com');
```

```sql
SELECT *
FROM   tmp_cliente;
```

Atualizar os dados dos clientes:

```sql
UPDATE tmp_cliente
SET    email = 'ana.nova@email.com'
WHERE  id = 1;

UPDATE tmp_cliente
SET    nome = 'Daniel Pereira'
WHERE  id = 4;
```

```sql
SELECT *
FROM   tmp_cliente;
```

Excluir clientes:

```sql
DELETE FROM tmp_cliente
WHERE       id = 5;
```

```sql
SELECT *
FROM   tmp_cliente;
```

**OBS**: o MySQL não aceita `CREATE TEMP TABLE`, apenas `CREATE TEMPORARY TABLE`.

---

## Criando tabela temporária a partir de uma consulta (SELECT)

Crie e preencha a tabela temporária com o resultado de uma consulta (query). Ela é populada no mesmo momento em que é criada:

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_produtos_vencidos;

CREATE TEMPORARY TABLE IF NOT EXISTS tmp_produtos_vencidos AS
SELECT id_produto, cnpj_filial, preco, quantidade, validade
FROM Estoque
WHERE validade < CURDATE();
```

```sql
SELECT *
FROM   tmp_produtos_vencidos;
```

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_produtos_vencidos_ontem;

CREATE TEMPORARY TABLE IF NOT EXISTS tmp_produtos_vencidos_ontem AS
SELECT id_produto, cnpj_filial, preco, quantidade, validade
FROM Estoque
WHERE validade = CURDATE() - INTERVAL 1 DAY;
```

```sql
SELECT *
FROM   tmp_produtos_vencidos_ontem;
```

---

## Copiando a estrutura de outra tabela (LIKE)

```sql
DROP TEMPORARY TABLE IF EXISTS temp_produto;

CREATE TEMPORARY TABLE IF NOT EXISTS temp_produto LIKE produto;
```

```sql
DESC produto;
```

```sql
DESC temp_produto;
```

```sql
SELECT *
FROM   produto;
```

```sql
SELECT *
FROM   temp_produto;
```

---

## Excluido uma tabela temporária manualmente

Para remover antes de encerrar a sessão, execute:

```sql
DROP TEMPORARY TABLE IF EXISTS temp_produto;
```

---

## Criando uma tabela temporária

No modelo do banco empresa de nosso projeto, a quantidade em estoque pertence à relação entre `Produto` e `Filial`, representada pela tabela `Estoque`. Este exemplo reúne os produtos cuja quantidade em alguma filial está abaixo de 10:

```sql
USE empresa;
```

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_reposicao_estoque;

CREATE TEMPORARY TABLE IF NOT EXISTS tmp_reposicao_estoque AS
SELECT p.id AS id_produto,
       p.nome AS produto,
       e.cnpj_filial,
       e.quantidade
FROM   empresa.Produto p
JOIN   empresa.Estoque e ON e.id_produto = p.id
WHERE  e.quantidade < 10
ORDER BY e.cnpj_filial, e.quantidade;
```

Consulte a tabela como qualquer outra tabela da conexão:

```sql
SELECT *
FROM   tmp_reposicao_estoque;
```

**OBS**:

1. A cláusula `AS SELECT` cria a estrutura com base nas colunas retornadas pela consulta e copia as linhas encontradas naquele momento. 

2. **Alterações posteriores na tabela de origem não atualizam automaticamente a tabela temporária**.

---

## Definindo a estrutura manualmente

Também é possível declarar as colunas e inserir os dados em etapas:

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_estoque_por_filial;

CREATE TEMPORARY TABLE IF NOT EXISTS tmp_estoque_por_filial (
    cnpj_filial VARCHAR(14) NOT NULL,
    filial VARCHAR(100) NOT NULL,
    quantidade_total BIGINT NOT NULL,
    valor_estimado DECIMAL(14, 2) NOT NULL
);
```

```sql
SELECT *
FROM   tmp_estoque_por_filial;
```

```sql
INSERT INTO tmp_estoque_por_filial (cnpj_filial, filial, quantidade_total, valor_estimado)
SELECT f.cnpj,
       f.nome,
       SUM(e.quantidade),
       SUM(e.quantidade * e.preco)
FROM   empresa.Filial f
JOIN   empresa.Estoque e ON e.cnpj_filial = f.cnpj
GROUP BY f.cnpj, f.nome
```

```sql
SELECT *
FROM   tmp_estoque_por_filial;
```

Os tipos de dados e restrições da tabela são definidos como em uma tabela comum. 

## Índices

Índices também podem ser criados quando forem necessários para as consultas da sessão:

```sql
CREATE INDEX idx_tmp_estoque_por_filial
ON tmp_estoque_por_filial (cnpj_filial);
```

Para consultar os índices da tabela temporária na sessão atual, use `SHOW INDEX`, como nos exemplos abaixo: 

```sql
SHOW INDEX FROM tmp_estoque_por_filial;
```

```sql
SHOW INDEX FROM tmp_estoque_por_filial FROM empresa;
```

Tabelas temporárias não são expostas em information_schema.statistics. Portanto, uma consulta a essa view não as retorna. A consulta abaixo lista índices apenas de tabelas permanentes do schema atual:

```sql
SELECT table_name, index_name, column_name, non_unique
FROM information_schema.statistics
WHERE table_schema = DATABASE()
ORDER BY table_name, index_name, seq_in_index;
```

---

## Escopo e ciclo de vida

- A tabela temporária existe apenas na conexão que executou `CREATE TEMPORARY TABLE`.
- Outra conexão não consegue consultar essa tabela, mesmo que use o mesmo usuário.
- Ao fechar a conexão, a tabela temporária é removida automaticamente.
- Uma tabela temporária não é uma tabela permanente e não aparece como objeto compartilhado por todas as conexões.
- Se uma tabela temporária e uma tabela permanente tiverem o mesmo nome, a tabela temporária oculta a permanente para a sessão que criou a temporária.

Uma tabela temporária pode ser removida antes do fim da conexão:

```sql
DROP TEMPORARY TABLE IF EXISTS tmp_reposicao_estoque;
```

```sql
SELECT *
FROM   tmp_reposicao_estoque;
```

**OBS**:
> 1. Use `DROP TEMPORARY TABLE` para deixar claro que a intenção é remover somente a tabela temporária. 
> 2. Essa instrução não remove uma tabela permanente que tenha o mesmo nome.

---

## Permissão necessária

Para criar tabelas temporárias em um banco, a conta precisa do privilégio `CREATE TEMPORARY TABLES` nesse banco. 

Um administrador pode concedê-lo assim:

```sql
DROP ROLE IF EXISTS 'analista'@'localhost';
CREATE ROLE IF NOT EXISTS 'analista'@'localhost'; 
GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.*
TO 'analista'@'localhost'; 
```

```sql
SHOW GRANTS FOR 'analista'@'localhost';
```

```sql
GRANT CREATE TEMPORARY TABLES
ON empresa.*
TO 'analista'@'localhost';
```

```sql
SHOW GRANTS FOR 'analista'@'localhost';
```

**OBS**:
1. O privilégio permite criar tabelas temporárias, mas não concede automaticamente acesso às tabelas permanentes consultadas. 

2. Para executar o exemplo que lê `empresa.Produto`, a conta também precisa dos privilégios adequados de leitura nessa tabela.

---

## Tabela Temporária e Stored Procedure

Para usar tabelas temporárias dentro de uma `Stored Procedure` no `MySQL`, você deve criar, manipular e excluir a tabela dentro do bloco `BEGIN ... END`. 

Uma boa prática é sempre usar `DROP TEMPORARY TABLE IF EXISTS` antes e depois do uso para evitar erros de "tabela já existe" caso a procedure seja chamada duas vezes na mesma sessão.

No exemplo abaixo, a procedure calcula o total em estoque e retorna um relatório consolidado:

```sql
DROP PROCEDURE IF EXISTS sp_relatorio_total_em_estoque;

DELIMITER $$

CREATE PROCEDURE IF NOT EXISTS sp_relatorio_total_em_estoque()
BEGIN
    -- 1. Garante que a tabela não exista antes de criar (evita erro a partr da 2ª execução):
    DROP TEMPORARY TABLE IF EXISTS tmp_total_em_estoque;

    -- 2. Cria e popula a tabela temporária:
    CREATE TEMPORARY TABLE IF NOT EXISTS tmp_total_em_estoque (
        cnpj_filial VARCHAR(14) NOT NULL,
        filial VARCHAR(100) NOT NULL,
        quantidade_total BIGINT NOT NULL,
        valor_estimado DECIMAL(14, 2) NOT NULL,
        valor_com_acrescimo DECIMAL(14, 2) NOT NULL DEFAULT 0
    );

    -- 3. Insere os dados processados:
    INSERT INTO tmp_total_em_estoque (cnpj_filial, filial, quantidade_total, valor_estimado)
    SELECT f.cnpj,
           f.nome,
           IFNULL(SUM(e.quantidade), 0),
           IFNULL(SUM(e.quantidade * e.preco), 0)
    FROM   empresa.Filial f
    JOIN   empresa.Estoque e ON e.cnpj_filial = f.cnpj
    GROUP BY f.cnpj, f.nome;

    -- 4. Faz operações adicionais (ex: aplicar um acréscimo de 10% para valores estimados acima de 1000):
    UPDATE tmp_total_em_estoque
    SET valor_com_acrescimo = valor_estimado * 1.10
    WHERE valor_estimado > 1000;

    -- 5. Retorna o resultado final da procedure:
    SELECT * FROM tmp_total_em_estoque 
    ORDER BY valor_estimado DESC;

    -- 6. Limpa a tabela da memória ao finalizar:
    DROP TEMPORARY TABLE IF EXISTS tmp_total_em_estoque;
END$$

DELIMITER ;
```

Após criá-la, basta chamá-la normalmente:

```sql
CALL sp_relatorio_total_em_estoque();
```

**OBS**:

Cuidados importantes em procedures:

1. **Limpeza obrigatória**: como a tabela temporária pertence à sessão (conexão) e não apenas à procedure, ela continua existindo mesmo após a procedure terminar. Sempre use `DROP TEMPORARY TABLE` no início e no fim.
2. **Limitação de auto-junção (Self-Join)**: no MySQL, você não pode referenciar a mesma tabela temporária mais de uma vez na mesma consulta (ex: `SELECT * FROM temp_a JOIN temp_a`). Se precisar fazer isso, crie uma segunda tabela temporária com os mesmos dados.
3. **Desempenho**: se a tabela tiver muitos registros e você for fazer `JOIN` ou `WHERE` nela dentro da procedure, adicione índices (`INDEX` ou `PRIMARY KEY`) na criação para manter a execução rápida.

---

## Tabela Temporária e Function

**OBS**:
> Não é possível criar ou manipular tabelas temporárias dentro de uma `FUNCTION` no MySQL. 

O MySQL proíbe instruções DDL (como CREATE, ALTER ou DROP) e transações implícitas dentro de funções armazenadas, retornando o erro `Explicit or implicit commit is not allowed in stored function or trigger`.

Como alternativa, você deve usar variáveis locais, CTEs (Common Table Expressions com `WITH`) ou subqueries para atingir o mesmo objetivo dentro de uma função.

### Alternativa 1: usando CTE (WITH)

A cláusula `WITH` funciona como uma "tabela temporária em memória" que existe apenas durante a execução daquela consulta.

```sql
DROP TABLE IF EXISTS fn_total_em_estoque;

DELIMITER $$

CREATE FUNCTION IF NOT EXISTS fn_total_em_estoque() 
RETURNS DECIMAL(10, 2)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE v_total_final DECIMAL(10, 2);

    -- O bloco WITH substitui a tabela temporária:
    WITH tmp_total_em_estoque AS (
        SELECT IFNULL(SUM(e.quantidade * e.preco), 0) as valor
        FROM     empresa.Filial f
        JOIN     empresa.Estoque e ON e.cnpj_filial = f.cnpj
        GROUP BY f.cnpj, f.nome
    );

    -- Inserir o valor na variável local:
    SELECT valor
    INTO v_total_final
    FROM tmp_total_em_estoque;

    RETURN v_total_final;
END$$

DELIMITER ;
```

Após criá-la, você a usa diretamente dentro de um SELECT:

```sql
SELECT fn_total_em_estoque() AS total;
```

---

## Tabela temporária não é tabela permanente

1. Use uma tabela temporária quando o resultado for intermediário e pertencer a uma conexão. 

2. Para compartilhar dados entre conexões ou mantê-los após o encerramento da sessão, crie uma tabela permanente.

```sql
SELECT id_produto, cnpj_filial, preco, quantidade, validade
FROM   empresa.Estoque;
```

* `empresa.Estoque` é uma tabela permanente do modelo. 

* Já `tmp_reposicao_estoque` é apenas um `resultado de trabalho` da conexão, que pode ser descartado ao fechá-la. 

* A tabela temporária não substitui os dados originais de `Estoque`.

---

## Boas práticas

1. Use nomes que indiquem que a tabela é de apoio (temporária), como `tmp_resumo_estoque`.
2. Remova a tabela temporária quando ela não for mais necessária, especialmente em conexões que permanecem abertas por muito tempo.
3. Não dependa de uma tabela temporária criada por outra conexão.
4. Verifique os privilégios da conta e o acesso às tabelas permanentes consultadas.
5. Não use o mesmo nome de uma tabela permanente, para evitar que ela fique oculta na sessão.

---

# Exercícios

1. Crie uma tabela temporária chamada `tmp_reservas_abertas` com os números e as datas das reservas de viagem que ainda não foram confirmadas.
2. A partir de uma tabela `farmacia.Medicamento`, crie `tmp_validade_proxima` com os medicamentos que vencem nos próximos 60 dias.
3. Declare manualmente uma tabela temporária `tmp_precos_revisados` para um supermerado com código do produto, preço atual e preço sugerido.
4. Insira na tabela do exercício anterior os produtos de um supermercado cujo preço atual esteja acima de um limite informado.
5. Crie uma tabela temporária com o total de itens vendidos por categoria em uma padaria.
6. Consulte os dados de `tmp_validade_proxima` em ordem crescente de vencimento.
7. Crie um índice na coluna de data de vencimento de `tmp_validade_proxima`.
8. Atualize uma linha de uma tabela temporária e explique por que a tabela permanente de origem não muda.
9. Exclua `tmp_reservas_abertas` antes de encerrar a conexão, sem afetar uma tabela permanente de mesmo nome.
10. Use `SHOW TABLES LIKE 'tmp_precos_revisados'` na mesma sessão para verificar se a tabela temporária existe.
11. Explique por que um usuário conectado em outra sessão não consegue consultar a sua tabela temporária.
12. Descreva o que acontece com as tabelas temporárias ao desconectar o cliente MySQL.
13. Duas sessões criam uma tabela temporária chamada `tmp_contagem`. Explique se uma sessão vê os registros da outra.
14. Explique qual tabela será consultada se a sessão tiver uma tabela temporária e uma permanente com o mesmo nome.
15. Identifique o privilégio necessário para criar uma tabela temporária no banco `empresa`.
16. Explique por que conceder `CREATE TEMPORARY TABLES` não concede, por si só, permissão para ler `empresa.Produto`.
17. Crie uma tabela temporária a partir de uma consulta que selecione somente viagens com lugares disponíveis.
18. Compare a finalidade de uma tabela temporária com a de uma tabela permanente de histórico de estoque.
19. Escreva um script SQL que crie, consulte e remova uma tabela temporária de resumo diário de vendas.
20. Analise um processo que precisa compartilhar resultados entre conexões e justifique por que uma tabela temporária não é adequada.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="20-deny.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="22-trigger.md">Próximo</a>
    </td>
  </tr>
</table>
