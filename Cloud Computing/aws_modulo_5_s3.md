# Módulo 5: Amazon S3 para objetos e backups

Neste módulo, nós usaremos o Amazon Simple Storage Service (S3) para armazenarmos artefatos do nosso laboratório. Nós trataremos o S3 como um armazenamento de objetos para arquivos, e não como um disco tradicional para instalar o PostgreSQL ou rodar a aplicação.

Para introduzirmos o uso do S3 de forma rápida e automatizada, a nossa atividade principal será o envio do arquivo de configuração do projeto diretamente pelo terminal, simulando uma esteira de automação.

Para preservarmos a segurança, nós manteremos o bucket totalmente privado e acessaremos os objetos via painel ou URLs pré-assinadas.

## 1. Criação de um bucket privado

No console da AWS, acesse **S3 > Create bucket**. Escolha a mesma região do restante do laboratório e use um nome globalmente único, por exemplo:

```text
forms-lab-ID_DA_CONTA
```

Configure as opções abaixo.

* acesso público: block all public access habilitado;
* controle de acesso: acls desabilitadas;
* criptografia: sse-s3 ou sse-kms;
* versionamento: conforme o objetivo da aula;
* tags: adicionar projeto=forms-simulator e ambiente=lab.

Clique em **Create bucket**. Nós não criaremos uma política pública apenas para facilitar o primeiro teste, pois a segurança vem em primeiro lugar.

## 2. Criação de permissões

Acesse **IAM > Policies > Create policy** e crie uma política restrita ao nosso bucket e à pasta `backups/`. Se nós executarmos a atividade no ECS, associaremos à task role da aplicação.

A política deverá conceder somente as ações necessárias. Para um laboratório de upload, um exemplo restrito seria este formato:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "allow",
      "Action": ["s3:putObject", "s3:getObject"],
      "Resource": "arn:aws:s3:::forms-lab-ID_DA_CONTA/backups/*"
    },
    {
      "Effect": "allow",
      "Action": "s3:listBucket",
      "Resource": "arn:aws:s3:::forms-lab-ID_DA_CONTA",
      "Condition": {"stringLike": {"s3:prefix": ["backups/*"]}}
    }
  ]
}
```

Nós evitaremos usar `AmazonS3FullAccess` quando uma política limitada for suficiente.

## 3. Upload automatizado pelo terminal

Para automatizarmos o processo, nós usaremos a interface de linha de comando da AWS. Nós enviaremos o arquivo `docker-compose.yml` do nosso projeto para a nuvem.

Abra o terminal na pasta do projeto e confirme a sua identidade. Em seguida, faça o envio executando as instruções abaixo.

```bash
aws sts get-caller-identity
aws s3 cp docker-compose.yml s3://forms-lab-ID_DA_CONTA/backups/docker-compose.yml
aws s3 ls s3://forms-lab-ID_DA_CONTA/backups/
```

Para baixarmos o objeto gerando a URL temporária por linha de comando, basta executar o código abaixo.

```bash
aws s3 presign s3://forms-lab-ID_DA_CONTA/backups/docker-compose.yml --expires-in 900
```

## 4. Backup lógico do PostgreSQL

Se nós quisermos validar um cenário real de banco, podemos gerar um dump a partir de uma máquina autorizada e enviá-lo pelo terminal de forma totalmente automatizada. Pela CLI, o comando seria este modelo:

```bash
pg_dump --host=ENDPOINT_DO_RDS --port=5432 --username=forms_backup --format=custom --file=forms.dump formdb
aws s3 cp forms.dump s3://forms-lab-ID_DA_CONTA/backups/forms-$(date +%Y-%m-%d).dump
```

Nós nunca colocaremos a senha direto na linha de comando. É importante lembrar que o backup no S3 não substitui as rotinas automáticas do RDS.

## 5. Validação de segurança

No console, nós confirmaremos as regras abaixo.

* bloqueio: um acesso anônimo ao objeto é negado;
* restrição: a role consegue executar apenas as operações autorizadas;
* proteção: a criptografia está habilitada;
* localidade: o bucket está na região esperada;
* organização: o objeto aparece na pasta correta;
* credenciais: a aplicação não contém chaves de acesso estáticas no código.

## 6. Limpeza e encerramento

No S3, nós apagaremos os objetos, as versões e os marcadores de exclusão antes de apagarmos o bucket. Se o versionamento estiver ativado, a AWS não deixará excluir o bucket caso ele contenha versões ocultas.

Depois, nós removeremos a política e a role do IAM criadas para este módulo, garantindo que não restem custos na conta.

## Resultado final da trilha

Parabéns! Ao final deste laboratório, nós teremos publicado o mesmo projeto em quatro modelos distintos, sendo uma máquina virtual EC2 com Docker Compose, contêineres escaláveis no ECS Fargate, uma rede segmentada com VPC e um banco de dados PostgreSQL gerenciado no RDS, tudo isso complementado pelo armazenamento seguro do S3.