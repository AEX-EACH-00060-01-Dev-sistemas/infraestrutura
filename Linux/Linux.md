# Linux

Linux é um kernel de código aberto desenvolvido por Linus Torvalds como um projeto pessoal de estudo e se tornou o sistema operacional mais utilizado do mundo, tendo sido utilizado em mais de 80% dos dispositivos de telefones celulares brasileiros com o sistema android, em torno de 70% do de servidores o utilizam, e sendo a escolha padrão para diversos dispositivos menores embarcados pelo seu baixo requisito.

Desenvolvido em linguagem C no começo da década de 90, o projeto é uma implementação de um núcleo de um sistema operacional, a parte responsável por controlar o computador e processos que rodem no mesmo. Entender o Linux como um sistema operacional completo é um pouco errôneo pela diferença entre o chamado kernel e o user-space, parte responsável pelas aplicações que o usuário interage no dia-a-dia.

## Kernel

O kernel é um programa especial rodado na inicialização do computador, é o núcleo controlador, que gere outros programas e processos, controla sistemas periféricos, memória, sistema de arquivos e está sempre em memória sendo executado para tratar trocas de contexto e chamadas especiais de programas do user-space (syscalls)

## Distros

Pela separação de funções entre kernel e user-space, o Linux é utilizado ao lado de outros tipos de programa, como compositores de telas, editores de texto, navegadores e outras aplicações, gráficas ou não. Essa junção do kernel com aplicações pro usuário é chamada de distro, ou distribuição, já que é uma das formas que o kernel Linux é distribuído para a comunidade.

![Árvore genética do Linux](https://tuxcare.com/wp-content/uploads/2025/01/05_linuxtree.png)

As distros se diferem primariamente na experiência do usuário visada e no objetivo geral da mesma, algumas buscam uma experiência leve e simples, outras buscam ser mais robustas ou parecidas com outros sistemas operacionais maiores do mercado, e algumas tem objetivos mais específicos de oferecer alguma função imediata

## Download

Para chegarmos a etapa de uso do sistema, precisamos escolher um método de colocar ele na nossa máquina, sendo o mais comum deles o download de uma distro em formato iso.

Para baixar a iso de sua escolha, vá para o site da distro escolhida (aqui será usado Fedora como exemplo) e procure pelo download que especifique o formato do arquivo como .iso para a arquitetura do seu processador (muito provavelmente x86 64bits). Ao terminar o download do arquivo, você precisará de um pendrive para a instalação e um programa q faça o pendrive utilizar o iso ao invés de simplesmente armazená-lo.

Ao instalar, é recomendável fazer o checksum da imagem baixada, que é um hash disponibilizado pela que diz se seu download veio com alguma diferença ou falha pelo download.

Para o exemplo foi utilizado o [ventoy](https://www.ventoy.net/en/download.html), aplicativo de código aberto que permite ter várias imagens em único usb, mas outras escolhas possíveis seriam a aplicação Balena Etcher ou um utilitário de linha de comando.

## Instalação

Após fazer a mídia bootavel com a imagem escolhida, conecte-a a máquina que você deseja e só então ligue ela. Uma das primeiras telas de início te permitirá apertar uma tecla como F12 para escolher a partir de qual dispositivo será feito o boot, escolha a mídia que tem a imagem recém-baixada.

O sistema escolhido normalmente apresenta um ambiente gráfico no qual você vai realizar escolhas para a instalação que você está fazendo, como fuso-horário do relógio, layout e idioma do teclado e particionamento do disco.

Após seguir as etapas de instalação, reinicialize a máquina e retire o pendrive para o boot acontecer a partir do disco recém formatado.

## Aprofundando

Entrando nas particularidades do sistema e algumas diferenças dele para outros, temos alguns tópicos importantes como o conceito de arquivos do mundo Unix, o sistema/árvore de arquivos, permissões de usuários e grupos, conceito e gerenciamento de processos, manutenção de drivers e áudio, e customização.

Para se aprofundar nesses tópicos e entender melhor as ramificações do SO, siga para o [próximo arquivo](./Aprofundando.md).
