# 03 - Projeto Empresa

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="02-mysql.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="04-banco-de-dados.md">Próximo</a>
    </td>
  </tr>
</table>

---

Projeto de uma empresa usado nos exemplos:

![DER Empresa](./img/empresa2.png)

1. As entidades do sistema são: Fornecedor, Produto, Identificação, Filial e Estoque.

2. A entidade Fornecedor possui os atributos: cnpj (chave primária), nome (NOT NULL e UNIQUE), telefone (NOT NULL e UNIQUE) e endereço (NOT NULL).

3. A entidade Produto possui os atributos: id (chave primária), cnpj_fornecedor (chave estrangeira que se refere à chave primária de fornecedor) e nome (NOT NULL).

4. A entidade Identificação possui os atributos: id (chave primária e também chave estrangeira que se refere à chave primária de Produto), descrição (NOT NULL), e observação (NOT NULL).

5. A entidade Filial possui os atributos: cnpj (chave primária), nome (NOT NULL e UNIQUE), telefone (NOT NULL e UNIQUE) e endereço (NOT NULL).

6. A entidade Estoque possui os atributos: id_produto (chave primária e chave estrangeira que se refere à chave primária de Produto), cnpj_filial (chave primária e chave estrangeira que se refere à chave primária de Filial), preço (NOT NULL), quantidade (NOT NULL) e validade (NOT NULL), que é a data de validade do produto em estoque. 
 
7. No projeto: 
  - Cada Fornecedor fornece vários produtos, mas cada produto é fornecido por apenas um fornecedor. 
  - Cada produto possui apenas uma única identificação e cada identificação refere-se a apenas um único produto. 
  - Cada Filial vende vários produtos e cada produto pode ser vendido em mais de uma filial. 
  - Entre as entidades Produto e Filial há a entidade-relacionamento Estoque. 

---

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="02-mysql.md">Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="04-banco-de-dados.md">Próximo</a>
    </td>
  </tr>
</table>
