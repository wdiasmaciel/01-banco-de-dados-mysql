# 05 - Projeto Empresa

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="03-banco-de-dados.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="05-produto.md">Próximo</a>
    </td>
  </tr>
</table>

---

# CREATE TABLE

1. Crie a tabela Fornecedor.

2. No console interativo do `MySQL`, informe o comando abaixo:

```sql
CREATE TABLE Fornecedor (
    id INT,
    nome VARCHAR(256),
    telefone VARCHAR(20),
    endereco VARCHAR(256)
);
```

2. Alternativamnte, informe o comando abaixo no console interativo do `MySQL`:

```sql
DROP TABLE IF EXISTS Fornecedor;

CREATE TABLE Fornecedor (
    id INT,
    nome VARCHAR(256),
    telefone VARCHAR(20),
    endereco VARCHAR(256)
);
```

---

# DESCRIBE

1. Observe a estrutura da tabela `Fornecedor` usando o comando `DESCRIBE` ou o seu atalho `DESC`.

```sql
DESCRIBE Fornecedor;
```

```sql
DESC Fornecedor;
```

---

# INSERT

1. Insira uma linha (registro) na tabela `Fornecedor`.

```sql
INSERT INTO Fornecedor (id, nome, telefone, endereco) VALUES 
(1, 'Logística Brasil Ltda', '(11) 98765-4321', 'Av. Paulista, 1000 - São Paulo, SP');
```

---

# SELECT

1. Apresente os dados inseridos na tabela `Fornecedor`:

```sql
SELECT id, nome, telefone, endereco FROM Fornecedor;
```

2. Apresente todas as colunas da linha inserida da tabela `Fornecedor`:

```sql
SELECT * FROM Fornecedor;
```

3. Apresente a coluna `nome` da linha inserida na tabela `Fornecedor`:

```sql
SELECT nome FROM Fornecedor;
```

4. Apresente a coluna `nome` e a coluna `telefone` da linha inserida na tabela `Fornecedor`:

```sql
SELECT nome, telefone FROM Fornecedor;
```

5. Apresente a coluna `nome` e a coluna `endereco` da linha inserida na tabela `Fornecedor`:

```sql
SELECT nome, endereco 
FROM Fornecedor;
```

6. Apresente a coluna `nome` como "Nome do Fornecedor" e a coluna `endereco` como "Endereço do Fornecedor":

```sql
SELECT nome AS "Nome do Fornecedor", endereco AS "Endereço do Fornecedor"
FROM Fornecedor;
```

7. Apresente a coluna `id` como 'Código do Fornecedor' e a coluna `nome` como 'Nome do Fornecedor':

```sql
SELECT id AS 'Código do Fornecedor', nome AS 'Nome do Fornecedor'
FROM Fornecedor;
```

---

# ID Repetido

1. Insira uma linha com `id` repetido:

```sql
INSERT INTO Fornecedor (id, nome, telefone, endereco) 
VALUES (1, 'Distribuidora Norte-Sul', '(21) 2555-1234', 'Rua das Marrecas, 45 - Rio de Janeiro, RJ');
```

2. Verifique a inserção indevida:

```sql
SELECT * FROM Fornecedor;
```

---

# Telefone Repetido

1. Insira uma linha com `telefone` repetido:

```sql
INSERT INTO Fornecedor (id, nome, telefone, endereco) 
VALUES (2, 'Tech Componentes Eletrônicos', '(21) 2555-1234', 'Av. Antônio Carlos, 6627 - Belo Horizonte, MG');
```

2. Verifique a inserção indevida:

```sql
SELECT * FROM Fornecedor;
```

# Dado Faltante

1. Insira uma linha sem ID:

```sql
INSERT INTO Fornecedor (nome, telefone, endereco) 
VALUES ('Embalagens Sustentáveis S.A.', '(41) 99111-2233', 'Rua das Flores, 123 - Curitiba, PR');
```

2. Insira uma linha sem nome:

