# Git & GitHub

Para criar uma branch nova a partir da branch atual e já mudar para ela:

```bash
git switch -c nome-da-branch
```

Por exemplo:

```bash
git switch -c temp
```


Para criá-la a partir de main, mesmo estando em outra branch:

```bash
git switch -c nome-da-branch main
```

Por exemplo:

```bash
git switch -c temp main
```

Mudar para a branch main:
```bash
git switch main
```

Criar a branch temp a partir da main:
```bash
git switch -c temp
```

Trabalhe em temp e faça commits normalmente. Quando quiser levar as alterações para main:


```bash
git switch main
```

```bash
git merge temp
```

Depois, volte para temp e continue trabalhando:

```bash
git switch temp
```


Posteriormente, repita o merge de temp em main. O Git levará apenas os commits novos. Se main também tiver recebido alterações próprias, você pode primeiro atualizá-las em temp com git merge main e então fazer o merge de temp em main.



```bash
git status --short --branch
```

```bash
git branch --show-current
```

Se a branch local temp já existe e tem ao menos um commit, publique-a com:

```bash
git switch temp
```

```bash
git push -u origin temp
```

Você pode fazer o merge de temp em main e continuar trabalhando em temp. O primeiro merge não apaga a branch temp.

```bash
git switch main
git merge temp

git switch temp
# faça novas alterações
git add .
git commit -m "Novas alterações"
```

Depois, você pode fazer outro merge:

```bash
git switch main
git merge temp
```

O segundo merge incluirá as novas alterações de temp que ainda não chegaram a main. 

Se main também tiver recebido alterações nesse intervalo, o merge pode exigir resolução de conflitos.

