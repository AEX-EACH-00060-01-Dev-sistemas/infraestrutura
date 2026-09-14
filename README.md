# Infraestrutura

Este repositório foi produzido como parte da atividade extensionista AEX-EACH-00060-01-Dev-sistemas promovida pela Universidade de São Paulo [USP]
e é dedicado a ensinar ao leitor sobre as principais ferramentas e serviços disponíveis para criação de uma infraestrutura utilizada
no desenvolvimento de software.

## Objetivo

Para o desenvolvimento de um projeto robusto e capaz de ser levado ao mundo real, é necessário uma infraestrutura também robusta. Assim,
esse repositório visa tornar uma pessoa capacitada na criação de um ambiente de desenvolvimento agnóstico que possibilite a criação de um
projeto com os conhecimentos obtidos [nessa atividade extensionista](https://github.com/AEX-EACH-00060-01-Dev-sistemas/).

## Conteúdos Disponíveis

Apresentaremos aqui trilhas de [Linux](./Linux/Linux.md), [Docker](./Docker/Docker.md) e [AWS](./Cloud%20Computing/Fundamentos.md). A trilha de AWS utilizará o projeto [forms-simulator](https://github.com/contygor/forms-simulator) como laboratório contínuo, permitindo comparar diferentes modelos de implantação da mesma aplicação.

## Regra do laboratório AWS

As atividades da trilha AWS foram planejadas para não exigir alterações no código do `forms-simulator`. Utilize as imagens e os arquivos do repositório como estão e faça as adaptações de cada ambiente por meio de variáveis de ambiente, task definitions, IAM roles, Secrets Manager, Security Groups e demais recursos da AWS.

Os comandos de build, push e publicação fazem parte da operação da infraestrutura e não significam edição do projeto. Quando uma configuração depender do ambiente, ela deverá ser informada na EC2, no ECS ou no serviço AWS correspondente.

## Rota Sugerida

Recomendamos estudar Linux, depois Docker e então seguir a trilha AWS nesta ordem:

1. [Fundamentos de computação em nuvem](./Cloud%20Computing/Fundamentos.md)
2. [EC2 e EBS: publicação com Docker Compose](./Cloud%20Computing/aws_modulo_1_ec2_ebs.md)
3. [ECR e ECS/Fargate: imagens e contêineres](./Cloud%20Computing/aws_modulo_2_ecr_ecs.md)
4. [VPC e Security Groups: isolamento de rede](./Cloud%20Computing/aws_modulo_3_vpc.md)
5. [RDS PostgreSQL: banco gerenciado](./Cloud%20Computing/aws_modulo_4_rds.md)
6. [S3: armazenamento de objetos e backups](./Cloud%20Computing/aws_modulo_5_s3.md)

Recomendamos encerrar os recursos ao fim de cada módulo para evitar cobranças desnecessárias. A ordem começa com a experiência mais concreta e evolui até uma arquitetura distribuída com rede segmentada, banco gerenciado e armazenamento de objetos.
