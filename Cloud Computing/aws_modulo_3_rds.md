# Módulo 3: Amazon RDS PostgreSQL

Neste módulo, nós retiraremos o PostgreSQL do Docker Compose e passaremos a utilizar o Amazon Relational Database Service (RDS). O banco de dados será criado como um serviço totalmente gerenciado dentro das subnets privadas da VPC que montamos no módulo anterior.

A nossa API continuará sendo executada no ECS, mas agora acessará o endpoint privado desse RDS.

## Objetivos

Ao terminarmos esta etapa, conseguiremos:

* criar um DB subnet group privado;
* provisionar uma instância PostgreSQL sem IP público;
* liberar o acesso exclusivamente para a nossa API;
* armazenar as credenciais de forma segura no AWS Secrets Manager;
* preparar a conexão da aplicação com o novo banco de dados;
* entender o funcionamento de backups, disponibilidade e custos desse modelo gerenciado.

## 1. Preparação da rede

No console da AWS, acesse **RDS > Subnet groups > Create DB subnet group**. Informe um nome, escolha a nossa rede `forms-vpc` e selecione as duas subnets privadas que estão em Availability Zones diferentes.

Na tela da VPC, confirme se a regra do Security Group do banco está correta: ela deve permitir TCP `5432` apontando somente para o Security Group da API. Não libere a porta `5432` para a internet.

## 2. Criação do banco de dados

Acesse **RDS > Databases > Create database** e configure:

* engine: postgreSQL;
* template: dev/Test ou Free tier, quando disponível;
* identificador: `forms-postgres`;
* database name: `formdb`;
* master username: o usuário administrativo dedicado ao laboratório;
* master password: a senha que armazenaremos no Secrets Manager;
* virtual private cloud: `forms-vpc`;
* subnet group: o grupo privado que acabamos de criar;
* public access: no;
* security group: o grupo de segurança específico do RDS;
* encryption: ativada;
* backups: configurados conforme a necessidade;
* deletion protection: desabilitada (apenas para facilitar a limpeza do laboratório).

Clique em **Create database** e aguarde o status mudar para `Available`. Em seguida, abra os detalhes da instância e copie o endpoint DNS. Esse endpoint substituirá o nome do host da conexão, já que ele não é um IP fixo.

## 3. Armazenamento seguro das credenciais

Acesse **Secrets Manager > Store a new secret**. Escolha a opção **Credentials for RDS database**, selecione a nossa nova instância, defina um nome como `forms/rds` e clique em **Store**.

Esse segredo armazenará a nossa senha do banco, evitando que valores sensíveis fiquem expostos no Git. Mais tarde, a permissão da API será ajustada no IAM para incluir `secretsmanager:GetSecretValue`, garantindo que a aplicação consiga ler esses dados.

## 4. Validação da infraestrutura

Verifique no console:

* o status do RDS está como `Available`;
* o acesso público está desativado;
* o DB subnet group realmente utiliza subnets privadas em duas Availability Zones;
* o Security Group permite entrada na porta `5432` apenas para a API;
* o segredo foi gerado corretamente no Secrets Manager;
* o backup automático está ativo.

No próximo módulo, usaremos esse endpoint DNS no lugar da palavra `postgres` que usávamos no Docker Compose.

## 5. Limpeza e encerramento

Observe no painel a diferença entre backup automático, snapshot manual e opções Multi-AZ. Para este laboratório, uma instância pequena em uma única AZ é suficiente, embora não ofereça a mesma resiliência de um ambiente de produção.

Para encerrar os recursos e evitar custos:

1. interrompa os serviços que estão utilizando o banco;
2. crie um snapshot final apenas se for necessário salvar os dados;
3. exclua a instância RDS diretamente pelo console;
4. apague os backups e snapshots que ficarem retidos;
5. exclua o segredo criado no Secrets Manager;
6. remova o Security Group e o DB subnet group.

## Resultado

Agora temos um banco PostgreSQL totalmente gerenciado, privado e seguro, aguardando as conexões da nossa aplicação. No próximo módulo, construiremos as imagens no ECR e publicaremos o frontend e a API usando o ECS e o Fargate.