# Fundamentos de Computação em Nuvem
A computação em nuvem revolucionou a forma como projetamos, implantamos e gerenciamos aplicações. Em vez de manter servidores físicos locais, a nuvem permite o acesso sob demanda a um pool compartilhado de recursos computacionais configuráveis.

## Características Essenciais
De acordo com o Instituto Nacional de Padrões e Tecnologia dos Estados Unidos, existem cinco características essenciais que definem o modelo de nuvem:

* Autoatendimento Sob Demanda: o consumidor pode provisionar recursos computacionais, como tempo de servidor e armazenamento em rede, de forma automática e sem interação humana com o provedor de serviços.
* Acesso Amplo a Rede: as capacidades estão disponíveis na rede e são acessadas por mecanismos padrões que promovem o uso por plataformas heterogêneas.
* Agrupamento de Recursos: os recursos computacionais do provedor são agrupados para atender a múltiplos consumidores usando um modelo de locação múltipla.
* Elasticidade Rápida: as capacidades podem ser provisionadas e liberadas elasticamente para escalar rapidamente para fora e para dentro conforme a demanda.
* Serviço Mensurado: os sistemas em nuvem controlam e otimizam automaticamente o uso de recursos alavancando uma capacidade de medição apropriada ao tipo de serviço.

## Modelos de Serviço
A arquitetura em nuvem é dividida em três modelos principais de serviço:

* IaaS (Infraestrutura como Serviço): fornece processamento, armazenamento e redes. Empresas e produtos de destaque incluem a Amazon Web Services com o Amazon EC2 e Amazon S3, a Microsoft Azure com suas Virtual Machines e Azure Blob Storage, e o Google Cloud Platform com o Compute Engine. No cenário de provedores focados em desenvolvedores, destaca se a DigitalOcean com seus Droplets e para estruturação de data centers privados em laboratórios e ambientes corporativos on premise, a plataforma de virtualização Proxmox é amplamente utilizada.
* PaaS (Plataforma como Serviço): oferece a infraestrutura e o ambiente de execução. Exemplos clássicos no mercado corporativo são o Heroku, o Google App Engine, o AWS Elastic Beanstalk e o Azure App Service. Para o ecossistema frontend, plataformas como Vercel e Netlify são dominantes. É muito comum o uso de bancos de dados gerenciados neste modelo, englobando motores robustos de PostgreSQL e MySQL através do Amazon RDS ou Google Cloud SQL, até bancos orientados a séries temporais fornecidos como plataforma, como o Timescale Cloud.
* SaaS (Software como Serviço): disponibiliza aplicações completas e prontas para o usuário final. Grandes nomes deste mercado são o Google Workspace com aplicações como o Gmail, além do Microsoft 365, Salesforce, GitHub, Slack, Zoom e ferramentas de gestão ágil como o Atlassian Jira.

## Sistemas Distribuídos
A compreensão de computação em nuvem requer estudar os conceitos de sistemas distribuídos. A computação em nuvem é a aplicação comercial e escalável dos princípios de sistemas distribuídos. Conforme abordado por Andrew S. Tanenbaum e Maarten van Steen na obra "Distributed Systems: Principles and Paradigms", um sistema distribuído é uma coleção de computadores independentes que se apresentam aos usuários como um sistema único e coerente.

### Virtualização
A virtualização é o motor da computação em nuvem, permitindo que hardwares físicos sejam divididos em vários recursos lógicos:

* Virtualização Total ou Máquinas Virtuais (VMs): ocorre quando um hipervisor simula um hardware completo para rodar um sistema operacional inteiro. Grandes corporações fornecem soluções para isso, como a VMware com o vSphere e o ESXi, a Microsoft com o Hyper V, a Oracle com o VirtualBox, provedores de nuvem com tecnologias proprietárias como o AWS Nitro System, e projetos de código aberto muito utilizados como o Proxmox KVM e o XenServer.
* Virtualização em Nível de Sistema Operacional (Contêineres): em vez de virtualizar o hardware de forma pesada, compartilha o kernel e virtualiza os espaços de usuário. O Docker é a principal ferramenta e o gigante absoluto desse mercado, junto com alternativas modernas como o Podman, permitindo empacotar a aplicação e suas dependências em um contêiner leve e portátil, ideal para arquiteturas de microsserviços, para orquestrar milhares desses contêineres, o Kubernetes é a tecnologia padrão, gerenciada na nuvem por serviços como Amazon EKS, Google Kubernetes Engine e Azure Kubernetes Service.

