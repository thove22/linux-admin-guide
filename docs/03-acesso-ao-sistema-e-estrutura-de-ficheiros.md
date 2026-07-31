# Acesso ao Sistema e Estrutura de Ficheiros


## Controlo de Acesso e Arquitetura de Privilégios

A segurança e a integridade de um sistema operativo Linux assentam na premissa de que nem todos os utilizadores possuem o mesmo nível de autoridade. Num ambiente de servidor, o controlo de acesso atua como o mecanismo central que avalia cada ação (como editar um ficheiro, reiniciar um serviço ou alterar configurações de rede) e emite um veredito sobre a sua permissibilidade. Compreender a hierarquia destas permissões é o primeiro passo crítico na administração de sistemas.

### O Superutilizador (root) e a Filosofia de Privilégios
Embora as distribuições modernas possuam múltiplos serviços de controlo, a base do modelo tradicional UNIX permanece inalterada: o sistema égovernado por uma conta administrativa omnipotente designada por root, também conhecida como o superutilizador.

Tecnicamente, a característica que define o root não é o nome de utilizador, mas sim o seu Identificador de Utilizador (UID) com o valor 0. Para o kernel (núcleo) do Linux, qualquer processo que opere com o UID 0 possui autoridade absoluta. 

#### Capacidades do Superutilizador:

- Pode contornar todas as restrições de leitura, escrita e execução do sistema de ficheiros.
- Pode alterar a titularidade de processos e ficheiros.
- Pode abrir portas de rede privilegiadas (portas abaixo da 1024, cruciais para serviços web e de email).
- Tem permissão exclusiva para invocar chamadas de sistema sensíveis, como a alteração do relógio do sistema ou a configuração de interfaces de rede.

##### Os Riscos da Operação Direta:

Historicamente, os administradores iniciavam sessão diretamente na conta root. No entanto, em infraestruturas modernas, esta prática é considerada um erro crítico de segurança. Operar permanentemente como root significa que qualquer erro de digitação pode destruir o sistema de forma irreversível. Além disso, se uma equipa de administradores partilhar a senha de root, perde-se totalmente a capacidade de auditoria: se ocorrer uma falha às 03:00 da manhã, os registos do sistema (logs) mostrarão apenas que o "root" executou a ação, impossibilitando a identificação do operador real.

### Delegação de Poderes e Transição de Identidade (su vs. sudo)

Para mitigar os riscos associados ao uso direto da conta root, os sistemas Linux implementaram ferramentas que permitem a delegação controlada de poderes administrativos. A distinção técnica e o caso de uso de cada uma são fundamentais.

#### O utilitário su (Substitute User):

O comando su permite a um utilizador alterar temporariamente a sua identidade no terminal para a de outro utilizador (por omissão, o root).

- **Mecanismo**: Ao executar su, o sistema solicita a palavra-passe do utilizador de destino (a senha do próprio root).
- **Problema**: Exige que a senha de root seja do conhecimento de vários administradores, mantendo a vulnerabilidade de partilha de credenciais e a ausência de um registo detalhado dos comandos executados após a transição.

#### O utilitário sudo (Superuser DO):

O sudo é o padrão moderno da indústria. Permite que um utilizador comum execute comandos específicos como se fosse o superutilizador.

- **Mecanismo**: Ao executar sudo [comando], o sistema solicita a palavra-passe do próprio utilizador que está a executar a ação (não a do root)
- **Vantagens**: O sistema verifica no ficheiro /etc/sudoers se esse utilizador tem autorização para executar aquela tarefa. Adicionalmente, o sudo gera um registo rigoroso (enviado para o syslog), documentando quem executou o comando, a partir de onde, e a que horas, garantindo rastreabilidade total.

#### A distinção estrutural: sudo vs. sudo su

Em certas ocasiões, um administrador necessita de executar não apenas um comando, mas uma longa sequência de tarefas administrativas, tornando moroso digitar sudo antes de cada linha. É aqui que surge a confusão com as transições de shell:
- **sudo su**: Esta combinação utiliza o poder do sudo para executar o comando su. Na prática, o administrador introduz a sua própria senha, e osistema abre uma consola permanente de root. Embora evite a necessidade de partilhar a senha do superutilizador, possui uma falha de auditoria: o sudo apenas regista que o utilizador iniciou o su. Todos os comandos destrutivos executados dentro dessa nova consola não ficam registados em nome do administrador original.

- **Boa Prática (sudo -i)**: A abordagem tecnicamente correta para obter uma sessão interativa de root é o comando sudo -i (ou sudo --login). Este comando simula um login limpo e completo com o perfil do superutilizador, carregando as suas variáveis de ambiente, mas fá-lo através dos mecanismos nativos do sudo, mantendo uma melhor integridade do sistema.

### Gestão de Credenciais e Palavras-passe (passwd)

Sendo o acesso validado mediante credenciais, a gestão destas é uma operação basilar. O comando passwd é o mecanismo utilizado para definir ou alterar senhas de acesso.

- Alteração Própria: Qualquer utilizador pode executar passwd no terminal para alterar a sua própria senha. O sistema exigirá a senha atual antes de permitir a definição da nova.

- Alteração Administrativa: Quando invocado com privilégios administrativos (ex: sudo passwd estudante), o administrador pode redefinir a senha de qualquer outro utilizador no sistema, dispensando a necessidade de conhecer a senha atual do mesmo. Esta funcionalidade é crucial para a recuperação de contas ou na rotatividade de credenciais de equipa.

