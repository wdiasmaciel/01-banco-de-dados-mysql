# 18 - Concessão de Permissões com GRANT

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="17-function.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="19-revoke.md">Próximo</a>
    </td>
  </tr>
</table>

---

# Data Control Language: GRANT

`GRANT` é um comando de Data Control Language (DCL). Ele concede privilégios a contas de usuário ou a papéis do MySQL.

Uma conta no MySQL pode conter usuário e origem da conexão, por exemplo:

```text
'ana'@'localhost'
'ana'@'192.168.1.%'
'ana'@'%'
```

Antes de conceder privilégios, recomenda-se criar a conta separadamente:

```sql
CREATE USER 'ana'@'localhost' 
IDENTIFIED BY 'SenhaForte123!';
```


```sql
SELECT User, Host
FROM mysql.user;
```

---

## Sintaxe geral

A forma geral do comando é:

```sql
GRANT privilégio
ON objeto
TO 'usuario'@'origem';
```

Ao conceder mais de um privilégio, separe os nomes por vírgula.

```sql
GRANT SELECT, INSERT, UPDATE
ON empresa.Fornecedor
TO 'ana'@'localhost';
```

Para consultar os privilégios concedidos a um usuário:

```sql
SHOW GRANTS 
FOR 'ana'@'localhost';
```

Para consultar os privilégios concedidos ao usuário corrente:
```sql
SHOW GRANTS;
```

---

## Papéis

O MySQL permite agrupar privilégios em papéis:

```sql
CREATE ROLE 'perfil_estoque';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT SELECT, INSERT, UPDATE
ON empresa.Produto
TO 'perfil_estoque';
```

```sql
SHOW GRANTS 
FOR 'perfil_estoque';
```

```sql
CREATE USER 'lucas'@'localhost' 
IDENTIFIED BY 'SenhaForte456!';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT 'perfil_estoque' 
TO 'lucas'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'lucas'@'localhost';
```

O usuário pode ativar seus papéis na sessão:

```sql
SET ROLE 'perfil_estoque';
```

Para ativar todos os papéis concedidos automaticamente em novas sessões:

```sql
SET DEFAULT 
ROLE ALL 
TO 'lucas'@'localhost';
```

---

## Verificando privilégios

Para consultar os privilégios de uma conta:

```sql
SHOW GRANTS 
FOR 'ana'@'localhost';
```

Para consultar a própria conta:

```sql
SHOW GRANTS;
```

O MySQL aplica privilégios considerando contas específicas, papéis ativos e a hierarquia dos objetos.

---

## Privilégios em nível global

Privilégios globais usam `*.*` e normalmente ficam restritos ao administrador do banco:

```sql
CREATE USER 'monitor'@'localhost' 
IDENTIFIED BY 'SenhaForte789!';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT PROCESS
ON *.* -- qualquer banco e qualquer tabela.
TO 'monitor'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'monitor'@'localhost';
```

**OBS**:
> PROCESS é um privilégio global do MySQL. Ele permite que a conta veja informações sobre as conexões e consultas em execução no servidor, inclusive as de outros usuários. Sem esse privilégio, em geral, a conta só consegue ver seus próprios processos. PROCESS não concede acesso para consultar ou alterar dados das tabelas. Exemplo:

```sql
SHOW PROCESSLIST;
```

O privilégio `ALL PRIVILEGES` concede todos os privilégios aplicáveis ao objeto indicado:

```sql
CREATE USER 'administrador_loja'@'localhost' 
IDENTIFIED BY 'SenhaForte123!';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT ALL PRIVILEGES
ON empresa.* -- banco empresa e qualquer tabela.
TO 'administrador_loja'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'administrador_loja'@'localhost';
```

**OBS**:
> Conceder `ALL PRIVILEGES ON *.*` é uma operação administrativa sensível. Prefira privilégios específicos.

---

## Privilégios em nível de banco

O exemplo abaixo concede permissões em todas as tabelas do banco `empresa`:

```sql
CREATE USER 'atendente'@'localhost' 
IDENTIFIED BY 'SenhaForte123!';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT SELECT, INSERT, UPDATE
ON empresa.* -- Concede as permissões em todas as tabelas do banco empresa.
TO 'atendente'@'localhost';
```

Não conceda privilégios mais amplos do que o necessário sem verificar cuidadosamente quais bancos serão alcançados.

---

## Privilégios em nível de tabela

```sql
CREATE ROLE 'consulta_produto';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT SELECT
ON empresa.Produto
TO 'consulta_produto'@'%';
```

```sql
SHOW GRANTS 
FOR 'consulta_produto';
```

Várias operações podem ser concedidas para uma tabela:

```sql
CREATE ROLE 'estoquista'@'localhost';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.Produto
TO 'estoquista'@'localhost';
```

```sql
GRANT 'estoquista' 
TO 'lucas'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'lucas'@'localhost';
```

Para permitir execução de consultas por uma visão e não por suas tabelas internas, conceda privilégio sobre a visão:


```sql
CREATE VIEW empresa.vw_produto_estoque_publico AS
SELECT p.id, p.nome, e.quantidade, e.preco, e.validade
FROM empresa.Produto AS p
JOIN empresa.Estoque AS e
  ON e.id_produto = p.id;
```

```sql
GRANT SELECT
ON empresa.vw_produto_estoque_publico
TO 'consulta_produto'@'%';
```

```sql
SHOW GRANTS 
FOR 'consulta_produto'@'%';
```

---

## Privilégios em nível de coluna

É possível conceder permissões para determinadas colunas:

