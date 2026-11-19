# 23 - Cursores no MySQL

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="22-trigger.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="24-drop.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Cursores em rotinas armazenadas

Um **cursor** permite percorrer, uma linha por vez, o conjunto de resultados de uma consulta dentro de uma procedure armazenada. Use-o quando uma operação realmente precisar tratar as linhas individualmente.

Sempre que possível, prefira uma única instrução SQL baseada em conjuntos — como `UPDATE`, `INSERT ... SELECT` ou uma consulta agregada. Em geral, essas operações são mais simples e eficientes do que processar cada linha com um cursor.

---

## Etapas de uso

Um cursor no MySQL segue estas etapas:

1. Declare as variáveis locais que receberão os valores.
2. Declare o cursor para uma consulta `SELECT`.
3. Declare um handler para reconhecer o fim dos resultados.
4. Abra o cursor com `OPEN`.
5. Leia cada linha com `FETCH`.
6. Encerre o laço ao chegar ao fim dos resultados.
7. Feche o cursor com `CLOSE`.

As declarações têm uma ordem obrigatória dentro de um bloco `BEGIN ... END`: variáveis e condições primeiro, cursores depois e handlers por último. As instruções executáveis vêm após as declarações.

---

## Calculando o valor do estoque

No modelo da empresa, a tabela `Estoque` registra quantidade e preço por produto e filial. A procedure abaixo percorre esses registros e calcula o valor total estimado do estoque:

```sql
DELIMITER $$

CREATE PROCEDURE empresa.calcular_valor_total_estoque()
BEGIN
    DECLARE fim INT DEFAULT 0;
    DECLARE quantidade_item INT;
    DECLARE preco_item DECIMAL(10,2);
    DECLARE total DECIMAL(12,2) DEFAULT 0;

    DECLARE cursor_estoque CURSOR FOR
        SELECT quantidade, preco
        FROM   empresa.Estoque;

    DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET fim = 1;

    OPEN cursor_estoque;

    leitura: LOOP
        FETCH cursor_estoque INTO quantidade_item, preco_item;

        IF fim = 1 THEN
            LEAVE leitura;
        END IF;

        SET total = total + (quantidade_item * preco_item);
    END LOOP;

    CLOSE cursor_estoque;

    SELECT total AS valor_total_estoque;
END$$

DELIMITER ;
```

Execute a procedure com `CALL`:

```sql
CALL empresa.calcular_valor_total_estoque();
```

O handler `NOT FOUND` altera `fim` quando o `FETCH` não encontra outra linha. O laço verifica essa condição e sai sem somar um valor inexistente. Para obter o mesmo total sem processar linha a linha, uma consulta baseada em conjuntos é mais simples e geralmente mais eficiente:

```sql
SELECT COALESCE(SUM(quantidade * preco), 0) AS valor_total_estoque
FROM   empresa.Estoque;
```

---

## Handler de fim dos resultados

O cursor não retorna uma linha especial para sinalizar o fim. O MySQL informa a condição `NOT FOUND`; um handler `CONTINUE` permite que a execução siga para a próxima instrução, onde o código pode verificar uma variável de controle.

```sql
DECLARE fim INT DEFAULT 0;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET fim = 1;
```

Declare o handler depois do cursor e das variáveis, mas antes de `OPEN` ou de qualquer outra instrução executável no bloco.

---

## Propriedades e cuidados

- Cursores do MySQL são somente para leitura: não atualizam a linha diretamente pelo cursor.
- São não roláveis: não permitem voltar a uma linha anterior ou pular para uma posição arbitrária.
- São assensíveis: o servidor pode usar uma tabela temporária interna para materializar o resultado.
- Devem ser abertos antes do primeiro `FETCH` e fechados depois do processamento.
- A consulta do cursor define as linhas e colunas disponíveis para o `FETCH`.
- A quantidade, ordem e tipos das variáveis informadas no `FETCH` devem corresponder às colunas retornadas pelo cursor.

Um cursor declarado dentro de uma procedure existe somente durante a execução do bloco que o declara.

---

## Limpando a demonstração

Remova a procedure de demonstração quando ela não for mais necessária:

```sql
DROP PROCEDURE IF EXISTS empresa.calcular_valor_total_estoque;
```

---

# Exercícios

1. Crie uma procedure com cursor para contar, um por um, os medicamentos de um lote que estão abaixo do estoque mínimo.
2. Percorra os itens de uma venda de supermercado e calcule o total aplicando a quantidade de cada item.
3. Use um cursor para calcular quantas reservas confirmadas existem para cada destino de uma agência de viagens.
4. Percorra os pães produzidos em um dia e some as quantidades destinadas a cada filial.
5. Declare variáveis adequadas para receber o código e o preço de cada medicamento retornado por um cursor.
6. Escreva a declaração de cursor que selecione somente produtos inativos de uma tabela de catálogo.
7. Crie um handler `NOT FOUND` que altere uma variável chamada `fim` para 1.
8. Ordene a consulta de um cursor para processar os fornecedores por nome.
9. Monte o esqueleto de uma procedure com as declarações na ordem obrigatória: variáveis, cursor e handler.
10. Acrescente `OPEN`, `FETCH`, uma condição de saída e `CLOSE` ao esqueleto do exercício anterior.
11. Explique o que ocorre quando o cursor não possui linhas e a procedure executa seu primeiro `FETCH`.
12. Explique por que o laço deve testar a variável de fim logo após `FETCH`.
13. Identifique o erro em um `FETCH` que fornece duas variáveis para um cursor cuja consulta retorna três colunas.
14. Reescreva um processamento linha a linha de descontos como uma única instrução `UPDATE`, se as regras permitirem.
15. Explique por que um cursor não é normalmente a primeira opção para somar valores de uma tabela.
16. Crie uma procedure que percorra as entregas pendentes de uma padaria e produza a quantidade total de caixas.
17. Crie uma procedure que liste, linha a linha, os códigos de viagens com vagas restantes.
18. Explique as diferenças entre cursor somente para leitura e um cursor que pudesse atualizar diretamente as linhas.
19. Indique por que o cursor deve ser fechado mesmo depois de concluir o laço normalmente.
20. Proponha um caso em que o processamento sequencial com cursor seja justificável e compare-o com uma solução baseada em conjuntos.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="22-trigger.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="24-drop.md">Próximo</a>
    </td>
  </tr>
</table>
