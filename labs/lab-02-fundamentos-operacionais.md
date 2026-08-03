# Laboratório 2 - Fundamentos Operacionais e Manipulação de Dados

## Objectivos

No fim deste laboratório, deverá ser capaz de:

- Identificar a natureza de um comando (executável, built-in, função ou alias) com `type` e `which`.
- Consultar a documentação do sistema com `man`, `--help`, `help`, `apropos` e `whatis`.
- Criar atalhos com `alias` e encadear comandos com `;`.
- Interpretar identidades e propriedades com `id`, `/etc/passwd` e `/etc/group`.
- Alterar permissões com `chmod` (notação octal e simbólica) e ajustar a máscara com `umask`.
- Reconhecer e aplicar as permissões especiais (setuid, setgid, sticky bit).
- Transferir propriedade de ficheiros com `chown` e `chgrp`.
- Redireccionar os fluxos de entrada, saída e erro (`>`, `>>`, `2>`, `&>`, `/dev/null`).
- Construir pipelines com filtros (`sort`, `uniq`, `wc`, `grep`, `head`, `tail`, `tee`).
- Arquivar e comprimir dados com `tar`, `gzip`, `bzip2` e `zip`.

## Pré-requisitos

- Ter lido o Capítulo 4.
- Uma máquina virtual com CentOS Stream. Todos os exercícios correm em modo texto, pelo que pode usar tanto a máquina sem interface gráfica como a máquina com interface gráfica (através de um terminal).
- Um utilizador comum com privilégios sudo (criado durante a instalação, ver Capítulo 2).
- Alguns exercícios beneficiam de duas sessões abertas em simultâneo — duas consolas virtuais, dois terminais, ou duas ligações SSH.


---

## Exercícios

### Parte A  (Natureza dos comandos e documentação)

#### Exercício 1 

Antes de confiar num comando, um administrador prudente verifica o que ele realmente é. Para cada um dos seguintes — `cd`, `ls`, `cp`, `pwd` e `cat` — descubra se se trata de um executável, um built-in do shell ou um alias, usando o `type`. Em seguida, para os que forem executáveis, descubra a sua localização física com o `which`. Porque é que o `which` não devolve nada para o `cd`?

#### Exercício 2 

1. Use o `apropos` (ou o equivalente `man -k`) para procurar comandos relacionados com `uptime` ou `boot`.
2. Escolha um candidato e confirme a sua finalidade numa única linha com o `whatis`.
3. Abra a man page completa desse comando, procure lá dentro pela palavra `load` usando `/`, e depois saia do paginador com `q`.

#### Exercício 3 

1. Obtenha o sumário rápido com `mkdir --help` e identifique essa opção.
2. Confirme a mesma informação na man page (`man mkdir`).
3. Para o built-in `cd`, obtenha a ajuda com o comando apropriado (que **não** é o `man`). Que comando usou e porquê?

### Parte B  (Aliases e encadeamento)

#### Exercício 4 

1. Execute essa sequência numa **única linha**, encadeando os três comandos com `;` (`cd /var/log`, `ls -lh`, `cd -`).
2. Antes de criar um atalho, verifique com o `type` que o nome `verlogs` ainda não está ocupado por outro comando.
3. Crie um alias chamado `verlogs` que execute a sequência anterior.
4. Teste o alias. Depois remova-o com `unalias` e confirme com o `type` que deixou de existir.

> Um alias criado assim dura apenas na sessão actual. Para o tornar permanente, a definição colocar-se-ia no ficheiro `~/.bashrc`.

### Parte C  (Identidade e propriedade)

#### Exercício 5 

1. Execute `id` e identifique o seu UID, o seu GID principal e os grupos secundários a que pertence.
2. Confirme a sua própria linha no ficheiro `/etc/passwd` (use o `grep` para filtrar).
3. Verifique a que grupos pertence, consultando o ficheiro `/etc/group`.

#### Exercício 6 

Um novo colega, o Bruno, precisa de receber um ficheiro de configuração seu. Este exercício reproduz um cenário clássico de transferência de propriedade.

