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

Uma senha administrativa robusta não deve ser baseada apenas na complexidade visual, mas principalmente na sua extensão (frequentemente implementada através de "passphrases" — frases-senha longas), minimizando a suscetibilidade a ataques de força bruta no ambiente de servidor.


## O Sistema de Ficheiros: Estrutura e Organização

A utilidade de um sistema operativo mede-se, em grande parte, pela eficiência com que gere e disponibiliza o acesso aos dados e aos recursos físicos da máquina. Em sistemas Linux e UNIX, o sistema de ficheiros transcende a mera função de armazenamento em disco; ele atua como o mecanismo central de organização. Compreende quatro componentes lógicos fundamentais: um espaço de nomes (namespace) hierárquico, uma interface de programação (API) para manipulação de objetos, um modelo de segurança e, por fim, a implementação de software que interliga este modelo abstrato ao hardware subjacente.

### A Filosofia "Tudo é um Ficheiro"

A premissa arquitetural mais célebre do UNIX — e por herança, do Linux — dita que "tudo é um ficheiro". Embora seja uma ligeira simplificação, esta máxima é a chave para a elegância do sistema.

No seu nível mais rudimentar, um ficheiro para o kernel é apenas uma sequência bidimensional de bytes. O sistema operativo não impõe qualquer estrutura rígida ou significado a estes dados; a interpretação do conteúdo é estritamente delegada aos programas que os leem.

O poder desta filosofia reside na sua universalidade. O conceito de ficheiro é estendido para mapear quase todos os recursos do sistema. Discos rígidos, terminais de linha de comandos, adaptadores de rede, processos em execução e até canais de comunicação entre processos (pipes esockets) são todos representados e acedidos como se fossem ficheiros de texto comuns. Isto significa que as mesmas ferramentas padronizadas usadas para ler ou escrever num documento de texto podem ser empregues para enviar dados para um dispositivo físico ou ler métricas do processador, simplificando drasticamente a administração e a programação no sistema.

## Conetividade e Acesso Remoto

### Fundamentação Teórica do Protocolo SSH (Secure Shell) 

O protocolo SSH (Secure Shell) constitui uma das abordagens baseadas em software mais robustas e amplamente adotadas para a segurança de comunicações em redes informáticas. A sua excelência reside na capacidade de fornecer **encriptação transparente**: sempre que dados são transmitidos de um nó para a rede, o protocolo encarrega-se da sua cifragem automática; ao alcançarem o destinatário, os dados são decifrados e restituídos à sua forma original.

Este mecanismo permite que a comunicação ocorra de forma fluida, sem que os utilizadores necessitem de intervir ativamente no processo criptográfico. Empregando algoritmos de cifra modernos e seguros, o SSH atinge um nível de fiabilidade que justifica a sua adoção mandatória em aplicações de missão crítica e nas maiores infraestruturas corporativas.

### Arquitetura Cliente-Servidor e Cifragem Ponto-a-Ponto

O SSH opera sob uma arquitetura estrita de **Cliente-Servidor**. Tipicamente, um programa servidor SSH (o daemon), gerido por um administrador de sistemas, opera em estado de escuta passiva, aceitando ou rejeitando conexões direcionadas à máquina hospedeira. Simultaneamente, utilizadores executam programas clientes SSH a partir de terminais remotos para submeter requisições ao servidor  tais como pedidos de autenticação, transferência de ficheiros ou execução direta de comandos. A característica vital deste modelo é que todo o tráfego transacionado entreo cliente e o servidor é encriptado e protegido contra modificações e interceção (sniffing) por terceiros.


<figure align="center">
  <img src="../assets/img/ssh.png" alt="Arquitetura SSH" width="800">
  <figcaption><b>img 1:</b> Arquitetura cliente-servidor do SSH </figcaption>
</figure>


### O Protocolo SSH: Arquitetura e Pilares de Segurança

É imperativo compreender que o SSH é, primordialmente, um **protocolo** e não um produto isolado. Trata-se de uma especificação técnica que define as diretrizes para a condução de comunicações seguras através de redes informáticas. Enquanto norma, o protocolo SSH endereça três dimensões críticas da segurança da informação: **Autenticação**, **Cifragem** e **Integridade**.