```sql
CREATE ROLE 'suporte'@'localhost';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT SELECT (cnpj, nome, telefone)
ON empresa.Fornecedor
TO 'suporte'@'localhost';
```

```sql
GRANT UPDATE (telefone)
ON empresa.Fornecedor
TO 'suporte'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'suporte'@'localhost';
```

Para operações que exigem outras colunas, os privilégios adicionais devem ser concedidos separadamente.

---

## Privilégios de rotina

Procedures e funções também podem receber privilégios específicos:

```sql
CREATE ROLE 'vendas'@'localhost';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
DELIMITER //

CREATE PROCEDURE empresa.calcular_total_estoque()
BEGIN
    SELECT COALESCE(SUM(quantidade * preco), 0) AS total
    FROM empresa.Estoque;
END//

DELIMITER ;
```

```sql
CALL empresa.calcular_total_estoque();
```

```sql
GRANT EXECUTE
ON PROCEDURE empresa.calcular_total_estoque
TO 'vendas'@'localhost';
```

```sql
DELIMITER //

CREATE FUNCTION empresa.calcular_desconto(valor DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN valor * 0.90;
END//

DELIMITER ;
```

```sql
SELECT empresa.calcular_desconto(100) AS desconto;
```

```sql
GRANT EXECUTE
ON FUNCTION empresa.calcular_desconto
TO 'vendas'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'vendas'@'localhost';
```
---

## WITH GRANT OPTION

`WITH GRANT OPTION` permite que o usuário repasse os privilégios que recebeu, desde que esses privilégios estejam dentro de seu próprio conjunto autorizado:

```sql
CREATE ROLE 'gerente'@'localhost';
```

```sql
SELECT User, Host
FROM mysql.user;
```

```sql
GRANT SELECT
ON empresa.Produto
TO 'gerente'@'localhost'
WITH GRANT OPTION;
```

```sql
SHOW GRANTS 
FOR 'gerente'@'localhost';
```

Esse recurso deve ser usado com cuidado, pois distribui autoridade para terceiros.

---
## Conceder vários papéis ao mesmo usuário

Conceder vários papéis ao mesmo usuário em um único comando, separados por vírgula:

```sql
GRANT 'consulta_produto'@'localhost',
      'perfil_estoque'@'localhost'
TO 'ana'@'localhost';
```

Para ativar todos os papéis concedidos por padrão em novas sessões:

```sql
SET DEFAULT ROLE ALL
TO 'ana'@'localhost';
```

```sql
SHOW GRANTS 
FOR 'ana'@'localhost';
```

---

# Exercícios

### Padaria
1. Crie uma conta de leitura para uma padaria chamada `consulta_pao` e conceda `SELECT` na tabela de produtos.
2. Conceda à conta de um caixa da padaria acesso de consulta e inserção na tabela de vendas.
3. Permita que um estoquista atualize a quantidade de produtos em uma tabela de estoque.

### Farmácia
4. Conceda a apenas um vendedor de uma farmácia privilégios de consulta sobre `vw_medicamentos_disponiveis`.
5. Crie uma conta para um supervisor de farmácia com `SELECT` e `UPDATE` na tabela de medicamentos.

### Agência de Viagens
6. Conceda ao atendente de uma agência privilégios de inserção na tabela de reservas.
7. Permita que um analista turístico consulte apenas as colunas de código, destino e preço de pacotes.
8. Conceda ao analista da agência privilégio de atualização somente na coluna `preco`.

### Supermercado
9. Crie uma conta que possa consultar todas as tabelas de um banco chamado `supermercado`.
10. Conceda privilégio de execução em uma procedure chamada `fechar_venda` para o usuário do caixa.
11. Conceda privilégio de execução em uma função chamada `calcular_frete` para um atendente.
12. Crie um papel chamado `perfil_operador` para consultar produtos e inserir vendas.
13. Conceda o papel criado aos usuários `joao` e `marcela`, em acessos vindos de `localhost`.
14. Configure o papel de operador para ser ativado automaticamente em novas sessões de `joao`.
15. Conceda a um gerente o privilégio de consulta na tabela de fornecedores com a opção de repassar essa permissão.
16. Crie uma conta com acesso global apenas ao privilégio `PROCESS` para monitorar processos.

<!--
CREATE USER 'monitor_processos'@'localhost'
IDENTIFIED BY 'SenhaForte123!';

GRANT PROCESS
ON *.*
TO 'monitor_processos'@'localhost';

SHOW GRANTS
FOR 'monitor_processos'@'localhost';

*.* indica todos os bancos e todas as tabelas, configurando o privilégio em nível global. O privilégio PROCESS permite visualizar informações sobre processos e threads, mas não concede privilégios de consulta ou alteração em tabelas.

Para entrar com essa conta e testar os privilégios:

mysql -u monitor_processos -p

Dentro da sessão:

SHOW PROCESSLIST;

Para conferir, a saída de SHOW GRANTS deverá conter aproximadamente:

GRANT PROCESS ON *.* TO 'monitor_processos'@'localhost'
-->

17. Consulte todos os privilégios concedidos ao usuário de relatórios do supermercado.
18. Explique por que não é adequado conceder `ALL PRIVILEGES ON *.*` a um vendedor.
19. Conceda privilégio de consulta e atualização em uma visão de clientes sem conceder acesso direto à tabela de clientes.
20. Crie um conjunto de contas e privilégios para os perfis caixa, estoque e gerente do supermercado.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="17-function.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="19-revoke.md">Próximo</a>
    </td>
  </tr>
</table>