Uma senha administrativa robusta não deve ser baseada apenas na complexidade visual, mas principalmente na sua extensão (frequentemente implementada através de "passphrases", frases-senha longas), minimizando a suscetibilidade a ataques de força bruta no ambiente de servidor.


## O Sistema de Ficheiros: Estrutura e Organização

A utilidade de um sistema operativo mede-se, em grande parte, pela eficiência com que gere e disponibiliza o acesso aos dados e aos recursos físicos da máquina. Em sistemas Linux e UNIX, o sistema de ficheiros transcende a mera função de armazenamento em disco; ele atua como o mecanismo central de organização. Compreende quatro componentes lógicos fundamentais: um espaço de nomes (namespace) hierárquico, uma interface de programação (API) para manipulação de objetos, um modelo de segurança e, por fim, a implementação de software que interliga este modelo abstrato ao hardware subjacente.

### A Filosofia "Tudo é um Ficheiro"

A premissa arquitetural mais célebre do UNIX, e por herança do Linux, dita que "tudo é um ficheiro". Embora seja uma ligeira simplificação, esta máxima é a chave para a elegância do sistema.

No seu nível mais rudimentar, um ficheiro para o kernel é apenas uma sequência bidimensional de bytes. O sistema operativo não impõe qualquer estrutura rígida ou significado a estes dados; a interpretação do conteúdo é estritamente delegada aos programas que os leem.

O poder desta filosofia reside na sua universalidade. O conceito de ficheiro é estendido para mapear quase todos os recursos do sistema. Discos rígidos, terminais de linha de comandos, adaptadores de rede, processos em execução e até canais de comunicação entre processos (pipes esockets) são todos representados e acedidos como se fossem ficheiros de texto comuns. Isto significa que as mesmas ferramentas padronizadas usadas para ler ou escrever num documento de texto podem ser empregues para enviar dados para um dispositivo físico ou ler métricas do processador, simplificando drasticamente a administração e a programação no sistema.

### A Árvore de Diretórios e a Norma FHS

O sistema de ficheiros é apresentado como uma hierarquia unificada, rigorosamente singular, que tem início num diretório de topo designado por raiz (representado pelo caractere / ou root directory). Em oposição a outros sistemas operativos que mantêm espaços de nomes segmentados por partições de disco (ex: C:\, D:\), o Linux anexa, ou "monta" (mounts), todos os sistemas de ficheiros e dispositivos em pastas vazias desta árvore única.

Para garantir a compatibilidade e a previsibilidade, as distribuições Linux aderem ao FHS (Filesystem Hierarchy Standard). Esta norma estabelece o propósito de cada subdiretório.
Abaixo, detalha-se a função dos diretórios padrão presentes em ambientes Linux:

- **/bin**: Contém os comandos nucleares do sistema operativo necessários para todos os utilizadores (ex: ls, cp, ping).

- **/boot**: Aloja o kernel do sistema operativo e os ficheiros estáticos cruciais para o processo de arranque (ex: carregador de arranque GRUB).

- **/dev**: Diretório de dispositivos (device nodes). Contém ficheiros especiais que representam o hardware, como discos, impressoras e pseudo-terminais.

- **/etc**: O centro nervoso da configuração do sistema. Contém exclusivamente ficheiros de configuração estáticos e scripts de inicialização globais. É aqui, por exemplo, que reside o diretório /etc/systemd, que aloja as configurações dos serviços geridos pelo daemon systemd.

- **/home**: A base de operações para os utilizadores do sistema. Contém os diretórios de trabalho pessoais (home directories), onde residem as configurações locais e os dados de cada utilizador.

- **/lib**: Bibliotecas partilhadas essenciais para os binários localizados em /bin e /sbin, e módulos do kernel. É também neste nível (em /lib/systemd) que residem as unidades de serviço padrão e a infraestrutura executável do systemd.

- **/media**: Pontos de montagem gerados automaticamente para sistemas de ficheiros em suportes removíveis (como drives USB ou CD-ROMs).

- **/mnt**: Pontos de montagem temporários, habitualmente utilizados pelo administrador do sistema para montar manualmente partições ou partilhas de rede provisórias.

- **/opt**: Diretório reservado para a instalação de pacotes de software opcionais ou de terceiros (aplicações monolíticas que não distribuem os seus ficheiros pela hierarquia padrão).

- **/proc**: Um sistema de ficheiros virtual em memória que expõe informações dinâmicas sobre os processos em execução e atua como uma interface para os parâmetros do kernel.

- **/root**: O diretório pessoal exclusivo do superutilizador (root). Por razões de segurança e integridade, não reside em /home.

- **/sbin**: Comandos e utilitários vitais para a operabilidade do sistema e tarefas de administração. Geralmente, requerem privilégios de superutilizador (ex: ferramentas de formatação ou configuração de rede).

- **/tmp**: Diretório para a criação de ficheiros temporários por aplicações ou utilizadores. O seu conteúdo é frequentemente expurgado durante o reinício do sistema.

#### A Hierarquia Secundária e Dados Variáveis:

Para além dos diretórios base na raiz, o sistema delega grande parte do software e dos dados operacionais a dois diretórios de elevada importância:

