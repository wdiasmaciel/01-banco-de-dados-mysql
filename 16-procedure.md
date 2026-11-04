# 16 - Procedures com CREATE PROCEDURE

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="15-view.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="17-function.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Rotinas armazenadas no MySQL  (stored procedure)

Uma **procedure armazenada** (stored procedure, ou procedimento armazenado) é um conjunto de comandos SQL associado ao banco de dados. Ela permite executar várias operações por meio de uma única chamada, facilitando a reutilização e padronização de tarefas.

Procedures podem receber parâmetros, consultar dados, alterar registros e controlar o fluxo da execução com estruturas como `IF`, `CASE`, `WHILE` e `LOOP`.

---

## Criando uma procedure simples

Considere as tabelas `Produto` e `Estoque` do projeto:

```sql
DELIMITER $$

CREATE PROCEDURE listar_produtos()
BEGIN
    SELECT id, nome
    FROM   Produto
    ORDER BY nome;
END$$

DELIMITER ;
```

**OBS**:
`DELIMITER $$`: muda temporariamente o delimitador padrão do MySQL, de `;` para `$$`.

`END$$`: indica o término da procedure ou função usando o novo delimitador.

`DELIMITER ;`: restaura o `;` como delimitador padrão.

Isso é necessário porque vários comandos internos de uma procedure terminam com `;`. 

Sem alterar o delimitador, o MySQL poderia encerrar a criação antes de chegar ao `END`.

```sql
DELIMITER $$

CREATE PROCEDURE exemplo()
BEGIN
    SELECT 1;
END$$

DELIMITER ;
```

---

## Executar

Para executar a procedure, usamos `CALL`:

```sql
CALL listar_produtos();
```

Não é necessário utilizar `CALL` se a procedure estiver definida para ser executada por uma aplicação externamente, mas essa é a forma usada diretamente pelo cliente MySQL.

---

## Parâmetros IN, OUT e INOUT

Os parâmetros de uma procedure podem ser classificados em:

- `IN`: envia um valor para a procedure;
- `OUT`: retorna um valor produzido pela procedure;
- `INOUT`: recebe um valor e pode devolvê-lo alterado.

### Parâmetro IN

```sql
DELIMITER $$

CREATE PROCEDURE consultar_produto(IN codigo INT)
BEGIN
    SELECT id, nome
    FROM   Produto
    WHERE  id = codigo;
END$$

DELIMITER ;
```

```sql
CALL consultar_produto(1);
```

### Parâmetro OUT

```sql
DELIMITER $$

CREATE PROCEDURE contar_produtos(OUT quantidade INT)
BEGIN
    SELECT COUNT(*)
    INTO   quantidade
    FROM   Produto;
END$$

DELIMITER ;
```

```sql
CALL contar_produtos(@total);
SELECT @total AS quantidade_produtos;
```

A variável iniciada com `@` é uma variável de usuário da sessão MySQL.

### Parâmetro INOUT

```sql
DELIMITER $$

CREATE PROCEDURE aplicar_desconto(INOUT valor DECIMAL(10,2))
BEGIN
    SET valor = valor * 0.90;
END$$

DELIMITER ;
```

```sql
SET @preco = 80.00;
CALL aplicar_desconto(@preco);
SELECT @preco AS preco_com_desconto;
```

---

## Uso de variáveis locais

Variáveis locais são declaradas dentro de uma procedure com `DECLARE`:

```sql
DELIMITER $$

CREATE PROCEDURE mostrar_quantidade_produto(IN codigo INT)
BEGIN
    DECLARE total INT DEFAULT 0;

    SELECT SUM(e.quantidade)
    INTO   total
    FROM   Estoque e
    WHERE  e.id_produto = codigo;

    SELECT codigo AS id_produto,
           COALESCE(total, 0) AS quantidade_total;
END$$

DELIMITER ;
```

**OBS**:
`COALESCE` retorna o primeiro valor diferente de `NULL`.

1. Se `total` possuir um valor, esse valor será exibido.
2. Se for `NULL`, será exibido `0`.
3. `AS` `quantidade_total` define o nome da coluna no resultado.
4. Sem o `COALESCE`, uma consulta sem registros poderia retornar `NULL`. Isso costuma acontecer quando o produto não possui itens cadastrados em `Estoque`.


```sql
CALL mostrar_quantidade_produto(1);
```

---

## Estruturas condicionais

### IF

