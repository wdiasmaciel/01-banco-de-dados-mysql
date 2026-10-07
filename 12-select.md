<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="11-join.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

## ORDER BY, funções agregadas, GROUP BY e HAVING

Nesta seção, o foco é a tabela `Produto` relacionada com a tabela `Estoque`. Como o preço não está na própria tabela `Produto` (ele varia por filial), é necessário relacionar as duas tabelas. 

### ORDER BY simples (uma coluna)

```sql
SELECT nome 
FROM Produto
ORDER BY nome ASC;
```

### ORDER BY decrescente

```sql
SELECT nome 
FROM Produto
ORDER BY nome DESC;
```

### GROUP BY: agrupar produtos por nome

```sql
SELECT p.nome
FROM Produto p
GROUP BY p.nome;
```

```sql
SELECT p.nome
FROM Produto p
GROUP BY p.id, p.nome;
```

```sql
SELECT p.id, p.nome
FROM Produto p
GROUP BY p.id, p.nome;
```

### COUNT(): em quantas filiais cada produto é vendido

```sql
SELECT p.nome AS produto, COUNT(*) AS 'quantidade de filiais'
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome;
```

### SUM(): quantidade total em estoque de cada produto (somando todas as filiais)

```sql
SELECT p.nome AS produto, SUM(e.quantidade) AS quantidade_total
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome;
```

### MAX() e MIN(): maior e menor preço praticado para cada produto

```sql
SELECT p.nome AS produto,
       MAX(e.preco) AS maior_preco,
       MIN(e.preco) AS menor_preco
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome;
```

### AVG(): preço médio de cada produto entre as filiais

```sql
SELECT p.nome AS produto, AVG(e.preco) AS preco_medio
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome;
```

### GROUP BY com ORDER BY combinados

Podemos ordenar o resultado de uma consulta agregada: por exemplo, mostrar os produtos do mais caro (em média) para o mais barato.

```sql
SELECT p.nome AS produto, AVG(e.preco) AS preco_medio
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome
ORDER BY preco_medio DESC;
```

### HAVING: filtrando grupos após a agregação

`HAVING` funciona como um `WHERE`, mas aplicado **depois** do `GROUP BY`.

Ou seja: filtra grupos com base no resultado da agregação, algo que o `WHERE` não consegue fazer diretamente. 

Aqui, mostramos apenas os produtos vendidos em **mais de uma** filial:

```sql
SELECT p.nome AS produto, COUNT(*) AS 'quantidade de filiais'
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome
HAVING COUNT(*) > 1;
```

> **OBS:** por que não poderíamos escrever `WHERE COUNT(*) > 1` no lugar do `HAVING`? 
>
> Resposta: 
>
> O `WHERE` é avaliado **antes** do agrupamento, linha a linha, então ele ainda não "conhece" o resultado de `COUNT(*)` naquele momento: 
>
> Só o `HAVING`, que roda depois do `GROUP BY`, tem acesso ao valor agregado.

### HAVING com condição sobre AVG()

Produtos cujo preço médio ultrapassa `R$ 20,00`:

```sql
SELECT p.nome AS produto, AVG(e.preco) AS "preco médio"
FROM Produto p
JOIN Estoque e 
ON p.id = e.id_produto
GROUP BY p.id, p.nome
HAVING AVG(e.preco) > 20.00
ORDER BY 'preco médio' DESC;
```

---

# Exercícios




<!--
SELECT nome, salario, 12*salario+100
FROM   emp

SELECT nome, salario, 12*(salario+100)
FROM   emp

SELECT nome + ' é um ' + cargo AS "Detalhes do Empregado"
FROM   emp


Funções de conversão 
de maiúsculas e minúsculas

SELECT codcliente, data, data + 5 AS "Data máxima de devolução"
FROM   locacao
WHERE  data >= '07-01-2019'


Utilize o operador LIKE para executar pesquisas curinga de valores de string de pesquisa válidos.
As condições de pesquisa podem conter caracteresliterais ou números.
        ·  % denota zero ou muitos caracteres.
        ·  _ denota um caractere.

SELECT codcliente, nome
FROM   cliente
WHERE  nome LIKE 'D%' 


SELECT nome
FROM   DVD
WHERE  nome LIKE 'O%n' 

