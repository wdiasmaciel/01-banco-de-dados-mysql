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
# Exercícios

## Exercício 1

Crie um banco de dados para uma empresa de aviaçao. No banco, crie a tabela "Voo". Crie as consultas abaixo. Insira dados na tabela "Voo" que atendam às consultas.
Apresente todos os voos com: 
1. Partida de Belo Horizonte.
2. Destino Fortaleza.
3. Viagem de volta agendada.
4. Valor abaixo de R$ 950,00.
5. Valor acima de R$ 1.200,00.
6. Valor entre R$ 500,00 e R$ 800,00.
7. Partida agendada para 2027.
8. Volta agendada para 2028.
9. Partida de Cuiabá e destino Curitiba.
10. Partida de Minas Gerais.
11. Destino Rio Grande do Sul.
12. Partida de Tocantins ou do Pará.
13. Destino Salvador ou Aracajú.
14. Viagem de volta não agendada.
15. Substring "es" na cidade/estado de partida. 
16. Substring "ta" na cidade/estado de destino.
17. Substring "or" na cidade/estado de partida ou de destino.

---

## Exercício 2

Crie um banco de dados para uma empresa de hotelaria. No banco, crie a tabela "Hotel". Crie as consultas abaixo. Insira dados na tabela "Hotel" que atendam às consultas.
Apresente todos os hotéis com:
1. Cidade de Minas Gerais.
2. Valor da diária abaixo de R$ 300,00.
3. Cidade São Luís.
4. Categoria 3 estrelas.
5. Substring "resort" no nome do hotel.
6. Cidade Recife ou Salvador.
7. Tipo de acomodação "Quarto duplo".
8. Valor da diária acima de R$ 100,00.
9. Substring "elite" no nome do hotel ou da acomodação.
10. Hotel com check-in agendado para 2027.
11. Cidade Curitiba e acomodação tipo "Suite".
12. Estado Rio Grande do Sul.
13. Estado São Paulo ou Paraná.
14. Hospedagem sem café incluso.
15. Substring "standard" no tipo da acomodação.
16. Valor da diária entre R$ 250,00 e R$ 450,00.
17. Hotel com check-out agendado para 2028.
18. Garagem inclusa na hospedagem.
19. Garagem inclusa, mas sem café incluso.
20. Café incluso, mas sem garagem inclusa.

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
