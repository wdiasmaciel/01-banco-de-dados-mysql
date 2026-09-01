# Projeto Empresa

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
