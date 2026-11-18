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

# Data Control Language: DENY

## DENY não existe no MySQL

No MySQL, o comando `DENY` não é um Data Control Language (DCL) disponível.

Essa instrução existe, por exemplo, no Microsoft SQL Server, em que se pode negar explicitamente um privilégio a um usuário. 

No MySQL, o controle é baseado no princípio de concessão explícita: o usuário recebe privilégios por `GRANT`, os privilégios não concedidos não ficam disponíveis e os privilégios concedidos são revogados por `REVOKE`.


Portanto, o comando abaixo não pode ser executado no MySQL:

```sql
DROP ROLE IF EXISTS 'usuario'@'localhost';
CREATE ROLE IF NOT EXISTS 'usuario'@'localhost'; 
GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.Produto
TO 'usuario'@'localhost'; 
```

```sql
SHOW GRANTS FOR 'usuario'@'localhost';
```

```sql
DENY DELETE -- Gera uma mensagem de erro.
ON empresa.Produto 
TO 'usuario'@'localhost';
```

Para impedir acesso, o MySQL não cria uma regra negativa correspondente chamada `DENY`. 

* Em vez disso, revisam-se ou removem-se as concessões com `REVOKE`, 
* E atribuem-se privilégios menores por meio de contas, papéis, visões e privilégios por coluna.

---

## Modelando uma restrição

Suponha que uma conta possa consultar todos os fornecedores, mas não deva modificar seus dados:

```sql
GRANT SELECT
ON empresa.Fornecedor
TO 'usuario'@'localhost';
```

```sql
SHOW GRANTS FOR 'usuario'@'localhost';
```

Como não foi concedido `UPDATE`, esse usuário não conseguirá alterar os registros da tabela. Também não conseguirá inserir e nem excluir.

Para revogar explicitamente permissões concedida anteriormente a um papel ou conta:

```sql
REVOKE INSERT, UPDATE, DELETE
ON empresa.Produto
FROM 'usuario'@'localhost';
```

```sql
SHOW GRANTS FOR 'usuario'@'localhost';
```

---

## Removendo privilégio de um usuário

Considere que um usuário receba privilégio de alteração em produtos:

```sql
DROP USER IF EXISTS 'joana';
CREATE USER IF NOT EXISTS 'joana'
IDENTIFIED BY 'senha-da-joana';

GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.Produto
TO 'joana';
```

```sql
SHOW GRANTS FOR 'joana';
```

Para impedir que o usuário altere produtos sem que seja necessário excluí-lo:

```sql
REVOKE UPDATE
ON empresa.Produto
FROM 'joana';
```

```sql
SHOW GRANTS FOR 'joana';
```

Para revogar demais privilégios:

```sql
REVOKE ALL PRIVILEGES, GRANT OPTION
FROM 'joana';
```

```sql
SHOW GRANTS FOR 'joana';
```

Depois, atribua ao usuário somente o privilégio necessário:

```sql
GRANT SELECT
ON empresa.Produto
TO 'joana';
```

```sql
SHOW GRANTS FOR 'joana';
```

---

É possível usar o comando `GRANT ALL PRIVILEGES` no MySQL, tanto para usuários individuais quanto para papéis (roles).

1. Em um banco de dados específico (mais comum):
> Concede controle total apenas sobre as tabelas, views e rotinas daquele banco, sem dar acesso administrativo ao servidor inteiro.

```sql
DROP USER IF EXISTS 'gerson'@'localhost';
CREATE USER IF NOT EXISTS 'gerson'@'localhost'
IDENTIFIED BY 'senha-do-gerson';
```

```sql
GRANT ALL PRIVILEGES 
ON empresa.* 
TO 'gerson'@'localhost';
```

```sql
SHOW GRANTS FOR 'gerson'@'localhost';
```

```sql
DROP ROLE IF EXISTS 'admin'@'localhost';
CREATE ROLE IF NOT EXISTS 'admin'@'localhost';
```

```sql
GRANT ALL PRIVILEGES 
ON empresa.* 
TO 'admin'@'localhost';
```

```sql
SHOW GRANTS FOR 'gerson'@'localhost';
```

2. Em nível global (superusuário):
> Concede controle irrestrito sobre todos os bancos de dados e configurações do servidor.

```sql
DROP USER IF EXISTS 'fatima'@'localhost';
CREATE USER IF NOT EXISTS 'fatima'@'localhost'
IDENTIFIED BY 'senha-da-fatima';
```

```sql
SHOW GRANTS FOR 'fatima'@'localhost';
```

