# Heroku buildpack: PHP

---
# Sobre

Este repositório foi criado a partir do PHP Heroku buildpack - https://github.com/heroku/heroku-buildpack-php.git

O squad possui uma aplicação Python 5 legada, mas importante, que precisa ser mantida no ar. Como o repositório oficial não oferecerá mais suporte ao Python 5 (em breve pode até ser descontinuado), foi criado este repo.

Os passos seguidos para a criação deste repo foram os seguintes:


1. Git clone e download do projeto original:
   - `git clone https://github.com/heroku/heroku-buildpack-php.git`


2. Criação de virtualenv com py2, necessário para execução dos scripts de cópia dos arquivos do *bucket* S3 original: 
   - `pyenv install 2.7.18`
   - `pyenv virtualenv 2.7.18 buildpack`
   - `pyenv activate buildpack`


3. Instação das dependencias:
   - `cd heroku heroku-buildpack-php`
   - `pip install -r requirements.txt`


4. Configuração do `AWSAccessKeyId` e `AWSSecretKey` do bucket destino, para ser possível usar o utilitário `s3cmd`:
   - `s3cmd --configure` *(informar a AWSAccessKeyId e AWSSecretKey (access-key obtida a partir da conta AWS: https://console.aws.amazon.com/iam/home?#/security_credentials)*


5. Execução dos scripts de cópia dos arquivos (sync, mkrepo) e enviar os arquivos `package.json` ajustados para as distribuições `heroku-16` e `heroku-18`:
   - `cd support/build/_util/`
   - `./sync.sh "magalu-php-buildpack" "dist-heroku-16-stable/" "s3" "lang-php" "dist-heroku-16-stable/"`
   - `./mkrepo.sh "magalu-php-buildpack" "dist-heroku-16-stable/"`
   - `s3cmd --ssl --acl-public -m application/json put packages.json s3://magalu-php-buildpack/dist-heroku-16-stable/packages.json`
   - `./sync.sh "magalu-php-buildpack" "dist-heroku-18-stable/" "s3" "lang-php" "dist-heroku-18-stable/"`
   - `./mkrepo.sh "magalu-php-buildpack" "dist-heroku-18-stable/"`
   - `s3cmd --ssl --acl-public -m application/json put packages.json s3://magalu-php-buildpack/dist-heroku-18-stable/packages.json`


6) Os arquivos das bibliotecas foram copiados para o *bucket* S3 `magalu-php-buildpack` da conta AWS da empresa, e disponibilizados para leitura pública em `https://magalu-php-buildpack.s3.amazonaws.com/` 


7) Até então, o git-clone do repositório somente serviu para a execução dos scripts e geração do novo bucket com a cópia das `libs`; 
   é necessário, então, clonar o repositório original para um novo repositório, e modificar os arquivos para referenciarem as *libs* no novo bucket - pode ser feito no próprio github; 
   em relação ao repo original, os arquivos alterados foram:
   - `bin/compile`
   - `README.md`
   
, e foram removidos os diretórios e arquivos:
   - `/support/_conf`
   - `/support/_docker`
   - `/support/_util`
   - `/test/*`
   - `/.github/*`
   - `requirements.txt`
   - `.travis.yml`


8) Também foi copiado para o diretório *bin/util* deste repositório o script `stdlib.sh` que, no buildpack original, é executado a partir de um local remoto no arquivo `bin/compile`;


9) Após ajustes e testes, o repositório foi clonado para o gilab do Magalu;


10) O uso do buildpack no projeto se dá informando o caminho do próprio repositório: <*caminho do repo aqui*>


Para referência sobre a api de buildpacks: https://devcenter.heroku.com/articles/buildpack-api

---

# Original README

This buildpack was created from the [Heroku buildpack - https://github.com/heroku/heroku-buildpack-php.git](https://github.com/heroku/heroku-buildpack-php) repo for PHP applications

It uses Composer for dependency management, supports PHP or HHVM (experimental) as runtimes, and offers a choice of Apache2 or Nginx web servers.

## Usage

You'll need to use at least an empty `composer.json` in your application.

    $ echo '{}' > composer.json
    $ git add composer.json
    $ git commit -m "add composer.json for PHP app detection"

If you also have files from other frameworks or languages that could trigger another buildpack to detect your application as one of its own, e.g. a `package.json` which might cause your code to be detected as a Node.js application even if it is a PHP application, then you need to manually set your application to use this buildpack:

    $ heroku buildpacks:set heroku/php

This will use the officially published version. To use the `master` branch from GitHub instead:

    $ heroku buildpacks:set https://github.com/heroku/heroku-buildpack-php

Please refer to [Dev Center](https://devcenter.heroku.com/categories/php) for further usage instructions.

## Custom Platform Repositories

The buildpack uses Composer repositories to resolve platform (`php`, `hhvm`, `ext-something`, ...) dependencies.

To use a custom Composer repository with additional or different platform packages, add the URL to its `packages.json` to the `HEROKU_PHP_PLATFORM_REPOSITORIES` config var:

    $ heroku config:set HEROKU_PHP_PLATFORM_REPOSITORIES="https://mybucket.s3.amazonaws.com/cedar-14/packages.json"

To allow the use of multiple custom repositories, the config var may hold a list of multiple repository URLs, separated by a space character, in ascending order of precedence.

If the first entry in the list is "`-`" instead of a URL, the default platform repository is disabled entirely. This can be useful when testing development repositories, or to forcefully prevent the use of unwanted packages from the default platform repository.

For instructions on how to build custom platform packages (and a repository to hold them), please refer to the instructions [further below](#custom-platform-packages-and-repositories).

**Please note that Heroku cannot provide support for issues related to custom platform repositories and packages.**

## Development

The following information only applies if you're forking and hacking on this buildpack for your own purposes.

### Pull Requests

Please submit all pull requests against `develop` as the base branch.

### Custom Platform Packages and Repositories

Please refer to the [README in `support/build/`](support/build/README.md) for instructions.