```sql
INSERT INTO Fornecedor (id, telefone, endereco) 
VALUES (3, '(61) 3222-0000', 'SCS Quadra 4, Bloco A - Brasília, DF');
```

3. Insira uma linha sem telefone e sem endereço:

```sql
INSERT INTO Fornecedor (id, nome) 
VALUES (3, 'Atacadista Central');
```

4. Verifique as inserções indevidas:

```sql
SELECT * FROM Fornecedor;
```

---

# Inserçao de Várias Linhas

1. Insira várias linhas simulteneamente:

```sql
INSERT INTO Fornecedor (id, nome, telefone, endereco) VALUES 
(1, 'Logística Brasil Ltda', '(11) 98765-4321', 'Av. Paulista, 1000 - São Paulo, SP'),
(2, 'Distribuidora Norte-Sul', '(21) 2555-1234', 'Rua das Marrecas, 45 - Rio de Janeiro, RJ'),
(3, 'Tech Componentes Eletrônicos', '(31) 3444-9876', 'Av. Antônio Carlos, 6627 - Belo Horizonte, MG'),
(4, 'Embalagens Sustentáveis S.A.', '(41) 99111-2233', 'Rua das Flores, 123 - Curitiba, PR'),
(5, 'Atacadista Central', '(61) 3222-0000', 'SCS Quadra 4, Bloco A - Brasília, DF');
```

2. Verifique as inserções:

```sql
SELECT * FROM Fornecedor;
```

---

# WHERE

1. Apresente o fornecedor com id igual a 1:

```sql
SELECT * 
FROM Fornecedor
WHERE id = 1;
```

2. Apresente os fornecedores com id's maiores que 2:

```sql
SELECT * 
FROM Fornecedor
WHERE id > 2;
```

3. Apresente os fornecedores com id's menores ou iguais a 3:

```sql
SELECT * 
FROM Fornecedor
WHERE id <= 3;
```

4. Apresente os fornecedores com id's diferentes de 3:

```sql
SELECT * 
FROM Fornecedor
WHERE id <> 3;
```

5. Apresente os fornecedores com id's diferentes de 1:

```sql
SELECT * 
FROM Fornecedor
WHERE id != 1;
```

6. Apresente o fornecedor com nome igual a "Embalagens Sustentáveis S.A.":

```sql
SELECT * 
FROM Fornecedor
WHERE nome = 'Embalagens Sustentáveis S.A.';
```

7. Apresente o fornecedor com nome igual a "Atacadista Central":

```sql
SELECT * 
FROM Fornecedor
WHERE nome LIKE 'Atacadista Central';
```

8. Apresente os fornecedores que são sociedades anônimas:

```sql
SELECT * 
FROM Fornecedor
WHERE nome LIKE '%S.A.';
```

9. Apresente os fornecedores cujo nome comece com a letra 'a':

```sql
SELECT * 
FROM Fornecedor
WHERE nome LIKE 'a%';
```

10. Apresente os fornecedores cujo nome termine com a letra 'a':

```sql
SELECT * 
FROM Fornecedor
WHERE nome LIKE '%a';
```

11. Apresente os fornecedores que tenham "Componentes" no nome:

```sql
SELECT * 
FROM Fornecedor
WHERE nome LIKE '%Componentes%';
```

12. Apresente os fornecedores cujo ID seja NULL:

```sql
SELECT * 
FROM Fornecedor
WHERE id IS NULL;
```

13. Apresente os fornecedores cujo ID seja diferente de NULL:

```sql
SELECT * 
FROM Fornecedor
WHERE id IS NOT NULL;
```

14. Apresente os fornecedores sem teleone e sem endereço:

```sql
SELECT * 
FROM Fornecedor
WHERE telefone IS NULL AND
      endereco IS NULL;
```

15. Apresente os fornecedores sem id ou sem teleone:

```sql
SELECT * 
FROM Fornecedor
WHERE id IS NULL OR
      telefone IS NULL;
```

---


<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="03-banco-de-dados.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="05-produto.md">Próximo</a>
    </td>
  </tr>
</table>
