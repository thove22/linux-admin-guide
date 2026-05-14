# Acesso ao Sistema e Estrutura de Ficheiros

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
- **Máquina Remota (Servidor Alvo)**: Uma VM configurada em modo texto (CLI/Minimal), que receberá as conexões. Representaremos o seu terminal como servidor>.

#### Verificação de Interfaces de Rede e Endereçamento

Antes de iniciar qualquer conexão SSH, é imperativo conhecer a topologia lógica, nomeadamente os endereços IP atribuídos a cada máquina. No ecossistema Linux, existem duas ferramentas primárias para este fim:


##### O comando ip (Padrão Moderno):

Parte do pacote iproute2, é a ferramenta contemporânea para administração de rede no Linux. Para listar todas as interfaces de rede e os seus respetivos endereços IP (IPv4 e IPv6), executa-se o seguinte comando no Servidor Alvo:

```bash
 ip addr show
```

Este comando revelará a interface de rede ativa (por exemplo, enp1s0 ou eth0) e o seu endereço inet (ex: 192.168.122.50).

##### O comando ifconfig (Ferramenta Legada):

Historicamente pertencente ao pacote net-tools, o ifconfig foi o padrão durante décadas. Embora considerado obsoleto em distribuições modernas como o CentOS Stream, ainda é amplamente referenciado na literatura técnica. A sua execução fornece um output formatado de forma diferente, mas com o mesmo propósito de identificação de endereços físicos (MAC) e lógicos (IP):

```bash
   ifconfig
```