```sql
DELIMITER $$

CREATE PROCEDURE verificar_disponibilidade(IN codigo INT)
BEGIN
    DECLARE quantidade_total INT DEFAULT 0;

    SELECT COALESCE(SUM(quantidade), 0)
    INTO   quantidade_total
    FROM   Estoque
    WHERE  id_produto = codigo;

    IF quantidade_total > 0 THEN
        SELECT 'Produto disponível' AS situacao,
               quantidade_total;
    ELSE
        SELECT 'Produto indisponível' AS situacao,
               quantidade_total;
    END IF;
END$$

DELIMITER ;
```

```sql
CALL verificar_disponibilidade(1);
```

### CASE

```sql
DELIMITER $$

CREATE PROCEDURE classificar_produto(IN codigo INT)
BEGIN
    DECLARE preco_atual DECIMAL(10,2);

    SELECT MIN(preco)
    INTO   preco_atual
    FROM   Estoque
    WHERE  id_produto = codigo;

    SELECT CASE
           WHEN preco_atual IS NULL THEN 'Sem preço cadastrado'
           WHEN preco_atual < 10 THEN 'Preço baixo'
           WHEN preco_atual < 50 THEN 'Preço intermediário'
           ELSE 'Preço alto'
           END AS classificacao;
END$$

DELIMITER ;
```

## Estruturas de repetição

### WHILE

A estrutura `WHILE` repete os comandos enquanto a condição definida estiver verdadeira. A condição é testada antes da primeira execução:

```sql
DELIMITER $$

CREATE PROCEDURE somar_numeros(IN limite INT)
BEGIN
    DECLARE contador INT DEFAULT 1;
    DECLARE total INT DEFAULT 0;

    WHILE contador <= limite DO
        SET total = total + contador;
        SET contador = contador + 1;
    END WHILE;

    SELECT total AS resultado;
END$$

DELIMITER ;
```

```sql
CALL somar_numeros(5);
```

Nesse exemplo, quando o valor do `contador` supera o limite, a condição deixa de ser verdadeira e o `WHILE` termina. Se o limite for menor que 1, os comandos internos não serão executados.

### LOOP

A estrutura `LOOP` repete os comandos indefinidamente até que um `LEAVE` encerre o bloco identificado pelo rótulo. Portanto, é necessário definir uma condição de saída:

```sql
DELIMITER $$

CREATE PROCEDURE somar_numeros_com_loop(IN limite INT)
BEGIN
    DECLARE contador INT DEFAULT 1;
    DECLARE total INT DEFAULT 0;

    repetir_numeros: LOOP
        IF contador > limite THEN
            LEAVE repetir_numeros;
        END IF;

        SET total = total + contador;
        SET contador = contador + 1;
    END LOOP;

    SELECT total AS resultado;
END$$

DELIMITER ;
```

```sql
CALL somar_numeros_com_loop(5);
```

`LEAVE` interrompe o `LOOP` identificado pelo rótulo `repetir_numeros`. O mesmo princípio pode ser aplicado a outros tipos de loop, mas deve-se evitar executar repetições que nunca alcancem a condição de saída.

---

## Procedure com alteração de dados

Uma procedure pode executar `INSERT`, `UPDATE` ou `DELETE`:

```sql
DELIMITER $$

CREATE PROCEDURE ajustar_preco(
    IN codigo INT,
    IN filial CHAR(14),
    IN novo_preco DECIMAL(10,2)
)
BEGIN
    UPDATE Estoque
    SET    preco = novo_preco
    WHERE  id_produto = codigo
    AND    cnpj_filial = filial;
END$$

DELIMITER ;
```

```sql
CALL ajustar_preco(1, '10101010000110', 25.95);
```

Antes de chamar uma procedure que altera dados, é importante confirmar os parâmetros e validar condições de negócio.

---

## Tratamento de ausência de resultados

Ao buscar um único valor, é importante tratar o caso em que nenhuma linha corresponde aos parâmetros. A saída pode ser inicializada como `NULL`, e o handler `NOT FOUND` pode indicar que o registro não foi localizado:

```sql
DELIMITER $$

CREATE PROCEDURE verificar_preco(
    IN codigo INT,
    IN filial CHAR(14),
    OUT preco_local DECIMAL(10,2),
    OUT registro_encontrado BOOLEAN
)
BEGIN
    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET registro_encontrado = FALSE;

    SET preco_local = NULL;
    SET registro_encontrado = TRUE;

    SELECT preco
    INTO   preco_local
    FROM   Estoque
    WHERE  id_produto = codigo
    AND    cnpj_filial = filial;
END$$

DELIMITER ;
```

