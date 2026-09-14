# Módulo 1: Publicação inicial com Amazon EC2

Neste módulo, usaremos o repositório [forms-simulator](https://github.com/contygor/forms-simulator) para apresentar a publicação de uma aplicação web em uma máquina virtual. Começaremos com uma arquitetura simples: uma única instância Amazon EC2 executará o Docker Compose com o frontend, a API ASP.NET Core e o PostgreSQL.

Não será necessário modificar os arquivos do projeto. A configuração permanecerá no ambiente Docker Compose e nos recursos da AWS.

Essa primeira versão é adequada para laboratório. Ela não representa a arquitetura final recomendada para produção, porque todos os componentes estarão no mesmo servidor e a instância terá um endereço público.

## Objetivos

Ao terminar este módulo, será possível:

- criar uma instância Ubuntu no Amazon EC2;
- conectar-se por SSH usando um par de chaves;
- instalar Docker e Git;
- subir os três contêineres do projeto;
- acessar o frontend pela internet;
- entender a relação entre EC2, EBS, Security Group e IP público.

## 1. Preparação da conta

Antes de criar recursos, recomendamos:

1. acessar a AWS com um usuário IAM administrativo protegido por MFA;
2. escolher uma única região para todo o laboratório, como `sa-east-1`;
3. configurar um orçamento e um alerta de cobrança;
4. confirmar que a região escolhida oferece o tipo de instância desejado.

Os nomes e os comandos deste material usam a região `sa-east-1` como exemplo. É possível trocar a região, mas recomendamos manter a mesma região nos módulos seguintes.

## 2. Criação da instância EC2

No console do Amazon EC2, selecione **Launch instance** e configure:

- **Name:** `forms-ec2-compose`;
- **AMI:** Ubuntu Server 24.04 LTS, 64 bits;
- **Instance type:** um tipo compatível com o laboratório e com o nível gratuito disponível para a conta;
- **Key pair:** um novo par, como `forms-lab`, no formato `.pem`;
- **Storage:** um volume EBS gp3 com pelo menos 20 GiB;
- **Network:** a VPC padrão inicialmente, com atribuição de IPv4 público;
- **Security Group:** regra SSH na origem do IP deles e regras TCP temporárias nas portas `3000` e `8080`, restritas ao IP deles durante a validação.

Não abra SSH para `0.0.0.0/0` quando for possível restringir a origem ao próprio endereço IP. A porta `5432` do PostgreSQL não deve ser aberta no Security Group.

Depois de criar a instância, copie o endereço IPv4 público e aguarde o estado `Running` e a conclusão dos status checks.

## 3. Acesso por SSH

No Linux ou macOS, proteja o arquivo da chave e conecte-se:

```bash
chmod 400 forms-lab.pem
ssh -i forms-lab.pem ubuntu@IP_PUBLICO_DA_EC2
```

No primeiro acesso, confirme a impressão digital do host somente depois de verificar que o endereço pertence à instância criada.

## 4. Instalação do ambiente

Dentro da EC2, instale o Docker Engine, o plugin Compose e o Git:

```bash
sudo apt update
sudo apt install -y ca-certificates curl git
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker "$USER"
```

Saia da sessão SSH e entre novamente para que a associação ao grupo `docker` seja aplicada. Depois, confirme a instalação:

```bash
docker --version
docker compose version
```

## 5. Download e publicação do projeto

Clone o repositório e inicie o Compose:

```bash
git clone https://github.com/contygor/forms-simulator.git
cd forms-simulator
docker compose up --build -d
docker compose ps
```

O primeiro build poderá demorar, porque a API e o frontend serão construídos na própria instância. Acompanhe os logs com:

```bash
docker compose logs -f
```

O Compose cria uma rede interna para os serviços. A API acessa o banco pelo nome `postgres`, e o frontend acessa a API pela URL configurada em `FRONTEND_API_URL`.

## 6. Acesso online

Como o Compose publica `3000:80` para o frontend e `8080:8080` para a API, permita temporariamente as portas `3000` e `8080` no Security Group, preferencialmente apenas para o seu IP. Depois, crie uma regra mais adequada com Nginx ou Load Balancer na porta `80`.

Acesse o frontend em:

```text
http://IP_PUBLICO_DA_EC2:3000
```

Para validar a API, acesse:

```text
http://IP_PUBLICO_DA_EC2:8080/health
```

O PostgreSQL não deve ser acessado diretamente pela internet. Teste o cadastro pelo frontend e verifique os serviços com `docker compose ps`.

## 7. EBS, parada e limpeza

O volume EBS contém o sistema operacional e os arquivos da instância. O volume Docker `postgres_data` mantém os dados do PostgreSQL enquanto o volume existir.

Para interromper os contêineres sem apagar os dados, execute:

```bash
docker compose down
```

Para parar a instância e interromper a cobrança de computação, use **Instance state > Stop instance**. O EBS continuará gerando cobrança enquanto existir.

Para remover o laboratório inteiro, apague a instância com **Terminate instance** e revise volumes EBS, snapshots, Elastic IPs e Security Groups que tenham sido criados separadamente.

## Resultado

Ao final, teremos uma aplicação funcional na internet usando uma única EC2. No próximo módulo, substituiremos o build realizado dentro da EC2 por imagens publicadas no Amazon ECR e contêineres executados pelo Amazon ECS com Fargate.
