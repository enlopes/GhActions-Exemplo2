
# Tutorial

## Como trabalhar no RStudio com GitHub e a funcionalidade GitHub Actions

### O que eu farei neste projeto?

Este é um exemplo de como criar um projeto R e conectá-lo ao GitHub.
Além disso, também utilizaremos a funcionalidade GitHub Actions para
sempre que um usuário realizar um push ou para um período de tempo
pré-determinado seja disparada uma action realizando determinada ação
pelo servidor do GitHub.

#### Criando o projeto R localmente e enviando para o GitHub

1.  O passo inicial é criar um projeto R no diretório de interesse. Para
    esse exemplo, criaremos o projeto em :
    `C:\Arquivos\Documents\GhActions-Exemplo2.Rproj`

2.  Quando da criação do projeto, selecione a opção de que o projeto
    seja acompanhado pelo Git.

3.  Precisaremos do pacote *usethis* para conectar o projeto ao GitHub.
    Assim, iremos realizar a instalação:

``` r
install.packages("usethis")
```

1.  Agora, precisaremos informar ao RStudio que vamos trabalhar com Git.

``` r
usethis::use_git()
```

1.  Após o comando acima, você receberá uma mensagem informando que
    alguns arquivos foram adicionados e que existem diversos arquivos
    que ainda não foram *commitados*. No fim da mensagem, o RStudio
    perguntará se você deseja *commitar* esses arquivos agora. Pode
    realizar o *commit* nesse momento.
2.  Ótimo. Agora os arquivos já estão prontos para serem enviados ao
    GitHub. No entanto, você pode verificar que o botão de Push, na guia
    Git do RStudio, ainda não está habilitado. Para resolver, vamos
    informar ao RStudio pra qual endereço do GitHub ele deve enviar os
    arquivos:

``` r
usethis::use_github()
```

1.  Após isso, você receberá mensagem informando que o repositório foi
    criado no GitHub. Observe que os arquivos já foram enviados para o
    repositório na web.

#### Atualizando nosso arquivo README.md automaticamente

Para nosso exemplo, vamos atualizar o arquivo README.md automaticamente.
Dessa forma, como o GitHub roda os arquivos .md e já apresenta o
resultado de forma formatada na sua própria página inicial, nós veremos
o resultado da nossa Action de forma mais rápida e transparente.

Assim, precisaremos criar nosso arquivo README.qmd, o qual gerará o
arquivo README.md:

A título de exemplo, vamos editar o README.qmd da seguinte forma:

``` r
---
title: "Atualização do Documento"
format: gfm
---

current_time <- Sys.time() |>
  format("%d/%m/%Y %H:%M")


Este documento foi atualizado pela última vez em: `r current_time`
```

#### Utilizando a funcionalidade GitHub Actions

O GitHub Actions é muito útil para automatizar tarefas. Em nosso caso,
vamos utilizá-lo para sempre que um usuário realizar um *push* (ou para
que em determinado período de tempo) um de nossos arquivos seja
atualizados automaticamente.

**Perceba que essa funcionalidade pode ser extremamente útil para
tarefas como atualização de bases de dados e dashboards.**

1.  Para que que o GitHub Actions funcione, nós deveremos criar, no
    repositório, uma pasta denominada *.github*. Dentro dessa pasta,
    cria-se outra pasta chamada *workflows.* É nesse endereço que o
    GitHub Actions vai procurar o arquivo .yaml, o qual conterá as
    instruções das ações que devem ser automatizadas.

2.  Exemplos de como o arquivo .yaml deve ser formatado pode ser
    encontrado [nesse
    repositório](https://github.com/r-lib/actions/tree/old/examples#readme).
    Em nosso caso, vamos utilizar como base [este
    documento](https://github.com/r-lib/actions/tree/old/examples#render-rmarkdown).

#### Criando o arquivo .yaml

Com base nos links apresentados acima, construímos o seguinte código
para ser incluído na pasta *workflows*:

    # Workflow derived from https://github.com/r-lib/actions/tree/master/examples
    # Need help debugging build failures? Start at https://github.com/r-lib/actions#where-to-find-help
    on:
      push:
        branches: master

    name: render-quarto

    jobs:
      render-quarto:
        runs-on: ubuntu-latest
        env:
          GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
        permissions:
            contents: write
        steps:
          - name: Checkout repo
            uses: actions/checkout@v4
            with:
              fetch-depth: 0
          - name: Setup pandoc
            uses: r-lib/actions/setup-pandoc@v2
          - name: Setup R
            uses: r-lib/actions/setup-r@v2
          - name: Setup Renv
            uses: r-lib/actions/setup-renv@v2
          - name: Install packages
            uses: r-lib/actions/setup-r-dependencies@v2
            with:
              packages: |
                any::quarto
          - name: Render quarto files
            run: |
              Rscript -e 'quarto::quarto_render("README.qmd")'

          - name: Commit results
            run: |
              git config --local user.name "$GITHUB_ACTOR"
              git config --local user.email "$GITHUB_ACTOR@users.noreply.github.com"
              git add .
              git commit -m "Hora atualizada" || echo "No changes to commit"
              git push origin master || echo "No changes to commit"

Agora já temos todos nossos arquivos configurados e o próximo passo é
realizar o commit e push na brain master do repositório e pode-se
verificar que o arquivo README da página inicial do GitHub foi
atualizado.
