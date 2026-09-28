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

Funções de conversão 
de maiúsculas e minúsculas

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
