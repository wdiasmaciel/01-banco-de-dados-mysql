<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="09-relacionamentos.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

## Exemplos de SELECT entre tabelas

### SELECT simples com WHERE e ORDER BY

Antes de combinar tabelas, revisamos o básico: filtrar e ordenar dados de uma única tabela.

```sql
SELECT nome, endereco
FROM Fornecedor
WHERE nome LIKE '%Ltda%'
ORDER BY nome ASC;
```

### SELECT com subquery (subconsulta) entre tabelas

1. Aqui buscamos todos os produtos de um fornecedor específico.

2. Mas, em vez de "sabermos de cor" o `cnpj`, usamos uma subquery que busca esse `cnpj` a partir do nome do fornecedor.

3. Uma forma de combinar informação de duas tabelas sem usar `JOIN` explicitamente.

```sql
SELECT nome
FROM Produto
WHERE cnpj_fornecedor = (
    SELECT cnpj 
    FROM Fornecedor 
    WHERE nome = 'Distribuidora Alfa Ltda'
);
```


Quais empregados possuem salário maior que o do empregado 7749?  
    
    SELECT empno, nome
    FROM   emp
    WHERE  salario > = 
                   (SELECT salario
                    FROM   emp
                    WHERE  empno > 7749)
            
    
    SELECT codDVD, nome
FROM   DVD 
WHERE  cor = 
  (SELECT cor
   FROM   DVD 
   WHERE  nome = 'Top Gang')
AND codgenero =
   (SELECT codgenero
    FROM   DVD
    WHERE  nome = 'O sexto sentido') 

Quais os empregados trabalham no mesmo departamento do
empregado 7654?

SELECT empno, nome, deptno
FROM   emp
WHERE  deptno = 
   (SELECT deptno 
    FROM   emp
    WHERE  empno = 7654)



Qual o empregado possui o maior salário na empresa?   
   
   SELECT empno, nome, salario
   FROM   emp
   WHERE  salario =
       (SELECT MAX(salario)
        FROM   emp)	


### SELECT combinando duas tabelas (sem join)

1. Sem o `JOIN` explícito, é possível combinar tabelas listando-as no `FROM` separadas por vírgula e relacionando-as no `WHERE`. 

2. Funciona, mas é considerado um estilo antigo.

```sql
SELECT p.nome AS produto, f.nome AS fornecedor
FROM Produto p, Fornecedor f
WHERE p.cnpj_fornecedor = f.cnpj;
```

> **OBS:** essa consulta produz exatamente o mesmo resultado que um `INNER JOIN`.
> Mas é considerada uma forma antiga e menos legível de escrever a mesma coisa. 
> Hoje em dia, prefira sempre a sintaxe explícita com `JOIN`.

### SELECT combinando três tabelas

Um exemplo mais completo: nome do produto, nome da filial, preço e quantidade em estoque, relacionando informação de `Produto`, `Filial` e `Estoque` em uma única consulta.

```sql
SELECT p.nome AS Produto, f.nome AS Filial, e.preco AS Preço, e.quantidade As Quantidade
FROM Produto p, Estoque e, Filial f
WHERE p.id = e.id_produto 
  AND e.cnpj_filial = f.cnpj
ORDER BY p.nome, f.nome;
```

### SELECT com DISTINCT entre tabelas

Para saber quais fornecedores **realmente têm** produtos cadastrados (sem repetir o mesmo fornecedor várias vezes, caso ele tenha mais de um produto):

```sql
SELECT DISTINCT f.nome
FROM Fornecedor f, Produto p
WHERE f.cnpj = p.cnpj_fornecedor;
```

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


---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="09-relacionamentos.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
