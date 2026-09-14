# Módulo 4: Amazon RDS PostgreSQL

Neste módulo, retiraremos o PostgreSQL do Docker Compose e usaremos o Amazon Relational Database Service (RDS). A API continuará no ECS, mas o banco passará a ser um serviço gerenciado.

Não será necessário modificar o código da API. O ASP.NET Core já lê a conexão do banco e as demais configurações por variáveis de ambiente fornecidas pelo ECS.

## Objetivos

Neste módulo, será possível aprender a:

- criar uma subnet group privada para o RDS;
- criar uma instância PostgreSQL sem IP público;
- permitir acesso somente a partir da API;
- guardar credenciais fora do código;
- apontar as migrations da aplicação para o RDS;
- distinguir disponibilidade, backup e alta disponibilidade.

## 1. Preparar a rede

Confirme que a VPC possui subnets privadas em pelo menos duas Availability Zones. No RDS, crie um **DB subnet group** usando essas subnets.

O Security Group do banco deve permitir TCP `5432` somente a partir do Security Group da API. Não libere `5432` para a internet nem para o próprio endereço residencial.

## 2. Criar o banco

No console do RDS, escolha **Create database** e configure:

- engine: PostgreSQL;
- template: Dev/Test ou Free tier, quando disponível para a conta;
- identificador: `forms-postgres`;
- database name: `formdb`;
- usuário administrativo dedicado ao laboratório;
- senha armazenada em um gerenciador de senhas;
- VPC e DB subnet group privados;
- **Public access:** `No`;
- encryption habilitada;
- backup automático conforme o objetivo da aula;
- deletion protection desabilitada somente para permitir a limpeza do laboratório.

Anote o endpoint DNS fornecido pelo RDS. O endpoint não será um IP fixo e deverá ser usado no lugar do nome `postgres` do Compose.

## 3. Guardar a conexão

Crie um segredo no AWS Secrets Manager com valores equivalentes a:

```json
{
  "Host": "ENDPOINT_DO_RDS",
  "Port": "5432",
  "Database": "formdb",
  "Username": "forms_app",
  "Password": "SENHA_FORTE"
}
```

Conceda à task role da API somente `secretsmanager:GetSecretValue` para esse segredo. A execution role continuará responsável por baixar imagens e enviar logs.

## 4. Atualizar a API

Na task definition da API, configure uma conexão compatível com o RDS por meio das variáveis de ambiente:

```text
ConnectionStrings__DefaultConnection=Host=ENDPOINT_DO_RDS;Port=5432;Database=formdb;Username=forms_app;Password=SENHA
```

Em uma prática mais segura, use a integração do ECS com Secrets Manager para injetar os valores sem colocá-los diretamente na task definition. Essa configuração será feita no ECS e no Secrets Manager, sem alteração no código.

Mantenha `Database__ApplyMigrations=true` apenas durante a primeira execução controlada. Depois que as migrations forem aplicadas, avalie uma etapa de migração separada para evitar que várias tasks tentem migrar o banco ao mesmo tempo.

## 5. Validar

Acompanhe os logs da API e procure a aplicação das migrations. Depois:

1. abrir o frontend pelo DNS do ALB;
2. cadastrar um formulário;
3. consultar o cadastro;
4. reiniciar a task da API;
5. confirmar que os dados continuam no RDS.

Verifique no console que o RDS está sem IP público e que o Security Group bloqueia conexões externas.

## 6. Backup e limpeza

Compare a diferença entre backup automático, snapshot manual e Multi-AZ. Para um laboratório, pode ser usada uma instância pequena e uma única AZ, mas é importante registrar que isso não oferece a mesma resiliência de produção.

Para encerrar o laboratório:

1. apagar ou parar os serviços que usam o banco;
2. apagar a instância RDS;
3. decidir conscientemente se criarão ou não um snapshot final;
4. apagar snapshots e backups retidos que não forem necessários;
5. apagar o segredo do Secrets Manager;
6. revisar o Security Group e o DB subnet group.

## Resultado

Ao final, teremos separado o ciclo de vida da aplicação do ciclo de vida dos dados. No próximo módulo, usaremos o S3 para armazenar objetos e enviar backups sem colocar credenciais estáticas na EC2 ou no código.