#### Tríade de Segurança do Protocolo

O SSH garante a proteção dos dados em trânsito através de mecanismos fundamentais:
- **Autenticação**: Consiste no processo de determinação fiável da identidade de uma entidade. Ao tentar aceder a um sistema remoto, o SSH exige a apresentação de provas digitais de identidade (sejam elas credenciais, chaves criptográficas ou certificados). Caso a validação seja bem-sucedida, a conexão é autorizada; caso contrário, o acesso é sumariamente rejeitado.
- **Cifragem (Encriptação)**: Encarrega-se de codificar os dados de modo a torná-los ininteligíveis para qualquer observador não autorizado. Este mecanismo assegura a confidencialidade da informação enquanto esta transita pela infraestrutura de rede, garantindo que apenas os destinatários pretendidos possam restaurar a inteligibilidade dos dados.
- **Integridade**: Garante que a informação que viaja pela rede chegue ao destino sem sofrer alterações. Através de algoritmos de verificação, o SSH é capaz de detetar se um terceiro capturou e modificou os dados em trânsito, asseverando que o conteúdo recebido é idêntico ao conteúdo enviado.

#### Taxonomia e Versões do Protocolo

Para uma correta documentação técnica, é necessário distinguir as versões do protocolo das suas implementações de software:
- **SSH (Termo Genérico)**: Refere-se de forma abrangente ao protocolo ou aos produtos de software que o implementam.
- **SSH-1: A primeira versão do protocolo**. Embora tenha passado por várias revisões (como a 1.3 e 1.5), é atualmente considerada obsoleta para fins de produção devido a vulnerabilidades de design conhecidas.
- **SSH-2**: A versão atual e padrão do protocolo, definida pelo grupo de trabalho SECSH do IETF (Internet Engineering Task Force). O SSH-2 introduziu melhorias significativas em termos de segurança, algoritmos de troca de chaves e eficiência.
- **ssh (em minúsculas)**: Refere-se especificamente ao programa cliente incluído na maioria das distribuições. É a ferramenta que o utilizador invoca no terminal para iniciar sessões remotas ou executar comandos.
- **OpenSSH**: Produto oriundo do projeto OpenBSD, constituindo a implementação mais difundida globalmente. É uma suite de software de código aberto que suporta ambas as versões do protocolo (SSH-1 e SSH-2), embora a utilização da versão 1 seja desencorajada por omissão.

### Funcionalidades e Aplicabilidades do Ecossistema SSH

O SSH transcende a simples conectividade, oferecendo um conjunto de funcionalidades que endereçam as vulnerabilidades críticas de redes inseguras. Estas capacidades podem ser categorizadas em quatro pilares principais de operação:

#### Interatividade e Execução Remota de Comandos

Historicamente, protocolos como o Telnet e o rsh transmitiam credenciais e sessões completas em texto plano (plaintext), permitindo a interceção de dados sensíveis por agentes maliciosos (sniffing).

- **Acesso Remoto Seguro (Login)**: Através do cliente ssh, estabelece-se uma sessão interativa onde o tráfego é cifrado antes de abandonar a máquina local. O processo é transparente para o utilizador, mantendo a experiência de uso mas elevando a segurança ao padrão industrial.
- **Invocação Remota de Comandos**: O SSH permite a execução de comandos isolados em múltiplos servidores de forma automatizada (scripts). Ao contrário do legado rsh, os resultados retornados via rede são protegidos por encriptação robusta, garantindo que informações sensíveis sobre o estado do sistema não sejam expostas.

#### Persistência e Transferência de Ativos (Dados)

A movimentação de ficheiros entre nós de rede é uma tarefa crítica que, através de protocolos tradicionais (FTP, rcp ou e-mail), carece de proteção nativa.
- **Transferência Cifrada com scp**: O comando scp (Secure Copy) permite o transporte de ficheiros com uma sintaxe simplificada, onde a cifragem e decifragem ocorrem de forma automática nas extremidades da conexão. Esta abordagem elimina a necessidade de cifrar ficheiros manualmente com ferramentas externas (como PGP) antes do envio, otimizando o fluxo de trabalho do administrador.