- **/usr**: Representa uma hierarquia secundária, de leitura obrigatória, para dados de sistema partilháveis.

    - **/usr/bin** e **/usr/sbin** : Maioria dos binários e comandos não essenciais para o arranque.

    - **/usr/include**: Ficheiros de cabeçalho (headers) para a compilação de programas em C/C++.

    - **/usr/lib** (e /usr/lib64): Bibliotecas e ficheiros de suporte a aplicações em espaço de utilizador.

    - **/usr/local**: Destinado a software compilado e instalado localmente pelo administrador, preservando-o de atualizações do sistema operativo.

    - **/usr/share**: Dados independentes da arquitetura de hardware, como manuais (man pages em /usr/share/man).

    - **/usr/src**: Código-fonte para software ou pacotes do kernel.

- **/var**: Destinado a ficheiros cujos tamanhos flutuam de forma dinâmica durante a operação do sistema. Colocar /var numa partição separada é uma prática comum para evitar que o crescimento de ficheiros esgote o espaço crítico do sistema.

    - **/var/log** (e historicamente /var/adm): Os registos do sistema (logs), fundamentais para a auditoria e resolução de problemas.

    - **/var/spool**: Filas de processamento para tarefas pendentes, como trabalhos de impressão ou correio eletrónico.

    - **/var/tmp**: Ficheiros temporários que, ao contrário do diretório /tmp principal, devem ser preservados após a reinicialização do sistema.

### Tipologia de Ficheiros

A flexibilidade do modelo de armazenamento do Linux baseia-se na consolidação de sete tipos distintos de ficheiros. Independentemente das inovações ou abstrações introduzidas no espaço de nomes do sistema (como a exposição de estruturas do núcleo em /proc), o kernel mascara estasentidades para que operem sob uma destas sete categorias fundamentais.

Para determinar a tipologia de um ativo existente no disco, utiliza-se o utilitário de listagem com a diretiva de detalhe e inspeção de diretórios:

```bash
    $ ls -ld [caminho]
```

O primeiro caractere da string de permissões gerada no terminal codifica de forma inequívoca a natureza do ficheiro.

#### A. Ficheiros Regulares (Símbolo: -)

Constituem a categoria mais comum no sistema, consistindo puramente numa sequência linear de bytes sobre a qual o sistema de ficheiros não impõe qualquer estrutura lógica interna. Esta categoria engloba documentos de texto plano, scripts, ficheiros de configuração, executáveis binários e bibliotecas partilhadas. O sistema permite tanto o acesso sequencial como o acesso aleatório aos dados contidos nestes ficheiros

#### B. Diretórios (Símbolo: d)

Um diretório é um ficheiro especial que atua como um contentor, armazenando referências nominativas (ligações ou links) para outros ficheiros ou subdiretórios. A criação é efetuada via mkdir e a remoção requer o comando rmdir (caso esteja vazio) ou rm -r (para remoção recursiva de estruturas não vazias).

Existem sempre duas entradas nativas em qualquer diretório: o ponto único (.), que referencia o próprio diretório, e os dois pontos (..), que referenciam o diretório pai. Na raiz do sistema (/), devido à inexistência de um nível superior, ambos os caminhos apontam para o próprio nó inicial.

#### C. Ficheiros de Dispositivo de Caractere (Símbolo: c)

Estes ficheiros servem como pontos de encontro (rendezvous) para a comunicação direta entre os programas em espaço de utilizador e os controladores de hardware (drivers) do kernel. Os dispositivos de caractere gerem o fluxo de entrada e saída (I/O) de forma linear e serial, caractere a caractere, delegando o encapsulamento e o varrimento de memória (buffering) ao próprio controlador. Exemplos típicos incluem terminais virtuais (/dev/ttyX) e portas série.

#### D. Ficheiros de Dispositivo de Bloco (Símbolo: b)

Semelhantes aos dispositivos de caractere, estabelecem a ponte com o hardware, mas são orientados à movimentação de dados em blocos de tamanho fixo. Neste modelo, os controladores exigem que o kernel execute e gira o buffering da informação em memória antes da escrita ou leitura física. As partições de discos rígidos e unidades de armazenamento (ex: /dev/sda1) são classificadas sob esta tipologia.

#### E. Links Simbólicos (Símbolo: l)

Um link simbólico (ou soft link) é um ficheiro distinto que contém, como única informação, uma string de texto representando o caminho (absoluto ou relativo) para outro ficheiro ou diretório alvo. Ao encontrar um link simbólico, o kernel redireciona a operação para o caminho de destino especificado. Por serem referências textuais, podem apontar para alvos localizados noutros sistemas de ficheiros ou mesmo para caminhos inexistentes (broken links).
A sua criação é feita com o comando: 

```bash
    $ ln -s [caminho_do_alvo] [caminho_do_link]

```
#### F. Pipes Nomeados (Símbolo: p)

Um pipe nomeado (também designado FIFO, de *First In, First Out*) é um ficheiro especial que estabelece um canal de comunicação unidireccional entre dois processos independentes. Ao contrário de um ficheiro regular, um pipe nomeado não armazena dados de forma persistente no disco: funciona como uma conduta, onde aquilo que um processo escreve numa extremidade é lido por outro processo na extremidade oposta, pela ordem exacta em que foi introduzido.

A sua utilidade reside em permitir que programas que não têm qualquer relação de parentesco directo troquem dados sem recorrer a ficheiros intermédios. A criação faz-se com o comando `mkfifo`.

#### G. Sockets (Símbolo: s)

Os sockets representam o mecanismo mais sofisticado de comunicação entre processos, permitindo a troca de dados bidireccional. Enquanto os pipes nomeados estabelecem um canal simples num único sentido, os sockets suportam diálogos complexos, sendo a base sobre a qual assentam inúmeros serviços do sistema.

