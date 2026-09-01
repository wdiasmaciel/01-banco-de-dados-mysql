# 03 - Criação do Banco de Dados

<table>
  <tr>
    <td align="left" style="border: none;">
      <a href="02-projeto.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="04-fornecedor.md">Próximo</a>
    </td>
  </tr>
</table>

---

1. Crie o banco de dados:

```sql
-- Cria o banco de dados:
CREATE DATABASE empresa;
```

2. Crie o banco de dados apenas se ele ainda não existir (evita erros):

```sql
-- Cria o banco de dados apenas se ele ainda não existir (evita erros)
CREATE DATABASE IF NOT EXISTS empresa;
```
3. Selecione (use) o banco de dados para os próximos comandos:

```sql
-- Seleciona o banco de dados para os próximos comandos:
USE empresa;
```

---

<table>
  <tr>
    <td align="left" style="border: none;">
      <a href="02-projeto.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="04-fornecedor.md">Próximo</a>
    </td>
  </tr>
</table>
