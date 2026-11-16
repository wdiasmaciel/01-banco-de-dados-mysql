# 19 - Remoção de Permissões com REVOKE

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="18-grant.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="20-deny.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Data Control Language: REVOKE

`REVOKE` é um comando de Data Control Language (DCL) usado para retirar privilégios concedidos anteriormente. Ele não exclui a conta nem apaga dados.

Criar o usuário:

```sql
CREATE USER 'ana'@'localhost' 
IDENTIFIED BY 'SenhaForte123!';
```

Conceder autorizações:

```sql
GRANT SELECT, INSERT, UPDATE
ON empresa.Fornecedor
TO 'ana'@'localhost';
```

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
ON empresa.Fornecedor
FROM 'ana'@'localhost';
```

```sql
SHOW GRANTS FOR 'ana'@'localhost';
```

**OBS**:
> O MySQL exige que os privilégios informados em `REVOKE` estejam concedidos à conta no escopo indicado. Se algum deles não estiver, a instrução pode falhar com o erro 1147 (`There is no such grant defined...`), em vez de remover apenas o privilégio existente. Confira o resultado de `SHOW GRANTS` e informe em `REVOKE` somente os privilégios que aparecem entre as concessões dessa conta.

---

## Removendo privilégios de banco

Criando um papel (perfil):

```sql
CREATE ROLE 'atendente'; -- Neste caso, o host é qualquer um (%).
```

Concedendo privilégios ao perfil criado:

```sql
GRANT SELECT, INSERT, UPDATE
ON empresa.*
TO 'atendente'; -- Neste caso, o host é qualquer um (%).
```

Verificando os privilégios do perfil criado:

```sql
SHOW GRANTS FOR 'atendente';
```

Revogando os privilégios concedidos:

```sql
REVOKE SELECT, INSERT, UPDATE
ON empresa.*
FROM 'atendente'; -- Neste caso, o host é qualquer um (%).
```

Para auditar um usuário, examine os resultados de `SHOW GRANTS`.

Verificando os privilégios do perfil:

```sql
SHOW GRANTS FOR 'atendente';
```

---

## Removendo privilégio de tabela

```sql
DROP ROLE IF EXISTS 'estoquista'@'localhost';
CREATE ROLE IF NOT EXISTS 'estoquista'@'localhost'; 
GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.Produto
TO 'estoquista'@'localhost'; 
```

```sql
REVOKE DELETE
ON empresa.Produto
FROM 'estoquista'@'localhost';
```

```sql
SHOW GRANTS FOR 'estoquista'@'localhost';
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
DROP ROLE IF EXISTS 'suporte'@'localhost';

CREATE ROLE IF NOT EXISTS 'suporte'@'localhost';

GRANT SELECT (telefone), INSERT (telefone), UPDATE (telefone)
ON empresa.Fornecedor
TO 'suporte'@'localhost';

GRANT DELETE
ON empresa.Fornecedor
TO 'suporte'@'localhost';
```

**OBS**:
> 1. A lista de colunas deve vir junto de cada privilégio que aceita escopo por coluna, como SELECT, INSERT e UPDATE, e não depois do nome da tabela.
> 2. DELETE não pode ser concedido por coluna: ele permite excluir linhas da tabela.

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

```sql
GRANT SELECT (telefone), INSERT (telefone), UPDATE (telefone)
ON empresa.Fornecedor
TO 'suporte'@'localhost';

