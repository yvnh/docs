---
title: Configurar o Docker para usar com o GitHub Packages
intro: 'Você pode configurar o cliente do Docker para usar o {% data variables.product.prodname_registry %} para publicar e recuperar imagens do Docker.'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /articles/configuring-docker-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-docker-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-docker-for-use-with-github-packages
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
---

{% data reusables.package_registry.packages-ghes-release-stage %}

**Note:** When installing or publishing a docker image, {% data variables.product.prodname_registry %} does not currently support foreign layers, such as Windows images.

### Autenticar-se no {% data variables.product.prodname_registry %}

{% warning %}

Using the Docker image ID, tag the docker image, replacing *OWNER* with the name of the user or organization account that owns the repository, *REPOSITORY* with the name of the repository containing your project, *IMAGE_NAME* with name of the package or image, and *VERSION* with package version at build time.

{% endwarning %}

Ao instalar ou publicar uma imagem do docker, o {% data variables.product.prodname_registry %} não é compatível camadas externas, como imagens do Windows.

{% if currentVersion != "free-pro-team@latest" %}

Antes de poder usar o registro do Docker no {% data variables.product.prodname_registry %}, o administrador do site para {% data variables.product.product_location_enterprise %} deve habilitar o suporte do Docker e o isolamento do subdomínio para a sua instância. Para obter mais informações, consulte "[Gerenciar pacotes do GitHub para a sua empresa](/enterprise/admin/packages)".

{% endif %}

### Autenticar-se no {% data variables.product.prodname_registry %}

{% data reusables.package_registry.authenticate-packages %}

#### Efetuando a autenticação com um token de acesso pessoal

{% data reusables.package_registry.required-scopes %}

Você pode efetuar a autenticação no {% data variables.product.prodname_registry %} usando o comando de login do `docker`.

Para manter suas credenciais seguras, recomendamos que você salve seu token de acesso pessoal em um arquivo local no seu computador e use o sinalizador `--password-stdin` do Docker que lê o seu token a partir de um arquivo local.

{% if currentVersion == "free-pro-team@latest" %}
{% raw %}
  ```shell
  $ cat <em>~/TOKEN.txt</em> | docker login https://docker.pkg.github.com -u <em>USERNAME</em> --password-stdin
  ```
{% endraw %}
{% endif %}

{% if currentVersion != "free-pro-team@latest" %}
{% raw %}
 ```shell
 $ cat <em>~/TOKEN.txt</em> | docker login docker.HOSTNAME -u <em>USERNAME</em> --password-stdin
```
{% endraw %}
{% endif %}

To use this example login command, replace `USERNAME` with your {% data variables.product.prodname_dotcom %} username and `~/TOKEN.txt` with the file path to your personal access token for {% data variables.product.prodname_dotcom %}.

Para obter mais informações, consulte "[Login do Docker](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin)".

#### Efetuando a autenticação com o `GITHUB_TOKEN`

{% data reusables.package_registry.package-registry-with-github-tokens %}

### Publicar um pacote

O {% data variables.product.prodname_registry %} aceita várias imagens do Docker de nível superior por repositório. Um repositório pode ter qualquer número de tags de imagem. Você poderá conhecer uma publicação de serviço degradada ou instalar imagens do Docker com tamanho superior a 10 GB. As camadas são limitadas em 5 GB cada. Para obter mais informações, consulte "[Tag do Docker](https://docs.docker.com/engine/reference/commandline/tag/)" na documentação Docker.

{% data reusables.package_registry.lowercase-name-field %}

{% data reusables.package_registry.viewing-packages %}

1. Determine o nome da imagem e o ID da sua imagem do docker usando `imagens do docker`.
  ```shell
  $ docker images
  > <&nbsp>
  > REPOSITORY        TAG        IMAGE ID       CREATED      SIZE
  > <em>IMAGE_NAME</em>        <em>VERSION</em>    <em>IMAGE_ID</em>       4 weeks ago  1.11MB
  ```
