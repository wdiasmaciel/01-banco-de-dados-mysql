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

Um **trigger** (gatilho) é um objeto associado a uma tabela. O MySQL executa automaticamente o gatilho quando uma linha é inserida, atualizada ou excluída.

Gatilhos são úteis para validar dados e registrar alterações. Eles não são chamados diretamente com `CALL`: são disparados pela instrução que modifica a tabela.

---

## Momento e evento

Um trigger é definido para um momento e um evento:

- `BEFORE INSERT`: antes de inserir cada linha;
- `AFTER INSERT`: depois de inserir cada linha;
- `BEFORE UPDATE`: antes de atualizar cada linha;
- `AFTER UPDATE`: depois de atualizar cada linha;
- `BEFORE DELETE`: antes de excluir cada linha;
- `AFTER DELETE`: depois de excluir cada linha.

No MySQL, os gatilhos são executados **uma vez por linha afetada**. Não existe trigger `FOR EACH STATEMENT` para executar apenas uma vez por comando.

---

## NEW e OLD

Dentro do gatilho, `NEW` e `OLD` identificam os valores da linha:

| Evento | `OLD` | `NEW` |
|---|---|---|
| `INSERT` | Não disponível | Valores que serão inseridos |
| `UPDATE` | Valores anteriores | Valores novos |
| `DELETE` | Valores excluídos | Não disponível |

Em gatilhos `BEFORE`, os valores de `NEW` podem ser alterados com `SET NEW.coluna = ...` para eventos de inserção e atualização, quando a coluna permite a alteração. Os valores de `OLD` são somente para leitura.

---

## Validando uma inserção em Estoque

No modelo da empresa, `Estoque` relaciona um produto a uma filial e armazena preço, quantidade e validade. A tabela já possui uma restrição `CHECK` para preço; como não há uma restrição equivalente para quantidade, o gatilho abaixo rejeita quantidades negativas. A quantidade zero continua válida, conforme o modelo. Pressupõe que `empresa.Estoque` já foi criada e que a conta tem o privilégio `TRIGGER` sobre ela.

```sql
DELIMITER $$

CREATE TRIGGER empresa.validar_estoque
BEFORE INSERT ON empresa.Estoque
FOR EACH ROW
BEGIN
    IF NEW.quantidade < 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'A quantidade em estoque nao pode ser negativa';
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

Em um `BEFORE UPDATE`, `NEW` contém os valores propostos e `OLD` contém os valores atuais. Este exemplo valida a quantidade proposta em `NEW`; `OLD` fica disponível para regras que dependam de comparar o valor anterior com o novo.

```sql
DELIMITER $$

CREATE TRIGGER empresa.validar_atualizacao_estoque
BEFORE UPDATE ON empresa.Estoque
FOR EACH ROW
BEGIN
    IF NEW.quantidade < 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'A quantidade em estoque nao pode ser negativa';
    END IF;
END$$

DELIMITER ;
```

Se a validação passar, a instrução de atualização prossegue:

```sql
UPDATE empresa.Estoque
SET    quantidade = 25
WHERE  id_produto = 1
AND    cnpj_filial = '10101010000110';
```

---

## Delimitador ao criar um trigger

O corpo de um trigger pode conter várias instruções terminadas em `;`. No cliente de linha de comando do MySQL, use `DELIMITER` para que o corpo seja enviado como uma única instrução `CREATE TRIGGER`.

```sql
DELIMITER $$
-- CREATE TRIGGER ... BEGIN ...; ... END$$
DELIMITER ;
```

`DELIMITER` é um comando do cliente MySQL, não uma instrução SQL enviada ao servidor. Algumas ferramentas gráficas, incluindo clientes SQL, podem executar a instrução inteira sem exigir essa configuração do mesmo modo; siga a forma de execução suportada pela ferramenta.

---

## Consultando e removendo triggers

Liste os gatilhos do banco selecionado:

```sql
SHOW TRIGGERS;
```

Consulte a definição de um gatilho:

```sql
SHOW CREATE TRIGGER empresa.validar_estoque;
```

Remova o gatilho quando ele não for mais necessário:

```sql
DROP TRIGGER IF EXISTS empresa.validar_estoque;
DROP TRIGGER IF EXISTS empresa.validar_atualizacao_estoque;
```

Remover um trigger não apaga as linhas da tabela associada. Remover a tabela associada também remove seus gatilhos.

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

1. Crie um `BEFORE INSERT` para impedir que uma reserva de viagem tenha preço negativo.
2. Faça um gatilho rejeitar a inclusão de um medicamento cuja quantidade inicial em estoque seja menor que zero.
3. Crie uma validação para impedir que um caixa de supermercado registre uma venda sem identificador de cliente quando essa informação for obrigatória.
4. Crie um `BEFORE UPDATE` que impeça a alteração do código de barras de um produto já cadastrado.
5. Registre em uma tabela de auditoria as mudanças de endereço de um fornecedor usando `OLD` e `NEW`.
6. Registre a data e o identificador do lote sempre que uma linha de lote de padaria for excluída.
7. Escreva um trigger `AFTER INSERT` que registre cada novo item de uma reserva em uma tabela de histórico.
8. Faça um `BEFORE UPDATE` que valide que a data de retorno de uma viagem não antecede a data de ida.
9. Em uma tabela de movimentação de farmácia, rejeite um débito que resulte em estoque negativo.
10. Crie um `AFTER UPDATE` que registre o preço antigo e o novo preço de um produto de supermercado, mas somente quando o preço mudar.
11. Explique a diferença entre `BEFORE INSERT` e `AFTER INSERT`.
12. Para um gatilho `BEFORE DELETE`, indique qual pseudotabela contém os valores da linha removida.
13. Indique quais pseudotabelas estão disponíveis em um trigger `AFTER UPDATE`.
14. Explique por que `FOR EACH ROW` é executado mais de uma vez quando uma instrução atualiza várias linhas.
15. Explique a função de `SIGNAL SQLSTATE '45000'` em uma regra de validação.
16. Consulte a definição de um gatilho chamado `validar_preco_pacote`.
17. Remova o trigger `auditar_movimento_estoque` sem excluir a tabela à qual ele pertence.
18. Explique por que um trigger não deve tentar atualizar a mesma tabela que disparou o evento.
19. Analise um sistema com regras de auditoria implementadas em vários gatilhos e proponha como tornar seus efeitos mais fáceis de rastrear.
20. Escolha um evento adequado (`INSERT`, `UPDATE` ou `DELETE`) e o momento (`BEFORE` ou `AFTER`) para validar e auditar a alteração de uma reserva; justifique a escolha.

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