Um socket de domínio UNIX permite a comunicação entre processos na mesma máquina através de um ficheiro no sistema de ficheiros. Serviços como bases de dados e o próprio daemon de gestão do sistema expõem sockets para que os clientes lhes enviem pedidos localmente, sem necessidade de atravessar a pilha de rede.

### Ligações Físicas e Simbólicas (Hard e Soft Links)

A introdução do link simbólico levanta uma questão natural: existirá uma outra forma de referenciar um mesmo ficheiro a partir de vários pontos da árvore de directórios? A resposta reside na distinção entre os dois tipos de ligações que o Linux disponibiliza, e compreendê-la exige um breve olhar sobre a forma como o sistema de ficheiros identifica internamente os seus objectos.

Cada ficheiro num sistema de ficheiros Linux é representado, ao nível interno, por uma estrutura chamada **inode**. O inode contém todos os metadados do ficheiro (permissões, dono, tamanho, datas e a localização dos dados no disco) com uma única excepção notável: o nome. O nome do ficheiro não pertence ao inode, mas sim à entrada de directório que aponta para ele. Esta separação entre o nome e a identidade real do ficheiro é a chave para compreender os dois tipos de ligação. Uma análise mais aprofundada dos inodes será apresentada no capítulo dedicado ao armazenamento.

#### Ligação Física (Hard Link)

Uma ligação física é uma segunda entrada de directório que aponta para o **mesmo inode** de um ficheiro existente. Na prática, o ficheiro passa a ter dois nomes, ambos com estatuto idêntico: nenhum é o "original" e nenhum é a "cópia". Ambos referenciam exactamente os mesmos dados no disco, e alterações feitas através de um nome são imediatamente visíveis através do outro.

O ficheiro só é efectivamente removido do disco quando a última entrada de directório que aponta para o seu inode é eliminada. Enquanto existir pelo menos uma ligação, os dados permanecem acessíveis.

```bash
    $ ln [ficheiro_alvo] [nome_da_ligacao]
```

As ligações físicas têm duas limitações estruturais importantes: não podem atravessar diferentes sistemas de ficheiros (porque um inode só tem significado dentro do sistema de ficheiros a que pertence) e não podem referenciar directórios.

#### Ligação Simbólica (Soft Link)

A ligação simbólica, já apresentada anteriormente, opera num nível diferente. Não partilha o inode do alvo: é um ficheiro autónomo, com o seu próprio inode, cujo conteúdo é meramente uma string de texto com o caminho para o alvo. Por esta razão, a ligação simbólica é flexível o suficiente para apontar para alvos noutros sistemas de ficheiros ou para directórios, mas fica inutilizável (*broken link*) se o alvo for removido ou movido.

A distinção prática resume-se ao seguinte: a ligação física é um segundo nome para os mesmos dados, robusto mas limitado ao sistema de ficheiros local; a ligação simbólica é um apontador para um caminho, flexível mas dependente da existência contínua do alvo.

| Característica | Ligação Física | Ligação Simbólica |
|----------------|----------------|-------------------|
| Partilha o inode do alvo | Sim | Não (tem inode próprio) |
| Atravessa sistemas de ficheiros | Não | Sim |
| Pode referenciar directórios | Não | Sim |
| Sobrevive à remoção do alvo | Sim | Não (fica quebrada) |
| Comando de criação | `ln` | `ln -s` |


## Navegação no Sistema de Ficheiros
 
Compreendida a estrutura estática da árvore de directórios, resta dominar o movimento através dela. A interacção com o sistema de ficheiros a partir da linha de comandos assenta num pequeno conjunto de comandos fundamentais e na noção de caminho.
 
### Caminhos Absolutos e Relativos
 
Sempre que se faz referência a um ficheiro ou directório, indica-se o seu **caminho** (*path*). Existem duas formas de o expressar, e a distinção é permanente no trabalho diário.
 
Um **caminho absoluto** descreve a localização de um objecto a partir da raiz do sistema, começando sempre pelo caractere `/`. O caminho `/usr/lib` é absoluto: identifica de forma inequívoca aquele directório, independentemente de onde o utilizador se encontre. Um caminho absoluto é como uma morada completa: válida a partir de qualquer ponto de partida.
 
Um **caminho relativo** descreve a localização de um objecto a partir do directório actual, e nunca começa por `/`. Se o utilizador estiver em `/usr`, o caminho relativo `lib` refere-se a `/usr/lib`. O mesmo caminho relativo, escrito a partir de outro directório, apontaria para outro sítio. Na prática, trabalha-se a maior parte do tempo com caminhos relativos, porque normalmente já se está no directório de interesse ou perto dele.
 
### Os Atalhos de Navegação
 
O sistema disponibiliza quatro símbolos que abreviam referências comuns e que aparecem constantemente em comandos e scripts.
 
| Símbolo | Referência |
|---------|-----------|
| `.` | O directório actual |
| `..` | O directório pai (o nível imediatamente acima) |
| `~` | O directório pessoal do utilizador (*home*) |
| `-` | O directório anterior (aquele onde se estava antes do último `cd`) |
 
O ponto único (`.`) refere-se ao directório actual. Se o utilizador estiver em `/usr/lib`, o caminho `.` continua a ser `/usr/lib`, e `./X11` equivale a `/usr/lib/X11`. Na maioria dos casos não é necessário usar o `.` explicitamente, uma vez que os comandos assumem por omissão o directório actual quando o caminho não começa por `/`.
 
