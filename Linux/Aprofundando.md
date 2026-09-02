# Linux Aprofundado

Aqui veremos alguns conceitos mais detalhados do Linux, começando com o papel do Kernel. Veremos também Ambientes comuns, Distros famosas e ideias como o conceito de arquivos do mundo Unix, o sistema/árvore de arquivos, permissões de usuários e grupos, conceito e gerenciamento de processos, manutenção de drivers e áudio, e customização.

## Kernel

O kernel é a parte principal e o que da nome à todos os sistemas que o usam, desenvolvido em C em 1991 por Linus Torvalds como um projeto pessoal, porém rapidamente disponibilizado publicamente por redes de FTP (File Transfer Protocol, sistema muito utilizado na época para compartilhamento de arquivos), onde seria então desenvolvido em conjunto com a comunidade interessada.

O sistema é fortemente inspirado por Unix e Minix (Mini Unix, desenvolvido pelo pesquisador de Sistemas Operacionais e escritor Andrew S. Tanenbaum). O Unix teve seu código fonte publicado para universidades e instituições, o que fez com que muitos pesquisadores o adotassem e o utilizassem, levando assim a virar um padrão. Fora herdado do sistema Unix características como tudo ser um arquivo, paradigma multitarefa e multiusuário.

## Ambientes

Os ambientes desktop (desktop environments, ou DEs), são responsáveis por oferecer interface gráfica, utilidades padrões do sistema, gerenciamento de janelas, gerenciadores de arquivos e outras utilidades gráficas comuns para o usuário.

Os ambientes mais famosos são Xfce, ambiente leve com baixo consumo de recursos, i3, um gerenciador de janelas simples baseado em X11 com tiling, Gnome, um sistema organizado e mínimo, KDE, ambiente moderno e rico em funcionalidades, e Hyprland, um gerenciador de janelas leve baseado em Wayland com tiling.

## Sistemas de arquivos

Como uma das filosofias principais do Linux é a questão de tudo ser um arquivo, partes importantes como gerenciamdento de entrada e saída, conexões de rede e informações de processos são todos arquivos no sistema

Durante a instalação, é comum o instalador oferecer diversas opções de sistemas de arquivos, como ext4, btrfs, zfs e ntfs, cada um com uma maneira diferente de manter os arquivos na memória persistente da máquina, mas mesmo com diversos sistemas, a estrutura de arquivos costuma ter uma estrutura comum entre si.

A estrutura padrão de arquivos em sistemas Linux segue uma estrutura de árvore, com a raíz da mesma sendo o diretório '/', que inclui diversos outros diretórios importantes para o sistema, como o '/home/', que mantém informações do usuário e onde a grande parte das instalações feitas vão ficar. Também é na raíz que fica a pasta de arquivos binários executáveis comuns para o sistema, a pasta '/bin/'. Outras pastas importantes são a '/dev/', onde ficam os dispositivos (_dev_ices) do sistema, e a pasta '/boot/', que guarda informações utilizadas pelo bootloader para iniciar o kernel e posteriormente o resto do sistema.

## Permissões

O acesso a arquivos no sistema é controlado pelo kernel dividindo o acesso em dono do arquivo, grupo do dono do arquivo e outros. Veja as permissões de um arquivo com 'ls -l', será possível ver uma seção com os dados num formato similar a 'rwxrwxrwx', onde r = read, w = write e x = execute. Por padrão, o usuário raiz pode ler e escrever todos os arquivos do sistema, e também poderá executar qualquer arquivo que seja definido como executável.

É comum as permissões serem vistas como um número de três dígitos equivalente à junção de cada tripla rwx como um número de 0 a 7: r = 4, w = 2, x = 1, assim, para definir que um arquivo pode ser lido, escrito e executável para todos usuários, pode ser usado o comando 'chmod 777 arquivo'.

## Processos

Toda tarefa/processo sendo executável recebe um PID, um identificador de processo único no sistema que serve para monitorar e "conversar" com o processo, com comandos como 'top' para visualização de processos e 'kill' para envio de sinais para eles.

## Drivers

Os drivers do Linux são implementados principalmente como módulos do kernel que podem ser carregados ou descarregados dinamicamente sem reiniciar o sistema. O comando 'lsmod' lista os módulos ativos, enquanto o 'modprobe' gerencia o carregamento. Há uma distinção importante entre drivers de código aberto integrados ao kernel e drivers proprietários (como os da NVIDIA).

## Áudio

Hoje em dia, é comum um sistema de uso diário normal ter integrado o sistema ALSA (Advanced Linux Sound Architecture), criado para substituir o Open Sound System, que conversa diretamente com os dispositivos de placas de som e áudio, junto ao sistema moderno PipeWire, um servidor de áudio de baixa latência.

## Customização

Um dos pontos mais comentados pela comunidade Linux é a alta customização, conhecida popularmente como ricing, é a prática de configurar 'dotfiles', arquivos que normalmente se encontram em pastas ou arquivos que comecem com ponto (muito comum para arquivos de configurações), além da instalação de diversos utilitários e temas para criar um ambiente único que encaixe especificamente o uso diário de um usuário.
