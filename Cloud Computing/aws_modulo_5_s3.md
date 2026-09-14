# Módulo 5: Amazon S3 para objetos e backups

Neste módulo, usaremos o Amazon Simple Storage Service (S3) para armazenar artefatos do laboratório. O S3 será tratado como armazenamento de objetos, não como um disco para instalar o PostgreSQL ou executar a aplicação.

Não será necessário modificar o código do projeto. O acesso ao bucket será configurado por IAM roles, políticas e comandos da AWS CLI executados no ambiente autorizado.

Escolha uma atividade principal:

- publicar um arquivo estático de demonstração;
- armazenar logs exportados;
- enviar um backup lógico do PostgreSQL do RDS.

Para preservar a segurança, mantenha o bucket privado e acesse os objetos com IAM e URLs pré-assinadas quando necessário.

## 1. Criar um bucket privado

Crie o bucket na mesma região do restante do laboratório, usando um nome globalmente único, por exemplo:

```text
forms-lab-ID_DA_CONTA
```

Configure:

- Block Public Access habilitado;
- ACLs desabilitadas;
- criptografia SSE-S3 ou SSE-KMS;
- versionamento conforme o objetivo da aula;
- tags como `Projeto=forms-simulator` e `Ambiente=lab`.

Não crie uma política pública para o bucket apenas para facilitar o primeiro teste.

## 2. Permissões por IAM role

Se a atividade for executada em uma EC2, associe uma IAM role à instância. Se for executada no ECS, use a task role da aplicação.

A política deverá conceder somente as ações necessárias ao bucket. Para um laboratório de upload, um exemplo restrito seria:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": "arn:aws:s3:::forms-lab-ID_DA_CONTA/backups/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::forms-lab-ID_DA_CONTA",
      "Condition": {"StringLike": {"s3:prefix": ["backups/*"]}}
    }
  ]
}
```

Evite `AmazonS3FullAccess` quando uma política limitada ao prefixo `backups/` for suficiente.

## 3. Teste pela AWS CLI

Com a role configurada, confirme a identidade e envie um arquivo de teste:

```bash
aws sts get-caller-identity
printf 'forms-simulator\n' > teste.txt
aws s3 cp teste.txt s3://forms-lab-ID_DA_CONTA/backups/teste.txt
aws s3 ls s3://forms-lab-ID_DA_CONTA/backups/
```

Para baixar um objeto sem torná-lo público, gere uma URL temporária:

```bash
aws s3 presign s3://forms-lab-ID_DA_CONTA/backups/teste.txt --expires-in 900
```

## 4. Backup lógico do PostgreSQL

Execute o `pg_dump` a partir de uma máquina que tenha conectividade com o RDS e credenciais apropriadas. Envie o arquivo gerado ao prefixo privado do bucket:

```bash
pg_dump --host=ENDPOINT_DO_RDS --port=5432 --username=forms_backup --format=custom --file=forms.dump formdb
aws s3 cp forms.dump s3://forms-lab-ID_DA_CONTA/backups/forms-$(date +%Y-%m-%d).dump
```

Não coloque a senha na linha de comando nem no repositório. Obtenha a senha por um mecanismo de segredo apropriado e mantenha o arquivo de backup protegido.

Lembre-se de que um backup no S3 não substitui os backups automáticos e snapshots do RDS. São camadas diferentes de recuperação.

## 5. Validação de segurança

Confirme que:

- um acesso anônimo ao objeto é negado;
- a role consegue executar apenas as operações autorizadas;
- a criptografia está habilitada;
- o bucket está na região esperada;
- o objeto aparece no prefixo correto;
- a aplicação não contém access key e secret key estáticas.

## 6. Limpeza

Apague os objetos, versões e delete markers antes de apagar o bucket. Se o versionamento estiver habilitado, o botão de apagar o bucket poderá não remover todas as versões sozinho.

Depois, remova a política IAM, a role ou a task role criada exclusivamente para a atividade e revise custos de armazenamento, requests e transferência.

## Resultado final da trilha

Ao final, teremos publicado o mesmo projeto em quatro modelos: uma EC2 com Docker Compose, imagens no ECR executadas pelo ECS, uma arquitetura segmentada por VPC e um banco PostgreSQL gerenciado no RDS, complementado por armazenamento privado no S3.