```sql
GRANT ALL PRIVILEGES 
ON *.* 
TO 'fatima'@'localhost' 
WITH GRANT OPTION;
```

```sql
SHOW GRANTS FOR 'fatima'@'localhost';
```

```sql
DROP ROLE IF EXISTS 'superusuario'@'localhost';
CREATE ROLE IF NOT EXISTS 'superusuario'@'localhost';
```

```sql
SHOW GRANTS FOR 'superusuario'@'localhost';
```

```sql
GRANT ALL PRIVILEGES 
ON *.* 
TO 'superusuario'@'localhost' 
WITH GRANT OPTION;
```

```sql
SHOW GRANTS FOR 'superusuario'@'localhost';
```

---

## Restringindo por objetos

Quando o usuário não deve consultar informações diretamente em uma tabela, retire o acesso a essa tabela:

```sql
DROP ROLE IF EXISTS 'gestor'@'localhost';
CREATE ROLE IF NOT EXISTS 'gestor'@'localhost';
```

```sql
SHOW GRANTS FOR 'gestor'@'localhost';
```

```sql
GRANT ALL PRIVILEGES 
ON empresa.Fornecedor
TO 'gestor'@'localhost' 
WITH GRANT OPTION;
```

```sql
SHOW GRANTS FOR 'gestor'@'localhost';
```

```sql
REVOKE ALL PRIVILEGES
ON empresa.Fornecedor
FROM 'gestor'@'localhost';

REVOKE GRANT OPTION
ON empresa.Fornecedor
FROM 'gestor'@'localhost';
```

```sql
SHOW GRANTS FOR 'gestor'@'localhost';
```

Depois, conceda acesso somente à visão:

```sql
DROP VIEW IF EXISTS empresa.vw_fornecedor_gestao;
CREATE VIEW IF NOT EXISTS empresa.vw_fornecedor_gestao AS
SELECT      nome, telefone, endereco
FROM        empresa.Fornecedor;
```

```sql
GRANT SELECT
ON empresa.vw_fornecedor_gestao
TO 'gestor'@'localhost';
```

```sql
SHOW GRANTS FOR 'gestor'@'localhost';
```

Essa visão pode evitar que dados como CPF ou observações internas sejam acessados.

---

## Restringindo por coluna

O acesso pode ser limitado à coluna necessária:

```sql
DROP ROLE IF EXISTS 'atendente'@'localhost';
CREATE ROLE IF NOT EXISTS 'atendente'@'localhost';
```

```sql
SHOW GRANTS FOR 'atendente'@'localhost';
```

```sql
GRANT SELECT (nome, telefone, endereco)
ON empresa.Fornecedor
TO 'atendente'@'localhost';
```

```sql
SHOW GRANTS FOR 'atendente'@'localhost';
```

```sql
GRANT UPDATE (telefone)
ON empresa.Fornecedor
TO 'atendente'@'localhost';
```

```sql
SHOW GRANTS FOR 'atendente'@'localhost';
```

O usuário não recebe concessão para consultar ou alterar outras colunas dessa tabela.

---

## Menor privilégio necessário

Em vez de conceder:

```sql
DROP ROLE IF EXISTS 'operador'@'localhost';
CREATE ROLE IF NOT EXISTS 'operador'@'localhost';
```

```sql
SHOW GRANTS FOR 'operador'@'localhost';
```

```sql
GRANT ALL PRIVILEGES 
ON empresa.* 
TO 'operador'@'localhost';
```

```sql
SHOW GRANTS FOR 'operador'@'localhost';
```

Prefira conceder apenas o necessário:

```sql
DROP ROLE IF EXISTS 'operador'@'localhost';
CREATE ROLE IF NOT EXISTS 'operador'@'localhost';
```

```sql
SHOW GRANTS FOR 'operador'@'localhost';
```

```sql
GRANT SELECT, INSERT
ON empresa.Produto
TO 'operador'@'localhost';
```

```sql
SHOW GRANTS FOR 'operador'@'localhost';
```

**OBS**:
> * A ausência da autorização é o mecanismo de bloqueio do MySQL. 
> * Não existe uma entrada `DENY` que sobreponha todas as concessões recebidas.

---

## Grant a múltiplos usuários e múltiplos papéis

No MySQL, não é possível conceder privilégios (como `SELECT`, `UPDATE`, `INSERT`, `DELETE`) a múltiplos usuários ou papéis (roles) em uma única instrução `GRANT`.
* Exemplo:

