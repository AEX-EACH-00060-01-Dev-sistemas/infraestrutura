# Módulo 2: Amazon ECR e Amazon ECS com Fargate

Neste módulo, substituiremos o build feito dentro da EC2 por imagens Docker versionadas no Amazon Elastic Container Registry (ECR). Depois, executaremos frontend e API como tasks do Amazon Elastic Container Service (ECS) com AWS Fargate.

Não será necessário alterar o código do projeto. O build utilizará os Dockerfiles já existentes, e as configurações específicas do ambiente serão informadas na task definition do ECS.

O banco continuará sendo o contêiner PostgreSQL do laboratório. Essa escolha mantém o módulo independente e prepara a migração do banco para o Amazon RDS no módulo 4.

## Objetivos

Neste módulo, será possível aprender a:

- construir as imagens da API e do frontend;
- criar repositórios privados no ECR;
- publicar imagens com tags;
- criar um cluster ECS;
- registrar uma task definition;
- executar uma task Fargate em uma subnet pública;
- validar o health check da API.

## 1. Construção local

Na máquina local, clone o projeto e defina variáveis para a região e para a conta:

```bash
git clone https://github.com/contygor/forms-simulator.git
cd forms-simulator
export AWS_REGION=sa-east-1
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```

Crie dois repositórios privados no ECR, um para cada imagem:

```bash
aws ecr create-repository --repository-name forms-api --region "$AWS_REGION"
aws ecr create-repository --repository-name forms-frontend --region "$AWS_REGION"
```

Se os repositórios já existirem, continue sem repetir a criação.

## 2. Login e publicação no ECR

Autentique o Docker e construa as imagens:

```bash
aws ecr get-login-password --region "$AWS_REGION" | docker login --username AWS --password-stdin "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"
docker build -t forms-api:1 .
docker build -t forms-frontend:1 ./frontend
```

Depois, crie tags com o endereço dos repositórios e faça o push:

```bash
docker tag forms-api:1 "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/forms-api:1"
docker tag forms-frontend:1 "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/forms-frontend:1"
docker push "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/forms-api:1"
docker push "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/forms-frontend:1"
```

Recomendamos evitar o uso exclusivo da tag `latest` durante as aulas. Tags como `1`, `2` ou um hash do commit tornam possível identificar e reverter versões.

## 3. Cluster ECS

No console do ECS, crie um cluster compatível com workloads Fargate. Em seguida, crie uma task definition compatível com **Fargate** e Linux.

Para a primeira execução, configure:

- CPU e memória compatíveis com o nível gratuito disponível;
- task role somente quando a aplicação precisar acessar outros serviços;
- execution role com `AmazonECSTaskExecutionRolePolicy` para baixar imagens e enviar logs;
- um contêiner `forms-api` usando a imagem do ECR;
- um contêiner `forms-frontend` usando a imagem do ECR;
- porta `8080` para a API e porta `80` para o frontend;
- logs em grupos separados no CloudWatch Logs.

Observe que a task definition não deve conter senhas reais no Git. Para este módulo introdutório, podem ser usados valores de laboratório na task definition, mas recomendamos migrá-los para o Secrets Manager antes de um ambiente real.

## 4. Primeira execução pública

Crie um serviço ECS ou execute uma task com:

- launch type **Fargate**;
- uma subnet pública da VPC padrão;
- atribuição de IP público habilitada;
- Security Group permitindo HTTP somente na porta escolhida para o teste;
- sem exposição da porta `5432`.

Para uma primeira demonstração, execute frontend e API separadamente e teste a API diretamente pelo IP público da task. Em uma implantação utilizável, coloque um Application Load Balancer diante dos serviços, porque o IP de uma task pode mudar.

Valide o funcionamento com:

```text
GET http://IP_DA_TASK:8080/health
```

Consulte também **ECS > Tasks > Logs** para investigar falhas de inicialização.

## 5. Limpeza

Depois do teste:

1. reduzir o serviço para zero tasks;
2. apagar serviços e task definitions que não serão reutilizados;
3. apagar o cluster quando ele não for mais necessário;
4. apagar imagens antigas e repositórios ECR se o laboratório for encerrado;
5. revisar grupos de logs, Elastic IPs e Load Balancers.

## Resultado

Ao final, teremos percorrido o ciclo de uma imagem Docker na AWS: build local, push para ECR e execução no ECS. No próximo módulo, construiremos uma VPC própria para separar componentes públicos e privados e colocaremos o ECS atrás de um Application Load Balancer.
