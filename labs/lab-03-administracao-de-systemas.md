
# Laboratório 3 - Administração de Sistemas Linux

## Objectivos

No fim deste laboratório, deverá ser capaz de:

- Criar e editar ficheiros com o `vi`/Vim, dominando os modos, a navegação, a edição e a pesquisa/substituição.
- Transformar texto de forma não interactiva com o `sed` (substituição, endereçamento, edição in-place).
- Gerir o ciclo de vida de contas com `useradd`, `passwd`, `usermod` e `userdel`, e de grupos com `groupadd`.
- Delegar privilégios com o grupo `wheel` e o `visudo`.
- Monitorizar sessões activas (`who`, `w`, `last`) e comunicar com utilizadores (`wall`, `write`, `mesg`).
- Observar e gerir processos (`ps`, `top`, jobs, sinais, `kill`/`pkill`/`killall`, `nice`/`renice`).
- Agendar tarefas recorrentes com o `cron` e tarefas únicas com o `at`.
- Analisar o sistema através dos logs (`journalctl`, `logrotate`) e da informação de arquitectura.

## Pré-requisitos

- Ter lido o Capítulo 5.
- Uma máquina virtual com CentOS Stream. Todos os exercícios correm em modo texto, pelo que servem tanto a máquina sem interface gráfica como a máquina com interface gráfica (através de um terminal).
- Um utilizador comum com privilégios sudo (criado durante a instalação, ver Capítulo 2).
- Alguns exercícios beneficiam de duas sessões abertas em simultâneo — duas consolas virtuais, dois terminais, ou duas ligações SSH.


---

## Exercícios

### Parte A  (Edição de texto com vi/Vim)

#### Exercício 1 

1. Abra um ficheiro novo com `vi ~/servidor_notas.txt`.
2. Entre em modo de inserção (`i`) e escreva quatro linhas: `hostname`, `ip`, `admin` e `estado` (com valores à sua escolha).
3. Regresse ao modo de comando com `Esc` e guarde e saia com `:wq`.
4. Reabra o ficheiro, acrescente uma linha nova por baixo da segunda com o comando `o`, e guarde de novo. Que diferença há entre `i`, `a`, `A` e `o`?

#### Exercício 2 

1. Copie o ficheiro `/etc/services` para `/tmp` (`cp /etc/services /tmp/`).
2. Abra `/tmp/services` no `vi` e procure o termo `WorldWideWeb` com `/WorldWideWeb`.
3. Altere essa ocorrência para que passe a ler `World Wide Web`.
4. Escolha um parágrafo de comentário (linhas começadas por `#`), corte-o com `dd` (ou `Ndd` para várias linhas), vá ao fim do ficheiro com `G` e cole-o aí com `p`.
5. Guarde as alterações.

#### Exercício 3 

Ainda em `/tmp/services`, pratique a substituição global em modo ex (comandos iniciados por `:`).

1. Substitua **todas** as ocorrências do termo `tcp` por `WHATEVER` em todo o ficheiro, distinguindo maiúsculas de minúsculas (`:%s/tcp/WHATEVER/g`).
2. Repita a mesma ideia noutra palavra, mas desta vez pedindo confirmação em cada ocorrência (flag `c`). O que fazem as respostas `y`, `n`, `a` e `q`?
3. Saia **sem guardar** (`:q!`), preservando o ficheiro original das experiências.

### Parte B  (Edição de fluxo com sed)

#### Exercício 4 

Quando é preciso alterar valores em ficheiros sem os abrir, o `sed` é a ferramenta certa. Trabalhe sobre uma cópia: `cp /etc/services /tmp/services2`.

1. Mostre no ecrã (sem alterar o ficheiro) o resultado de substituir `tcp` por `TCP`, apenas para pré-visualizar.
2. Confirmado o resultado, aplique a alteração **directamente ao ficheiro**, guardando automaticamente uma cópia de segurança `.bak` (opção `-i.bak`).
3. Confirme que existem agora dois ficheiros: o modificado e o `.bak` original.

> A opção `-i` sobrescreve o ficheiro sem confirmação. Teste sempre o comando **sem** `-i` primeiro e garanta que existe uma cópia de segurança antes de o aplicar a ficheiros importantes.

#### Exercício 5 

1. Usando `sed`, mostre `/etc/services` **sem** as linhas de comentário (as que começam por `#`) e **sem** as linhas em branco, num único comando (`sed -e '/^#/d' -e '/^$/d'`).
2. Usando `sed -n` com o comando `p`, imprima **apenas** as linhas 10 a 20 desse ficheiro.
3. Encaminhe o resultado do ponto 1 para o `wc -l` e diga quantas linhas "úteis" o ficheiro tem.