SELECT nome
FROM   DVD
WHERE  nome LIKE '_o%'


É possível usar o identificador ESCAPE para procurar por “ % “ ou “_“.

SELECT nome
FROM   dept
WHERE  nome LIKE '%/_%' ESCAPE '/'

SELECT nome
FROM   dept
WHERE  nome LIKE '%?%%' ESCAPE '?'

Teste se um valor é nulo utilizando o operador IS NULL.

SELECT codcliente, codDVD, data 
FROM   locacao
WHERE  datadevolucao IS NULL

SELECT nome, estado 
FROM   cliente
WHERE  estado NOT IN ('RJ', 'SP')

SELECT nome, cor, status, codgenero
FROM   DVD
WHERE  cor = 'PR' 
OR     status = 'L'
AND    codgenero = 3

SELECT nome, cor, status, codgenero
FROM   DVD
WHERE  (cor = 'PR' 
OR     status = 'L')
AND    codgenero = 3

SELECT    cor, valor * 1.5 AS "Novo valor" 
FROM      preco
ORDER BY "Novo valor"

SELECT   codgenero, descricao
FROM     genero
ORDER BY 2


SELECT   nome, codgenero 
FROM     DVD
ORDER BY codgenero, nome

SELECT   nome, cidade
FROM     cliente
ORDER BY estado

SELECT   nome, cor
FROM     DVD
ORDER BY cor, nome DESC




UPPER
LOWER


Funções de manipulação
de caracteres

SUBSTRING
LEN
CHARINDEX
LEFT
RIGHT
LTRIM
RTRIM
REPLACE

SELECT nome, SUBSTRING(nome, 1, 5) AS sub
FROM   DVD

SELECT nome, LEN(nome) AS tamanho
FROM   paciente

Retorna a posição inicial da expressão especificada em uma string de caracteres

CHARINDEX(expressao1, expressao2 [,pos_inicial])
expressao1 – é uma expressão contendo uma seqüência de caracteres a ser encontrada
expressao2 – é uma expressão a ser procurada pela seqüência especificada
pos_inicial – é a posição do caractere de início para a pesquisa da expressao1 na expressao2. Se este argumento for omitido, ou se for um valor negativo ou zero, a procura começa no início da expressao2.

SELECT nome, CHARINDEX('i', nome, 3) As "Posição" 
FROM   DVD

Retornam parte de uma string de caracteres começando em um número específico de caracteres a partir da esquerda/direita da string.
LEFT(expressao, pos_inicial)
RIGHT(expressao, pos_inicial)
expressao – é uma string ou expressão envolvendo uma coluna
pos_inicial – é a posição inicial

SELECT nome, LEFT(nome, 3) AS Esquerda, RIGHT(nome, 3) AS DIREITA 
FROM   DVD

Removem espaços em branco no início ou no fim de uma string de caracteres.
LTRIM(expressao)
RTRIM(expressao)
expressao – é uma string ou expressão envolvendo uma coluna

SELECT LTRIM('   STR1   ') Esq, RTRIM('   STR2   ') Dir     

Substitui todas as ocorrências de uma determinada string de caracteres em uma expressão por outra string de caracteres.
REPLACE(expressao1, expressao2, expressao3)
expressao1 – é uma string de caracteres ou expressão envolvendo uma coluna onde a string será substituída
expressao2 – é a string a ser substituída
expressao3 – é a string de substituição

SELECT nome, REPLACE(nome, 'A', 'O') AS Troca
FROM   DVD


ROUND
ABS
CEILING
FLOOR
SIGN

Retorna uma expressão numérica arredondada para um tamanho ou precisão especificada.  
ROUND(expressao, tamanho [,funcao])  
expressao – expressão numérica a ser arredondada
tamanho – precisão de arredondamento. Quando positivo, a expressão é arredondada para o número de casas decimais especificadas pelo tamanho. Quando negativo, expressão é arredondada do lado esquerdo do ponto decimal, conforme especificado pelo tamanho.
funcao – tipo de operação a ser realizada. Quando omitido ou com valor 0 (default), a expressão é arredondada. Quando diferente de zero, a expressão é truncada.


SELECT ROUND(748.58, -1) R1, ROUND(748.58, -2) R2, ROUND(748.58, 0) R3, ROUND(748.58, 1) R4 
R1      R2      R3      R4  
------- ------- ------- ------- 
750.00  700.00  749.00  748.60