#### Mecanismos Avançados de Autenticação: Chaves e Agentes

Uma das maiores vulnerabilidades em sistemas distribuídos é a dependência exclusiva de palavras-passe, que podem ser fracas ou expostas por erro humano.
- **Autenticação Baseada em Chaves**: O SSH introduz o conceito de identidades digitais únicas através de pares de chaves criptográficas. O acesso é concedido mediante a prova de posse de uma chave privada, protegida por uma passphrase.
- **Agentes de Autenticação (ssh-agent)** : Para mitigar a fadiga de autenticação em infraestruturas complexas, o uso de agentes permite carregar as chaves em memória uma única vez. 

#### Extensibilidade e Segurança de Protocolos Terceiros

O SSH pode atuar como uma camada de transporte segura para outras aplicações baseadas em TCP/IP que não possuem segurança nativa ou que são bloqueadas por firewalls.
- **Encaminhamento de Portas (Port Forwarding / Tunelamento)**: Esta técnica permite redirecionar conexões de rede através de um túnel SSH. Por exemplo, é possível encapsular o tráfego de um servidor de base de dados ou de notícias (porta 119) através de uma porta local segura, contornando restrições de firewall e garantindo a encriptação ponto-a-ponto de protocolos que, de outra forma, seriam vulneráveis.
- **Controlo de Acesso Granular**: O sistema permite a delegação de tarefas específicas a terceiros (ex: permitir apenas a execução de um programa de e-mail) sem a necessidade de partilhar a palavra-passe principal ou conceder privilégios de super-utilizador, reforçando o princípio do privilégio mínimo.

### Uso Básico do Cliente SSH: Exemplo Prático


O protocolo SSH assenta numa premissa simples, mas engloba múltiplos componentes complexos. Esta secção foi desenhada para iniciar a operação prática do SSH de forma ágil, abordando as suas funcionalidades mais utilitárias e imediatas. O foco incidirá sobre dois eixos fundamentais:
- O estabelecimento de sessões de terminal remoto através de ligações seguras.
- A transferência de ficheiros entre nós de rede sob um canal cifrado.

#### O Cenário do Exemplo

Para demonstrar estas capacidades, estabeleceremos um ambiente de testes controlado. Suponha que atua como administrador de sistemas e necessita de gerir um servidor remotamente. Para este laboratório, utilizaremos duas instâncias virtuais (VMs) alocadas na mesma sub-rede virtual, embora o procedimento seja aplicável a quaisquer duas máquinas físicas ou virtuais com conectividade de rede:
- **Máquina Local (Estação de Gestão)**: Uma VM equipada com Interface Gráfica (GUI), a partir da qual os comandos serão originados. Representaremos a linha de comandos desta máquina com o símbolo $.
- **Máquina Remota (Servidor Alvo)**: Uma VM configurada em modo texto (CLI/Minimal), que receberá as conexões. Representaremos o seu terminal como servidor >.

#### Verificação de Interfaces de Rede e Endereçamento

Antes de iniciar qualquer conexão SSH, é imperativo conhecer a topologia lógica, nomeadamente os endereços IP atribuídos a cada máquina. No ecossistema Linux, existem duas ferramentas primárias para este fim:


##### O comando ip (Padrão Moderno):

Parte do pacote iproute2, é a ferramenta contemporânea para administração de rede no Linux. Para listar todas as interfaces de rede e os seus respetivos endereços IP (IPv4 e IPv6), executa-se o seguinte comando no Servidor Alvo:

```bash
 $ ip addr show
```

Este comando revelará a interface de rede ativa (por exemplo, enp1s0 ou eth0) e o seu endereço inet (ex: 192.168.122.50).

##### O comando ifconfig (Ferramenta Legada):

Historicamente pertencente ao pacote net-tools, o ifconfig foi o padrão durante décadas. Embora considerado obsoleto em distribuições modernas como o CentOS Stream, ainda é amplamente referenciado na literatura técnica. A sua execução fornece um output formatado de forma diferente, mas com o mesmo propósito de identificação de endereços físicos (MAC) e lógicos (IP):