1. Crie o utilizador `bruno` (precisará de privilégios administrativos) e defina-lhe uma password.
2. Com a sua conta normal, crie um ficheiro `~/config_partilha.conf` com uma linha de texto lá dentro.
3. Copie-o para o directório pessoal do Bruno usando `sudo` (`sudo cp ~/config_partilha.conf ~bruno`).
4. Verifique com `sudo ls -l ~bruno/config_partilha.conf` quem é o dono do ficheiro copiado. Ele conseguirá editá-lo? Porquê?
5. Corrija a situação, transferindo a propriedade (utilizador **e** grupo) para o Bruno com um único `chown`. Confirme o resultado.

#### Exercício 7 

Uma equipa vai partilhar ficheiros através de um grupo comum.

1. Crie o grupo `projecto` (privilégios administrativos).
2. Crie um ficheiro `~/nota_equipa.txt`.
3. Altere apenas o **grupo** do ficheiro para `projecto`, primeiro com o `chgrp` e depois, num ficheiro novo, usando a sintaxe equivalente do `chown` (`chown :projecto ...`). Confirme com `ls -l` que ambos os métodos produzem o mesmo efeito.

### Parte D  (Permissões)


#### Exercício 8 

1. No seu directório pessoal, crie um directório `~/laboratorio` e, dentro dele, um ficheiro `relatorio.txt` com algum conteúdo.
2. Observe as permissões actuais com `ls -l`.
3. Usando **notação octal**, defina as permissões do ficheiro para: leitura e escrita para o dono, apenas leitura para o grupo, e nada para os outros. (Que número octal corresponde a esta combinação?)
4. Confirme o resultado com `ls -l`.

#### Exercício 9 

Sobre o mesmo `relatorio.txt`, usando agora a **notação simbólica**:

1. Adicione permissão de execução apenas ao dono.
2. Remova a permissão de leitura aos outros (repare no que muda, ou não muda, em relação ao estado anterior).
3. Num único comando, defina o grupo e os outros com **apenas** leitura (use o operador `=`).

#### Exercício 10 


1. Crie um ficheiro `~/laboratorio/backup.sh` com uma linha: `echo "backup executado"`.
2. Tente executá-lo com `./backup.sh`. Observe a mensagem de erro.
3. Conceda permissão de execução ao dono e corra-o novamente. Funciona agora?

#### Exercício 11


1. Liste `/tmp` com `ls -ld /tmp` e repare no caractere especial no fim das permissões. Que atributo é esse e que problema resolve num directório partilhado por todos?
2. Crie um directório partilhado `/srv/equipa` (com `sudo`), atribua-lhe o grupo `projecto` e aplique-lhe o bit **setgid**.
3. Crie um ficheiro dentro de `/srv/equipa` e confirme com `ls -l` que ele herdou o grupo `projecto` em vez do seu grupo principal. Porque é que isto é útil para uma equipa?

### Parte E  (Redirecionamento de fluxos)

#### Exercício 12 

1. Guarde a listagem detalhada de `/usr/bin` no ficheiro `~/laboratorio/binarios.txt` (redireccionamento de saída).
2. **Acrescente** ao mesmo ficheiro a listagem de `/usr/sbin`, sem apagar o conteúdo anterior.
3. Confirme com `wc -l` quantas linhas o ficheiro passou a ter no total.

### Parte F  (Pipelines e filtros)

#### Exercício 15 

Quantos programas únicos existem entre `/bin` e `/usr/bin`?

1. Construa uma pipeline que junte as duas listagens, as ordene, remova duplicados e conte as linhas (`sort | uniq | wc -l`).
2. Modifique a pipeline para mostrar **apenas** os nomes que aparecem nos dois directórios (as linhas duplicadas), com `uniq -d`.

#### Exercício 16 


1. Liste todas as linhas que contêm `http`, ignorando maiúsculas/minúsculas.
2. Repita, mas destacando o termo a cor (`--color`).
3. Conte quantas linhas contêm o termo `tcp`.
4. Mostre todas as linhas que **não** são comentários (as que não começam por `#`). Dica: `grep -v '^#'`.
5. Por fim, em `/etc`, liste apenas os **nomes** dos ficheiros que contêm a palavra `server` (procura recursiva, apenas nomes de ficheiro, erros de permissão descartados).