(1 row(s) affected)


Utilizando a função ROUND para truncar

SELECT ROUND(150.98, 0, 1) T1, ROUND(150.98, 1, 1) T2
T1      T2 
------- ------- 
150.00  150.90

(1 row(s) affected)

Retorna o valor absoluto (positivo) de uma expressão numérica.
ABS(expressao)
Expressao – expressão numérica a ser avaliada

SELECT ABS(-1.0) A1, ABS(0.0) A2, ABS(1.0) A3

A1   A2   A3   
---- ---- ---- 
1.0  .0   1.0

(1 row(s) affected)


CEILING - Retorna o menor inteiro maior ou igual a uma determinada expressão
CEILING(expressao)
expressao – expressão numérica a ser avaliada

FLOOR – Retorna o maior inteiro menor ou igual a uma determinada expressão
FLOOR(expressao)
expressao – expressão numérica a ser avaliada

SELECT valor, CEILING(valor) Teto, FLOOR(valor) Piso
FROM   preco
valor                          Teto                     Piso                                                  
------------------------------ ------------------------ --------------
4.0                            4.0                      4.0
2.5                            3.0                      2.0
2.0                            2.0                      2.0
3.0                            3.0                      3.0
3.5                            4.0                      3.0
4.5                            5.0                      4.0

(6 row(s) affected)

Retorna o sinal positivo (+1), zero (0) ou negativo (-1) de uma determinada expressão numérica
SIGN(expressao) 
expressao – expressão numérica a ser avaliada

SELECT SIGN(-10.0) Neg, SIGN(0.0) Zero, SIGN(10.0) Pos
Neg   Zero Pos   
----- ---- ----- 
-1.0  .0   1.0

(1 row(s) affected)

SELECT cidade, estado, 
 CASE estado
   WHEN 'MG' THEN 'Minas Gerais'
   WHEN 'RJ' THEN 'Rio de Janeiro'
   WHEN 'SP' THEN 'São Paulo'
   ELSE 'estado desconhecido'
  END "Nome do Estado"
FROM paciente
Cidade			estado		Nome do Estado
----------------  	---------	------------------
Belo Horizonte		MG		Minas Gerais
Sete Lagoas		MG		Minas Gerais
Sete Lagoas		MG		Minas Gerais
Rio de Janeiro		RJ		Rio de Janeiro
Jacareí			SP		São Paulo

SELECT codDVD, datadevolucao,
 CASE 
   WHEN datadevolucao BETWEEN '01-01-2019' AND '01-30-2019' THEN 'Dev. janeiro'
   WHEN datadevolucao BETWEEN '02-01-2019' AND '02-28-2019' THEN 'Dev. fevereiro'
   WHEN datadevolucao IS NULL THEN 'DVD não devolvido'
   ELSE 'data desconhecida'
  END "Data de devolução"
FROM locacao

select	codConta
,		ABS(saldo) saldo
,		case 
            when saldo < 0 then 'D'
            when saldo >= 0 then 'C'
        end as 'D\C'
,		case numeroAgencia
            when 1010 then 'Savassi'
            when 1020 then 'Centro'
            when 1030 then 'Minas Shopping'
            when 1040 then 'Cidade Nova'
        end "Nome da Agência"
from	conta


SELECT comissao, salario, ISNULL(comissao, 0) + salario AS "Comissão + Salário"
FROM   emp

Convertem explicitamente um tipo de dados para um outro tipo de dados.
CAST(expressao AS tipo_de_dados)
CONVERT(tipo_de_dados[(tam)], expressao [,estilo]) 
expressao – expressão a ser convertida
tipo_de_dados – tipo de dados para o qual a expressao será convertida
tam – parâmetro opcional para o tamanho do tipo de dados
estilo – formato de data a ser utilizado para converter dados do tipo DATETIME ou SMALLDATETIME para dados do tipo caractere.

SELECT cor, 'O valor é ' + CAST(VALOR AS varchar) valor
FROM   preco 

SELECT codDVD, CONVERT(VARCHAR, data, 100) Data
FROM   locacao



INSERT INTO mineiro (codigo, nome, cidade)
(SELECT codcliente, nome, cidade
 FROM   cliente
 WHERE  estado = 'MG')
-->


---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="11-join.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