```sql
GRANT SELECT 
ON empresa.* 
TO 'user1'@'localhost', 'user2'@'localhost'; -- Isso gerará um erro de sintaxe.
```

---

A melhor prática no MySQL é criar um role (papel), conceder os privilégios a essa role, e depois atribuir o role a todos os usuários desejados:

```sql
DROP USER IF EXISTS 'usuario1';
CREATE USER IF NOT EXISTS 'usuario1'
IDENTIFIED BY 'senha-do-usuario1';

DROP USER IF EXISTS 'usuario2'@'%';
CREATE USER IF NOT EXISTS 'usuario2'@'%'
IDENTIFIED BY 'senha-do-usuario2';

DROP USER IF EXISTS 'usuario3'@'localhost';
CREATE USER IF NOT EXISTS 'usuario3'@'localhost'
IDENTIFIED BY 'senha-do-usuario3';
```

```sql
SHOW GRANTS FOR 'usuario1';
SHOW GRANTS FOR 'usuario2'@'%';
SHOW GRANTS FOR 'usuario3'@'localhost';
```

```sql
-- 1. Criar o papel (role):
DROP ROLE IF EXISTS 'diretor';
CREATE ROLE IF NOT EXISTS 'diretor';
```

```sql
SHOW GRANTS FOR 'diretor';
```

```sql
-- 2. Conceder os privilégios ao papel:
GRANT SELECT, INSERT 
ON empresa.* 
TO 'diretor';
```

```sql
SHOW GRANTS FOR 'diretor';
```

```sql
-- 3. Atribuir o papel a múltiplos usuários:
GRANT 'diretor' 
TO    'usuario1', 'usuario2'@'%', 'usuario3'@'localhost';
```

```sql
SHOW GRANTS FOR 'diretor';
```

```sql
SHOW GRANTS FOR 'usuario1';
SHOW GRANTS FOR 'usuario2'@'%';
SHOW GRANTS FOR 'usuario3'@'localhost';
```

---

```sql
-- 1. Criar outro papel (role):
DROP ROLE IF EXISTS 'atendente';
DROP ROLE IF EXISTS 'comercial'@'localhost';
CREATE ROLE IF NOT EXISTS 'atendente';
CREATE ROLE IF NOT EXISTS 'comercial'@'localhost';
```

```sql
SHOW GRANTS FOR 'atendente';
SHOW GRANTS FOR 'comercial'@'localhost';
```

```sql
-- 2. Conceder os privilégios ao papel:
GRANT UPDATE, DELETE 
ON empresa.* 
TO 'atendente';

GRANT SELECT 
ON empresa.* 
TO 'comercial'@'localhost';
```

```sql
SHOW GRANTS FOR 'diretor';
SHOW GRANTS FOR 'atendente';
SHOW GRANTS FOR 'comercial'@'localhost';
```

```sql
-- 3. Atribuir os papéis a múltiplos usuários:
GRANT 'diretor', 'atendente', 'comercial'@'localhost'
TO    'usuario1', 'usuario2'@'%', 'usuario3'@'localhost';
```

```sql
SHOW GRANTS FOR 'diretor';
SHOW GRANTS FOR 'atendente';
SHOW GRANTS FOR 'comercial'@'localhost';
```

```sql
SHOW GRANTS FOR 'usuario1';
SHOW GRANTS FOR 'usuario2'@'%';
SHOW GRANTS FOR 'usuario3'@'localhost';
```

```sql
SELECT CURRENT_USER();
SHOW GRANTS;
```

--- 

### SET DEFAULT ROLE e SET ROLE

Atribuir um papel não garante que ele seja ativado automaticamente ao conectar. 

Sem o papel ativo, os privilégios dele podem não estar disponíveis, e o Workbench pode não listar o banco.

Como administrador (`root`), defina `diretor` como papel padrão para as contas `usuario1`, `usuario2` e `usuario3`:

```sql
SET DEFAULT ROLE 'diretor'
TO 'usuario1', 'usuario2'@'%', 'usuario3'@'localhost';
```

Depois, desconecte e reconecte cada usuário no Workbench e atualize (refresh) a lista de schemas. 

```sql
SELECT CURRENT_USER();
SHOW GRANTS;
```
**OBS**: para testar sem reconectar, o usuário pode executar:

```sql
SET ROLE 'diretor';
```

Se não aparecer, confira qual conta o MySQL reconheceu, executando `SELECT CURRENT_USER();` e verifique as concessões com `SHOW GRANTS` para essa conta e para o papel `diretor`.


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
