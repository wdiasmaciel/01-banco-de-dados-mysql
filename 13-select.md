<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="12-grup-by-having.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

# Select 


## Operadores aritméticos e precedência

```sql
SELECT id_produto, cnpj_filial, preco, 12 * preco + 100
FROM   Estoque;
```

```sql
SELECT id_produto, cnpj_filial, preco, 12 * (preco + 100)
FROM   Estoque;
```

```sql
SELECT (SELECT nome FROM Produto WHERE id = e.id_produto) AS nome,
       e.preco AS "preço",
       12 * e.preco + 100 AS "novo preço"
FROM   Estoque e;
```

```sql
SELECT (SELECT nome FROM Produto WHERE id = e.id_produto) AS nome,
       e.preco As "preço",
       12 * (e.preco + 100) AS "novo preço"
FROM   Estoque e;
```

---

## Concatenação de strings

No MySQL, `+` é **somente aritmético**. A função correta é `CONCAT()`.

```sql
SELECT CONCAT(nome, ' está localizado em ', endereco) AS "Detalhes do Fornecedor"
FROM   Fornecedor;
```

---

## Operações com datas

No MySQL, a função `DATE_ADD()` soma um intervalo de tempo a uma data. 

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL 5 DAY) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL -5 DAY) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL 1 MONTH) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL 2 YEAR) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

```sql
SELECT id_produto, cnpj_filial, validade,
       DATE_ADD(validade, INTERVAL 30 MINUTE) AS "Data limite para descarte"
FROM   Estoque
WHERE  validade >= '2026-07-01';
```

```sql
SELECT NOW() AS hora_atual,
       DATE_ADD(NOW(), INTERVAL 30 MINUTE) AS daqui_a_30_minutos;
```
 
```sql
SELECT DATE_FORMAT(NOW(), '%d/%m/%Y %H:%i:%s') AS hora_atual,
       DATE_FORMAT(
           DATE_ADD(NOW(), INTERVAL 30 MINUTE),
           '%d/%m/%Y %H:%i:%s'
       ) AS daqui_a_30_minutos;
```

---

## Operador LIKE e curingas

Caracteres curingas no MySQL: 
1. `%` representa zero ou vários caracteres.
2. `_` representa exatamente um caractere.

```sql
SELECT cnpj, nome
FROM   Fornecedor
WHERE  nome LIKE 'D%'; -- nome tem que iniciar com a letra maiúscula D seguida de qualquer coisa.
-- Retorna: Distribuidora Alfa Ltda, Delta Bebidas Ltda.
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE 'S%L'; -- nome tem que iniciar com a letra maiúscula S seguida de qualquer coisa e terminar com a letra maiúscula L.
-- Retorna: Suco de Laranja 1L (começa com S, termina com L)
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '_o%'; -- nome tem que iniciar com uma única letra qualquer antes da letra minúscula "o" seguida de qualquer coisa. 
-- Retorna: Fone de Ouvido Bluetooth (segunda letra = 'o')
```

---

## ESCAPE no LIKE

Usado para procurar literalmente pelo caractere `%` ou `_` dentro de um texto (em vez de tratá-los como curinga). 

No MySQL, a sintaxe `ESCAPE` é `\`. 

Como nenhum dado do projeto contém `%` (`\%`) ou `_` (`\_`) no nome, inserimos registros de teste para demonstrar:

```sql
INSERT INTO Produto (id, cnpj_fornecedor, nome) VALUES 
(7, '55555555000105', 'Kit_Degustação'),
(8, '55555555000105', 'Iogurte 0% Lactose');
``` 

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '%\_%' ESCAPE '\\';
-- Retorna: Kit_Degustacao (o '\' antes do '_' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '%#_%' ESCAPE '\#';
-- Retorna: Kit_Degustacao (o '\' antes do '_' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```


```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '%\%%' ESCAPE '\\';
-- Retorna: Iogurte 0% Lactose (o '\' antes do '%' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```

```sql
SELECT nome
FROM   Produto
WHERE  nome LIKE '%@%%' ESCAPE '\@';
-- Retorna: Iogurte 0% Lactose (o '\' antes do '%' indica que ele deve
-- ser tratado como caractere literal, não como curinga)
```

