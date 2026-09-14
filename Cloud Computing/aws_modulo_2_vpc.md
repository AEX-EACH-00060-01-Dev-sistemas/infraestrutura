# Módulo 2: VPC e isolamento de rede

Neste módulo, deixaremos de usar a rede padrão da AWS para criar uma infraestrutura sob medida para o [forms-simulator](https://github.com/contygor/forms-simulator). Nosso objetivo é separar completamente o ponto de entrada público dos nossos serviços internos, aumentando a segurança.

Antes de começarmos, vamos alinhar os conceitos dessa nova arquitetura:
- **VPC (Virtual Private Cloud):** É a nossa rede particular e isolada dentro da nuvem da Amazon.
- **ALB (Application Load Balancer):** Funcionará como o "recepcionista" do sistema, recebendo o tráfego da internet e distribuindo para a nossa aplicação.
- **ECS/Fargate:** O serviço onde os nossos contêineres do frontend e da API rodarão no futuro, sem precisarmos gerenciar máquinas EC2 virtuais.
- **RDS (Relational Database Service):** O nosso banco de dados PostgreSQL totalmente gerenciado pela AWS.

A nossa arquitetura didática funcionará assim:

```text
Internet
   |
Application Load Balancer (Subnets Públicas)
   |
ECS/Fargate: frontend e API (Subnets Privadas)
   |
RDS PostgreSQL (Subnets Privadas)
```

## Objetivos

Ao terminarmos este módulo, conseguiremos:

- criar uma VPC com subnets públicas (com acesso à internet) e privadas (sem acesso externo);
- configurar Security Groups (firewalls virtuais) específicos para cada função;
- entender a comunicação segura entre componentes públicos e privados;
- preparar a rede e as portas exatas para o banco gerenciado do próximo módulo;
- acompanhar os custos de NAT Gateways e conexões na nuvem.

## 1. Criação da VPC

No Console da AWS, acesse **VPC > Your VPCs > Create VPC** e selecione a opção **VPC and more**. Essa opção abre um assistente visual prático. Configure com os seguintes valores:

- **Name tag auto-generation:** `forms-vpc`
- **IPv4 CIDR block:** `10.20.0.0/16`
- **Number of Availability Zones (AZs):** 2 (Garante alta disponibilidade, exigência para o Load Balancer e para bancos RDS).
- **Number of public subnets:** 2
- **Number of private subnets:** 2
- **NAT gateways:** `None` (Deixaremos desabilitado inicialmente para evitar custos contínuos enquanto construímos a rede).
- **VPC endpoints:** `None`

Clique em **Create VPC** e aguarde. O assistente criará automaticamente as subnets, as tabelas de roteamento e o Internet Gateway.

## 2. Configuração dos Security Groups

Os Security Groups agem como seguranças de porta para os nossos recursos. Em **VPC > Security Groups**, selecione **Create security group** e crie um grupo para cada componente abaixo. 

*Dica importante: Quando a comunicação for interna, em vez de digitar um IP na regra, selecione o nome do outro Security Group no campo **Source**.*

### ALB (Load Balancer)
- **Regras de Entrada:** HTTP (porta `80`) e HTTPS (porta `443`) com Source para `0.0.0.0/0` (internet aberta).
- **Regras de Saída:** Liberar apenas para o Security Group do frontend.

### Frontend
- **Regras de Entrada:** Porta `80` (ou a porta configurada no contêiner) com Source exclusivo do Security Group do ALB.
- **Regras de Saída:** Liberar acesso para o Security Group da API.

### API
- **Regras de Entrada:** Porta `8080` com Source apontando para os Security Groups do ALB e do Frontend.
- **Regras de Saída:** Liberar acesso para o Security Group do RDS na porta `5432`.

### RDS (Banco de Dados)
- **Regras de Entrada:** Porta `5432` com Source exclusivo do Security Group da API. Não pode ter nenhum acesso público.
- **Atenção ao projeto:** Nós estamos criando este bloqueio de rede para garantir que o banco de dados receba tráfego **exclusivamente na porta 5432**. Essa infraestrutura foi desenhada para espelhar a porta padrão exigida pelo `forms-simulator`, garantindo que a *connection string* e as migrações da nossa API funcionem nativamente quando o banco for instanciado no próximo módulo.

## 3. Validação do ambiente

Verifique pelo console se a infraestrutura foi montada corretamente:

- a VPC `forms-vpc` está disponível;
- existem duas subnets públicas (conectadas ao Internet Gateway) e duas privadas;
- os Security Groups possuem somente as permissões estritas configuradas acima;
- nenhuma faixa ampla (`0.0.0.0/0`) foi liberada para a API ou para o RDS.

Neste momento, não é necessário criar o Load Balancer ou subir a aplicação. O nosso objetivo aqui é apenas preparar o terreno de rede.

## 4. Limpeza e encerramento

Se você não for prosseguir imediatamente para o próximo módulo, lembre-se de limpar os recursos para evitar surpresas. Pelo console, exclua os Security Groups e, em seguida, delete a VPC (o que removerá as subnets e o Internet Gateway junto).

## Resultado

Ao final desta etapa, temos uma rede corporativa estruturada e segura. No próximo módulo, instanciaremos o nosso banco PostgreSQL privado protegido por esta VPC, pronto para se conectar à nossa API do forms-simulator.