Os dois pontos (`..`) referem-se ao directório pai. A partir de `/usr/lib`, o caminho `..` aponta para `/usr`, e `../bin` aponta para `/usr/bin`. Na raiz do sistema, por não existir um nível superior, tanto `.` como `..` apontam para a própria raiz.
 
### pwd: identificar o directório actual
 
Cada processo, incluindo a shell, mantém a noção de um **directório de trabalho actual** (*current working directory*): o directório em que se encontra num dado momento. O comando `pwd` (*print working directory*) revela esse directório, expresso como caminho absoluto.
 
```bash
    $ pwd
    /home/estudante
```
 
Ainda que o prompt da maioria das distribuições já indique o directório actual, o `pwd` fornece sempre o caminho absoluto completo e inequívoco, o que é útil em scripts e quando o prompt está configurado para mostrar apenas o nome curto do directório.
 
### cd: mudar de directório
 
O comando `cd` (*change directory*) altera o directório de trabalho actual da shell.
 
```bash
    $ cd /etc/systemd          # navegar para um caminho absoluto
    $ cd documentos            # navegar para um subdirectório (caminho relativo)
    $ cd ..                    # subir um nível
    $ cd                       # regressar ao directório pessoal (home)
    $ cd -                     # regressar ao directório anterior
```
 
Executado sem qualquer argumento, o `cd` devolve o utilizador ao seu directório pessoal, o mesmo que `cd ~`. O `cd -` alterna para o directório onde se estava imediatamente antes, o que é prático para saltar repetidamente entre dois locais de trabalho.
 
Convém notar um detalhe técnico: o `cd` é um comando interno da própria shell (*built-in*), e não um programa autónomo. A razão é estrutural. Se o `cd` fosse um programa externo, correria como um subprocesso da shell, e um subprocesso não pode, em circunstâncias normais, alterar o directório de trabalho do processo que o invocou. A mudança de directório teria efeito no subprocesso e desapareceria com ele, deixando a shell exactamente onde estava. Por isso, a capacidade de mudar de directório tem de residir na própria shell.
 
### mkdir e rmdir: criar e remover directórios
 
O comando `mkdir` (*make directory*) cria um novo directório:
 
```bash
    $ mkdir projectos
    $ mkdir -p projectos/2025/relatorios    # criar toda a hierarquia de uma vez
```
 
A opção `-p` cria os directórios intermédios em falta, permitindo construir uma estrutura completa num único comando em vez de criar cada nível separadamente.
 
O comando `rmdir` (*remove directory*) remove um directório, mas apenas se este estiver vazio:
 
```bash
    $ rmdir projectos
```
 
Se o directório contiver ficheiros ou subdirectórios, o `rmdir` falha. Esta recusa é uma protecção deliberada. Para remover um directório e todo o seu conteúdo de uma vez, recorre-se ao `rm` com a opção de remoção recursiva `-r`, que apaga repetidamente tudo o que se encontra no interior.
 
```bash
    $ rm -r projectos
```
 
> **O `rm -r` é um dos poucos comandos capazes de causar danos graves e irreversíveis**, sobretudo quando executado com privilégios de superutilizador. Não existe lixeira nem forma de desfazer. Deve-se verificar sempre o comando antes de o executar, e evitar em absoluto combinar o `-r` com wildcards como o asterisco (`*`), uma combinação que pode apagar muito mais do que se pretendia num instante.

## Wildcards (Globbing)
 
Ao trabalhar na linha de comandos, é frequente querer aplicar uma operação a um conjunto de ficheiros em vez de a um único. Listar todos os ficheiros de texto de um directório, apagar todos os ficheiros de registo antigos, ou copiar todas as imagens de uma pasta são tarefas em que especificar cada ficheiro individualmente seria impraticável. Os **wildcards** (ou *globbing*) resolvem este problema, permitindo descrever padrões que a shell expande para os nomes de ficheiros correspondentes.
 
### O mecanismo: expansão pela shell
 
O ponto fundamental, e a origem de muita confusão, é que **os wildcards são processados pela shell, não pelo comando**. Antes de executar qualquer comando, a shell examina os argumentos, substitui os padrões pelos nomes de ficheiros que lhes correspondem, e só depois entrega ao comando a linha já expandida. O comando nunca chega a ver o wildcard: recebe apenas a lista de nomes que a shell encontrou.
 
Esta substituição designa-se por **expansão**. Um exemplo simples torna o mecanismo visível:
 
```bash
    $ echo *
```
 
O comando `echo` limita-se a imprimir os seus argumentos. No entanto, ao correr `echo *`, o que aparece no ecrã é a lista de todos os ficheiros do directório actual. Isto acontece porque a shell expandiu o `*` para os nomes dos ficheiros antes de invocar o `echo`, que recebeu já a lista completa e se limitou a imprimi-la.
 
### Os caracteres de correspondência
 
#### O asterisco (`*`)
 
O asterisco corresponde a qualquer sequência de caracteres, incluindo nenhum. É o wildcard mais usado.
 
```bash
    $ ls at*        # nomes que começam por "at"
    $ ls *at        # nomes que terminam em "at"
    $ ls *at*       # nomes que contêm "at" em qualquer posição
    $ ls *.txt      # todos os ficheiros com extensão .txt
```
 
