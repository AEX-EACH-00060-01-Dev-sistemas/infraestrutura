# Módulo 3: VPC e isolamento de rede

Neste módulo, deixaremos de usar a rede padrão e criaremos uma arquitetura de rede específica para o `forms-simulator`. O ponto de entrada público será separado dos serviços internos.

Não será necessário modificar o código do projeto. O endereço do ALB e as permissões de rede serão configurados nos serviços da AWS e nas variáveis de ambiente das tasks.

A arquitetura didática será:

```text
Internet
   |
Application Load Balancer público
   |
ECS/Fargate: frontend e API
   |
RDS PostgreSQL privado
```

O ALB ficará em subnets públicas. As tasks do ECS e o RDS ficarão em subnets privadas. O RDS não terá IP público.

## 1. Criar a VPC

No console da VPC, selecione **Create VPC > VPC and more** e use valores semelhantes a:

- nome: `forms-vpc`;
- IPv4 CIDR: `10.20.0.0/16`;
- duas Availability Zones;
- duas subnets públicas;
- duas subnets privadas;
- Internet Gateway habilitado;
- NAT Gateway inicialmente desabilitado para evitar custo contínuo;
- VPC endpoint para S3 somente quando a atividade exigir.

É importante entender que duas Availability Zones são importantes para o ALB e para o RDS Multi-AZ, mas que os recursos continuam gerando custos conforme o serviço escolhido.

## 2. Security Groups

Crie regras por função, sem permitir que todos os componentes conversem livremente:

### ALB

- entrada HTTP `80` e HTTPS `443` a partir da internet;
- saída para o Security Group do frontend.

### Frontend

- entrada da porta `80` somente a partir do Security Group do ALB;
- saída para a API.

### API

- entrada da porta `8080` somente a partir do Security Group do ALB ou do frontend, conforme o desenho escolhido;
- saída para o RDS na porta `5432`.

### RDS

- entrada da porta `5432` somente a partir do Security Group da API;
- nenhum acesso público.

Prefira referências entre Security Groups em vez de liberar faixas amplas como `0.0.0.0/0`.

## 3. Application Load Balancer

Crie um ALB internet-facing nas duas subnets públicas, com:

- um target group para o frontend na porta `80`;
- um target group para a API na porta `8080`;
- health check da API em `/health`;
- listener HTTP `80` encaminhando o tráfego conforme as regras de caminho.

Como o frontend chama a API por uma URL configurável, informe `FRONTEND_API_URL` na configuração da task do frontend com o endereço público do ALB. O container já gera sua configuração durante a inicialização, portanto não será necessário reconstruir a imagem nem editar o repositório.

Na task da API, informe também a origem pública do frontend usando a configuração de ambiente `Cors__AllowedOrigins__0`. Use o endereço real publicado pelo ALB, por exemplo:

```text
FRONTEND_API_URL=http://DNS_DO_ALB
Cors__AllowedOrigins__0=http://DNS_DO_ALB
```

Essa configuração permite que o navegador faça chamadas entre frontend e API sem modificar `Program.cs`.

## 4. ECS nas subnets privadas

Atualize o serviço ECS para:

- usar as duas subnets privadas;
- desabilitar IP público das tasks;
- associar o Security Group correto;
- registrar as tasks no target group do ALB.

Sem NAT Gateway ou VPC endpoints adequados, tasks privadas poderão não conseguir baixar imagens do ECR ou enviar logs. Escolha uma destas alternativas didáticas:

1. usar NAT Gateway temporariamente, sabendo que ele tem custo por hora e por processamento;
2. criar endpoints privados para ECR API, ECR DKR, S3 e CloudWatch Logs;
3. realizar a primeira execução em subnets públicas apenas para comparação, sempre com Security Groups restritivos.

## 5. Validação

Verifique:

- o ALB está `active`;
- os target groups mostram targets `healthy`;
- `/health` retorna `{"status":"ok"}`;
- o frontend abre pelo DNS do ALB;
- a API e o RDS não possuem endereço público;
- conexões diretas da internet para `8080` e `5432` falham.

## 6. Limpeza

Apague serviços ECS e tasks, depois o ALB e target groups, NAT Gateways e Elastic IPs. Só então remova subnets, route tables, endpoints, Internet Gateway e a VPC.

## Resultado

Ao final, teremos comparado uma implantação pública simples com uma arquitetura segmentada. No próximo módulo, substituiremos o PostgreSQL do Compose por uma instância Amazon RDS em subnets privadas.