### Parte C  (Contas de utilizador e grupos)

#### Exercício 6 

1. Crie o utilizador `jbaxter` com o nome completo "John Baxter", usando `/bin/sh` como shell de login e deixando o UID ser atribuído por defeito. (Lembre-se das opções `-m`, `-s` e `-c`.)
2. Defina-lhe uma password.
3. Force a mudança de password no primeiro login.
4. Confirme a entrada da conta em `/etc/passwd` e identifique os sete campos dessa linha.

#### Exercício 7 

1. Crie um grupo chamado `testing` com o GID `315` (`groupadd -g 315 testing`).
2. Adicione o `jbaxter` aos grupos `testing` e `wheel`, **sem** o remover dos grupos que já tem (atenção à diferença entre `-aG` e `-G`).
3. Confirme a pertença consultando `/etc/group` e/ou `id jbaxter`.
4. Abra uma sessão como `jbaxter` (`su - jbaxter`), torne o `testing` o grupo activo com `newgrp testing`, crie um ficheiro `~/ficheiro.txt` com `touch` e confirme com `ls -l` que o grupo atribuído ao ficheiro é `testing`.

#### Exercício 8 


1. Anote o UID atribuído ao `jbaxter` (`id -u jbaxter`).
2. Bloqueie a conta com `usermod -L jbaxter` (boa prática antes de remover).
3. Elimine a conta **preservando** o directório home (`userdel` **sem** `-r`).
4. Procure em `/home` todos os ficheiros que ainda pertencem ao UID que era do `jbaxter` (`find /home -uid <UID>`). Porque é que o `find` mostra agora um número em vez de um nome?

> `userdel -r` remove o home e o correio do utilizador; sem `-r`, os ficheiros ficam órfãos com o UID numérico visível. Confirme sempre que ninguém tem processos activos antes de eliminar uma conta.

#### Exercício 9 

1. Copie o ficheiro `/etc/services` para `/etc/skel/` (com `sudo`).
2. Crie um novo utilizador `mjones`, com o nome completo "Mary Jones" e directório home em `/home/maryjones` (opção `-d`, mais `-m`).
3. Confirme que o ficheiro que colocou em `/etc/skel/` apareceu automaticamente em `/home/maryjones/`.
4. Encontre todos os ficheiros abaixo de `/home` que pertencem ao `mjones` (`find /home -user mjones`). Há algum que não esperava encontrar?

### Parte E  (Sessões e comunicação)

#### Exercício 11 

1. Veja as sessões activas com o `who` e depois, com mais detalhe de actividade, com o `w`. Interprete a primeira linha do `w` (uptime e *load average*).
2. Consulte o histórico de logins com o `last`.
3. Consulte as tentativas de login **falhadas** com `sudo lastb`. Porque é que este comando exige privilégios administrativos e que valor de segurança tem?

#### Exercício 12 

1. Envie um aviso a todos os utilizadores activos com `sudo wall "Manutenção às 22h"`.
2. Numa segunda sessão (outra consola/terminal), com outro utilizador, envie uma mensagem direccionada com o `write`, terminando com Ctrl+D.
3. Desactive a recepção de mensagens no seu terminal com `mesg n`, peça a alguém (ou à outra sessão) para lhe escrever, e observe o resultado. Reactive com `mesg y`.

### Parte F  (Processos)

#### Exercício 13 

1. Liste **todos** os processos com o conjunto completo de colunas e encaminhe o resultado para o `less`, para poder percorrê-lo (`ps aux | less`).
2. Liste todos os processos ordenados pelo **nome do utilizador** que os corre (dica: `ps aux --sort=user`, ou `ps -ef | sort`).
3. Liste os processos mostrando apenas estas colunas: PID, utilizador, grupo, memória virtual, memória residente e comando (`ps -eo pid,user,group,vsz,rss,comm`).

#### Exercício 14 

1. Lance o `top` e observe o cabeçalho (tarefas, uso de CPU, memória, *load average*).
2. Alterne a ordenação entre consumo de CPU (`P`) e consumo de memória (`M`).
3. Filtre para ver apenas os processos de um utilizador (`u`).
4. Saia com `q`.

#### Exercício 15 