```bash
 $ ifconfig
```
Nota: Para os exemplos subsequentes, assumiremos que o comando acima revelou que o Servidor Alvo possui o endereço IP 192.168.122.50 .

##### Validação de Conectividade com ping
A partir da Estação de Gestão (Local), a verificação da conectividade é efetuada mediante o seguinte comandoConhecido o endereço de destino, o passo lógico seguinte na engenharia de redes é validar a alcançabilidade do nó remoto. Para tal, recorre-se ao comando ping, que utiliza pacotes ICMP (Internet Control Message Protocol) de Echo Request e aguarda por um Echo Reply.

A partir da Estação de Gestão (Local), a verificação da conectividade é efetuada mediante o seguinte comando:

```bash
 $ ping -c 4 192.168.122.50
```
O parâmetro -c 4 instrui o sistema a enviar exatamente quatro pacotes. Uma resposta bem-sucedida (indicando 0% de perda de pacotes) confirma que a rota de rede entre a estação de gestão e o servidor está funcional e pronta para estabelecer o túnel TCP necessário para o SSH.


#### Sessões de Terminal Remoto com ssh

Suponha que a conta de utilizador no Servidor Alvo foi batizada como admin_sys. Para iniciar a sessão remota a partir da sua máquina de gestão local, utiliza-se a ferramenta cliente ssh, combinando o nome de utilizador e o endereço IP do destino. A sintaxe moderna e mais comum utiliza o formato utilizador@host:

```bash
$ ssh admin_sys@192.168.122.50
The authenticity of host '192.168.122.50 (192.168.122.50)' can't be established.
ED25519 key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
admin_sys@192.168.122.50's password: ******
Last login: Mon May 25 10:15:30 2026 from 192.168.122.10
servidor>

```

Ao primeiro contacto, o cliente SSH alerta que a autenticidade do servidor não é conhecida e apresenta a sua impressão digital (fingerprint). Ao aceitar (yes), estabelece-se um canal seguro e as comunicações passam a ser integralmente cifradas. De seguida, o cliente solicita a palavra-passe, que já transita pela rede de forma criptografada.
Uma vez autenticado, o servidor autoriza o acesso, alterando o prompt para o contexto da máquina remota. A partir deste momento, todos os comandos digitados localmente são executados no servidor de forma transparente, protegidos contra interceção.


#### Transferência de Ficheiros com scp

Após a consolidação da conectividade interativa, a etapa subsequente consiste na movimentação de dados entre os nós da rede. O utilitário scp (Secure Copy) é a ferramenta padrão para este fim, substituindo métodos legados e vulneráveis, como o FTP. O scp utiliza o protocolo SSH para assegurar que a integridade e a confidencialidade dos arquivos sejam preservadas durante o trânsito.

Para fins de demonstração, deve-se inicialmente gerar um artefato técnico na Estação de Gestão (Local). O exemplo abaixo ilustra a criação de um arquivo de configuração simplificado:

```bash
    $ echo "Parâmetros de configuração do sistema" > config_teste.txt
```

A estrutura lógica do comando scp é análoga à do comando cp (copy) nativo do Unix, seguindo a sintaxe: scp [opções] [origem] [destino]. Paratransferir o arquivo config_teste.txt para o diretório de usuário no Servidor Alvo, executa-se:

```bash
    $ scp config_teste.txt admin_sys@192.168.122.50:~/
```

À semelhança do ssh, o scp invoca a solicitação da palavra-passe remota. Uma vez verificada a credencial pelo daemon SSH no servidor, o cliente copia o ficheiro local através da rede, apresentando uma barra de progresso. O processo garante que o ficheiro é automaticamente cifradoao abandonar a máquina local e decifrado no momento em que é escrito no disco do servidor remoto.

A versatilidade desta ferramenta permite ainda omitir definições de caminho se pretendermos utilizar diretivas padrão (como ~/ para o diretório base do utilizador) ou até mesmo descarregar ficheiros do servidor para a máquina local invertendo a ordem dos argumentos na sintaxe.
