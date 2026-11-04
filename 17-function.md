# 17 - Funções com CREATE FUNCTION

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="16-procedure.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="18-grant.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Funções armazenadas no MySQL

Uma **função armazenada** é uma rotina que recebe argumentos e retorna um único valor. Diferentemente das procedures, as funções podem ser chamadas dentro de expressões SQL, por exemplo em `SELECT`, `WHERE` ou `ORDER BY`.

A sintaxe básica informa o tipo do valor retornado com `RETURNS` e contém o valor que deverá ser devolvido por meio de `RETURN`:

```text
CREATE FUNCTION nome(parâmetros)
RETURNS tipo
corpo_da_função;
```

---

## Configuração necessária

Em algumas instalações do MySQL, a criação de funções pode exigir privilégios adicionais ou a configuração de segurança para rotinas que realizam alterações. O usuário deve possuir privilégios administrativos (admin).

---

## Função com parâmetro

Considere a tabela `Estoque` do projeto:

```sql
DELIMITER $$

CREATE FUNCTION calcular_valor_estoque(
    quantidade INT,
    preco DECIMAL(10,2)
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN quantidade * preco;
END$$

DELIMITER ;
```

A função pode ser usada em uma consulta:

```sql
SELECT id_produto,
       quantidade,
       preco,
       calcular_valor_estoque(quantidade, preco) AS valor_total
FROM   Estoque;
```

Uma chamada simples também pode ser feita com `SELECT`:

```sql
SELECT calcular_valor_estoque(12, 15.50) AS resultado;
```

`DETERMINISTIC` informa que, para os mesmos argumentos, o resultado da função não muda. Ele deve ser usado somente quando essa afirmação é verdadeira.

---

## Função que consulta uma tabela

Uma função pode consultar dados armazenados em tabelas:

```sql
DELIMITER $$

CREATE FUNCTION nome_produto(codigo INT)
RETURNS VARCHAR(100)
READS SQL DATA
BEGIN
    DECLARE resultado VARCHAR(100);

    SELECT nome
    INTO   resultado
    FROM   Produto
    WHERE  id = codigo;

    RETURN resultado;
END$$

DELIMITER ;
```

```sql
SELECT nome_produto(1) AS nome;
```

Se não houver produto com o código informado, a consulta pode terminar com `NULL`. Funções chamadas em consultas podem deixar o resultado indefinido nesse caso, portanto é importante validar a entrada ou prever esse resultado na aplicação.

---

## Variáveis e condicionais

É possível utilizar uma variável local dentro da função:

```sql
DELIMITER $$

CREATE FUNCTION classificar_preco(preco DECIMAL(10,2))
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    IF preco < 10 THEN
        RETURN 'Acessível';
    ELSEIF preco < 50 THEN
        RETURN 'Intermediário';
    ELSE
        RETURN 'Premium';
    END IF;
END$$

DELIMITER ;
```

```sql
SELECT preco,
       classificar_preco(preco) AS faixa
FROM   Estoque;
```

---

## CASE dentro de uma função

```sql
DELIMITER $$

CREATE FUNCTION descricao_quantidade(quantidade INT)
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE descricao VARCHAR(20);

    CASE
        WHEN quantidade = 0 THEN
            SET descricao = 'Esgotado';
        WHEN quantidade <= 10 THEN
            SET descricao = 'Pouco estoque';
        ELSE
            SET descricao = 'Estoque adequado';
    END CASE;

    RETURN descricao;
END$$

DELIMITER ;
```

```sql
SELECT id_produto,
       quantidade,
       descricao_quantidade(quantidade) AS situacao
FROM   Estoque;
```

---

## Substituindo uma função existente

Uma função pode ser removida e recriada quando é necessário alterar a definição:

```sql
DROP FUNCTION IF EXISTS calcular_valor_estoque;
```

```sql
DELIMITER $$

CREATE FUNCTION calcular_valor_estoque(
    quantidade INT,
    preco DECIMAL(10,2)
)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN quantidade * preco;
END$$

DELIMITER ;
```

O MySQL não disponibiliza `CREATE OR REPLACE FUNCTION` na mesma forma de `CREATE OR REPLACE VIEW`. Portanto, a remoção precede uma nova criação.

---

## Consultando funções

Para listar as funções armazenadas:

```sql
SHOW FUNCTION STATUS
WHERE Db = DATABASE();
```