1. Lance `sleep 600` e suspenda-o de imediato com Ctrl+Z. Confirme o estado com `jobs`.
2. Retome-o em segundo plano com `bg %1` e verifique que ficou `Running`.
3. Descubra o PID (`jobs -l` ou `ps`), envie-lhe o sinal de paragem `STOP` (`kill -STOP <PID>`), confirme que passou a `Stopped`/`T`, e volte a activá-lo com o sinal `CONT` (`kill -CONT <PID>`).
4. Termine-o de forma ordenada com `kill <PID>` (sinal `TERM`, o padrão).

#### Exercício 16 

1. Lance três processos em segundo plano: `sleep 500 &` três vezes.
2. Confirme que existem com `pgrep -a sleep` (ou `ps aux | grep sleep`).
3. Termine-os **todos de uma vez** pelo nome, com `pkill sleep` (ou `killall sleep`).
4. Explique por que razão se deve tentar sempre o sinal `TERM` antes de recorrer ao `KILL` (`-9`).

> `kill -9` não dá ao processo a hipótese de fechar ficheiros nem de guardar estado. Em serviços como bases de dados, um `KILL` forçado pode deixar dados inconsistentes. Use-o só como último recurso.

#### Exercício 17 


1. Lance um processo com uma prioridade reduzida: `nice -n 5 sleep 600 &`.
2. Confirme o valor de *niceness* com `ps -o pid,ni,comm -p <PID>` (coluna `NI`).
3. Altere a prioridade desse processo para `7` com o `renice` (`sudo renice 7 <PID>`) e confirme a mudança.
4. Recorde a lógica: um valor de *niceness* mais **alto** significa prioridade mais **baixa**. Porque é que só o root pode atribuir valores negativos?

### Parte G  (Agendamento de tarefas)

#### Exercício 18 


1. Edite o seu crontab com `crontab -e` e agende um script (ou comando) para correr **todos os dias às 2:30 da manhã**, descartando a saída (`> /dev/null 2>&1`).
2. Adicione uma segunda linha que registe o espaço em disco **de hora em hora** num ficheiro de log (`0 * * * * df -h >> ~/disk_usage.log`).
3. Liste o seu crontab com `crontab -l` e confirme as duas entradas. Identifique os cinco campos de tempo de cada linha.

#### Exercício 19 


1. Agende um comando para correr daqui a 5 minutos (`echo "tarefa concluida" > ~/at_resultado.txt` via `at now + 5 minutes`).
2. Liste as tarefas pendentes com `atq`.
3. Cancele a tarefa com `atrm <número>` antes de ela correr — ou deixe-a correr e confirme o ficheiro criado.

### Parte H  (Observabilidade do sistema)

#### Exercício 20 

1. Com o `journalctl`, veja os logs do serviço SSH (`journalctl -u sshd`), depois apenas as mensagens de nível de erro ou mais urgente (`-p err`) e, por fim, as mensagens desde as 06:00 de hoje (`--since`).
2. Acompanhe os logs em **tempo real** com `journalctl -f` (termine com Ctrl+C).
3. Faça uma simulação de rotação de logs, sem alterar nada, com `sudo logrotate -d /etc/logrotate.conf` (*dry run*).
4. Recolha informação de arquitectura do sistema: versão do kernel (`uname -r`), CPU (`lscpu`), memória (`free -h`) e discos (`lsblk`).

---

## Desafio 

Cenário: acabou de assumir a administração de um servidor CentOS e precisa de preparar contas, automatizar uma verificação e produzir um pequeno relatório de estado.

1. Crie um grupo `operacoes` e dois utilizadores, `op1` e `op2`, ambos com home próprio, shell Bash e pertencentes a esse grupo. Force ambos a mudar a password no primeiro login.
2. Conceda ao `op1`, através do `visudo`, permissão para executar **apenas** `systemctl restart sshd` como root — e mais nada.
3. Usando o `sed`, crie a partir de `/etc/services` um ficheiro `~/servicos_limpos.txt` sem comentários nem linhas em branco, e diga quantas linhas úteis ficaram.
4. Agende, no crontab, uma tarefa que **de 10 em 10 minutos** acrescente a saída do `w` (quem está ligado) a `~/sessoes.log`, com data incluída (dica: `date; w`).
5. Identifique o processo que mais CPU está a consumir neste momento (com `ps` ou `top`) e registe o seu PID, utilizador e comando num ficheiro `~/relatorio_processos.txt`.
6. Com o `journalctl`, extraia para `~/erros_boot.txt` todas as mensagens de erro (`-p err`) do arranque actual (`-b`).
7. Produza um resumo do sistema (kernel, CPU, memória, discos) e acrescente-o ao fim de `~/relatorio_processos.txt`.

Verifique cada passo à medida que avança. Se algo não funcionar como esperado, investigue antes de consultar a solução.
