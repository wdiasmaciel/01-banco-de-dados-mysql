# 17 - Remoção de Permissões com REVOKE

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="16-grant.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="18-deny.md">Próximo</a>
    </td>
  </tr>
</table>

---

# REVOKE e Data Control Language

`REVOKE` é um comando de DCL usado para retirar privilégios concedidos anteriormente. Ele não exclui a conta nem apaga dados.

Antes de remover privilégios, confira os privilégios atuais:

```sql
SHOW GRANTS FOR 'ana'@'localhost';
```

---

## Sintaxe geral

```sql
REVOKE privilégio
ON objeto
FROM 'usuario'@'origem';
```

Para remover privilégios concedidos em uma tabela:

```sql
REVOKE INSERT, UPDATE
ON empresa.Produto
FROM 'ana'@'localhost';
```

Caso existam dois privilégios separados e o usuário já possua apenas um deles, o MySQL remove somente aquele que foi solicitado.

---

## Removendo privilégios de banco

```sql
REVOKE SELECT, INSERT
ON empresa.*
FROM 'atendente'@'localhost';
```

Essa operação não remove privilégios concedidos diretamente em nível global. Para auditar um usuário, examine os resultados de `SHOW GRANTS`.

---

## Removendo privilégio de tabela

```sql
REVOKE DELETE
ON empresa.Produto
FROM 'estoquista'@'localhost';
```

```sql
REVOKE ALL PRIVILEGES
ON empresa.Produto
FROM 'estoquista'@'localhost';
```

`ALL PRIVILEGES` representa os privilégios aplicáveis ao objeto informado. O usuário pode continuar tendo outros privilégios em outros objetos.

---

## Removendo privilégio de coluna

```sql
REVOKE UPDATE (telefone)
ON empresa.Fornecedor
FROM 'suporte'@'localhost';
```

Para remover privilégios de consulta em colunas específicas:

```sql
REVOKE SELECT (telefone)
ON empresa.Fornecedor
FROM 'suporte'@'localhost';
```

---

## Removendo WITH GRANT OPTION

A opção de repassar privilégios pode ser retirada separadamente:

```sql
REVOKE GRANT OPTION
ON empresa.Produto
FROM 'gerente'@'localhost';
```

Essa forma remove a capacidade de conceder; ela não necessariamente remove os privilégios próprios da conta.

---

## Removendo privilégios de rotina

```sql
REVOKE EXECUTE
ON PROCEDURE empresa.fechar_venda
FROM 'caixa'@'localhost';
```

```sql
REVOKE EXECUTE
ON FUNCTION empresa.calcular_frete
FROM 'atendente'@'localhost';
```

---

## Revogando papéis

Para retirar um papel de uma conta:

```sql
REVOKE 'perfil_estoque'
FROM 'lucas'@'localhost';
```

Para remover os privilégios associados ao próprio papel:

```sql
REVOKE SELECT, UPDATE
ON empresa.Produto
FROM 'perfil_estoque';
```

Para excluir o papel após avaliar dependências:

```sql
DROP ROLE 'perfil_estoque';
```

---

## Revogando todos os privilégios

Quando uma conta ainda existe, mas não deve possuir privilégio algum:

```sql
REVOKE ALL PRIVILEGES, GRANT OPTION
FROM 'ex_usuario'@'localhost';
```

Os privilégios `ALL PRIVILEGES` e `GRANT OPTION` são retirados sem indicar um objeto específico. Essa forma é útil antes de desativar ou excluir uma conta. Para remover completamente uma conta:

```sql
DROP USER 'ex_usuario'@'localhost';
```

---

## Atualização efetiva dos privilégios

O MySQL normalmente aplica alterações de privilégios imediatamente. Em situações administrativas específicas, o servidor pode ser instruído a recarregá-los:

```sql
FLUSH PRIVILEGES;
```

Não é necessário executar `FLUSH PRIVILEGES` após os comandos comuns `GRANT` e `REVOKE`. Seu uso é típico ao manipular diretamente tabelas de concessão.

---

# Exercícios

1. Remova o privilégio de inserção da conta de um caixa da padaria.
2. Remova simultaneamente os privilégios de inserir e excluir vendas do usuário da padaria.
3. Retire o privilégio de alteração da quantidade de produtos de um estoquista.
4. Remova o acesso direto à tabela de fornecedores de um analista que deve usar uma visão.
5. Remova o privilégio de consulta sobre uma visão de medicamentos de uma conta temporária.
6. Retire o privilégio de executar a procedure `gerar_relatorio_diario`.
7. Remova o privilégio de atualização da coluna `telefone` de um atendente.
8. Remova a opção de repassar privilégios de uma gerente de viagens.
9. Confira os privilégios de uma conta antes de retirar sua permissão de exclusão.
10. Retire todos os privilégios aplicáveis à tabela de reservas de uma agência.
11. Revogue o papel `perfil_caixa` de um colaborador que mudou de setor.
12. Remova os privilégios de consulta e inserção associados ao papel `perfil_atendente`.
13. Exclua o papel do setor de estoque depois de retirar todas as suas associações.
14. Deixe uma conta de auditoria sem privilégios de banco, mas sem excluir a conta.
15. Revogue apenas privilégios concedidos em nível de banco, preservando um privilégio em nível global.
16. Retire o acesso de atualização de uma conta enquanto mantém o acesso de consulta.
17. Remova o acesso à função `calcular_comissao` de uma conta que não deve fechar vendas.
18. Explique por que `REVOKE SELECT` pode não ser suficiente quando o usuário recebeu privilégios por um papel.
19. Planeje a remoção segura dos privilégios de uma conta de funcionário desligado.
20. Depois de revogar e atualizar os privilégios, escreva um comando que confirme o resultado.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="16-grant.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="18-deny.md">Próximo</a>
    </td>
  </tr>
</table>
