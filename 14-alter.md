# 14 - Alteração de Estrutura com ALTER

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="13-select.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>

---

# Comando ALTER TABLE

O comando `ALTER TABLE` pertence à Data Definition Language (DDL) e altera a estrutura de uma tabela já existente. 

Ele pode adicionar, remover ou modificar colunas, alterar restrições (constraints), índices e nomes de objetos.

Antes de alterar uma tabela importante, consulte seus dados, faça um backup e analise como as alterações podem afetar consultas, aplicações e outros objetos relacionados.

```sql
ALTER TABLE nome_da_tabela
    operação;
```

Também é possível executar várias operações em um único comando, separando-as por vírgulas.

```sql
ALTER TABLE Produto
        ADD COLUMN codigo_barras VARCHAR(20) NULL,
        ADD COLUMN ativo BOOLEAN NOT NULL DEFAULT TRUE;
```

---

## Adicionando uma coluna

A cláusula `ADD COLUMN` cria uma nova coluna na tabela.

```sql
ALTER TABLE Produto
        ADD COLUMN peso DECIMAL(8,3) NULL;
```

Por padrão, a nova coluna é adicionada ao final da tabela.

Para posicionar uma coluna, o MySQL permite usar `FIRST`, que coloca o campo no início, ou `AFTER`, que o coloca depois de outra coluna.

```sql
ALTER TABLE Produto
        ADD COLUMN codigo VARCHAR(20) NULL
      FIRST;
```

```sql
ALTER TABLE Produto
        ADD COLUMN unidade_medida VARCHAR(20) NOT NULL DEFAULT 'unidade'
      AFTER nome;
```

> **OBS:** `FIRST` e `AFTER` são recursos específicos do MySQL.

---

## Modificando uma coluna

### ALTER COLUMN

Em `ALTER COLUMN`, é possível alterar o valor padrão ou remover o valor padrão de uma coluna.

```sql
ALTER TABLE Produto
      ALTER COLUMN unidade_medida SET DEFAULT 'caixa';
```

```sql
ALTER TABLE Produto
      ALTER COLUMN unidade_medida DROP DEFAULT;
```

### MODIFY COLUMN

`MODIFY COLUMN` altera o tipo de dados, o tamanho, a nulabilidade ou o valor padrão. O nome da coluna permanece o mesmo.

```sql
ALTER TABLE Produto
     MODIFY COLUMN nome VARCHAR(150) NULL;
```

### CHANGE COLUMN

`CHANGE COLUMN` altera o tipo da coluna e permite renomeá-la ao mesmo tempo.

```sql
  ALTER TABLE Produto
CHANGE COLUMN peso peso_gramas INT NULL;
```

Antes de executar mudanças de tipo, verifique se os valores existentes podem ser convertidos sem perda de dados. 

Por exemplo, converter texto em número pode falhar ou gerar resultados inesperados.

---

## Renomeando uma coluna

O uso de `CHANGE COLUMN` com o mesmo nome conserva o campo, mas altera sua definição.

```sql
  ALTER TABLE Produto
CHANGE COLUMN codigo_barras codigo_barras VARCHAR(30) NULL;
```

Para alterar apenas o nome e manter o mesmo tipo e as mesmas restrições, o MySQL também permite:

```sql
  ALTER TABLE Produto
RENAME COLUMN codigo_barras TO codigo_produto;
```

---

## Removendo uma coluna

A cláusula `DROP COLUMN` remove a coluna e seus dados.

```sql
ALTER TABLE Produto
       DROP COLUMN unidade_medida;
```

> **OBS:** 
1. `DROP COLUMN` é uma operação destrutiva. 
2. Antes de executá-la, confirme que a coluna não é usada por consultas, índices, chaves estrangeiras ou aplicações.

---

## Adicionando e removendo restrições

### NOT NULL

Para aplicar `NOT NULL`, todos os registros existentes devem ter valor nessa coluna.

```sql
ALTER TABLE Produto
     MODIFY COLUMN peso_gramas INT NOT NULL;
```

Se registros existentes estiverem nulos, preencha esses valores antes de executar a alteração:

```sql
UPDATE Produto
SET    peso_gramas = 0
WHERE  peso_gramas IS NULL;
```

### CHECK

O MySQL permite criar uma restrição `CHECK` com um nome específico:

```sql
ALTER TABLE Produto
        ADD CONSTRAINT chk_produto_peso
      CHECK (peso_gramas >= 0);
```

Para remover essa restrição:

```sql
ALTER TABLE Produto
       DROP CHECK chk_produto_peso;
```

### UNIQUE

Uma restrição de unicidade impede valores repetidos na coluna.