> **OBS:** 
1. No MySQL, a barra invertida (`\`) já é o caractere de escape padrão.
2. Então, `ESCAPE '\\'` costuma ser redundante.
3. O exemplo acima funcionaria mesmo sem a cláusula `ESCAPE`. 
4. Ainda assim, vale mostrar a sintaxe explícita, já que ela é exigida em outros bancos (como o SQL Server).

---

## NOT IN

Encontrar fornecedores que **não têm** algum produto associado:

```sql
SELECT nome
FROM   Fornecedor
WHERE  cnpj NOT IN (SELECT DISTINCT cnpj_fornecedor FROM Produto);
```

Encontrar o nome das filiais que não estejam entre ('Filial Centro', 'Filial Savassi'):

```sql
SELECT nome
FROM   Filial
WHERE  nome NOT IN ('Filial Centro', 'Filial Savassi');
-- Retorna: Filial Contagem
```

---

## Precedência de AND / OR

**OBS:** 
1. O `AND` tem precedência maior que o `OR`. 

2. Na primeira consulta, a condição é avaliada como `preco < 10 OR (quantidade > 50 AND cnpj_filial = '10101010000110')`. Ou seja: 
    * Qualquer item com `preco < 10` aparece, **mesmo que não seja da Filial Centro**. 

3. Na segunda consulta, os parênteses forçam que a filial seja sempre `'Filial Centro'`. 

```sql
SELECT id_produto, cnpj_filial, preco, quantidade
FROM   Estoque
WHERE  preco < 10
OR     quantidade > 50
AND    cnpj_filial = '10101010000110';
```

```sql
SELECT id_produto, cnpj_filial, preco, quantidade
FROM   Estoque
WHERE  (preco < 10
OR      quantidade > 50)
AND     cnpj_filial = '10101010000110';
```

---

## ORDER BY

1. **Por alias** (sem aspas duplas, que no MySQL têm outro significado por padrão):

```sql
SELECT id_produto, preco, preco * 1.5 AS `novo preço`
FROM   Estoque
ORDER BY `novo preço`;
```

**OBS**:
No MySQL padrão, aspas duplas ("), por padrão, podem representar strings literais, da mesma forma que aspas simples ('), quando o modo `SQL ANSI_QUOTES` não está ativo.

Exemplo:
```sql
SELECT "Produto";  -- Tratado como string literal: Produto
```
```sql
SELECT "nome";     -- Em alguns casos, também pode funcionar como identificador
```

Por isso, para gerar aliases com espaços ou caracteres especiais, é mais seguro usar aspas simples ou, preferencialmente, crases para identificadores:

```sql
SELECT nome AS 'Nome do produto'  -- Aliases com string
  FROM produto;
```

```sql
SELECT nome AS `Nome do produto`  -- Aliases como identificador MySQL
  FROM produto; 
```

2. **Por posição da coluna no SELECT:**

```sql
SELECT id, descricao
FROM   Identificacao
ORDER BY 2;
```

3. **Por mais de uma coluna:**

```sql
SELECT nome, cnpj_fornecedor
FROM   Produto
ORDER BY cnpj_fornecedor, nome;
```

4. **Por uma coluna que não está no SELECT**:

```sql
SELECT nome, endereco
FROM   Filial
ORDER BY cnpj;
```

5. **Múltiplas colunas com direções diferentes:**

```sql
SELECT id_produto, cnpj_filial
FROM   Estoque
ORDER BY cnpj_filial, id_produto DESC;
```

---

## Funções de manipulação de caracteres

| Função | Observação |
|---|---|
| `LENGTH()` / `CHAR_LENGTH()` | `LENGTH()` conta bytes; `CHAR_LENGTH()` conta caracteres (importante com acentos) |
| `LOCATE(busca, texto, pos)` | Localiza a primeira ocorrência da substring `busca` em `texto`, começando pela posição `pos`; retorna 0 se não encontrar |
| `LEFT()` / `RIGHT()` | Retorna a quantidade de caracteres indicada à esquerda (`LEFT`) ou à direita (`RIGHT`) de um texto |
| `LTRIM()` / `RTRIM()` | Remove espaços em branco do início (`LTRIM`) ou do fim (`RTRIM`) de um texto |
| `REPLACE()` | Substitui todas as ocorrências de um trecho por outro dentro do texto informado |
| `SUBSTRING()` | Extrai uma parte do texto a partir de uma posição inicial e, opcionalmente, com um comprimento definido |

```sql
SELECT "pão novo", LENGTH("pão novo") AS bytes, CHAR_LENGTH("pão novo") AS tamanho;
```

```sql
SELECT nome, LENGTH(nome) AS bytes, CHAR_LENGTH(nome) AS tamanho
FROM   Fornecedor;
```

```sql
SELECT nome, LOCATE('i', nome, 3) AS posicao
FROM   Produto;
```

```sql
SELECT nome, LEFT(nome, 3) AS esquerda, RIGHT(nome, 3) AS direita
FROM   Produto;
```

```sql
SELECT LTRIM('   Arroz   ') AS esquerda, RTRIM('   Feijao   ') AS direita;
```

```sql
SELECT nome, REPLACE(nome, 'a', 'o') AS troca
FROM   Produto;
```

```sql
SELECT nome, SUBSTRING(nome, 1, 5) AS substring
FROM   Produto;
```

---

## Funções numéricas

| Função | Descrição |
|---|---|
| `ABS()` | Retorna o valor absoluto de um número, ou seja, o mesmo número sem sinal negativo. |
| `CEILING()` | Arredonda um número para cima, retornando o menor inteiro maior ou igual ao valor informado. |
| `FLOOR()` | Arredonda um número para baixo, retornando o maior inteiro menor ou igual ao valor informado. |
| `SIGN()` | Retorna `-1` para números negativos, `0` para zero e `1` para números positivos. |
| `ROUND()` | Arredonda um número para uma quantidade específica de casas decimais. |
| `TRUNCATE()` | Remove casas decimais de um número sem realizar arredondamento. |

```sql
SELECT ABS(-1.0) AS abs1, ABS(0.0) AS abs2, ABS(1.0) AS abs3;
```

```sql
SELECT preco, CEILING(preco) AS teto, FLOOR(preco) AS piso
FROM   Estoque;
```

```sql
SELECT SIGN(-10.0) AS negativo, SIGN(0.0) AS zero, SIGN(10.0) AS positivo;
```

```sql
SELECT ROUND(748.58, -1) AS round1, 
       ROUND(748.58, -2) AS round2,
       ROUND(748.58, 0)  AS round3, 
       ROUND(748.58, 1)  AS round4;
```

**OBS**:
1. `ROUND()` usa casas decimais negativas para arredondar partes inteiras. Exemplo:
```sql
ROUND(748.58, -1);  -- dezenas: 750
ROUND(748.58, -2);  -- centenas: 700
ROUND(748.58, 0);   -- unidades: 749
ROUND(748.58, 1);   -- uma casa decimal: 748.6
```
2. No caso do exemplo:

```sql
SELECT ROUND(748.58, -1) AS round1;
```

* 748.58: número a arredondar.
* -1: arredonda à esquerda da vírgula decimal, ou seja, para dezenas.
* Como 748,58 está mais próximo de 750 do que de 740, o resultado é: 750.

```sql
SELECT TRUNCATE(150.98, 0) AS truncate1, 
       TRUNCATE(150.98, 1) AS truncate2;
```

---

## CASE WHEN

Classificar o preço dos itens de estoque e traduzir o código da filial para o nome:

```sql
SELECT id_produto,
       cnpj_filial,
       preco,
       CASE 
           WHEN preco < 10 THEN 'Baixo'
           ELSE 'Alto'
       END AS "Faixa de Preço",
       CASE cnpj_filial
           WHEN '10101010000110' THEN 'Filial Centro'
           WHEN '20202020000120' THEN 'Filial Savassi'
           WHEN '30303030000130' THEN 'Filial Contagem'
       END AS "Nome da Filial"
FROM   Estoque;
```

Para preço baixo, médio e alto:

```sql
SELECT id_produto,
       cnpj_filial,
       preco,
       CASE
           WHEN preco < 10 THEN 'Baixo'
           WHEN preco <= 20 THEN 'Médio'
           ELSE 'Alto'
       END AS "Faixa de Preço",
       CASE cnpj_filial
           WHEN '10101010000110' THEN 'Filial Centro'
           WHEN '20202020000120' THEN 'Filial Savassi'
           WHEN '30303030000130' THEN 'Filial Contagem'
       END AS "Nome da Filial"
FROM Estoque;
```

### Diferença entre as formas de `CASE`

No exemplo anterior, usamos o **`CASE` pesquisado**:

```sql
CASE
    WHEN preco < 10 THEN 'Baixo'
    WHEN preco <= 20 THEN 'Médio'
    ELSE 'Alto'
END
```

Nesse formato, cada condição escrita depois de `WHEN` é avaliada separadamente.

Já este formato inicia um **`CASE` simples**:

```sql
CASE preco
    WHEN preco < 10 THEN 'Baixo'
    WHEN preco <= 20 THEN 'Médio'
    ELSE 'Alto'
END
```

Ao escrever `CASE preco`, o MySQL compara o valor de `preco` com cada expressão escrita em `WHEN`. Assim, ele tenta equivalências semelhantes a estas:

```text
preco = (preco < 10)
preco = (preco <= 20)
```

Como essas expressões podem gerar resultados booleanos (`0` ou `1`), esse formato não produz corretamente as classificações de preço baixas, médias e altas. 

Para condições comparativas, deve-se omitir a expressão entre `CASE` e o primeiro `WHEN`.

O `CASE` simples é adequado quando a expressão possui valores conhecidos, por exemplo:

```sql
CASE preco
    WHEN 10 THEN 'Preço igual a dez'
    WHEN 20 THEN 'Preço igual a vinte'
    ELSE 'Outro preço'
END
```

Classificando a validade por mês de vencimento:

```sql
SELECT id_produto, cnpj_filial, validade,
       CASE
           WHEN validade BETWEEN '2026-10-01' AND '2026-10-31' THEN 'Vence em outubro'
           WHEN validade BETWEEN '2026-11-01' AND '2026-11-30' THEN 'Vence em novembro'
           WHEN validade BETWEEN '2026-12-01' AND '2026-12-31' THEN 'Vence em dezembro'
           ELSE 'Vence em outro período'
       END AS "Período de vencimento"
FROM   Estoque;
```

---

## IFNULL 

O projeto não tem, no momento, nenhuma coluna numérica que aceite `NULL`. 

Para demonstrar `IFNULL()` de forma prática, adicionamos temporariamente uma coluna opcional `desconto` a `Estoque`:

```sql
ALTER 
TABLE Estoque 
  ADD COLUMN desconto DECIMAL(10,2) NULL;
```

```sql
UPDATE Estoque 
   SET desconto = 2.00 
 WHERE id_produto = 1 
   AND cnpj_filial = '10101010000110';
-- Os demais registros permanecem com desconto = NULL
```

```sql
SELECT id_produto, cnpj_filial, preco, desconto,
       preco - IFNULL(desconto, 0) AS preco_final
FROM   Estoque;
```

> **OBS:** 
1. Sem o `IFNULL()`, `preco - desconto` resultaria em `NULL` para todas as linhas em que `desconto` não foi informado.
2. Porém, qualquer operação aritmética com `NULL` retorna `NULL`.

---

## CAST, CONVERT e formatação de datas

```sql
SELECT id_produto, 
       preco,
       CONCAT('O preço é R$ ', CAST(preco AS CHAR)) AS preco_formatado
FROM   Estoque;
```

Para formatar datas, o MySQL usa `DATE_FORMAT()` com máscaras de formato:

```sql
SELECT DATE_FORMAT(DATE '2026-10-05', '%d/%m/%Y');
```

```sql
SELECT id_produto, 
       cnpj_filial, 
       validade,
       DATE_FORMAT(validade, '%d/%m/%Y') AS data_formatada
FROM   Estoque;
```

Outros formatos úteis:

```sql
-- Ano-mês-dia, adequado para exibição internacional (padrão ISO):
SELECT id_produto, 
       cnpj_filial, 
       validade,
       DATE_FORMAT(validade, '%Y-%m-%d') AS data_iso
FROM   Estoque;
```

```sql
SELECT DATE_FORMAT(DATE '2026-10-05', '%Y-%m-%d');
```

```sql
-- Dia da semana, dia, mês por extenso e ano:
SELECT id_produto, 
       cnpj_filial, 
       validade,
       DATE_FORMAT(validade, '%W, %d de %M de %Y') AS data_por_extenso
FROM   Estoque;
```

```sql
SELECT DATE_FORMAT(DATE '2026-10-05', '%W, %d de %M de %Y');
```

```sql
-- Dia, mês abreviado e ano:
SELECT id_produto, 
       cnpj_filial, 
       validade,
       DATE_FORMAT(validade, '%d/%b/%Y') AS data_abreviada
FROM   Estoque;
```

```sql
SELECT DATE_FORMAT(DATE '2026-10-05', '%d/%b/%Y');
```

---

## INSERT INTO ... SELECT

```sql
CREATE TABLE FornecedorBH (
    cnpj      VARCHAR(14)  NOT NULL,
    nome      VARCHAR(100) NOT NULL,
    endereco  VARCHAR(200) NOT NULL,
    PRIMARY KEY (cnpj)
);

INSERT INTO FornecedorBH (cnpj, nome, endereco)
SELECT cnpj, nome, endereco
FROM   Fornecedor
WHERE  endereco LIKE '%Belo Horizonte%';
```

---

# Exercício

1. Calcule o valor do estoque multiplicando o preço pela quantidade disponível; em seguida, adicione uma taxa fixa de embalagem de R$ 25,00 ao total.

2. Calcule 15% de imposto sobre o preço unitário e depois some esse valor ao próprio preço.

3. Exiba o endereço de cada fornecedor seguido de duas barras e do respectivo nome, utilizando `CONCAT_WS()`. `CONCAT_WS()` significa Concatenate With Separator: concatena vários textos, inserindo um separador entre eles. Sintaxe: 

```sql
CONCAT_WS(separador, valor1, valor2, ...);
```
Exemplo:

```sql
SELECT CONCAT_WS(' - ', nome, endereco)
FROM Fornecedor;
```

4. Consulte os produtos cuja validade no estoque vencerá em exatamente 30 dias a partir da data `'2026-10-29'`, usando `DATE_ADD()`.

5. Liste os fornecedores cujo nome termine com `Ltda.` e tenha qualquer quantidade de caracteres antes dessa expressão.

6. Liste os fornecedores cujo endereço comece com `Rua`, mas termine com uma cidade diferente de Belo Horizonte, utilizando `LIKE`.

7. Localize o caractere literal `%` no nome dos produtos com `LIKE`, preparando o padrão com `ESCAPE`.

8. Consulte os fornecedores cujo CNPJ não esteja associado a nenhum produto.

9. Aplique parênteses em uma consulta que atenda às duas regras: preço até 12 ou validade posterior a `'2026-12-31'`, mas somente em itens com quantidade diferente de zero.

10. Mostre os produtos com o respectivo CNPJ e nome de suas filiais e ordene o resultado pelo CNPJ e pelo nome do produto em ordem crescente.

11. Converta o nome de cada produto para maiúsculas antes de contar quantos caracteres ele possui. Apresente os resultados.

12. Exiba a posição da primeira vogal `a` do nome dos produtos, pesquisando a partir da quarta posição.

13. Substitua a palavra `carioca` por `especial` nos nomes dos produtos e combine o resultado com suas cinco primeiras letras.

14. Remova os espaços laterais do texto `'   Produtos em promoção   '` e apresente o resultado ao lado do texto original.

15. Arredonde o número 1234,567 para duas casas decimais, para a unidade e para a centena mais próximas.

16. Determine o teto e o piso de 19,01 e informe o sinal de `-24`, `0` e `150`.

17. Classifique cada item do estoque como `Esgotado`, `Baixa`, `Normal` ou `Alta`, conforme a quantidade: zero, até 20, até 50 ou acima de 50.

18. Consulte os itens de estoque e classifique seu prazo como `Vencido`, `Vence em breve` ou `Prazo regular`, comparando as datas com `'2026-10-29'`.

19. Substitua um desconto ausente por 10% do preço e apresente o valor líquido após essa regra de negócio.

20. Crie uma lista com validade formatada como `aaaa` ('%Y'), `mês/ano` ('%m/%Y') e `nome do dia por extenso` ('%W').

* Ano: %Y
* Mês e ano: %m/%Y
* Nome do dia da semana por extenso: %W

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="12-grup-by-having.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>