```sql
CALL verificar_preco(1, '10101010000110',
                    @preco_local, @registro_encontrado);

SELECT @preco_local AS preco_local,
       @registro_encontrado AS registro_encontrado;
```

Quando nenhuma linha é encontrada, o `handler` altera `registro_encontrado` para `FALSE`. Assim, o chamador pode decidir como tratar o resultado, por exemplo exibindo uma mensagem ou interrompendo a operação.

Outra alternativa é interromper a execução com `SIGNAL`:

```sql
DELIMITER $$

CREATE PROCEDURE verificar_preco_obrigatorio(
    IN codigo INT,
    IN filial CHAR(14),
    OUT preco_local DECIMAL(10,2)
)
BEGIN
    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Registro de preço não encontrado';

    SELECT preco
    INTO   preco_local
    FROM   Estoque
    WHERE  id_produto = codigo
    AND    cnpj_filial = filial;
END$$

DELIMITER ;
```

O uso de `SIGNAL` dentro de `CONTINUE HANDLER` provoca uma exceção porque o comando interrompe a execução da rotina. Para escolher entre as duas abordagens, considere: `OUT + handler` permite continuar e decidir como tratar o resultado; `SIGNAL` informa diretamente ao chamador que a operação não pode continuar.

---

## Consultando e removendo procedures

Para listar os objetos armazenados do banco atual:

```sql
SHOW PROCEDURE STATUS
WHERE Db = DATABASE();
```

Para visualizar o comando usado na criação:

```sql
SHOW CREATE PROCEDURE consultar_produto;
```

Para remover uma procedure:

```sql
DROP PROCEDURE consultar_produto;
```

Use `IF EXISTS` quando o objeto pode não existir:

```sql
DROP PROCEDURE IF EXISTS consultar_produto;
```

---

## Boas práticas

1. Escolha nomes que indiquem a ação realizada.
2. Valide os parâmetros antes de alterar dados.
3. Utilize `IN` por padrão e `OUT` ou `INOUT` somente quando necessário.
4. Use `DELIMITER` para evitar que o cliente interprete os pontos e vírgula internos antes de terminar a criação.
5. Não misture várias responsabilidades sem necessidade, como alteração de estoque, emissão de relatórios e manutenção de usuários.
6. Confira os privilégios necessários, pois executar uma procedure não fornece automaticamente privilégios sobre suas tabelas internas.

---

# Exercícios

### Padaria
1. Crie uma procedure sem parâmetros que liste o nome e o preço dos pães disponíveis.
2. Crie uma procedure com `IN` para consultar todos os ingredientes fornecidos por um código de fornecedor.
3. Crie uma procedure com `OUT` para informar a quantidade total de produtos de uma categoria.
4. Crie uma procedure com `INOUT` que aplique 5% de acréscimo ao preço recebido por parâmetro.
5. Crie uma procedure que receba o código de uma venda e classifique o pedido como pequeno, médio ou grande com base na quantidade vendida.

### Farmácia
6. Crie uma procedure que receba o código de um medicamento e liste lotes com validade ainda não vencida.
7. Crie uma procedure que receba o código do medicamento e use uma variável local para calcular o valor total em estoque.
8. Crie uma procedure que atualize o preço de um medicamento usando parâmetros de código e novo valor.
9. Crie uma procedure que receba a quantidade vendida e informe por `OUT` se ela está dentro ou fora do estoque disponível.
10. Crie uma procedure que registre a retirada de um medicamento apenas quando houver quantidade suficiente.

### Agência de Viagens
11. Crie uma procedure sem parâmetros que liste os pacotes turísticos com vagas disponíveis.
12. Crie uma procedure com `IN` que consulte reservas feitas por um código de cliente.
13. Crie uma procedure com `OUT` que informe o número de destinos cadastrados por região.
14. Crie uma procedure que atualize o preço de um pacote usando código, preço atual e novo preço.
15. Crie uma procedure que classifique um pacote como econômico, padrão ou premium com base no valor recebido.

### Supermercado
16. Crie uma procedure que recebe o código de um setor e retorna os produtos cadastrados nele.
17. Crie uma procedure com `INOUT` para converter uma quantidade de unidades em pacotes de dez unidades.
18. Crie uma procedure que receba um produto e atualize sua quantidade em estoque somente se o novo valor não for negativo.
19. Crie uma procedure que consulte uma procedure criada anteriormente, exibindo seu código SQL completo.
20. Crie uma procedure que liste todas as procedures armazenadas no banco atual e explique a diferença entre executar e remover essa procedure.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="15-view.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="17-function.md">Próximo</a>
    </td>
  </tr>
</table>
