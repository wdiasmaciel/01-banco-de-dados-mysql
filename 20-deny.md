# 20 - Controle de Negação de Permissões: DENY

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="19-revoke.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="21-drop.md">Próximo</a>
    </td>
  </tr>
</table>

---

# DENY não existe no MySQL

No MySQL, o comando `DENY` não é um comando DCL disponível.

Essa instrução existe, por exemplo, em Microsoft SQL Server, onde pode negar explicitamente um privilégio a um usuário. No MySQL, o controle é baseado no princípio de concessão explícita: o usuário recebe privilégios por `GRANT`, e os privilégios não concedidos não estão disponíveis.

Portanto, o comando abaixo não pode ser executado no MySQL:

```sql
DENY SELECT ON empresa.Produto TO 'usuario'@'localhost';
```

Para impedir acesso, o MySQL não cria uma regra negativa correspondente chamada `DENY`. Em vez disso, revisam-se ou removem-se as concessões com `REVOKE`, e atribuem-se privilégios menores por meio de contas, papéis, visões e privilégios por coluna.

---

## Modelando uma restrição

Suponha que uma conta possa consultar todos os fornecedores, mas não deva modificar seus dados:

```sql
GRANT SELECT
ON empresa.Fornecedor
TO 'leitura_fornecedor'@'localhost';
```

Como não foi concedido `UPDATE`, esse usuário não deve conseguir alterar os registros da tabela.

Para impedir explicitamente uma permissão recebida anteriormente por um papel ou por outra conta:

```sql
REVOKE UPDATE
ON empresa.Fornecedor
FROM 'leitura_fornecedor'@'localhost';
```

```sql
SHOW GRANTS FOR 'leitura_fornecedor'@'localhost';
```

---

## Verificando as concessões

```sql
SHOW GRANTS FOR 'leitura_fornecedor'@'localhost';
```

A conta também pode receber privilégios por papéis ativos. Por isso, verifique o papel e seus privilégios:

```sql
SHOW GRANTS FOR 'leitura_fornecedor'@'localhost' USING 'perfil_operador';
```

Se o usuário não tiver o privilégio esperado, a operação será negada naturalmente pelo servidor.

---

## Removendo o privilégio de um papel

Considere que um papel concede alteração em produtos:

```sql
CREATE ROLE 'perfil_estoque';

GRANT SELECT, UPDATE
ON empresa.Produto
TO 'perfil_estoque';
```

```sql
SHOW GRANTS FOR 'perfil_estoque';
```

Para impedir o papel de alterar produtos sem apagar todo o papel:

```sql
REVOKE UPDATE
ON empresa.Produto
FROM 'perfil_estoque';
```

Depois, atribua ao papel somente o privilégio necessário:

```sql
GRANT SELECT
ON empresa.Produto
TO 'perfil_estoque';
```

---

## Restringindo por objetos

Quando o usuário não deve consultar informações diretamente em uma tabela, retire o acesso a essa tabela:

```sql
REVOKE ALL PRIVILEGES
ON empresa.Cliente
FROM 'atendente'@'localhost';
```

Depois, conceda acesso somente à visão:

```sql
CREATE VIEW vw_clientes_atendimento AS
SELECT id, nome, telefone
FROM   Cliente;

GRANT SELECT
ON empresa.vw_clientes_atendimento
TO 'atendente'@'localhost';
```

Essa visão pode evitar que dados como CPF ou observações internas sejam acessados.

---

## Restringindo por coluna

O acesso pode ser limitado à coluna necessária:

```sql
GRANT SELECT (id, nome, telefone)
ON empresa.Cliente
TO 'atendente'@'localhost';
```

```sql
GRANT UPDATE (telefone)
ON empresa.Cliente
TO 'atendente'@'localhost';
```

O usuário não recebe concessão para consultar ou alterar outras colunas dessa tabela.

---

## Menor privilégio necessário

Em vez de conceder:

```sql
GRANT ALL PRIVILEGES ON empresa.* TO 'operador'@'localhost';
```

Prefira conceder apenas o necessário:

```sql
GRANT SELECT, INSERT
ON empresa.Venda
TO 'operador'@'localhost';
```

A ausência da autorização é o mecanismo de bloqueio do MySQL. Não existe uma entrada `DENY` que sobreponha todas as concessões recebidas.

---

## Boas práticas

1. Modele perfis com privilégios mínimos.
2. Não compartilhe contas administrativas.
3. Verifique privilégios individuais e papéis.
4. Use visões e privilégios por coluna para informações sensíveis.
5. Não confunda `DENY` com `REVOKE` em MySQL.

---

# Exercícios

1. Explique por que o comando `DENY SELECT` não deve ser usado em um roteiro MySQL.
2. Conceda apenas consulta na tabela de produtos para um funcionário da padaria.
3. Retire um privilégio de exclusão que tenha sido concedido anteriormente ao usuário da padaria.
4. Verifique se uma conta possui privilégio de alteração antes de concluir que ela está proibida.
5. Crie uma conta que não possa alterar vendas, mas possa consultá-las.
6. Remova o privilégio de atualização de uma pessoa que usa o papel `perfil_estoque`.
7. Crie uma visão para mostrar nome e quantidade de medicamentos, sem mostrar preços de aquisição.
8. Conceda ao atendente de farmácia acesso à visão criada no exercício anterior.
9. Remova da conta de atendimento o acesso direto à tabela de medicamentos.
10. Conceda privilégio de consulta apenas para nome, destino e data em uma tabela de viagens.
11. Permita que uma agência atualize somente o telefone de um cliente.
12. Desative um papel na sessão atual sem excluir o papel do banco.
13. Explique por que uma conta pode permanecer proibida de excluir dados mesmo antes de ser executado qualquer `DENY`.
14. Conceda a um caixa o mínimo necessário para registrar uma venda.
15. Revise os privilégios globais, de banco e de tabela de uma conta administrativa temporária.
16. Crie um papel com consulta a produtos e remova dele o privilégio de atualização.
17. Planeje a restrição de acesso aos dados pessoais dos clientes de um supermercado.
18. Explique como privar o acesso direto a uma tabela sensível usando uma visão.
19. Verifique quais papéis ativos podem conceder privilégios indevidos a um usuário.
20. Compare em uma tabela os comportamentos de `DENY`, `REVOKE` e ausência de privilégio no contexto do MySQL.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="19-revoke.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="21-drop.md">Próximo</a>
    </td>
  </tr>
</table>