### Sincronização de Relógios
Saber a ordem e o tempo exato dos eventos é um grande desafio quando temos servidores espalhados pelo mundo:

* Sincronização Física (NTP): o Protocolo de Tempo de Rede sincroniza os relógios dos computadores com uma referência global. Empresas como Google (com o serviço Google TrueTime que utiliza relógios atômicos e receptores GPS), AWS (com o Amazon Time Sync Service) e Microsoft (com o Azure Time Sync Service) oferecem soluções altamente precisas para garantir que as máquinas possuam a mesma marcação de horário físico.
* Relógios Lógicos (Algoritmo de Lamport e Relógios Vetoriais): foco na ordem cronológica dos eventos, determinando o que aconteceu antes do que, mesmo na ausência de uma sincronização de horário físico exato. É um conceito fundamental para ordenar eventos e garantir transações distribuídas consistentes.

### Bancos de Dados e Sincronização
Sistemas em nuvem exigem que os dados estejam consistentes e altamente disponíveis em múltiplas localidades:

* Replicação Líder e Seguidor: mm nó principal recebe todas as gravações e repassa as alterações para nós de leitura. É o padrão de bancos relacionais robustos e muito visto em serviços como o Amazon RDS e o Amazon Aurora, rodando instâncias otimizadas de PostgreSQL e MySQL, além do Azure Database for PostgreSQL e do Google Cloud SQL.
* Sincronização Sem Líder ou Ponto a Ponto: todos os nós podem aceitar leituras e gravações paralelamente. O Apache Cassandra é um excelente exemplo de banco NoSQL amplamente utilizado neste modelo para suportar massivos volumes de dados, assim como o ScyllaDB e o Riak, escalando horizontalmente de maneira incrível sem depender de um ponto único de falha.
* Replicação Multi Líder: vários nós principais processam gravações simultaneamente em data centers espalhados por regiões diferentes. Ideal para aplicações globais, com serviços como o Amazon DynamoDB oferecendo funcionalidades de tabelas globais, o Azure Cosmos DB com sua arquitetura de distribuição global nativa, e o banco Couchbase, que replicam dados quase instantaneamente entre continentes diferentes.

## Referências e Leituras Recomendadas
Esta seção está dividida entre materiais de acesso público ou gratuito, que estão disponíveis na pasta de bibliografias deste repositório, e obras consagradas de leitura indispensável para aprofundamento teórico.

## Referências e Leituras Recomendadas

### Materiais de Acesso Livre e Domínio Público (disponíveis no repositório)
* [Publicação Especial 800-145 do NIST](./Bibliografia/NIST%20SP%20800-145,%20The%20NIST%20Definition%20of%20Cloud%20Computing.pdf): o documento oficial do governo dos Estados Unidos que define os padrões e modelos de nuvem adotados globalmente.
* [Publicação Especial 500-292 do NIST](./Bibliografia/NIST%20SP%20500-292.pdf): a arquitetura de referência oficial para computação em nuvem, detalhando os atores e componentes do ecossistema.
* [AWS Well-Architected Framework](./Bibliografia/AWS%20Well-Architected%20Framework%202022.pdf): o material oficial da Amazon que reúne as melhores práticas divididas em pilares para projetar arquiteturas resilientes, seguras e eficientes na nuvem.

### Obras Consagradas e Manuais (Leituras Recomendadas)
* Distributed Systems Principles and Paradigms (Andrew S. Tanenbaum e Maarten van Steen): obra clássica fundamental para entender os conceitos de replicação de dados e tolerância a falhas.
* Designing Data Intensive Applications (Martin Kleppmann): livro amplamente aclamado e indispensável na engenharia de software moderna, que explica profundamente como bancos de dados, particionamento e replicação funcionam nos bastidores de aplicações escaláveis em nuvem.
* Site Reliability Engineering (Niall Richard Murphy, Betsy Beyer, Chris Jones e Jennifer Petoff): o manual oficial criado por engenheiros do Google detalhando como eles constroem, operam e mantêm sistemas massivamente escaláveis e altamente disponíveis.
* Cloud Computing Concepts, Technology and Architecture (Thomas Erl): um guia detalhado sobre a arquitetura corporativa em ambientes de nuvem, ideal para entender modelos de serviço em infraestruturas empresariais complexas.
* Normas ISO IEC 27017 e ISO IEC 27018: manuais oficiais da Organização Internacional de Padronização que tratam especificamente das diretrizes de segurança da informação e privacidade de dados em ambientes de computação em nuvem.