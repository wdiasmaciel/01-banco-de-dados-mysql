# Projeto Empresa

---

# Tabela Fornecedor

---

## Crie a tabela Fornecedor

```sql
DROP TABLE IF EXISTS Fornecedor;

CREATE TABLE Fornecedor (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(256) NOT NULL,
    telefone VARCHAR(20),
    endereco VARCHAR(256)
);
```