Se nenhum ficheiro corresponder ao padrão, a shell não faz qualquer expansão e o comando recebe o wildcard literal. Correr `echo *dfkdsafh` num directório sem ficheiros correspondentes imprime a própria string `*dfkdsafh`, porque não houve nada para substituir.

#### O ponto de interrogação (`?`)
 
O ponto de interrogação corresponde a exactamente um caractere arbitrário, nem mais nem menos.
 
```bash
    $ ls b?at       # corresponde a "boat" e "brat", mas não a "bat" nem a "boueat"
    $ ls ficheiro?.txt   # ficheiro1.txt, ficheiro2.txt, mas não ficheiro10.txt
```
 
#### Os parênteses rectos (`[ ]`)
 
Os parênteses rectos correspondem a um único caractere de entre um conjunto especificado. Permitem maior precisão do que o `?`.
 
```bash
    $ ls ficheiro[123].txt      # ficheiro1.txt, ficheiro2.txt, ficheiro3.txt
    $ ls relatorio[a-z].pdf     # qualquer letra minúscula: relatorioa.pdf ... relatorioz.pdf
    $ ls log[0-9].txt           # qualquer dígito: log0.txt ... log9.txt
```
 
Dentro dos parênteses, um hífen define um intervalo (`a-z`, `0-9`), e a correspondência aplica-se a um único caractere que esteja dentro desse conjunto.
 
### Proteger os wildcards da expansão
 
Por vezes é necessário que o wildcard chegue intacto ao comando, sem ser expandido pela shell. Isto consegue-se envolvendo o padrão em aspas simples.
 
```bash
    $ echo '*'      # imprime um asterisco literal, sem expansão
```
 
Esta técnica é essencial para comandos como o `find`, que fazem a sua própria interpretação de padrões e precisam de receber o wildcard tal como foi escrito, como veremos a seguir.
 
> **Cuidado extremo ao combinar wildcards com comandos destrutivos.** Como a shell expande o padrão antes de o comando correr, um `rm *` apaga todos os ficheiros do directório num instante, sem confirmação. Um erro de digitação, como um espaço acidental em `rm * .txt` em vez de `rm *.txt`, transforma a intenção de apagar ficheiros `.txt` na eliminação de tudo. Verifique sempre o padrão, e em caso de dúvida use primeiro `ls` com o mesmo padrão para ver exactamente o que será afectado.

## Localizar Ficheiros
 
Saber que um ficheiro existe algures na árvore de directórios mas não saber onde é uma frustração comum. O Linux oferece duas ferramentas para o encontrar, com abordagens fundamentalmente diferentes: o `find`, que procura em tempo real, e o `locate`, que consulta um índice previamente construído.
 
### find: procura em tempo real
 
O `find` percorre efectivamente a árvore de directórios no momento em que é executado, examinando cada ficheiro segundo os critérios indicados. A forma essencial da sua utilização é:
 
```bash
    $ find [directório] -name [nome]
```
 
O primeiro argumento é o directório a partir do qual a procura começa, e desce recursivamente por toda a sua subárvore.
 
```bash
    # Procurar um ficheiro pelo nome, a partir do directório actual
    $ find . -name relatorio.pdf
 
    # Procurar em todo o sistema (requer privilégios para alguns directórios)
    $ find / -name sshd_config
 
    # Procurar no directório pessoal
    $ find ~ -name notas.txt
```
 
O `find` aceita padrões com wildcards, mas há um cuidado importante. Como a shell expande os wildcards antes de o `find` correr, é necessário protegê-los com aspas simples, para que seja o `find`, e não a shell, a interpretá-los:
 
```bash
    # As aspas garantem que o find recebe o padrão intacto
    $ find . -name '*.log'
    $ find /var -name 'erro*'
```
 
Sem as aspas, a shell tentaria expandir o `*.log` no directório actual antes de o `find` sequer arrancar, o que produziria resultados errados ou uma mensagem de erro.
 
O `find` é uma ferramenta com muitas capacidades adicionais, incluindo a procura por tamanho, data de modificação, dono ou permissões, e a execução de acções sobre os ficheiros encontrados. No entanto, é aconselhável dominar primeiro a forma essencial acima e compreender bem o papel do `-name` antes de avançar para opções mais complexas.
 
Alguns exemplos do alcance da ferramenta:
 
```bash
    # Ficheiros modificados nas últimas 24 horas
    $ find /var/log -mtime -1
 
    # Ficheiros maiores que 100 MB
    $ find / -size +100M
 
    # Directórios (e não ficheiros) com um determinado nome
    $ find . -type d -name backup
```
 
### locate: procura por índice
 
O `locate` resolve o mesmo problema por uma via oposta. Em vez de percorrer o disco em tempo real, consulta uma base de dados que o sistema constrói e actualiza periodicamente. O resultado é uma procura quase instantânea, mesmo em sistemas com milhões de ficheiros.
 
```bash
    $ locate sshd_config
    $ locate relatorio.pdf
```
 
Esta velocidade tem um custo: o `locate` só conhece os ficheiros que existiam quando o índice foi construído pela última vez. Um ficheiro criado há minutos pode ainda não constar do índice, e o `locate` não o encontrará. O índice é normalmente actualizado uma vez por dia por uma tarefa agendada, mas pode ser forçado manualmente:
 
```bash
    $ sudo updatedb
```
 
Em muitas distribuições, incluindo o CentOS, o `locate` não vem instalado por omissão e faz parte do pacote `mlocate`:
 
```bash
    $ sudo dnf install mlocate -y
```
 