2. If you haven't already built a docker image for the package, build the image, replacing *OWNER* with the name of the user or organization account that owns the repository, *REPOSITORY* with the name of the repository containing your project, *IMAGE_NAME* with name of the package or image, *VERSION* with package version at build time, and *PATH* to the image if it isn't in the current working directory.
{% if currentVersion != "free-pro-team@latest" %} *HOSTNAME* com o nome de host de {% data variables.product.product_location_enterprise %},{% endif %} e *VERSÃO* com a versão do pacote no momento da criação.
  {% if currentVersion == "free-pro-team@latest" %}
  ```shell
  $ docker tag <em>IMAGE_ID</em> docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% else %}
  ```shell
  $ docker tag <em>IMAGE_ID</em> docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% endif %}
3. Se você ainda não criou uma imagem do docker para o pacote, crie a imagem, substituindo *PROPRIETÁRIO* pelo nome do usuário ou conta de organização proprietária do repositório, *REPOSITÓRIO* pelo o nome do repositório que contém o seu projeto, *IMAGE_NAME* pelo o nome do pacote ou imagem, *VERSÃO* com a versão do pacote no momento da criação,
{% if currentVersion != "free-pro-team@latest" %} *HOSTNAME* pelo o nome de host de {% data variables.product.product_location_enterprise %},{% endif %} e *PATH* para a imagem se não estiver no diretório de trabalho atual.
  {% if currentVersion == "free-pro-team@latest" %}
  ```shell
  $ docker build -t docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% else %}
  ```shell
  $ docker build -t docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% endif %}
4. Publicar a imagem no {% data variables.product.prodname_registry %}.
  {% if currentVersion == "free-pro-team@latest" %}
  ```shell
  $ docker push docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% else %}
  ```shell
  $ docker push docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% endif %}
  {% note %}

  **Observação:** Você deve enviar sua imagem usando `IMAGE_NAME:VERSION` e não `IMAGE_NAME:SHA`.

  {% endnote %}

#### Exemplo de publicação de uma imagem do Docker

Você pode publicar a versão 1.0 da imagem de `monalisa` para o repositório `octocat/octo-app` usando um ID de imagem.

{% if currentVersion == "free-pro-team@latest" %}
```shell
$ docker images

> REPOSITORY           TAG      IMAGE ID      CREATED      SIZE
> monalisa             1.0      c75bebcdd211  4 weeks ago  1.11MB

# Tag the image with <em>OWNER/REPO/IMAGE_NAME</em>
$ docker tag c75bebcdd211 docker.pkg.github.com/octocat/octo-app/monalisa:1.0

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0
```

{% else %}

```shell
$ docker images

> REPOSITORY           TAG      IMAGE ID      CREATED      SIZE
> monalisa             1.0      c75bebcdd211  4 weeks ago  1.11MB

# Tag the image with <em>OWNER/REPO/IMAGE_NAME</em>
$ docker tag c75bebcdd211 docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0
```

{% endif %}

Você pode publicar uma nova imagem do Docker pela primeira vez e nomeá-la como `monalisa`.

{% if currentVersion == "free-pro-team@latest" %}
```shell
# Build the image with docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
# Assumes Dockerfile resides in the current working directory (.)
$ docker build -t docker.pkg.github.com/octocat/octo-app/monalisa:1.0 .
$ docker build -t docker.pkg.github.com/octocat/octo-app/monalisa:1.0 .

# Faça push da imagem no {% data variables.product.prodname_registry %}
$ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0
```

{% else %}
```shell
# Build the image with docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
# Assumes Dockerfile resides in the current working directory (.)
$ docker build -t docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0 .

# Push the image to {% data variables.product.prodname_registry %}
$ docker push docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0
```
{% endif %}

### Instalar um pacote

You can use the `docker pull` command to install a docker image from {% data variables.product.prodname_registry %}, replacing *OWNER* with the name of the user or organization account that owns the repository, *REPOSITORY* with the name of the repository containing your project, *IMAGE_NAME* with name of the package or image, *TAG_NAME* with tag for the image you want to install. {% data reusables.package_registry.lowercase-name-field %}


{% if currentVersion == "free-pro-team@latest" %}
```shell
$ docker pull docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% else %}
```shell
$ docker pull docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% endif %}

{% note %}

**Nota:** Você deve fazer pull da imagem usando `IMAGE_NAME:VERSION` e não usar `IMAGE_NAME:SHA`.

{% endnote %}


### Leia mais

- "[Excluir um pacote](/packages/publishing-and-managing-packages/deleting-a-package/)"