REVOKE SELECT (telefone), INSERT (telefone), UPDATE (telefone)
ON empresa.Fornecedor
FROM 'suporte'@'localhost';
```

---

## Lock, Unlock, Expired

```sql
DROP ROLE 'atendente';
CREATE ROLE IF NOT EXISTS 'atendente'; -- Neste caso, o host é qualquer um (%).
GRANT SELECT, INSERT, UPDATE
ON empresa.*
TO 'atendente'; -- Neste caso, o host é qualquer um (%).
```

Os estados (status) de um usuário ou pefil são:

1. **Lock**: indica que a conta está bloqueada para login. 
2. **Unlock**: indica que a conta está desbloqueada para login. 
3. **Expired**: a senha está expirada. A senha precisa ser redefinida.

Se você quiser consultar o status da conta do usuário (atendente), execute a consulta:

```sql
SELECT user, host, account_locked, password_expired
FROM mysql.user
WHERE user = 'atendente';
```

No MySQL, **locked** indica que a conta está bloqueada para login. Isso não apaga o usuário nem remove seus privilégios; apenas impede que ele se conecte ao MySQL.

Para verificar o estado e os detalhes da conta:

```sql
SHOW CREATE USER 'atendente'@'localhost';
```

```sql
SHOW CREATE USER 'atendente';  -- Neste caso, o host é qualquer um (%).
```

Para desbloquear:

```sql
ALTER USER 'atendente'@'localhost' ACCOUNT UNLOCK;
```

```sql
ALTER USER 'atendente' ACCOUNT UNLOCK; -- Neste caso, o host é qualquer um (%).
```

```sql
SELECT user, host, account_locked, password_expired
FROM mysql.user
WHERE account_locked = 'N';
```

Para redefinir a senha:

```sql
ALTER USER 'atendente'@'localhost' IDENTIFIED BY 'uma_nova_senha';
```

```sql
ALTER USER 'atendente' IDENTIFIED BY 'uma_nova_senha'; -- Neste caso, o host é qualquer um (%).
```

```sql
SELECT user, host, account_locked, password_expired
FROM mysql.user
WHERE password_expired = 'Y';
```

```sql
SELECT user, host, account_locked, password_expired
FROM mysql.user
WHERE password_expired = 'N';
```

Para bloquear:

```sql
ALTER USER 'atendente'@'localhost' ACCOUNT LOCK;
```

```sql
ALTER USER 'atendente' ACCOUNT LOCK; -- Neste caso, o host é qualquer um (%).
```

```sql
SELECT user, host, account_locked, password_expired
FROM mysql.user
WHERE account_locked = 'Y';
```

---

## Removendo WITH GRANT OPTION

```sql
DROP ROLE IF EXISTS 'gerente'@'localhost';

CREATE ROLE IF NOT EXISTS 'gerente'@'localhost';

GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.Produto
TO 'gerente'@'localhost'
WITH GRANT OPTION;
```

```sql
SHOW GRANTS FOR 'gerente'@'localhost';
```

A opção de repassar privilégios pode ser retirada separadamente:

```sql
REVOKE GRANT OPTION
ON empresa.Produto
FROM 'gerente'@'localhost';
```

```sql
SHOW GRANTS FOR 'gerente'@'localhost';
```

**OBS**:
1. Essa forma remove a capacidade de conceder.
2. Ela não necessariamente remove os privilégios próprios da conta.

---

## Removendo privilégios de rotina

```sql
DELIMITER //

CREATE PROCEDURE empresa.fechar_venda()
BEGIN
    SELECT 'Venda fechada!' AS mensagem;
END//

DELIMITER ;
```

```sql
CALL empresa.fechar_venda();
```

```sql
DROP ROLE IF EXISTS 'caixa'@'localhost';

CREATE ROLE IF NOT EXISTS 'caixa'@'localhost';

GRANT EXECUTE
ON PROCEDURE empresa.fechar_venda
TO 'caixa'@'localhost';
```

```sql
SHOW GRANTS FOR 'caixa'@'localhost';
```

```sql
REVOKE EXECUTE
ON PROCEDURE empresa.fechar_venda
FROM 'caixa'@'localhost';
```

```sql
SHOW GRANTS FOR 'caixa'@'localhost';
```

---

```sql
DELIMITER //

CREATE FUNCTION empresa.calcular_frete()
RETURNS VARCHAR(16)
DETERMINISTIC
BEGIN
    RETURN 'Frete calculado!';
END//

DELIMITER ;
```

```sql
SELECT empresa.calcular_frete() AS mensagem;
```

```sql
DROP ROLE IF EXISTS 'caixa'@'localhost';

CREATE ROLE IF NOT EXISTS 'caixa'@'localhost';

GRANT EXECUTE
ON FUNCTION empresa.calcular_frete
TO 'caixa'@'localhost';
```

```sql
SHOW GRANTS FOR 'caixa'@'localhost';
```

```sql
REVOKE EXECUTE
ON FUNCTION empresa.calcular_frete
FROM 'atendente'@'localhost';
```

```sql
SHOW GRANTS FOR 'caixa'@'localhost';
```

---

## Revogando papéis

Para retirar um papel de uma conta:

```sql
REVOKE 'perfil_estoque'
FROM 'lucas'@'localhost';
```

```sql
SHOW GRANTS FOR 'lucas'@'localhost';
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
      <a href="18-grant.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="20-deny.md">Próximo</a>
    </td>
  </tr>
</table>
