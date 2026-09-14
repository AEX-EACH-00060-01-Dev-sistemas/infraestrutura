# Módulo 1: publicação inicial com Amazon EC2

Neste módulo, usaremos o repositório [forms-simulator](https://github.com/contygor/forms-simulator) para apresentar a publicação de uma aplicação web na nuvem. O Amazon EC2 (Elastic Compute Cloud) é um serviço da AWS que permite alugar máquinas virtuais para rodar as nossas aplicações. Começaremos com uma arquitetura simples. Uma única instância EC2 executará o Docker Compose com o frontend, a API ASP.NET Core e o PostgreSQL.

Não precisaremos modificar os arquivos do projeto. Manteremos a configuração no ambiente Docker Compose e nos recursos da AWS.

Consideramos essa primeira versão adequada para laboratório. Ela não representa a arquitetura final recomendada para produção, porque colocaremos todos os componentes no mesmo servidor e a instância terá um endereço público.

## Objetivos

Ao terminarmos este módulo, conseguiremos realizar os passos abaixo.

- criar uma instância Ubuntu no Amazon EC2;
- fazer a conexão por SSH usando um par de chaves;
- instalar Docker e Git;
- subir os três contêineres do projeto;
- acessar o frontend pela internet;
- entender a relação entre EC2, EBS, Security Group e IP público.

## 1. Preparação da conta

Antes de criarmos os recursos, recomendamos o seguinte: acessar a AWS com um usuário IAM administrativo protegido por MFA e escolher uma única região para todo o laboratório. Usamos a região `sa-east-1` como exemplo nos nomes e nos comandos deste material. Podemos trocar a região, mas recomendamos manter a mesma nos módulos seguintes. Outros passos importantes são configurar um orçamento de cobrança e confirmar se a região escolhida oferece o tipo de instância desejado.

## 2. Criação da instância EC2

No console do Amazon EC2, selecionaremos **Launch instance** e configuraremos as opções a seguir.

- **Name:** forms-simulator-v1;
- **AMI:** ubuntu Server 24.04 LTS, 64 bits;
- **Instance type:** um tipo compatível com o laboratório e com o nível gratuito disponível para a conta;
- **Key pair:** um novo par chamado `forms-simulator-key-pair`, no formato `.pem`;
- **Storage:** um volume EBS gp3 com pelo menos 20 GiB;
- **Network:** a VPC padrão inicialmente, com atribuição de IPv4 público;
- **Security Group:** regra SSH na origem do nosso IP e regras TCP temporárias nas portas `3000` e `8080`, restritas ao nosso IP durante a validação.

Não abriremos SSH para `0.0.0.0/0` quando for possível restringir a origem ao nosso próprio endereço IP. Não devemos abrir a porta `5432` do PostgreSQL no Security Group.

Depois de criarmos a instância, copiaremos o endereço IPv4 público e aguardaremos o estado `Running` e a conclusão das verificações de status.

## 3. Acesso por SSH

No Linux ou macOS, protegeremos o arquivo da chave e faremos o acesso executando as linhas abaixo.

```bash
chmod 400 forms-simulator-key-pair.pem
ssh -i forms-simulator-key-pair.pem ubuntu@IP_PUBLICO_DA_EC2
```

No Windows 11, acessaremos a EC2 pelo **PowerShell** ou pelo **Terminal do Windows** usando o OpenSSH, que normalmente já vem instalado. Na pasta onde o arquivo `.pem` foi salvo, ajustaremos as permissões da chave e faremos a conexão com estes comandos.

```powershell
cd C:\caminho\onde\a\chave\foi\salva

icacls .\forms-simulator-key-pair.pem /inheritance:r

icacls .\forms-simulator-key-pair.pem /grant:r "$($env:USERNAME):(R)"

ssh -i .\forms-simulator-key-pair.pem ubuntu@IP_PUBLICO_DA_EC2
```

Se `ssh` não for reconhecido, instalaremos o recurso **OpenSSH Client** em **Configurações > Sistema > Recursos opcionais > Exibir recursos**. Como alternativa, usaremos o PuTTY. Converteremos o arquivo `.pem` para `.ppk` com o PuTTYgen e informaremos o arquivo convertido no campo de chave privada da sessão SSH.

No primeiro acesso, confirmaremos a impressão digital do host somente depois de verificar que o endereço pertence à instância criada.

## 4. Instalação do ambiente

Dentro da EC2, instalaremos o Docker e o Git de forma muito mais enxuta executando o script oficial da plataforma.

```bash
sudo apt update
sudo apt install -y git
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker "$USER"
```

Sairemos da sessão SSH e entraremos novamente para que a nossa associação ao grupo `docker` seja aplicada. Depois, confirmaremos a instalação executando estas verificações.

```bash
docker --version
docker compose version
```

## 5. Download e publicação do projeto

Clonaremos o repositório e iniciaremos o Compose executando o código a seguir.

```bash
git clone https://github.com/contygor/forms-simulator.git
cd forms-simulator
docker compose up --build -d
docker compose ps
```

O primeiro build poderá demorar, porque construiremos a API e o frontend na própria instância. Acompanharemos os logs executando este comando.

```bash
docker compose logs -f
```

Veremos que o Compose cria uma rede interna para os serviços. A API acessa o banco pelo nome `postgres`, e o frontend acessa a API pela URL configurada em `FRONTEND_API_URL`.

## 6. Acesso online

Como o Compose publica `3000:80` para o frontend e `8080:8080` para a API, permitiremos temporariamente as portas `3000` e `8080` no Security Group, preferencialmente apenas para o nosso IP. Depois, criaremos uma regra mais adequada com Nginx ou Load Balancer na porta `80`.

Acessaremos o frontend no endereço abaixo.

```text
http://IP_PUBLICO_DA_EC2:3000
```

Para validarmos a API, acessaremos a rota a seguir.

```text
http://IP_PUBLICO_DA_EC2:8080/health
```

Não acessaremos o PostgreSQL diretamente pela internet. Testaremos o cadastro pelo frontend e verificaremos os serviços rodando o painel do Compose.

## 7. Conferir os dados pelo terminal

Depois de cadastrarmos um formulário pelo site, confirmaremos pelo terminal da EC2 se a API gravou o registro no PostgreSQL.

Na sessão SSH, entraremos na pasta do projeto e abriremos o terminal do banco executando este código.

```bash
cd ~/forms-simulator
docker compose exec postgres psql -U postgres -d formdb
```

Dentro do PostgreSQL, listaremos as tabelas com esta instrução.

```sql
\dt
```

A tabela de cadastros criada pelo projeto é `form_responses`. Consultaremos os registros rodando a query abaixo.

```sql
SELECT * FROM form_responses;
```

Para visualizarmos somente os últimos cadastros de forma mais organizada, usaremos este comando.

```sql
SELECT "Id", "Nome", "Sobrenome", "DataNascimento", "CidadeNascimento", "CreatedAtUtc"
FROM form_responses
ORDER BY "Id" DESC
LIMIT 10;
```

Se o cadastro aparecer no resultado, confirmaremos que a requisição da API chegou ao PostgreSQL e os dados foram persistidos corretamente. A tabela `__EFMigrationsHistory` também aparecerá, mas sabemos que ela guarda apenas o histórico das migrações do Entity Framework, não os cadastros.

Para sairmos do PostgreSQL, executaremos a letra de saída.

```sql
\q
```

## 8. EBS, parada e limpeza

O volume EBS contém o sistema operacional e os arquivos da instância. O volume Docker `postgres_data` mantém os dados do PostgreSQL enquanto o volume existir.

Para interrompermos os contêineres sem apagar os dados, executaremos o código abaixo.

```bash
docker compose down
```

Para pararmos a instância e interromper a cobrança de computação, usaremos a opção **Instance state > Stop instance**. O EBS continuará gerando cobrança enquanto existir.

Como usaremos VPC, RDS, ECR, ECS e S3 nos módulos seguintes, não precisaremos da EC2 deste módulo. Depois de validarmos a aplicação, encerraremos os contêineres e terminaremos a instância para evitar custos adicionais.

### Excluir a EC2 pelo Console da AWS

1. Na sessão SSH, entraremos na pasta do projeto e encerraremos os contêineres executando as linhas a seguir.

```bash
cd ~/forms-simulator
docker compose down
exit
```

2. No **Console da AWS**, abriremos **EC2 > Instances**.
3. Selecionaremos a instância `forms-simulator-v1`.
4. Escolheremos a opção **Instance state > Terminate instance**.
5. Confirmaremos a exclusão quando o console solicitar.
6. Em **EC2 > Volumes**, verificaremos se não ficou nenhum volume EBS associado ao laboratório.
7. Em **EC2 > Elastic IPs**, liberaremos qualquer Elastic IP criado exclusivamente para este laboratório.
8. Em **EC2 > Security Groups**, removeremos o Security Group criado exclusivamente para a instância, caso você tenha criado um security groups que não era o default, depois de confirmar que ele não é usado por outro recurso.

Lembramos que a opção **Terminate instance** exclui a instância e o volume raiz associado. Antes de confirmar, salvaremos fora da EC2 qualquer arquivo ou dado que precisemos reutilizar. Também perderemos o volume Docker do PostgreSQL, pois ele pertence ao laboratório desta instância.