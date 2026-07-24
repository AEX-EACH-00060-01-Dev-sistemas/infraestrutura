# Docker

Docker é uma ferramenta de containerização que utiliza imagens para automatizar o processo de implantação de aplicações.

A ferramenta é baseada no kernel do linux e no uso de namegroups para segregação de processos para manter independência, autonomia e segurança
entre as diversas aplicações utilizadas.

## Download

- Windows:
    - Siga as instruções especificadas na [documentação oficial](https://docs.docker.com/desktop/setup/install/windows-install/)
- Linux:
    - Siga as instruções especificadas na [documentação oficial](https://docs.docker.com/desktop/setup/install/linux/)
    - Para adicionar seu usuário ao grupo de permitidos ao docker: `$ sudo usermod -aG docker $USER`

## Estrutura do projeto

Vamos primeiro criar uma pasta para o projeto (Aqui utilizamos a pasta "Docker_Exemplo", siga as instruções por aqui
e se baseie no código disponibilizado nela se precisar de um guia).

- Dê o nome que quiser para ela, pode escolher um nome que combine com seu projeto (no caso do exemplo, todo-list
seria outro nome bom).

- Crie dentro dela uma pasta para os arquivos de lógica do projeto (src) e uma pasta para a infraestrutura que criaremos (infra)

- Na src criaremos um script python que roda a lógica de uma lista de afazeres simples (app.py). Um bom exercício de programação
é fazer sua própria implementação dela para linha de comando, mas temos o exemplo para web para pularmos essa parte.

- Junto ao arquivo python iremos precisar também de um arquivo requirements.txt para identificar o que deve ser instalado para esse projeto e uma pasta
para o html.

- Criaremos na pasta infra o nosso arquivo Docker. Aqui, utilizaremos um arquivo compose.yml com as informações que precisamos
para rodar o código python escrito.

- Assim, a pasta do seu projeto deve estar com essa cara agora:

```
todo-list/
├── infra
│   └── compose.yml
└── src
    ├── app.py
    ├── requirements.txt
    └── templates
        └── index.html
```


## Criando um exemplo mínimo

Para o exemplo do projeto, utilizaremos Python, uma linguagem vista no [curso de linguagens](https://github.com/AEX-EACH-00060-01-Dev-sistemas/linguagem-de-programacao/),
com o framework Flask para programação web.

- Criaremos uma aplicação que manterá em memória suas tarefas a serem feitas:

```python
import os
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)

# dict: {'id': int, 'title': str, 'completo': bool}
todos = []
id_counter = 1

@app.route('/')
def index():
    return render_template('index.html', todos=list(todos))

@app.route('/add', methods=['POST'])
def add_todo():
    global id_counter
    title = request.form.get('title')
    if title:
        todos.append({
            'id': id_counter,
            'title': title,
            'completo': False
        })
        id_counter += 1
    return redirect(url_for('index'))


@app.route('/toggle/<int:todo_id>')
def toggle_todo(todo_id):
    for todo in todos:
        if todo['id'] == todo_id:
            todo['completo'] = not todo['completo']
            break
    return redirect(url_for('index'))


@app.route('/delete/<int:todo_id>')
def delete_todo(todo_id):
    global todos
    todos = [todo for todo in todos if todo['id'] != todo_id]

    return redirect(url_for('index'))


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

- Para ela funcionar, utilize o template disponibilizado na pasta "Docker_Exemplo"

- Criaremos então um arquivo compose que rodará essa versão mínima:

```yaml
services:
  web_app:
    image: python:latest
    container_name: todo_flask
    restart: always
    ports:
      - "5000:5000"
    volumes:
      - ../src:/app:z
    working_dir: /app
   
    command: sh -c "pip install --no-cache-dir -r requirements.txt && python app.py"
```

## Rodando a aplicação

Com o arquivo compose que criamos podemos ir direto para o terminal e rodar o seguinte comando:

```
docker compose up -d
```

- Você pode necessitar de privilégios de administrador ou ser incluído no grupo Docker, para escalada de privilégio em ambientes linux
usamos o comando sudo precedendo o comando que queremos executar. A adição do usuário ao grupo Docker já foi coberta no começo desse arquivo.

- Após rodar o comando, será possível ver algumas informações sobre o container sendo criado. O padrão da mensagem é o resultado positivo
para o download da imagem, para a criação da conexão e do container.

- Se você quiser entrar no ambiente, pode ser utilizado o comando `docker attach NOME_DO_CONTAINER` e sair com CTRL+P + CTRL+Q.

## Aumentando o escopo

- A aplicação atual roda mantendo todas as variáveis relacionadas à lista em memória, o que impede que certos benefícios que vem com o uso
de um banco de dados para o seu projeto.

- Desse modo, criaremos uma nova versão da nossa aplicação utilizando chamadas ao banco de dados: [arquivo final](./Docker_Exemplo/src/app.py)

- Não será necessário mudar o template utilizado para essa nova versão da aplicação.

- Assim, seu arquivo compose final deve se parecer com isso:

```yaml
services:
  database:
    image: mariadb:11.2
    container_name: todo_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: todo_db
      MYSQL_USER: todo_user
      MYSQL_PASSWORD: todo_password
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql

  web_app:
    image: python:latest
    container_name: todo_flask
    restart: always
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=database
      - DB_USER=todo_user
      - DB_PASSWORD=todo_password
      - DB_NAME=todo_db
    volumes:
      - ../src:/app:z
    working_dir: /app
   
    command: sh -c "pip install --no-cache-dir -r requirements.txt && python app.py"
    depends_on:
      - database

volumes:
  mariadb_data:
```

dica: use variáveis de ambiente no lugar das variaveis de banco de dados para evitar acidentes e vazamentos de informações sensíveis.

## Referências

- [Docker](https://www.docker.com)
- [RedHat](https://www.redhat.com/pt-br/topics/containers/what-is-docker)
- [Wikipedia](https://en.wikipedia.org/wiki/Docker_(software))
- [Compose](https://docs.docker.com/compose)