### find ou locate: quando usar cada um
 
A escolha entre as duas ferramentas depende do que se procura e das circunstâncias.
 
O `locate` é a escolha certa para procuras rápidas de ficheiros que existem há algum tempo, sobretudo quando não se sabe sequer em que parte do sistema procurar. É ideal para responder à pergunta "onde está o ficheiro X?" de forma imediata.
 
O `find` é indispensável quando o ficheiro é recente e pode ainda não estar indexado, quando é preciso procurar por critérios que vão além do nome (tamanho, data, dono, permissões), ou quando se quer agir sobre os resultados. É também a única opção quando o `locate` não está disponível ou o seu índice não é de confiança.
 
| Critério | `find` | `locate` |
|----------|--------|----------|
| Método | Percorre o disco em tempo real | Consulta um índice pré-construído |
| Velocidade | Lento em árvores grandes | Quase instantâneo |
| Ficheiros recentes | Encontra sempre | Só após actualização do índice |
| Critérios de procura | Nome, tamanho, data, dono, tipo, permissões | Apenas nome/caminho |
| Disponibilidade | Sempre presente | Requer instalação e índice actualizado |

## Gestão de Pacotes: rpm, yum e dnf

Instalar software é uma das primeiras necessidades ao administrar um sistema. Num sistema Linux, isto raramente se faz descarregando executáveis avulsos de sítios na internet, como é hábito noutros sistemas operativos. Em vez disso, o software distribui-se em **pacotes**, geridos por um sistema centralizado que resolve dependências, verifica autenticidade e mantém um registo rigoroso de tudo o que está instalado.

### O conceito de gestor de pacotes e repositórios

Um **pacote** é um arquivo que contém não só os ficheiros de um programa, mas também os seus metadados: a versão, a descrição, a lista de outros pacotes de que depende para funcionar, e assinaturas que garantem a sua autenticidade.

Este modelo resolve um problema que noutros sistemas é fonte constante de dificuldades: as **dependências**. Um programa raramente funciona isolado; precisa de bibliotecas e componentes que outros pacotes fornecem. Um gestor de pacotes, ao instalar um programa, identifica automaticamente tudo aquilo de que ele depende e instala-o também, garantindo que o sistema fica num estado coerente.

Os pacotes não são procurados individualmente na internet. São obtidos a partir de **repositórios**: colecções organizadas de pacotes, mantidas pela distribuição ou por terceiros de confiança, alojadas em servidores espelhados por todo o mundo. O sistema conhece uma lista de repositórios e sabe onde procurar quando lhe é pedido um programa. Esta centralização traz três vantagens decisivas: o software vem de uma fonte confiável e assinada, as actualizações de segurança chegam a todo o sistema de forma coordenada, e a instalação resume-se a um único comando.

Na família do CentOS e do Red Hat, este sistema assenta em três camadas: o formato base `rpm`, e as ferramentas de alto nível `yum` e `dnf` que o gerem.

### rpm: o formato base

O **RPM** (*RPM Package Manager*) é simultaneamente o formato dos pacotes (ficheiros com extensão `.rpm`) e a ferramenta de baixo nível que os manipula. É a fundação sobre a qual tudo o resto assenta.

A característica que define o `rpm` é também a sua limitação: **não resolve dependências**. O `rpm` instala, remove e consulta ficheiros de pacote individuais, mas se um pacote precisar de outro que não está presente, o `rpm` limita-se a recusar e a indicar o que falta, sem tentar obter o que é necessário. Este comportamento é deliberado: o `rpm` é uma ferramenta determinística e de baixo nível, e a resolução de dependências é tarefa das camadas superiores.

Por esta razão, o `rpm` raramente é usado para instalar software no dia-a-dia. A sua utilidade reside sobretudo na **consulta**: interrogar a base de dados de pacotes para saber o que está instalado.

```bash
    # Ver todos os pacotes instalados
    $ rpm -qa

    # Verificar se um pacote específico está instalado
    $ rpm -q httpd

    # Descobrir a que pacote pertence um determinado ficheiro
    $ rpm -qf /usr/sbin/sshd

    # Listar todos os ficheiros que um pacote instalou
    $ rpm -ql httpd

    # Ver informação detalhada sobre um pacote
    $ rpm -qi httpd
```

A opção `-q` significa *query* (consulta), e combina-se com as restantes: `-qa` para todos (*all*), `-qf` por ficheiro (*file*), `-ql` para listar (*list*), `-qi` para informação (*info*). Estas consultas são ferramentas de diagnóstico valiosas, sobretudo o `rpm -qf`, que responde à pergunta frequente "de onde veio este ficheiro?".

### dnf e yum: gestão completa

Se o `rpm` é a fundação, o **dnf** é a ferramenta com que efectivamente se trabalha. O `dnf` (*Dandified YUM*) opera por cima do `rpm` e acrescenta precisamente aquilo que lhe falta: consulta os repositórios, resolve dependências automaticamente, descarrega os pacotes necessários, verifica as suas assinaturas, e executa a transacção de forma segura.

Uma nota sobre a nomenclatura, que é fonte de confusão. O **yum** (*Yellowdog Updater, Modified*) foi durante muitos anos a ferramenta padrão nas distribuições Red Hat. A partir do CentOS 8 e RHEL 8, foi substituído pelo `dnf`, que oferece melhor desempenho e uma resolução de dependências mais robusta, assente na biblioteca `libsolv`. Para manter a compatibilidade, o comando `yum` continua a existir, mas é hoje apenas uma ligação para o `dnf`: escrever `yum` ou `dnf` produz exactamente o mesmo resultado. No CentOS Stream, `dnf` é a forma correcta e recomendada, e é a que usaremos.

