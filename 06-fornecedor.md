# 06 - Fornecedor

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="05-fornecedor.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="07-produto.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Criar o Banco de Dados

1. Crie o banco de dados `empresa`:

```sql
DROP DATABASE IF EXISTS empresa;
CREATE DATABASE empresa;
USE empresa;
```

ou

```sql
DROP SCHEMA IF EXISTS empresa;
CREATE SCHEMA empresa;
USE empresa;
```


# CREATE TABLE

1. Crie a tabela Fornecedor.

```sql
DROP TABLE IF EXISTS Fornecedor;

CREATE TABLE Fornecedor (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(256) NOT NULL UNIQUE,
    telefone VARCHAR(20) NOT NULL UNIQUE,
    endereco VARCHAR(255) NOT NULL
);

```

---

# DESCRIBE

1. Observe a estrutura da tabela `Fornecedor` usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Fornecedor;
```

ou

```sql
DESC Fornecedor;
```

---

# INSERT

1. Insira uma linha (registro) na tabela `Fornecedor`. O ID é gerado automaticamente:

```sql
INSERT INTO Fornecedor (nome, telefone, endereco) VALUES 
('Logística Brasil Ltda', '(11) 98765-4321', 'Av. Paulista, 1000 - São Paulo, SP');
```

---

# SELECT

1. Apresente os dados inseridos na tabela `Fornecedor`:

```sql
SELECT id, nome, telefone, endereco FROM Fornecedor;
```

---

# ID Repetido

1. Insira uma linha com `id` repetido:

```sql
INSERT INTO Fornecedor (id, nome, telefone, endereco) VALUES 
(1, 'Logística Brasil Ltda', '(11) 98765-4321', 'Av. Paulista, 1000 - São Paulo, SP');
```

O comando dispara um erro, porque não permite a inserção de IDs repetidos.

---

# Nome Repetido

1. Insira uma linha com `nome` repetido:

```sql
INSERT INTO Fornecedor (nome, telefone, endereco) VALUES 
('Logística Brasil Ltda', '(11) 98765-4321', 'Av. Paulista, 1000 - São Paulo, SP');
```

O comando dispara um erro, porque não permite a inserção de nomes repetidos.

---

# Telefone Repetido

1. Insira uma linha com `telefone` repetido:

```sql
INSERT INTO Fornecedor (nome, telefone, endereco) VALUES 
('Nova Logística Brasil Ltda', '(11) 98765-4321', 'Av. Paulista, 1000 - São Paulo, SP');
```

O comando dispara um erro, porque não permite a inserção de telefones repetidos.

---

# Endereço Repetido

1. Insira uma linha com `endereço` repetido:

```sql
INSERT INTO Fornecedor (nome, telefone, endereco) VALUES 
('Nova Logística Brasil Ltda', '(31) 98154-3227', 'Av. Paulista, 1000 - São Paulo, SP');
```

O comando é executado com sucesso, porque permite a inserção de endereços repetidos.

---

# SELECT

1. Apresente os dados inseridos na tabela `Fornecedor`:

```sql
SELECT * FROM Fornecedor;
```

--- 

# Dado Faltante

1. Insira uma linha sem nome:

```sql
INSERT INTO Fornecedor (telefone, endereco) 
VALUES ('(61) 3222-0000', 'SCS Quadra 4, Bloco A - Brasília, DF');
```

O comando dispara um erro, porque o nome é obrigatório, não pode ser nulo.

2. Insira uma linha sem telefone:

```sql
INSERT INTO Fornecedor (nome, endereco) 
VALUES ('Atacadista Central', 'Avenida Amazonas, n. 436, Barro Preto, Belo Horizonte, MG');
```

O comando dispara um erro, porque o telefone é obrigatório, não pode ser nulo.

3. Insira uma linha sem endereço:

```sql
INSERT INTO Fornecedor (nome, telefone) 
VALUES ('Atacadista Central', '(31) 9 9751-4523');
```

O comando dispara um erro, porque o endereço é obrigatório, não pode ser nulo.

---

# Inserçao de Várias Linhas

1. Insira várias linhas simulteneamente:

```sql
INSERT INTO Fornecedor (nome, telefone, endereco) VALUES 
('Distribuidora Norte-Sul', '(21) 2555-1234', 'Rua das Marrecas, 45 - Rio de Janeiro, RJ'),
('Tech Componentes Eletrônicos', '(31) 3444-9876', 'Av. Antônio Carlos, 6627 - Belo Horizonte, MG'),
('Embalagens Sustentáveis S.A.', '(41) 99111-2233', 'Rua das Flores, 123 - Curitiba, PR'),
('Atacadista Central', '(61) 3222-0000', 'SCS Quadra 4, Bloco A - Brasília, DF');
```

---

# BETWEEN ... AND ... 

1. Apresente os fornecedores com ID entre 2 e 4, inclusive:

```sql
SELECT * 
FROM Fornecedor
WHERE id BETWEEN 2 and 4;
```
---

# IN

1. Apresente os fornecedores com ID em 1, 3 e 5:

```sql
SELECT * 
FROM Fornecedor
WHERE id IN (1, 3, 5);
```

---

# NOT IN

1. Apresente os fornecedores cujos IDs não estejam em 1, 3 e 5:

```sql
SELECT * 
FROM Fornecedor
WHERE id NOT IN (1, 3, 5);
```

---

# Ordenando por Apelido de Coluna

1. No MySQL, para nos referirmos a apelidos (aliases) ou colunas que possuem espaços ou caracteres especiais, usamos a `crase`:

```sql
SELECT id as "código do fornecedor", nome, telefone, endereco 
FROM Fornecedor
ORDER BY `código do fornecedor` DESC;
```

ou

```sql
SELECT id as `código do fornecedor`, nome, telefone, endereco 
FROM Fornecedor
ORDER BY `código do fornecedor` DESC;
```

---

# Ordenando pela Posição da Coluna

1. Apresentar o fornecedores em ordem decrescente de nome:

```sql
SELECT *
FROM Fornecedor
ORDER BY 2 DESC;
```

---

# Ordenando por uma Coluna Ausente na Cláusula SELECT

1. Apresentar o nome e telefone dos fornecedores, mas apresentar em ordem decrescente de ID:

```sql
SELECT nome, telefone 
FROM Fornecedor
ORDER BY id DESC;
```

---

# Exercícios

## Exercício 1 (Inserção e Restrições):

Escreva o comando SQL para inserir um novo fornecedor chamado 'Suprimentos Globais', com o telefone '(11) 91111-2222' e endereço 'Av. das Nações, 500 - São Paulo, SP'. Em seguida, responda: O que acontecerá se você tentar executar esse mesmo comando uma segunda vez? Por quê?

---

## Exercício 2 (Filtro por Intervalo):

Escreva uma consulta que selecione todos os campos da tabela Fornecedor, mas retorne apenas os registros cujos códigos (id) estejam no intervalo de 3 a 5 (inclusive). Utilize o operador de intervalo adequado visto em aula.

---

## Exercício 3 (Seleção de Valores Específicos):

Escreva uma consulta que apresente todas as colunas dos fornecedores que possuem os IDs 1, 2 ou 4. Não utilize o operador OR nesta solução.

---

## Exercício 4 (Ordenação por Posição):

Escreva uma consulta que retorne todas as colunas da tabela Fornecedor. O resultado deve ser ordenado de forma decrescente utilizando o nome do fornecedor, porém você deve obrigatoriamente fazer essa ordenação referenciando a posição numérica da coluna na tabela.

---

## Exercício 5 (Ordenação Oculta):

Escreva uma consulta que exiba apenas as colunas nome e endereco de todos os fornecedores. Garanta que o resultado venha ordenado do último ID gerado para o primeiro (ordem decrescente de id), mesmo que a coluna id não apareça no resultado final do SELECT.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="05-fornecedor.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="07-produto.md">Próximo</a>
    </td>
  </tr>
</table>