#### Exercício 17 


1. Na primeira sessão, acompanhe em tempo real o log de autenticação com `sudo tail -f /var/log/secure`.
2. Na segunda sessão, gere alguma actividade de autenticação (por exemplo, execute um `sudo` qualquer, ou faça login numa consola virtual).
3. Observe as novas linhas a surgir instantaneamente na primeira sessão. Termine a monitorização com Ctrl+C.

#### Exercício 18 

Guarde a listagem completa de `/usr/bin` no ficheiro `~/laboratorio/inventario.txt` e, em simultâneo, mostre no ecrã apenas os programas cujo nome contém `zip` — tudo num único comando, usando o `tee`.

### Parte G  (Arquivamento e compressão)

#### Exercício 19 

1. Comprima o ficheiro `~/laboratorio/binarios.txt` com o `gzip` e compare o tamanho antes e depois com `ls -l`.
2. Inspeccione o conteúdo do ficheiro comprimido **sem** o descomprimir para o disco (`zcat` ou `zless`).
3. Descomprima-o de novo.

#### Exercício 20 

Faça uma cópia de `~/laboratorio/relatorio.txt` (ou reutilize o `binarios.txt`) e comprima-a com o `bzip2`. Compare o tamanho resultante com o do ficheiro `.gz` do exercício anterior. Qual das ferramentas comprimiu mais? Qual lhe pareceu mais lenta?

#### Exercício 21 

1. Crie um arquivo `~/laboratorio_backup.tar` de todo o directório `~/laboratorio/`, usando os modos de criação, verbosidade e destino.
2. Liste o conteúdo do arquivo **sem** o extrair.
3. Crie agora, num único passo, um arquivo comprimido `~/laboratorio_backup.tar.gz` (com a opção de compressão `z`).
4. Compare os tamanhos do `.tar` e do `.tar.gz`.
5. Extraia o `.tar.gz` para um directório novo `~/restauro/` e confirme que a estrutura foi reconstruída correctamente.

#### Exercício 22 

1. Crie um arquivo `~/laboratorio.zip` com todo o conteúdo de `~/laboratorio/` (atenção ao modificador de recursividade `-r`).
2. Descomprima-o para outro local com o `unzip` e confirme o conteúdo.
3. O que teria acontecido se tivesse esquecido a opção `-r`?

---

## Desafio 

Cenário: é responsável por um servidor e precisa de produzir um pacote de diagnóstico e, ao mesmo tempo, organizar as permissões de uma pasta de equipa.

1. Crie a estrutura `~/diagnostico/sistema/`, `~/diagnostico/logs/` e `~/diagnostico/relatorios/` num único comando.
2. Guarde em `~/diagnostico/sistema/binarios.txt` a lista ordenada e sem duplicados de todos os programas em `/bin` e `/usr/bin`. Acrescente ao fim de um relatório `~/diagnostico/relatorios/resumo.txt` uma linha com o número total desses programas.
3. A partir do log do sistema (`/var/log/messages` ou `/var/log/secure`), extraia para `~/diagnostico/logs/erros_recentes.txt` todas as linhas que contenham `error` ou `fail` (ignorando maiúsculas/minúsculas), descartando as mensagens de permissão negada.
4. Crie um grupo `suporte` e um directório `~/diagnostico/partilha/` cujo grupo seja `suporte` e que tenha o bit **setgid** activo, de modo a que os ficheiros criados lá dentro herdem automaticamente o grupo.
5. Ajuste as permissões de todo o `~/diagnostico/`, de forma recursiva, para que: o dono tenha controlo total, o grupo tenha apenas leitura (e execução/entrada nos directórios), e os outros não tenham qualquer acesso. Pense na diferença entre ficheiros e directórios ao escolher o comando.
6. Empacote e comprima todo o `~/diagnostico/` num único ficheiro cujo nome inclua a data do dia — por exemplo `diagnostico_$(date +%F).tar.gz`.
7. Verifique o conteúdo do arquivo final sem o extrair.

Verifique cada passo à medida que avança. Se algo não funcionar como esperado, investigue antes de consultar a solução.
