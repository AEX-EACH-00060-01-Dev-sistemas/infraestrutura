# AWS

Amazon Web Services (AWS) é uma plataforma de computação em nuvem oferecida pela Amazon
que oferece uma gama de diferentes serviços, alguns deles sendo de infraestrutura,
permitindo seus usuários definirem para uma aplicação própria como ela deve rodar.

Um desses serviços oferecidos é o chamado EC2 (Elastic Compute Cloud), um serviço que
possibilita o upload de ambietes containerizados virtuais, como o Docker.

Assim, veremos aqui como colocar um container Docker no ar utilizando a ferramenta EC2
disponível no site da AWS.

# Criando o container

Para esse projeto, vamos precisar de uma imagem Docker para subir para o EC2, aqui usaremos
a imagem utilizada na [trilha de Docker](./Docker.md), mas você pode baixar/buildar uma
própria, para checar as imagens que você possui, utilize o seguinte comando:

```bash
$ docker images
```

Esse comando lista as imagens baixadas que estão disponíveis para serem usadas. Nesse caso,
o output após testar o projeto web de python feito na outra trilha é o seguinte:

```
IMAGE              ID             DISK USAGE   CONTENT SIZE   EXTRA
mariadb:11.2       ff87d49107a1        554MB          127MB
python:latest      09b29c360b84       1.63GB          432MB
```

# Criando um repositório para a imagem

Agora, é necessário uma conta no [site do docker](https://app.docker.com/signup).
Ao entrar no hub, dirija-se para a [criação de um novo repositório](https://hub.docker.com/repository/create?)
que será utilizado como base para as imagens do seu arquivo compose.

Após criar sua conta, faça login via terminal para poder se conectar com o site:

```bash
$ docker login
```

siga as instruções de login e após concluído, suba as imagens com o comando:

```bash
$ docker tag ID_IMAGEM SEU_USUARIO/NOME_REPOSITORIO
```

no caso das imagens utilizadas no exemplo criado, fica assim:

```bash
$ docker tag ff87d49107a1 meu_usuario/repo_py
$ docker tag 09b29c360b84 meu_usuario/repo_db
```

agora só temos que colocar as mudanças feitas no repositório remoto:

```bash
$ docker push meu_usuario/repo_py:latest
$ docker push meu_usuario/repo_db:latest
```

Esses repositórios estarão visíveis na sua página dos [seus repositórios do Docker Hub](https://hub.docker.com/repositories/)

Temos então que fazer algumas alterações no nosso arquivo compose.yml:
a linha que especifíca a imagem utilizada em cada serviço apontará agora para
o repositório que você criou, e incluiremos uma linha nova para o argumento build

```yaml
build: .
image: ID_DOCKER_HUB/NOME_REPOSITORIO:latest
```

Para as duas imagens, os serviços vão ficar resumidamente assim:

```yaml
services:
  database:
    build: .
    image: username/repo_db:latest

  web_app:
    build: .
    image: username/repo_py:latest
```

Agora, sempre que uma mudança ocorrer na imagem, seja uma alteração local ou remota,
você pode receber/enviar essas alterações utilizando o arquivo compose:

```bash
$ docker compose pull # Para pegar as mudanças feitas remotamente
$ docker compose push # Para enviar as mudanças feitas localmente
```

Isso é especialmente útil para imagens construídas por você mesmo, menos útil
para imagens estáticas baixadas, como o caso da imagem do python.

# AWS

Crie uma conta na [AWS](https://signin.aws.amazon.com/signup?request_type=register),
navegue para o serviço EC2 e lance uma nova instância, o que executará uma máquina virtual.
Para escolha do sistema operacional, é indicado um leve e também simples para usar docker,
assim, recomenda-se o uso de uma instância Ubuntu. Configure agora para liberar tráfego HTTP e HTTPS
caso seu serviço seja baseado na web, como o aplicativo exemplo criado.

Agora, conecte-se a sua instância, instale o docker `$ sudo snap install docker`
e clone o projeto, rode o docker com o compose do projeto e pronto, seu projeto estará
acessível a partir do ip público dado a sua instância na porta mapeada.
