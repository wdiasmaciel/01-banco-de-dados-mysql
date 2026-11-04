# 16 - Concessão de Permissões com GRANT

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="15-view.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="17-revoke.md">Próximo</a>
    </td>
  </tr>
</table>

---

# GRANT e Data Control Language

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

```sql
SHOW GRANTS FOR 'ana'@'localhost';
```

---

## Privilégios em nível global

Privilégios globais usam `*.*` e normalmente ficam restritos ao administrador do banco:

```sql
GRANT PROCESS
ON *.*
TO 'monitor'@'localhost';
```

```sql
SHOW GRANTS FOR 'monitor'@'localhost';
```

O privilégio `ALL PRIVILEGES` concede todos os privilégios aplicáveis ao objeto indicado:

```sql
GRANT ALL PRIVILEGES
ON empresa.*
TO 'administrador_loja'@'localhost';
```

Conceder `ALL PRIVILEGES ON *.*` é uma operação administrativa sensível. Em aulas, prefira privilégios específicos.

---

## Privilégios em nível de banco

O exemplo concede permissões em todas as tabelas do banco:

```sql
GRANT SELECT, INSERT, UPDATE
ON empresa.*
TO 'atendente'@'localhost';
```

Não conceda privilégios mais amplos do que o necessário sem verificar cuidadosamente quais bancos serão alcançados.

---

## Privilégios em nível de tabela

```sql
GRANT SELECT
ON empresa.Produto
TO 'consulta_produto'@'localhost';
```

Várias operações podem ser concedidas para uma tabela:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON empresa.Produto
TO 'estoquista'@'localhost';
```

Para permitir execução de consultas por uma visão e não por suas tabelas internas, conceda privilégio sobre a visão:

```sql
GRANT SELECT
ON empresa.vw_produto_estoque_publico
TO 'consulta_produto'@'localhost';
```

---

## Privilégios em nível de coluna

É possível conceder permissões para determinadas colunas:

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

Para operações que exigem outras colunas, os privilégios adicionais devem ser concedidos separadamente.

---

## Privilégios de rotina

Procedures e funções também podem receber privilégios específicos:

```sql
GRANT EXECUTE
ON PROCEDURE empresa.calcular_total_pedido
TO 'vendas'@'localhost';
```

```sql
GRANT EXECUTE
ON FUNCTION empresa.calcular_desconto
TO 'vendas'@'localhost';
```

---

## WITH GRANT OPTION

`WITH GRANT OPTION` permite que o usuário repasse os privilégios que recebeu, desde que esses privilégios estejam dentro de seu próprio conjunto autorizado:

```sql
GRANT SELECT
ON empresa.Produto
TO 'gerente'@'localhost'
WITH GRANT OPTION;
```

Esse recurso deve ser usado com cuidado, pois distribui autoridade para terceiros.

---

## Papéis

O MySQL permite agrupar privilégios em papéis:

```sql
CREATE ROLE 'perfil_estoque';

GRANT SELECT, INSERT, UPDATE
ON empresa.Produto
TO 'perfil_estoque';

CREATE USER 'lucas'@'localhost' IDENTIFIED BY 'SenhaForte456!';
GRANT 'perfil_estoque' TO 'lucas'@'localhost';
```

```sql
SHOW GRANTS FOR 'lucas'@'localhost';
```

O usuário pode ativar seus papéis na sessão:

```sql
SET ROLE 'perfil_estoque';
```

Para ativar todos os papéis concedidos automaticamente em novas sessões:

```sql
SET DEFAULT ROLE ALL TO 'lucas'@'localhost';
```

---

## Verificando privilégios

Para consultar os privilégios de uma conta:

```sql
SHOW GRANTS FOR 'ana'@'localhost';
```

Para consultar a própria conta:

```sql
SHOW GRANTS;
```

O MySQL aplica privilégios considerando contas específicas, papéis ativos e a hierarquia dos objetos.


---

# Exercícios

1. Crie uma conta de leitura para a padaria chamada `consulta_pao` e conceda `SELECT` na tabela de produtos.
2. Conceda à conta de um caixa da padaria acesso de consulta e inserção na tabela de vendas.
3. Permita que um estoquista atualize a quantidade de produtos em uma tabela de estoque.
4. Conceda a apenas um vendedor da farmácia privilégios de consulta sobre `vw_medicamentos_disponiveis`.
5. Crie uma conta para um supervisor de farmácia com `SELECT` e `UPDATE` na tabela de medicamentos.
6. Conceda ao atendente de uma agência privilégios de inserção na tabela de reservas.
7. Permita que um analista turístico consulte apenas as colunas de código, destino e preço de pacotes.
8. Conceda ao analista da agência privilégio de atualização somente na coluna `preco`.
9. Crie uma conta que possa consultar todas as tabelas de um banco chamado `supermercado`.
10. Conceda privilégio de execução em uma procedure chamada `fechar_venda` para o usuário do caixa.
11. Conceda privilégio de execução em uma função chamada `calcular_frete` para um atendente.
12. Crie um papel chamado `perfil_operador` para consultar produtos e inserir vendas.
13. Conceda o papel criado aos usuários `joao` e `marcela`, em acessos vindos de `localhost`.
14. Configure o papel de operador para ser ativado automaticamente em novas sessões de `joao`.
15. Conceda a um gerente o privilégio de consulta na tabela de fornecedores com a opção de repassar essa permissão.
16. Crie uma conta com acesso global apenas ao privilégio `PROCESS` para monitorar processos.
17. Consulte todos os privilégios concedidos ao usuário de relatórios da padaria.
18. Explique por que não é adequado conceder `ALL PRIVILEGES ON *.*` a um vendedor.
19. Conceda privilégio de consulta e atualização em uma visão de clientes sem conceder acesso direto à tabela de clientes.
20. Crie um conjunto de contas e privilégios para os perfis caixa, estoque e gerente de uma loja.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="15-view.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="17-revoke.md">Próximo</a>
    </td>
  </tr>
</table>