Para obter a instrução usada na criação:

```sql
SHOW CREATE FUNCTION calcular_valor_estoque;
```

Os metadados das funções também podem ser consultados em `information_schema.routines`:

```sql
SELECT routine_name, routine_type, data_type
FROM   information_schema.routines
WHERE  routine_schema = DATABASE()
AND    routine_type = 'FUNCTION'
ORDER BY routine_name;
```

Para remover uma função:

```sql
DROP FUNCTION IF EXISTS calcular_valor_estoque;
```

---

## Procedure e function

| Característica | Procedure | Function |
|---|---|---|
| Invocação | `CALL procedimento();` | Geralmente dentro de uma expressão SQL |
| Retorno | Não é obrigatório; pode usar parâmetros `OUT` ou `INOUT` | Obrigatoriamente retorna um valor |
| Uso em `SELECT` | Não é usado diretamente | Pode ser usado diretamente |
| Alteração de dados | Pode executar operações de manipulação | Deve respeitar restrições da rotina e dos privilégios |

---

## Privilégios e características da rotina

Uma função armazenada pode receber características de execução como:

- `DETERMINISTIC`: o mesmo conjunto de argumentos sempre produz o mesmo resultado;
- `NOT DETERMINISTIC`: o resultado pode variar, por exemplo com funções de tempo;
- `READS SQL DATA`: a função lê dados do banco, mas não os altera;
- `NO SQL`: a função não utiliza SQL;
- `MODIFIES SQL DATA`: a função pode alterar dados.

Dependendo das configurações do servidor e da versão do MySQL, o usuário pode precisar declarar uma dessas características ou informar explicitamente uma cláusula `SQL SECURITY`.

---

## Boas práticas

1. Dê preferência a funções que tenham uma responsabilidade pequena e bem definida.
2. Não utilize uma função para executar operações complexas de `INSERT`, `UPDATE` ou `DELETE` quando uma procedure é mais adequada.
3. Declare `DETERMINISTIC` somente quando o resultado realmente depende apenas dos argumentos.
4. Evite funções que consultem várias tabelas em cada linha de uma consulta extensa, pois podem reduzir o desempenho.
5. Confirme se o valor retornado possui tamanho suficiente, especialmente em funções que usam `VARCHAR`.
6. Remova funções antigas antes de recriar uma definição corrigida, conforme necessário.

---

# Exercícios

### Padaria
1. Crie uma função que calcule o preço com 10% de acréscimo e retorne um valor decimal.
2. Crie uma função que receba uma quantidade vendida e retorne `UNITÁRIO` ou `ATACADO` a partir de um limite informado.
3. Crie uma função que receba o código do fornecedor e consulte seu nome em uma tabela.
4. Crie uma função que informe se o saldo de farinha no estoque está abaixo de cinco unidades.
5. Crie uma função que calcule o tempo, em dias, entre a fabricação e a validade de um lote.

### Farmácia
6. Crie uma função que calcule o total de um item a partir da quantidade vendida e do preço unitário.
7. Crie uma função que informe se a quantidade de um medicamento está esgotada, baixa ou suficiente.
8. Crie uma função que receba a data atual e a validade de um medicamento e informe se ele está vencido.
9. Crie uma função que retorne o nome de uma classe terapêutica pelo código cadastrado.
10. Crie uma função que calcule o desconto aplicado a uma compra farmacêutica, com limite máximo definido.

### Agência de Viagens
11. Crie uma função que calcule o valor final de uma passagem com acréscimo de taxa aeroportuária.
12. Crie uma função que informe a categoria de um pacote pelo número de vagas disponíveis.
13. Crie uma função que receba nome e sobrenome separadamente e devolva o nome completo em maiúsculas.
14. Crie uma função que calcule quantos dias faltam para a data da viagem.
15. Crie uma função que classifique uma diária como econômica ou premium a partir do preço.

### Supermercado
16. Crie uma função que calcule o preço de uma venda com base em quantidade e preço unitário.
17. Crie uma função que determine se um produto está com estoque crítico.
18. Crie uma função que receba o código de uma categoria e retorne o percentual de imposto associado.
19. Crie uma função que consulte uma função existente e liste todas as funções armazenadas do banco atual.
20. Crie uma função que remova espaços extras do nome de um produto e devolva o texto em formato padronizado.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="16-procedure.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="18-grant.md">Próximo</a>
    </td>
  </tr>
</table>
