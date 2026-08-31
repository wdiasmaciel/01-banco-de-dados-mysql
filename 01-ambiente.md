
# Ambiente

<table width="100%" style="border: none; border-collapse: collapse;">
  <tr style="border: none;">
    <td align="left" style="border: none;">
      <a href="arquivo-anterior.md">← Anterior</a>
    </td>
    <td align="right" style="border: none;">
      <a href="proximo-arquivo.md">Próximo →</a>
    </td>
  </tr>
</table>

1. Nós vamos utilizar o **GitHub Codespaces** para realizar as nossas práticas. 

2. Isso significa que você não precisa instalar programas no seu computador pessoal. 

3. Tudo vai rodar diretamente no seu navegador.

---

# Abrindo o Ambiente (Codespaces)

1. No topo desta página do repositório no GitHub, clique no botão verde **`< > Code`**.

2. Clique na aba **Codespaces**.

3. Clique no botão verde **Create codespace on main**.

4. Aguarde alguns instantes até que o VS Code abra no seu navegador.

---

# Configuração do Ambiente

Instalar o `Node`, o `MySQL` e a `Database Client`.

1. No Codespaces, adicione o `Node.js`, o `MySQL` e a extensão `Database Client` do `VS Code` no container do `Codespace`, usando um arquivo de configuração `devcontainer.json`. 

Além disso, inicie o serviço do `MySQL` de forma automática assim que o `Codespaces` abrir.

1. Crie uma pasta chamada `.devcontainer` na raiz do seu repositório do GitHub. 

2. Dentro da pasta `.devcontainer`, crie um arquivo chamado `devcontainer.json` com o código abaixo:

```json
{
  "name": "Aulas de Banco de Dados (MySQL)",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "customizations": {
    "vscode": {
      "extensions": [
        "cweijan.vscode-mysql-client2"
      ]
    }
  },
  "onCreateCommand": "sudo apt update && sudo apt install -y nodejs npm && sudo apt-get install -y mysql-server git-lfs && git lfs install",
  "postStartCommand": "sudo mkdir -p /var/run/mysqld && sudo chown mysql:mysql /var/run/mysqld && sudo /usr/sbin/mysqld --user=mysql &"
}
```

3. No terminal do `VS Code`, execute o comando abaixo:

```bash
git add . && git commit -m "Exemplo" && git push
```

---

# Rebuild

1. No `VS Code` do `Codespaces`, digite `CTRL + SHIFT + P`. 

2. Na barra de pesquisa digite `rebuild`. 

3. Selecione a opção `Codespaces: Rebuild Container`. 

4. Clique no botão `Rebuild`. 

5. Aguarde o término do processamento.

---

# Versão

1. No terminal, execute os comandos abaixo, para visualizar a versão instalada do `Node.js`, do `Node Package Manager (NPM)` e do `MySQL`:  :

```bash
node -v 
```

```bash
npm -v
```

```bash
mysql --version
```

**OBS**: no caso do `MySQL`, pode ser que seja necessário executar `mysql -V` ou executar o comando `SELECT VERSION()`; após acessar o prompt do banco de dados.

---