```sql
ALTER TABLE Produto
        ADD CONSTRAINT uq_produto_codigo
     UNIQUE (codigo_produto);
```

Para remover essa restrição:

```sql
ALTER TABLE Produto
 DROP INDEX uq_produto_codigo;
```

---

## Adicionando uma chave estrangeira

Uma chave estrangeira exige que os valores da coluna referenciada exista na tabela estrangeira.

```sql
ALTER TABLE Produto
        ADD CONSTRAINT fk_produto_fornecedor
            FOREIGN KEY (cnpj_fornecedor)
            REFERENCES Fornecedor (cnpj);
```

Para remover a chave estrangeira:

```sql
ALTER TABLE Produto
       DROP FOREIGN KEY fk_produto_fornecedor;
```

Após remover uma chave estrangeira, o índice criado pelo MySQL pode continuar existindo e, se necessário, deve ser removido separadamente:

```sql
ALTER TABLE Produto
 DROP INDEX fk_produto_fornecedor;
```

---

## Adicionando e removendo índices

Índices podem acelerar consultas, mas aumentam o custo de inserção e atualização dos dados.

```sql
ALTER TABLE Produto
  ADD INDEX idx_produto_nome (nome);
```

```sql
ALTER TABLE Produto
 DROP INDEX idx_produto_nome;
```

---

## Modificando a chave primária

Remover uma chave primária pode permitir valores duplicados ou nulos. Sempre confira os dados antes dessa operação.

```sql
ALTER TABLE Produto
       DROP PRIMARY KEY;
```

Se os registros forem únicos e válidos, uma nova chave primária pode ser criada:

```sql
ALTER TABLE Produto
        ADD PRIMARY KEY (id);
```

---

## Renomeando uma tabela

O comando `RENAME TO` altera o nome da tabela sem mover ou copiar seus dados.

```sql
ALTER TABLE Produto
     RENAME TO Item;
```

Nomes usados por aplicações e consultas também precisam ser atualizados.

---

## Boas práticas

1. Faça backup antes de mudanças estruturais relevantes.

2. Verifique se há dados que impedem a aplicação de `NOT NULL`, `UNIQUE`, `CHECK` ou chave estrangeira.

3. Prefira executar uma alteração por vez quando estiver aprendendo.

4. Não remova colunas sem confirmar se existem consultas ou processos que ainda utilizam essas colunas.

5. Após alterar nomes, tipos ou restrições, execute consultas para validar os dados existentes.

---

# Exercícios

1. Em uma tabela chamada `Padaria_Pao`, adicione uma coluna `data_fabricacao` do tipo `DATE` que não permita valor nulo.

2. Na mesma tabela, adicione uma coluna opcional `observacao`, com tamanho máximo de 200 caracteres.

3. Altere o tamanho da coluna `nome` para permitir 120 caracteres.

4. Renomeie a coluna `valor_venda` para `preco` sem alterar seu tipo.

5. Adicione a coluna `peso_gramas` logo após a coluna `nome`.

6. Na tabela da padaria, altere o preço padrão do produto para o valor 0,00.

7. Remova o valor padrão da coluna `peso_gramas`.

8. Remova da tabela de padaria uma coluna chamada `codigo_interno` que não é mais utilizada.

9. Na tabela `Farmacia_Medicamento`, torne a coluna `principio_ativo` obrigatória.

10. Adicione à tabela de medicamentos uma restrição que impeça o estoque de receber valores negativos.

11. Crie uma restrição única para impedir medicamentos com o mesmo código de barras.

12. Remova da tabela de medicamentos a restrição única criada no exercício anterior.

13. Adicione à tabela `Medicamento_Venda` uma chave estrangeira relacionando o medicamento ao respectivo registro da tabela `Farmacia_Medicamento`.

14. Remova essa chave estrangeira sem remover a coluna que contém o identificador do medicamento.

15. Crie um índice para acelerar consultas pelo nome da agência de viagens.

16. Adicione uma chave primária à tabela `Viagem` formada pelas colunas `codigo_pacote` e `data_saida`.

17. Remova temporariamente a chave primária da tabela `Viagem` e explique por que essa operação pode ser perigosa.

18. Renomeie a tabela `Cliente_Viagem` para `Turista`.

19. Adicione simultaneamente uma coluna `ativo` e um índice pelo campo `destino` na tabela de pacotes turísticos.

20. Escreva uma sequência de alterações que aumente o tamanho de um campo de e-mail, renomeie esse campo para `email_contato` e adicione uma restrição única.

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="13-select.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="#">Próximo</a>
    </td>
  </tr>
</table>
