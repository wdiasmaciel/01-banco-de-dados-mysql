# 19 - Remoção de Objetos com DROP

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="18-deny.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

# Comando DROP

`DROP` pertence à Data Definition Language (DDL). Ele remove objetos do banco de dados.

Essa operação é diferente de `DELETE`: `DELETE` remove linhas, enquanto `DROP` remove o objeto e sua estrutura.

---

## Removendo um banco de dados

```sql
DROP DATABASE empresa;
```

O comando remove permanentemente o banco e todos os objetos nele contidos. Para evitar erro caso o banco não exista:

```sql
DROP DATABASE IF EXISTS empresa;
```

Verifique se o banco ainda aparece na lista de bancos acessíveis:

```sql
SHOW DATABASES;
```

Antes da execução, faça backup e confirme o banco selecionado:

```sql
SELECT DATABASE();
```

---

## Removendo uma tabela

```sql
DROP TABLE Produto;
```

Ao excluir uma tabela com registros, todos os dados são removidos com ela:

```sql
DROP TABLE IF EXISTS Produto;
```

Confirme a remoção consultando o banco no catálogo:

```sql
SELECT table_name
FROM   information_schema.tables
WHERE  table_schema = DATABASE()
AND    table_name = 'Produto';
```

É possível remover várias tabelas em uma única instrução:

```sql
DROP TABLE Produto, Estoque, Fornecedor;
```

A ordem pode importar quando existirem chaves estrangeiras. O MySQL pode impedir a remoção de uma tabela pai enquanto houver uma restrição dependente.

Exemplo seguro de remoção com dependência:

```sql
ALTER TABLE Estoque
DROP FOREIGN KEY fk_estoque_produto;

DROP TABLE Produto;
```

Também é possível impedir temporariamente a verificação de chaves estrangeiras em uma sessão, mas essa opção é administrativa e não remove dependências:

```sql
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE Produto;
SET FOREIGN_KEY_CHECKS = 1;
```

Em scripts reais, prefira remover objetos dependentes na ordem correta e restaurar a configuração.

---

## Tabelas temporárias

Uma tabela temporária pertence à sessão atual e é removida com:

```sql
DROP TEMPORARY TABLE IF EXISTS clientes_relatorio;
```

`DROP TEMPORARY TABLE` não remove uma tabela permanente com o mesmo nome.

---

## Removendo uma visão

```sql
DROP VIEW IF EXISTS vw_produto_estoque;
```

```sql
SELECT table_name
FROM   information_schema.views
WHERE  table_schema = DATABASE()
AND    table_name = 'vw_produto_estoque';
```

Remover uma visão não remove os dados e a tabela que servem como base da consulta.

---

## Removendo índices

Em MySQL, o índice é removido com sintaxe de tabela:

```sql
ALTER TABLE Produto
DROP INDEX idx_produto_nome;
```

Também é possível remover um índice com `DROP INDEX`:

```sql
DROP INDEX idx_produto_nome ON Produto;
```

```sql
SHOW INDEX FROM Produto
WHERE  Key_name = 'idx_produto_nome';
```

Chaves estrangeiras, restrições únicas ou índices usados por outras estruturas podem exigir retirada planejada.

---

## Removendo um usuário ou papel

```sql
DROP USER 'ana'@'localhost';
```

```sql
DROP ROLE 'perfil_estoque';
```

```sql
SELECT user, host
FROM   mysql.user
WHERE  user = 'perfil_estoque';
```

Para remover vários objetos de usuário ou papel, cada objeto é separado por vírgula:

```sql
DROP USER 'ana'@'localhost', 'joao'@'localhost';
```

Ao remover um papel, ele deixa de estar atribuído aos usuários.

---

## Removendo procedimentos e funções

```sql
DROP PROCEDURE IF EXISTS calcular_desconto;
```

