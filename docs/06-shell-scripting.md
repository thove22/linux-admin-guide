# Capítulo 6 — Automação com Shell Scripting
 
## 1. O Kernel, a Shell e o Terminal
 
### 1.1 O papel do kernel
 
O kernel é o núcleo do sistema operativo Linux. É o componente que arranca primeiro quando o sistema liga, que gere directamente o hardware  CPU, memória, discos, interfaces de rede — e que serve de árbitro entre todos os programas que precisam de aceder a esses recursos. Nenhum programa de utilizador comunica directamente com o hardware: tudo passa pelo kernel, através de um mecanismo controlado chamado chamadas de sistema (*system calls*).
 
Um sistema Linux completo é composto por várias camadas: o kernel em baixo, as bibliotecas e utilitários GNU em cima, e no topo um shell que une tudo e o expõe ao utilizador de forma utilizável. Sem o kernel, nada corre. Sem o shell, o utilizador não tem forma prática de interagir com o que o kernel oferece.
 
### 1.2 O que é o shell
 
O shell é um interpretador de comandos. Recebe texto digitado pelo utilizador (ou lido de um ficheiro), interpreta-o e traduz-o em instruções que o sistema operativo executa. É a camada de ligação entre o ser humano e o kernel.
 
Quando se escreve `ls -l /etc` num terminal, é o shell que analisa essa linha, identifica que `ls` é um programa externo, constrói os argumentos correctos e pede ao kernel que execute o programa. O kernel cria um processo, o programa corre, escreve para o stdout, e o shell apresenta o resultado.
 
O shell não é apenas um executor de comandos isolados. Tem a sua própria linguagem de programação, com variáveis, condicionais, ciclos, funções e substituições. É esta capacidade que transforma o shell numa ferramenta de automação poderosa.
 
### 1.3 Os diferentes tipos de shell
 
Existem várias shells disponíveis em sistemas Linux, cada uma com características próprias:
 
- **Bash** (*Bourne Again SHell*) é o shell por defeito na maioria das distribuições Linux, incluindo o CentOS. É retro-compatível com o shell `sh` original do UNIX e incorpora funcionalidades úteis do Korn Shell e do C Shell. É o shell que cobre este capítulo.
 
- **sh** (*Bourne Shell*) é o shell original do UNIX, desenvolvida por Stephen Bourne nos anos 70. O Bash é essencialmente uma extensão moderna do `sh`. Muitos scripts de sistema são escritos em `sh` puro para garantir portabilidade máxima entre sistemas Unix e Linux.
 
- **zsh** (*Z Shell*) é umo shell moderna com funcionalidades avançadas de auto-completação, temas e personalização. É o shell por defeito no macOS desde 2019 e é popular entre utilizadores que passam muito tempo no terminal.
 
- **ksh** (*Korn Shell*) foi desenvolvida na Bell Labs e é comum em sistemas AIX e Solaris. Introduziu várias funcionalidades que mais tarde foram incorporadas no Bash.
 
- **csh** e **tcsh** (*C Shell* e *TENEX C Shell*) têm uma sintaxe inspirada na linguagem C. São menos comuns em scripting de sistema mas ainda aparecem em ambientes Unix mais antigos.
 
Para saber qual o shell activa na sessão actual:
 
```bash
$ echo $SHELL
/bin/bash
 
$ echo $0
bash
```
 
Para listar todas as shells disponíveis no sistema:
 
```bash
$ cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/zsh
/usr/bin/zsh
```
 
---
