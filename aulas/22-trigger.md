# 22 - Gatilhos (Triggers) no MySQL

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="21-tabela-temporaria.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="23-cursor.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Gatilhos no MySQL

1. Um **trigger** (gatilho) é um objeto associado a uma tabela. 

2. O MySQL executa automaticamente o gatilho quando uma linha é inserida, atualizada ou excluída.

3. Gatilhos são úteis para validar dados e registrar alterações. 

4. Eles não são chamados diretamente com `CALL`: são disparados pela instrução que modifica a tabela.

---

## Momento e evento

Um trigger é definido para um momento e um evento:

- `BEFORE INSERT`: antes de inserir cada linha;
- `AFTER INSERT`: depois de inserir cada linha;
- `BEFORE UPDATE`: antes de atualizar cada linha;
- `AFTER UPDATE`: depois de atualizar cada linha;
- `BEFORE DELETE`: antes de excluir cada linha;
- `AFTER DELETE`: depois de excluir cada linha.

No MySQL, os gatilhos são executados **uma vez por linha afetada**. 

---

## NEW e OLD

Dentro do gatilho, `NEW` e `OLD` identificam os valores da linha:

| Evento | `OLD` | `NEW` |
|---|---|---|
| `INSERT` | Não disponível | Valores que serão inseridos |
| `UPDATE` | Valores anteriores | Valores novos |
| `DELETE` | Valores excluídos | Não disponível |

1. Em gatilhos `BEFORE`, os valores de `NEW` podem ser alterados com `SET NEW.coluna = ...` para eventos de inserção e atualização, quando a coluna permite a alteração. 

2. Os valores de `OLD` são somente para leitura.

---

## Validando uma inserção em Estoque

1. O gatilho abaixo rejeita quantidades negativas na tabela estoque do banco empresa. 

2. A quantidade zero continua válida, conforme o modelo. 

3. Pressupõe que `empresa.Estoque` já foi criada e que a conta do usuário tem o privilégio `TRIGGER` sobre ela.

```sql
DROP TRIGGER IF EXISTS empresa.validar_estoque;

DELIMITER $$

CREATE TRIGGER IF NOT EXISTS empresa.validar_estoque
BEFORE INSERT ON empresa.Estoque
FOR EACH ROW
BEGIN
    IF NEW.quantidade < 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'A quantidade em estoque não pode ser negativa.';
    END IF;
END$$

DELIMITER ;
```

Ao inserir um item em estoque, o MySQL executa o gatilho. Os valores de `id_produto` e `cnpj_filial` precisam corresponder a registros existentes em `Produto` e `Filial`:

Este exemplo pressupõe que o produto de código `1` já exista e que a filial use o CNPJ padrão apresentado no modelo.

```sql
INSERT INTO empresa.Estoque (id_produto, cnpj_filial, preco, quantidade, validade)
VALUES (1, '10101010000110', 8.50, -1, '2027-12-31');
```

`SQLSTATE '45000'` é usado para sinalizar uma condição de erro definida pela aplicação.

---

## Validando alterações em Estoque

1. Em um `BEFORE UPDATE`, `NEW` contém os valores propostos e `OLD` contém os valores atuais. 

2. Este exemplo valida a quantidade proposta em `NEW`.

3. `OLD` fica disponível para regras que dependam de comparar o valor anterior com o novo.

```sql
DROP TRIGGER IF EXISTS empresa.validar_atualizacao_estoque;

DELIMITER $$

CREATE TRIGGER IF NOT EXISTS empresa.validar_atualizacao_estoque
BEFORE UPDATE ON empresa.Estoque
FOR EACH ROW
BEGIN
    IF NEW.quantidade < 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'A quantidade em estoque não pode ser negativa.';
    END IF;
END$$

DELIMITER ;
```

Se a validação passar, a instrução de atualização prossegue:

```sql
UPDATE empresa.Estoque
SET    quantidade = -1
WHERE  id_produto = 1
AND    cnpj_filial = '10101010000110';
```

---

## Exemplos de `AFTER INSERT`, `AFTER UPDATE`, `BEFORE DELETE` e `AFTER DELETE`

Os exemplos a seguir usam uma tabela de clientes e uma tabela de auditoria.

