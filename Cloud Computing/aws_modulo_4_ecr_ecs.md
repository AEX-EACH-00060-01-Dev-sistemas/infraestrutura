# Módulo 4: Amazon ECR e Amazon ECS com Fargate

Neste módulo, nós publicaremos as imagens Docker no Amazon Elastic Container Registry (ECR) e executaremos o frontend e a API no Amazon Elastic Container Service (ECS) com AWS Fargate. Para contextualizar: o ECR funciona como um cofre onde guardamos as nossas imagens Docker de forma segura; o ECS atua como o maestro que gerencia os nossos contêineres; e o Fargate é o motor que roda tudo isso sem precisarmos administrar servidores virtuais.

A nossa VPC, as subnets, os grupos de segurança e o RDS PostgreSQL já foram preparados nos módulos anteriores. Portanto, a API será configurada para usar o endpoint privado do RDS em vez de um banco em contêiner.

## Objetivos

Ao terminarmos este módulo, nós conseguiremos:

* criar repositórios privados no ECR;
* publicar imagens Docker com tags;
* criar um cluster ECS;
* registrar task definitions para a API e para o frontend;
* conectar a API ao RDS usando o Secrets Manager;
* executar os serviços em subnets privadas atrás de um ALB.

## 1. Preparação do ambiente local

Instale o Docker Desktop e a AWS CLI na nossa máquina local antes de continuarmos. Vamos clonar o projeto executando o código abaixo.

```bash
git clone [https://github.com/contygor/forms-simulator.git](https://github.com/contygor/forms-simulator.git)
cd forms-simulator
```

## 2. Criação dos repositórios no ECR

No console da AWS, abra **Amazon ECR > Repositories > Create repository**.

Crie um repositório privado chamado `forms-api` e clique em **Create repository**. Repita o processo para criarmos outro repositório privado chamado `forms-frontend`.

Abra cada repositório e clique em **View push commands**. A AWS exibirá os comandos exatos que precisaremos para autenticar o Docker, construir a imagem, adicionar a tag e fazer o push.

## 3. Construção e publicação das imagens

No terminal, dentro da pasta do projeto, nós executaremos os comandos mostrados na tela para cada repositório. O fluxo da API será semelhante aos comandos a seguir.

```bash
aws ecr get-login-password --region <REGIAO> | docker login --username AWS --password-stdin <REGISTRO_ECR>
docker build -t forms-api:1 .
docker tag forms-api:1 <URI_DO_REPOSITORIO_FORMS_API>:1
docker push <URI_DO_REPOSITORIO_FORMS_API>:1
```

Nós repetiremos o procedimento para o `forms-frontend`, garantindo o uso do Dockerfile correto (localizado na pasta `frontend`) e o URI do repositório correspondente. Use tags claras, como `1`, `2` ou o hash do commit.

## 4. Criação do cluster ECS

Acesse **Amazon ECS > Clusters > Create cluster**. Escolha a opção compatível com o **AWS Fargate**, informe um nome como `forms-cluster` e crie o cluster.

Em seguida, vá até **Amazon ECS > Task definitions > Create new task definition**. Selecione **Fargate** e o sistema operacional **Linux**. Nós criaremos uma definição de tarefa (task definition) para a API e outra para o frontend.

## 5. Configuração da task definition da API

Na task definition da API, nós configuraremos os parâmetros abaixo.

* cpu e memória: valores compatíveis com o nosso laboratório;
* perfil de execução: usar a política base AmazonECSTaskExecutionRolePolicy;
* contêiner: usar a imagem do `forms-api` no ECR;
* porta: definir como 8080;
* logs: habilitados no CloudWatch Logs;
* rede: grupo de segurança da API;
* variáveis: aplicar Database__ApplyMigrations=true (apenas na primeira execução para criar as tabelas).

Na seção de segredos ou variáveis de ambiente, nós configuraremos a conexão com o endpoint do RDS. É uma boa prática injetarmos as credenciais usando o segredo que criamos no Secrets Manager, sem expor a senha diretamente no texto.

## 6. Configuração da task definition do frontend

Na task definition do frontend, nós definiremos os parâmetros a seguir.

* contêiner: apontar para a imagem do `forms-frontend` no ECR;
* porta: definir como 80;
* logs: ativados no CloudWatch Logs;
* rede: usar o grupo de segurança do frontend;
* variáveis de ambiente: preencher FRONTEND_API_URL, que nós configuraremos assim que criarmos o ALB.

## 7. Criação do Application Load Balancer

Abra **EC2 > Target Groups > Create target group** e crie dois target groups escolhendo **IP** como tipo de destino (uma exigência técnica do Fargate).

* frontend: usar a porta 80;
* api: usar a porta 8080, com health check apontando para a rota `/health`.

Agora, acesse **EC2 > Load Balancers > Create Load Balancer > Application Load Balancer**. Nós configuraremos o ALB com as propriedades abaixo.

* esquema: internet facing (voltado para a internet);
* rede: selecionar as duas subnets públicas da nossa `forms-vpc`;
* segurança: selecionar o grupo de segurança do ALB;
* listener: configurar para requisições HTTP na porta 80.

Copie o DNS do ALB criado. Volte à task definition do frontend e atualize a variável `FRONTEND_API_URL` com esse endereço. Na task definition da API, adicione a origem pública do frontend na variável `Cors__AllowedOrigins__0`. Salve as novas revisões de ambas as tarefas.

## 8. Criação dos serviços no ECS

Dentro do nosso cluster ECS, clique em **Create service** para a API e depois repita o processo para o frontend. Para cada serviço, nós ajustaremos as seguintes opções.

* launch type: selecionar fargate;
* rede: alocar nas subnets privadas da `forms-vpc`;
* ip público: manter desabilitado;
* segurança: vincular ao grupo de segurança correspondente;
* balanceamento: apontar para o target group correspondente do ALB;
* quantidade: preencher com o número desejado de tasks rodando.

*Atenção técnica importante:* como as tasks ficarão em subnets privadas sem IP público, o ECS precisará de um NAT Gateway ou VPC Endpoints para conseguir baixar as imagens do ECR e enviar os logs para o CloudWatch. Sem isso, as tasks falharão ao iniciar.

## 9. Validação do ambiente

No console, nós verificaremos as etapas abaixo.

* tasks: devem estar com o status em execução (Running);
* target groups: devem marcar os alvos como saudáveis (healthy);
* logs: não devem apresentar falhas de conexão com o banco;
* acesso web: o DNS do ALB deve carregar o frontend corretamente;
* integridade: a rota `/health` da API deve responder com o status de sucesso;
* usabilidade: o cadastro e a consulta devem funcionar perfeitamente pela interface web;
* segurança: o nosso RDS deve continuar protegido, sem endereço público.

## 10. Limpeza e encerramento

Após o teste, para evitarmos custos desnecessários, nós seguiremos estes passos.

1. edite os serviços e reduza as tasks para zero;
2. exclua os serviços e as revisões que não usaremos mais;
3. delete o cluster se ele não for mais necessário;
4. remova as imagens e os repositórios do ECR ao fim do laboratório;
5. revise e apague os grupos de logs, o ALB, o NAT Gateway e os Elastic IPs soltos.

## Resultado

Ao final, o nosso frontend e a nossa API estarão executando de forma escalável no ECS Fargate, puxando imagens do ECR, rodando em uma VPC segmentada e conectados ao PostgreSQL gerenciado. No próximo módulo, nós usaremos o S3 para objetos e backups.