```sql
SELECT routine_name
FROM   information_schema.routines
WHERE  routine_schema = DATABASE()
AND    routine_name = 'calcular_desconto';
```

```sql
DROP FUNCTION IF EXISTS calcular_frete;
```

```sql
SELECT routine_name
FROM   information_schema.routines
WHERE  routine_schema = DATABASE()
AND    routine_name = 'calcular_frete';
```

---

## Removendo gatilhos

```sql
DROP TRIGGER IF EXISTS atualizar_estoque;
```

```sql
SELECT trigger_name
FROM   information_schema.triggers
WHERE  trigger_schema = DATABASE()
AND    trigger_name = 'atualizar_estoque';
```

O gatilho pode ter sido criado para outro banco; confirme o contexto antes da remoção:

```sql
USE empresa;
DROP TRIGGER IF EXISTS atualizar_estoque;
```

---

## Removendo eventos

```sql
DROP EVENT IF EXISTS fechar_dia;
```

```sql
SELECT event_name
FROM   information_schema.events
WHERE  event_schema = DATABASE()
AND    event_name = 'fechar_dia';
```

---

## Diferenças entre comandos de remoção

| Comando | O que remove | Estrutura permanece? |
|---|---|---|
| `DELETE FROM tabela` | Linhas selecionadas | Sim |
| `TRUNCATE TABLE tabela` | Todas as linhas | Sim |
| `DROP TABLE tabela` | Tabela, estrutura e dados | Não |
| `DROP VIEW view` | A visão | Tabela base permanece |
| `DROP DATABASE banco` | Banco e seus objetos | Não |

`TRUNCATE TABLE` remove todos os dados de modo mais direto que `DELETE`, mas não deve ser confundido com `DROP TABLE`.

---

## Boas práticas

1. Faça backup antes de remover objetos relevantes.
2. Use `IF EXISTS` em scripts que possam ser executados várias vezes.
3. Verifique dependências como visões, procedimentos, chaves estrangeiras e gatilhos.
4. Remova primeiro os objetos dependentes ou atualize referências.
5. Confirme o banco e o objeto antes de executar `DROP DATABASE` ou `DROP TABLE`.
6. Prefira desativar uma aplicação ou limitar acessos ao excluir dados usados por ela.

---

# Exercícios

1. Remova uma visão chamada `vw_pao_em_falta` sem apagar a tabela de produtos.
2. Exclua uma tabela temporária chamada `resumo_horario` da sessão atual.
3. Remova uma tabela permanente chamada `produto_teste` somente se ela existir.
4. Apague o banco chamado `padaria_teste` depois de confirmar que não é usado pelo sistema real.
5. Remova duas tabelas auxiliares da padaria com um único comando.
6. Exclua uma procedure chamada `atualizar_preco_medio`.
7. Remova uma função chamada `calcular_validade_promocao`.
8. Exclua um gatilho chamado `baixar_estoque_medicamento`.
9. Remova um evento chamado `gerar_relatorio_noturno`.
10. Exclua o papel `perfil_caixa_temporario`.
11. Remova uma conta que não pertence mais à equipe da farmácia.
12. Remova um índice chamado `idx_medicamento_principio_ativo`.
13. Remova um índice segundo a sintaxe própria do MySQL para a tabela de uma agência.
14. Explique por que `DROP TABLE` é mais destrutivo que `DELETE FROM tabela`.
15. Crie uma sequência que remova uma chave estrangeira e depois a tabela referenciada.
16. Escreva a diferença prática entre `TRUNCATE TABLE` e `DROP TABLE`.
17. Remova uma visão que depende de uma tabela, mas mantenha essa tabela.
18. Apague objetos temporários que não tenham sido usados pelo restante da sessão.
19. Planeje a exclusão segura de um banco de dados de ambiente de testes.
20. Classifique em uma tabela se `DROP`, `DELETE` ou `TRUNCATE` deve ser usado para cinco situações administrativas.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="18-deny.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