```sql
DROP TABLE IF EXISTS ClienteTriggerExemplo; 
CREATE TABLE IF NOT EXISTS ClienteTriggerExemplo (
    id INT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

DROP TABLE IF EXISTS AuditoriaTriggerExemplo; 
CREATE TABLE IF NOT EXISTS AuditoriaTriggerExemplo (
    id INT AUTO_INCREMENT PRIMARY KEY,
    evento VARCHAR(20) NOT NULL,
    cliente_id INT NOT NULL,
    nome_antigo VARCHAR(100),
    nome_novo VARCHAR(100),
    registrado_em TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

---

O `AFTER INSERT` registra os valores recém-inseridos, disponíveis em `NEW`:

```sql
DROP TRIGGER IF EXISTS exemplo_cliente_depois_inserir;

DELIMITER $$

CREATE TRIGGER IF NOT EXISTS exemplo_cliente_depois_inserir
AFTER INSERT ON ClienteTriggerExemplo
FOR EACH ROW
BEGIN
    INSERT INTO AuditoriaTriggerExemplo (evento, cliente_id, nome_novo)
    VALUES ('INSERT', NEW.id, NEW.nome);
END$$

DELIMITER ;
```

---

O `AFTER UPDATE` registra os valores anteriores (`OLD`) e os novos (`NEW`):

```sql
DROP TRIGGER IF EXISTS exemplo_cliente_depois_atualizar;

DELIMITER $$

CREATE TRIGGER IF NOT EXISTS exemplo_cliente_depois_atualizar
AFTER UPDATE ON ClienteTriggerExemplo
FOR EACH ROW
BEGIN
    INSERT INTO AuditoriaTriggerExemplo (evento, cliente_id, nome_antigo, nome_novo)
    VALUES ('UPDATE', NEW.id, OLD.nome, NEW.nome);
END$$

DELIMITER ;
```

---

O `BEFORE DELETE` registra a linha antes de ela ser excluída. Em um `DELETE`, os valores da linha estão disponíveis em `OLD`:

```sql
DROP TRIGGER IF EXISTS exemplo_cliente_antes_excluir;

DELIMITER $$

CREATE TRIGGER IF NOT EXISTS exemplo_cliente_antes_excluir
BEFORE DELETE ON ClienteTriggerExemplo
FOR EACH ROW
BEGIN
    INSERT INTO AuditoriaTriggerExemplo (evento, cliente_id, nome_antigo)
    VALUES ('BEFORE DELETE', OLD.id, OLD.nome);
END$$

DELIMITER ;
```

---

O `AFTER DELETE` também pode consultar `OLD`, embora a linha já tenha sido excluída da tabela:

```sql
DROP TRIGGER IF EXISTS exemplo_cliente_depois_excluir;

DELIMITER $$

CREATE TRIGGER IF NOT EXISTS exemplo_cliente_depois_excluir
AFTER DELETE ON ClienteTriggerExemplo
FOR EACH ROW
BEGIN
    INSERT INTO AuditoriaTriggerExemplo (evento, cliente_id, nome_antigo)
    VALUES ('AFTER DELETE', OLD.id, OLD.nome);
END$$

DELIMITER ;
```

---

Execute os comandos abaixo para disparar cada evento:

```sql
-- Dispara o AFTER INSERT.
INSERT INTO ClienteTriggerExemplo (id, nome)
VALUES (1, 'Ana');

-- Dispara o AFTER UPDATE.
UPDATE ClienteTriggerExemplo
SET nome = 'Ana Silva'
WHERE id = 1;

-- Dispara o BEFORE DELETE e, em seguida, o AFTER DELETE.
DELETE FROM ClienteTriggerExemplo
WHERE id = 1;
```

---

Consulte os registros produzidos pelos gatilhos:

```sql
SELECT evento, cliente_id, nome_antigo, nome_novo, registrado_em
FROM AuditoriaTriggerExemplo
ORDER BY id;
```


---

## Delimitador ao criar um trigger

1. O corpo de um trigger pode conter várias instruções terminadas em `;`. 

2. No cliente de linha de comando do MySQL, use `DELIMITER` para que o corpo seja enviado como uma única instrução `CREATE TRIGGER`:

```sql
DELIMITER $$
-- CREATE TRIGGER ... BEGIN ...; ... END$$
DELIMITER ;
```

3. `DELIMITER` é um comando do cliente MySQL, não uma instrução SQL enviada ao servidor. 

4. Algumas ferramentas gráficas, incluindo clientes SQL, podem executar a instrução inteira sem exigir essa configuração do mesmo modo; siga a forma de execução suportada pela ferramenta.

---

## Consultando triggers

Liste os gatilhos (triggers) do banco selecionado:

```sql
SHOW TRIGGERS;
```

Consulte a definição de um gatilho:

```sql
SHOW CREATE TRIGGER empresa.validar_estoque;
```

---

## Removendo triggers

Remova o gatilho quando ele não for mais necessário:

```sql
DROP TRIGGER IF EXISTS empresa.validar_estoque;
DROP TRIGGER IF EXISTS empresa.validar_atualizacao_estoque;