#### Actualizar o sistema

Antes de instalar software, é boa prática garantir que a informação dos repositórios e os pacotes instalados estão actualizados.

```bash
    # Actualizar a lista de pacotes disponíveis e instalar as actualizações
    $ sudo dnf update

    # Ver que actualizações estão disponíveis sem as instalar
    $ sudo dnf check-update
```

O comando `dnf update` refresca os metadados dos repositórios e actualiza todos os pacotes instalados para as versões mais recentes disponíveis. É um comando que aparece no início de quase todos os guias de instalação, e a sua execução regular é a base da manutenção de segurança do sistema, como visto no Capítulo 7.

#### Instalar pacotes

```bash
    # Instalar um pacote (pede confirmação)
    $ sudo dnf install httpd

    # Instalar sem pedir confirmação (útil em scripts)
    $ sudo dnf install httpd -y

    # Instalar vários pacotes de uma vez
    $ sudo dnf install httpd mariadb-server php
```

Ao instalar, o `dnf` apresenta primeiro um resumo da transacção: o pacote pedido, todas as dependências que serão instaladas com ele, e o espaço em disco necessário. Só depois da confirmação procede. A opção `-y` responde "sim" automaticamente, o que é conveniente em scripts mas deve ser usado com atenção em comandos interactivos, para não saltar a revisão do que vai ser instalado.

#### Procurar pacotes

Quando não se sabe o nome exacto de um pacote, o `dnf` permite procurá-lo.

```bash
    # Procurar pacotes por palavra-chave no nome ou descrição
    $ dnf search apache

    # Ver informação detalhada sobre um pacote antes de o instalar
    $ dnf info httpd

    # Descobrir que pacote fornece um determinado ficheiro ou comando
    $ dnf provides /usr/sbin/httpd
```

O `dnf provides` é particularmente útil: quando um comando não existe no sistema, permite descobrir qual o pacote que o fornece, para depois o instalar.

#### Remover pacotes

```bash
    # Remover um pacote
    $ sudo dnf remove httpd

    # Remover pacotes que foram instalados como dependências e já não são necessários
    $ sudo dnf autoremove
```

Ao remover um pacote, o `dnf` verifica se outros pacotes dependem dele e avisa em conformidade. O `dnf autoremove` limpa dependências órfãs: pacotes que foram instalados automaticamente para satisfazer outro pacote e que deixaram de ser necessários após a remoção deste.

#### Listar e consultar

```bash
    # Listar todos os pacotes instalados
    $ dnf list installed

    # Listar pacotes disponíveis mas não instalados
    $ dnf list available

    # Ver o histórico de transacções (instalações, remoções, actualizações)
    $ sudo dnf history
```

O `dnf history` mantém um registo de todas as operações realizadas, o que é valioso para auditoria e para perceber o que mudou no sistema ao longo do tempo.

### Instalar um pacote .rpm local

Ocasionalmente, é necessário instalar um pacote obtido directamente como ficheiro `.rpm`, fora dos repositórios. Embora o `rpm -i` o consiga fazer, não resolve dependências. A abordagem correcta é usar o `dnf`, que instala o ficheiro local **e** resolve as suas dependências a partir dos repositórios:

```bash
    $ sudo dnf install ./pacote-exemplo.rpm
```

### Repositórios e os ficheiros .repo

O `dnf` sabe onde procurar pacotes porque consulta uma lista de repositórios definida em ficheiros de configuração. Estes ficheiros, com extensão `.repo`, residem no directório `/etc/yum.repos.d/`.

```bash
    # Ver os repositórios configurados
    $ ls /etc/yum.repos.d/

    # Listar os repositórios activos e a sua situação
    $ dnf repolist
```

Cada ficheiro `.repo` pode definir um ou mais repositórios. A estrutura de uma definição é a seguinte:

```ini
[nome-do-repositorio]
name=Descrição legível do repositório
baseurl=https://servidor.exemplo.com/centos/9/x86_64/
enabled=1
gpgcheck=1
gpgkey=https://servidor.exemplo.com/RPM-GPG-KEY-exemplo
```

Os campos principais:

| Campo | Significado |
|-------|-------------|
| `[nome]` | Identificador curto e único do repositório |
| `name` | Descrição legível |
| `baseurl` | O endereço onde os pacotes estão alojados |
| `enabled` | `1` para activo, `0` para desactivado |
| `gpgcheck` | `1` para verificar assinaturas dos pacotes (segurança) |
| `gpgkey` | Localização da chave usada para verificar as assinaturas |

O campo `gpgcheck=1` merece destaque. Activa a verificação criptográfica das assinaturas de cada pacote antes da instalação, garantindo que o pacote vem realmente da fonte que alega e que não foi adulterado em trânsito. **Não deve ser desactivado**, pois é uma protecção fundamental contra a instalação de software comprometido.

Muitas ferramentas de terceiros fornecem o seu próprio ficheiro `.repo`, e um repositório muito comum no ecossistema CentOS é o **EPEL** (*Extra Packages for Enterprise Linux*), mantido pela comunidade Fedora, que disponibiliza software adicional não incluído nos repositórios base:

```bash
    # Instalar o repositório EPEL
    $ sudo dnf install epel-release
```