DROP TRIGGER IF EXISTS empresa.exemplo_cliente_depois_inserir;
DROP TRIGGER IF EXISTS empresa.exemplo_cliente_depois_atualizar;
DROP TRIGGER IF EXISTS empresa.exemplo_cliente_antes_excluir;
DROP TRIGGER IF EXISTS empresa.exemplo_cliente_depois_excluir;
```

1. Remover um trigger não apaga as linhas da tabela associada. 

2. Remover a tabela associada também remove seus gatilhos (triggers).

---

## Cuidados e limitações

1. A conta que cria o gatilho precisa do privilégio `TRIGGER` na tabela associada.
2. Um trigger só pode ser associado a uma tabela e a um dos eventos `INSERT`, `UPDATE` ou `DELETE`.
3. Um gatilho é disparado por linha, inclusive quando um comando altera várias linhas.
4. A alteração que disparou o trigger e as operações feitas por ele participam da mesma instrução/transação, conforme o mecanismo de armazenamento e as condições da operação.
5. Um trigger não pode iniciar ou encerrar transações com comandos como `START TRANSACTION`, `COMMIT` ou `ROLLBACK`.
6. Não tente alterar a mesma tabela que está sendo modificada pela instrução que disparou o trigger; esse padrão é proibido e pode gerar erro.
7. Mantenha gatilhos pequenos e documente os efeitos que não ficam visíveis na instrução original.

---

# Exercícios

1. No banco de uma agência de viagens, crie um gatilho `BEFORE INSERT` para impedir que uma reserva de viagem tenha preço negativo.
2. No banco de uma farmácia, faça um gatilho rejeitar a inclusão de um medicamento cuja quantidade inicial em estoque seja menor que zero.
3. Crie um gatilho de validação para impedir que um caixa de supermercado registre uma venda sem identificador de cliente quando essa informação for obrigatória.
4. Crie um gatilho `BEFORE UPDATE` que impeça a alteração do código de barras de um produto já cadastrado no banco de um supermercado.
5. No banco de um supermercado, registre em uma tabela de auditoria as mudanças de endereço de um fornecedor usando `OLD` e `NEW`.
6. Como um gatilho, registre a data e o identificador do lote sempre que uma linha de lote de uma padaria for excluída.
7. No banco de uma agência de viagens, escreva um trigger `AFTER INSERT` que registre cada novo item de uma reserva em uma tabela de histórico.
8. No banco de uma agência de viagens, faça um `BEFORE UPDATE` que valide que a data de retorno de uma viagem não antecede a data de ida.
9. Em uma tabela de movimentação de uma farmácia, rejeite um débito que resulte em estoque negativo.
10. Crie um gatilho `AFTER UPDATE` que registre o preço antigo e o novo preço de um produto de supermercado, mas somente quando o preço mudar.
11. Explique a diferença entre `BEFORE INSERT` e `AFTER INSERT`.
12. Para um gatilho `BEFORE DELETE`, indique qual pseudotabela (`NEW` / `OLD`) contém os valores da linha removida.
13. Indique quais pseudotabelas (`NEW` / `OLD`) estão disponíveis em um trigger `AFTER UPDATE`.
14. Explique por que `FOR EACH ROW` é executado mais de uma vez quando uma instrução atualiza várias linhas.
15. Explique a função de `SIGNAL SQLSTATE '45000'` em uma regra de validação.
16. Consulte (apresente) a definição de um gatilho chamado `validar_preco_pacote`.
17. Remova o trigger `auditar_movimento_estoque` sem excluir a tabela à qual ele pertence.
18. Explique por que um trigger não deve tentar atualizar a mesma tabela que disparou o evento.
19. Analise um sistema com regras de auditoria implementadas em vários gatilhos e proponha como tornar seus efeitos mais fáceis de rastrear.
20. Escolha um evento adequado (`INSERT`, `UPDATE` ou `DELETE`) e o momento (`BEFORE` ou `AFTER`) para validar e auditar a alteração de uma reserva em uma agência de viagens; justifique a escolha.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="21-tabela-temporaria.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="23-cursor.md">Próximo</a>
    </td>
  </tr>
</table>
