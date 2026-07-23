# Redes, Serviços e Segurança

## 1. Fundamentos de Rede em Linux

### 1.1 A questão central das redes

Quando se escreve um endereço numa carta e se a coloca no correio, assume-se que um sistema complexo de classificação, transporte e entrega vai tratar do resto. Não é necessário saber quantos aviões, camiões ou carteiros estão envolvidos. A carta chega. Redes de computadores funcionam por um princípio semelhante: esconde-se a complexidade por baixo de camadas de abstracção, de forma a que uma aplicação possa enviar dados para o outro lado do mundo sem saber nada sobre cabos, routers ou protocolos de baixo nível.

Mas ao contrário do correio, as redes de computadores são engenharia pura. Cada decisão tem uma razão, cada número tem um significado, e quando algo falha, não existe carteiro a quem telefonar. É o administrador de sistemas que tem de perceber exactamente o que aconteceu, em que camada o problema está, e como corrigi-lo.

Para isso, duas perguntas fundamentais precisam de ser respondidas com precisão em qualquer comunicação entre computadores: como é que o emissor sabe para onde enviar os dados, e quando os dados chegam ao destino, como sabe o receptor o que acabou de receber? A resposta não vem de um único mecanismo, mas de um conjunto de componentes organizados em camadas independentes, cada uma responsável por um aspecto específico da comunicação. Compreender estas camadas não é teoria académica. É a diferença entre um administrador que configura serviços às cegas e um que sabe exactamente o que está a fazer e porquê.

### 1.2 A arquitectura em camadas: o modelo de rede

Uma rede funcional é construída sobre um conjunto de camadas empilhadas, onde cada camada depende dos serviços da camada abaixo e oferece serviços à camada acima. No contexto da internet, as camadas relevantes são quatro:

- A **camada de aplicação** contém os protocolos de alto nível que as aplicações usam para comunicar: HTTP para a web, FTP para transferência de ficheiros, SMTP para email, TLS para encriptação. O processamento desta camada acontece no espaço de utilizador.

- A **camada de transporte** define as características de transmissão de dados: verificação de integridade, portas de origem e destino, fragmentação de dados em pacotes e a sua remontagem no destino. Os protocolos mais comuns são o TCP (*Transmission Control Protocol*), que garante entrega ordenada e fiável, e o UDP (*User Datagram Protocol*), mais rápido mas sem garantias de entrega. No Linux, esta camada é gerida principalmente pelo kernel.

- A **camada de rede** define como mover pacotes de um host de origem para um host de destino, possivelmente através de múltiplas redes intermédias. O protocolo que governa a internet é o IP (*Internet Protocol*), nas versões 4 e 6.

- A **camada física** define como transmitir dados brutos através de um meio físico: cabos Ethernet, sinais wireless, fibra óptica. É a camada do hardware.

Quando se envia dados de um computador para outro, os dados percorrem estas camadas do topo para baixo no emissor, atravessam o meio físico, e sobem novamente as camadas no receptor. Este percurso acontece pelo menos duas vezes antes de os dados chegarem à aplicação de destino.

### 1.3 Rede local e acesso à internet

A configuração de rede mais comum em ambientes de escritório, laboratório e virtualização é a rede local com acesso à internet através de um router.


<figure align="center">
  <img src="../assets/img/Lan.jpg" alt="Topologia de uma rede local típica" width="500">
  <figcaption><b>img 1:</b> Topologia de uma rede local típica </figcaption>
</figure>

---

Neste modelo, todos os computadores ligados entre si formam uma **LAN** (*Local Area Network*). Um deles é o **router**, um host especial que sabe mover dados entre a rede local e o exterior. A ligação do router à internet chama-se **uplink** ou ligação WAN (*Wide Area Network*).

Cada máquina ligada à rede chama-se **host**. Os hosts numa LAN partilham normalmente o mesmo esquema de endereçamento e têm acesso uns aos outros directamente. Para comunicar com o exterior, enviam o tráfego para o router, que se encarrega de o encaminhar.

### 1.4 Endereçamento IP

#### IPv4

Cada host ligado à internet tem pelo menos um endereço IP. No IPv4, esse endereço é composto por 32 bits representados em quatro grupos de números decimais separados por pontos, como `10.23.2.37`. A esta representação chama-se notação dotted-quad.

Um endereço IP não identifica apenas o host: identifica também a rede a que o host pertence. Esta informação está codificada em conjunto com a máscara de sub-rede.

#### Sub-redes e máscaras

Uma sub-rede é um grupo de hosts cujos endereços IP partilham um prefixo comum. Para definir uma sub-rede são necessárias duas informações: o prefixo de rede e a máscara de sub-rede.

A máscara de sub-rede é um número de 32 bits onde os bits a 1 identificam a parte do endereço que pertence à rede, e os bits a 0 identificam a parte que pertence ao host. Por exemplo, a máscara `255.255.255.0` em binário é:

```
11111111 11111111 11111111 00000000
```

Os primeiros 24 bits identificam a rede, os últimos 8 bits identificam o host dentro dessa rede. Aplicando esta máscara ao endereço `10.23.2.37`, a parte de rede é `10.23.2` e a parte de host é `37`. Qualquer endereço que comece com `10.23.2` pertence à mesma sub-rede.

#### Notação CIDR

A notação CIDR (*Classless Inter-Domain Routing*) é a forma compacta e moderna de representar sub-redes. Em vez de escrever a máscara completa, escreve-se apenas o número de bits a 1 da máscara, precedido de uma barra:

`10.23.2.0/24` é equivalente a `10.23.2.0` com máscara `255.255.255.0`

A tabela seguinte mostra as máscaras mais comuns e as suas representações CIDR:

| Forma longa | Forma CIDR |
|-------------|------------|
| 255.0.0.0 | /8 |
| 255.255.0.0 | /16 |
| 255.240.0.0 | /12 |
| 255.255.255.0 | /24 |
| 255.255.255.192 | /26 |

A máscara `/24` é a mais comum em redes locais de utilizadores. Uma rede `/24` comporta 254 hosts (os endereços com a parte de host a `0` e a `255` são reservados).

#### IPv6

O IPv4 com 32 bits permite cerca de 4,3 mil milhões de endereços, um número que se revelou insuficiente para a escala actual da internet. O IPv6 usa endereços de 128 bits, representados em oito grupos de quatro dígitos hexadecimais separados por dois pontos:

```
2001:0db8:0a0b:12f0:0000:0000:0000:8b6e
```

Os zeros iniciais em cada grupo podem ser omitidos, e um único bloco contíguo de grupos de zeros pode ser substituído por `::`:

```
2001:db8:a0b:12f0::8b6e
```

Os hosts IPv6 têm normalmente dois endereços em simultâneo. O **endereço unicast global** é válido em toda a internet e começa sempre com `2` ou `3`. O **endereço link-local** é válido apenas na rede local e tem sempre o prefixo `fe80::/64`.

A maioria das redes modernas corre em modo **dual-stack**, onde IPv4 e IPv6 coexistem na mesma interface. Os dois protocolos são independentes e a aplicação é que escolhe qual usar para cada ligação.

### 1.5 Rotas e a tabela de encaminhamento

O kernel Linux mantém uma tabela de encaminhamento que define como chegar a cada destino de rede. Para cada pacote enviado, o kernel consulta esta tabela e decide: este destino está na rede local e pode ser alcançado directamente, ou tem de ser enviado para um router intermédio?

Para ver a tabela de encaminhamento actual:

```bash
$ ip route show
default via 10.23.2.1 dev enp0s31f6 proto static metric 100
10.23.2.0/24 dev enp0s31f6 proto kernel scope link src 10.23.2.4 metric 100
```

A segunda linha diz que a rede `10.23.2.0/24` é acessível directamente através da interface `enp0s31f6`. A primeira linha é a **rota por defeito**: qualquer destino que não corresponda a nenhuma outra regra é encaminhado para `10.23.2.1`, que é o router da rede local. Este endereço chama-se **gateway por defeito**.

#### O gateway por defeito

O gateway por defeito é o router através do qual todo o tráfego destinado ao exterior da rede local é enviado. Em notação CIDR, a rota por defeito é `0.0.0.0/0` para IPv4, porque corresponde a qualquer endereço possível. Quando nenhuma outra rota é mais específica, esta é sempre a que se aplica.

Por convenção, o router de uma rede `/24` está normalmente no primeiro endereço utilizável da sub-rede. Numa rede `10.23.2.0/24`, o gateway estará tipicamente em `10.23.2.1`. Trata-se apenas de uma convenção amplamente seguida, não de uma regra técnica.

### 1.6 Interfaces de rede

O kernel Linux mantém uma separação clara entre a camada física e a camada de rede, e liga-as através das **interfaces de rede**. Cada interface tem um nome que identifica o hardware subjacente.

Nas distribuições modernas com systemd, os nomes das interfaces são **preditivos**: derivam da localização física do hardware no sistema, o que garante que o nome não muda entre reboots. Um nome como `enp0s31f6` identifica uma placa Ethernet num slot PCI específico. Os nomes tradicionais `eth0`, `eth1` ainda aparecem em sistemas mais antigos ou em certas configurações de virtualização.

Existem também interfaces especiais sem hardware físico:

`lo` é a interface de loopback, sempre com o endereço `127.0.0.1`. É usada pelo sistema para comunicar consigo próprio. Processos locais comunicam entre si através do loopback sem que os dados saiam da máquina. Esta interface está sempre presente e activa.

Para ver todas as interfaces e os seus endereços:

```bash
$ ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host

2: enp0s31f6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 40:8d:5c:fc:24:1f brd ff:ff:ff:ff:ff:ff
    inet 10.23.2.4/24 brd 10.23.2.255 scope global enp0s31f6
    inet6 2001:db8:8500:e:52b6:59cc:74e9:8b6e/64 scope global
    inet6 fe80::d05c:97f9:7be8:bca/64 scope link
```

Cada interface tem um número sequencial. A interface `lo` é sempre a 1. Para cada interface activa, os campos mais importantes a verificar são:

`UP` indica que a interface está activa e operacional. A ausência desta palavra significa que a interface está desactivada.

`link/ether` seguido de seis pares hexadecimais é o **endereço MAC** (*Media Access Control*), o identificador único atribuído ao hardware de rede pelo fabricante. É usado na camada física para identificar dispositivos dentro da mesma rede local.

`inet` seguido de um endereço e máscara em CIDR é o endereço IPv4 da interface.

`inet6` com `scope global` é o endereço IPv6 válido em toda a internet. `inet6` com `scope link` é o endereço link-local, válido apenas na rede local.

A forma abreviada `ip a` produz o mesmo output:

```bash
$ ip a
```

### 1.7 Ferramentas de diagnóstico de rede

Um administrador de sistemas usa regularmente um conjunto de ferramentas para verificar a conectividade, diagnosticar problemas e analisar o comportamento da rede. Estas ferramentas formam um fluxo de diagnóstico progressivo: começa-se por verificar se a interface está activa, depois se há conectividade local, depois se há conectividade com o exterior, e finalmente qual o caminho que o tráfego percorre.

#### ping: verificar conectividade

O `ping` é a ferramenta de diagnóstico mais básica. Envia pacotes ICMP *echo request* para um destino e aguarda a resposta *echo reply*. É o equivalente a bater à porta de um host e ver se responde:

```bash
$ ping -c 4 10.23.2.1
PING 10.23.2.1 (10.23.2.1) 56(84) bytes of data.
64 bytes from 10.23.2.1: icmp_seq=1 ttl=64 time=1.76 ms
64 bytes from 10.23.2.1: icmp_seq=2 ttl=64 time=2.35 ms
64 bytes from 10.23.2.1: icmp_seq=4 ttl=64 time=1.69 ms
64 bytes from 10.23.2.1: icmp_seq=5 ttl=64 time=1.61 ms
```

A opção `-c 4` limita o número de pacotes enviados a 4. Sem ela, o `ping` corre indefinidamente até ser interrompido com `Ctrl+C`.

Os campos mais importantes do output são o número de sequência `icmp_seq` e o tempo de ida e volta `time`. Uma lacuna nos números de sequência, como a que existe entre `2` e `4` no exemplo, indica perda de pacotes. Numa rede local por cabo, o valor esperado é 0% de perda e tempos abaixo de 1ms. Se o destino for inacessível, o router mais próximo responde com um pacote ICMP "host unreachable".

Numa sequência de diagnóstico, o `ping` serve para:

```bash
# 1. Verificar que a interface local está activa (loopback)
$ ping -c 1 127.0.0.1

# 2. Verificar conectividade com o gateway
$ ping -c 4 10.23.2.1

# 3. Verificar conectividade com a internet
$ ping -c 4 8.8.8.8

# 4. Verificar resolução de nomes DNS
$ ping -c 4 google.com
```

Se o passo 3 funciona mas o passo 4 falha, o problema está na resolução de DNS, não na conectividade de rede.

>**Nota:** muitos firewalls e routers bloqueiam pacotes ICMP por razões de segurança. Um host que não responde ao `ping` pode estar activo mas configurado para ignorar estes pacotes. A ausência de resposta não é prova definitiva de inacessibilidade.

#### traceroute: mapear o caminho até ao destino

O `traceroute` mostra cada salto (*hop*) que o tráfego percorre entre o host local e o destino. Para cada router intermédio, mostra o endereço IP, o nome de host (quando disponível) e o tempo de resposta em três medições:

```bash
$ traceroute google.com
traceroute to google.com (142.250.184.110), 30 hops max, 60 byte packets
 1  gateway (192.168.1.1)          1.1 ms   1.3 ms   1.7 ms
 2  * * *
 3  10.200.4.1                    14.6 ms  14.9 ms  15.2 ms
 4  212.48.96.241                 17.6 ms  17.6 ms  17.9 ms
 5  74.125.243.113                22.8 ms  14.2 ms  14.4 ms
 6  142.250.184.110               21.9 ms  21.1 ms  21.6 ms
```

Os asteriscos na linha 2 indicam que esse router não respondeu às sondas do `traceroute`, por configuração ou por firewall. Isto é comum e não significa necessariamente um problema, especialmente nos primeiros saltos dentro da rede do ISP.

O `traceroute` é útil para identificar onde a conectividade falha: se os primeiros saltos respondem mas a partir de certo ponto aparecem apenas asteriscos, o problema está algures nessa parte do percurso.

#### ip: o comando central de configuração de rede

O `ip` é a ferramenta moderna para visualizar e configurar todos os aspectos de rede no Linux. Substituiu os comandos `ifconfig` e `route`, que ainda aparecem em sistemas mais antigos mas estão obsoletos. Os subcomandos mais usados são:

```bash
# Ver todas as interfaces e endereços
$ ip address show
$ ip a

# Ver apenas uma interface específica
$ ip a show enp0s31f6

# Ver a tabela de encaminhamento
$ ip route show
$ ip r

# Ver a tabela de encaminhamento IPv6
$ ip -6 route show

# Adicionar um endereço a uma interface (requer root)
$ sudo ip address add 192.168.1.100/24 dev enp0s31f6

# Activar ou desactivar uma interface
$ sudo ip link set enp0s31f6 up
$ sudo ip link set enp0s31f6 down

# Adicionar uma rota por defeito
$ sudo ip route add default via 192.168.1.1
```

As alterações feitas com `ip` são temporárias: não sobrevivem a um reboot. Para configurações permanentes, usa-se o NetworkManager ou os ficheiros de configuração em `/etc/sysconfig/network-scripts/` em CentOS.

#### ss e netstat: conexões e portas activas

O `ss` (*socket statistics*) é a ferramenta moderna para examinar as conexões de rede activas e as portas em escuta. O `netstat` é o seu predecessor, ainda amplamente usado e disponível na maioria dos sistemas:

```bash
# Ver todas as conexões TCP activas
$ ss -t

# Ver todas as portas em escuta (TCP e UDP)
$ ss -tuln

# Ver portas em escuta com o processo que as abriu (requer root)
$ sudo ss -tulnp

# Equivalente com netstat
$ netstat -tuln
$ sudo netstat -tulnp
```

As opções mais úteis do `ss`:

| Opção | Efeito |
|-------|--------|
| `-t` | Mostrar conexões TCP |
| `-u` | Mostrar conexões UDP |
| `-l` | Mostrar apenas sockets em escuta |
| `-n` | Mostrar números em vez de nomes de serviços |
| `-p` | Mostrar o processo associado a cada socket |

Para um administrador, `ss -tulnp` é o comando mais útil: mostra todas as portas em escuta com o processo responsável por cada uma. Permite verificar quais os serviços activos e em que portas estão a escutar, o que é informação essencial para diagnóstico e para auditoria de segurança:

```bash
$ sudo ss -tulnp
Netid  State   Recv-Q Send-Q  Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0      128           0.0.0.0:22      0.0.0.0:*          users:(("sshd",pid=1089))
tcp    LISTEN  0      128           0.0.0.0:80      0.0.0.0:*          users:(("nginx",pid=1203))
tcp    LISTEN  0      128           0.0.0.0:3306    0.0.0.0:*          users:(("mysqld",pid=1445))
```

Este output mostra que o servidor tem SSH (porta 22), Nginx (porta 80) e MySQL (porta 3306) em escuta.

## 2. Transferência de Ficheiros e Acesso Remoto

Administrar um servidor remotamente é, na prática, a maior parte do trabalho de um sysadmin. Raramente se tem acesso físico à máquina. Tudo acontece através da rede: configurar serviços, transferir ficheiros, executar comandos, monitorizar o estado do sistema. A qualidade e a segurança das ferramentas que se usam para isso determinam directamente a segurança de toda a infraestrutura.

Esta secção cobre o ecossistema completo de acesso remoto e transferência de ficheiros, desde a ferramenta central que torna tudo possível — o SSH — até às alternativas para cada caso de uso específico.

---

## 2.1 SSH: Acesso Remoto Seguro

O protocolo SSH (Secure Shell) constitui uma das abordagens baseadas em software mais robustas e amplamente adotadas para a segurança de comunicações em redes informáticas. A sua excelência reside na capacidade de fornecer encriptação transparente: sempre que dados são transmitidos de um nó para a rede, o protocolo encarrega-se da sua cifragem automática; ao alcançarem o destinatário, os dados são decifrados e restituídos à sua forma original.

Este mecanismo permite que a comunicação ocorra de forma fluida, sem que os utilizadores necessitem de intervir ativamente no processo criptográfico. Empregando algoritmos de cifra modernos e seguros, o SSH atinge um nível de fiabilidade que justifica a sua adoção mandatória em aplicações de missão crítica e nas maiores infraestruturas corporativas.

### A tríade de segurança do SSH

O protocolo SSH assenta em três pilares que, em conjunto, garantem a segurança da comunicação:

**Autenticação** é o processo de verificar a identidade de quem está a ligar. O SSH suporta autenticação por senha e, mais importante, autenticação por par de chaves criptográficas. Sem autenticação bem-sucedida, a ligação é recusada.

**Cifragem** codifica todos os dados transmitidos de forma a tornarem-se ilegíveis para qualquer observador não autorizado. Apenas as duas extremidades da ligação, cliente e servidor, possuem as chaves necessárias para decifrar o tráfego.

**Integridade** garante que os dados chegam exactamente como foram enviados. Os algoritmos de verificação do SSH detectam qualquer alteração ao conteúdo em trânsito, protegendo contra ataques de man-in-the-middle onde um terceiro tenta modificar os dados sem ser detectado.

### Arquitectura cliente-servidor

O SSH opera sob uma arquitetura estrita de Cliente-Servidor. Tipicamente, um programa servidor SSH (o daemon), gerido por um administrador de sistemas, opera em estado de escuta passiva, aceitando ou rejeitando conexões direcionadas à máquina hospedeira. Simultaneamente, utilizadores executam programas clientes SSH a partir de terminais remotos para submeter requisições ao servidor tais como pedidos de autenticação, transferência de ficheiros ou execução direta de comandos. A característica vital deste modelo é que todo o tráfego transacionado entreo cliente e o servidor é encriptado e protegido contra modificações e interceção (sniffing) por terceiros.

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

### Ficheiro de configuração do cliente SSH

O cliente SSH também tem um ficheiro de configuração: `~/.ssh/config`. Permite definir atalhos e configurações por servidor, eliminando a necessidade de repetir opções longas em cada ligação:

```
# ~/.ssh/config

Host producao
    HostName 203.0.113.10
    User deploy
    IdentityFile ~/.ssh/id_ed25519_producao
    Port 2222

Host bastion
    HostName 203.0.113.1
    User carlos
    IdentityFile ~/.ssh/id_ed25519

Host interno
    HostName 10.0.0.50
    User carlos
    ProxyJump bastion
```

Com esta configuração, `ssh producao` é equivalente a `ssh -p 2222 -i ~/.ssh/id_ed25519_producao deploy@203.0.113.10`. A directiva `ProxyJump` permite ligar a servidores internos através de um servidor de salto (bastion host) numa única linha de comando.

---

## 2.2 Telnet

O Telnet foi durante décadas o standard para acesso remoto a servidores. Funciona de forma simples: estabelece uma ligação TCP à porta 23 do servidor e transmite o input do teclado directamente, recebendo o output do terminal em troca.

O problema é exactamente essa simplicidade: o Telnet não cifra nada. Credenciais, comandos e respostas passam pela rede em texto plano. Com ferramentas como o `tcpdump` ou o Wireshark, qualquer pessoa com acesso à rede entre o cliente e o servidor pode ler toda a sessão em tempo real, incluindo a senha de login.

```bash
# Captura de tráfego Telnet — a senha aparece em texto claro
$ sudo tcpdump -i eth0 -A port 23
```

O SSH surgiu em 1995 precisamente para resolver este problema, e desde então o Telnet tornou-se obsoleto para administração de sistemas. Não deve existir em nenhum servidor de produção.

Para verificar e desactivar o Telnet em CentOS:

```bash
# Verificar se o telnet-server está instalado
$ rpm -q telnet-server

# Desactivar e parar o serviço se estiver activo
$ sudo systemctl disable telnet.socket
$ sudo systemctl stop telnet.socket

# Remover o pacote
$ sudo dnf remove telnet-server
```

O único uso legítimo do cliente Telnet nos dias de hoje é como ferramenta de diagnóstico para testar se uma porta TCP está aberta e se um serviço está a responder:

```bash
# Testar se o servidor web está a responder na porta 80
$ telnet 192.168.1.10 80
Trying 192.168.1.10...
Connected to 192.168.1.10.
Escape character is '^]'.
```

Para este uso específico, `nc` (netcat) ou `curl` são alternativas mais adequadas, mas o Telnet ainda aparece frequentemente em contextos de diagnóstico rápido.

---

## 2.3 wget e curl

### wget: transferência directa e recursiva

O `wget` é uma ferramenta de linha de comandos para transferir ficheiros via HTTP, HTTPS e FTP. É não-interactivo: recebe um URL, descarrega o conteúdo, e termina. Isto torna-o ideal para scripts e automação.

A utilização mais básica descarrega um ficheiro para o directório actual:

```bash
$ wget https://exemplo.com/ficheiro.tar.gz
--2025-10-15 11:02:51--  https://exemplo.com/ficheiro.tar.gz
Resolving exemplo.com... 203.0.113.10
Connecting to exemplo.com|203.0.113.10|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15728640 (15M) [application/gzip]
Saving to: 'ficheiro.tar.gz'

ficheiro.tar.gz   100%[===================>]  15.00M  2.34MB/s  in 6.4s
```

Opções frequentes em contexto de administração:

```bash
# Descarregar para um nome de ficheiro específico
$ wget -O /tmp/pacote.tar.gz https://exemplo.com/ficheiro.tar.gz

# Continuar um download interrompido
$ wget -c https://exemplo.com/ficheiro-grande.iso

# Descarregar em background (útil para ficheiros grandes)
$ wget -b https://exemplo.com/ficheiro-grande.iso
# O progress é guardado em wget-log

# Descarregar com autenticação HTTP básica
$ wget --user=carlos --password=senha https://servidor-interno/ficheiro

# Limitar a velocidade de download (para não saturar a ligação)
$ wget --limit-rate=5m https://exemplo.com/ficheiro.tar.gz

# Descarregar recursivamente um site (com profundidade limitada)
$ wget -r -l 2 https://exemplo.com/docs/
```

### curl: transferência e interacção com APIs

O `curl` é mais versátil que o `wget`. Enquanto o `wget` é especializado em descarregar ficheiros, o `curl` suporta dezenas de protocolos e é a ferramenta standard para interagir com APIs HTTP. Escreve o output para stdout por defeito, o que o torna ideal para pipelines:

```bash
# Ver o conteúdo de uma URL
$ curl https://exemplo.com

# Descarregar para um ficheiro
$ curl -o ficheiro.tar.gz https://exemplo.com/ficheiro.tar.gz

# Seguir redireccionamentos
$ curl -L https://exemplo.com/download

# Ver os cabeçalhos HTTP de resposta
$ curl -I https://exemplo.com

# Fazer um pedido POST com dados JSON
$ curl -X POST \
       -H "Content-Type: application/json" \
       -d '{"utilizador": "carlos", "acção": "login"}' \
       https://api.exemplo.com/auth

# Fazer um pedido com autenticação por token
$ curl -H "Authorization: Bearer eyJhb..." https://api.exemplo.com/dados

# Ver o progresso detalhado da ligação (útil para diagnóstico SSL)
$ curl -v https://exemplo.com

# Testar se um serviço está a responder (sem output)
$ curl -s -o /dev/null -w "%{http_code}" https://exemplo.com
200
```

A última linha é particularmente útil em scripts de monitorização: verifica o código HTTP de resposta de um serviço sem precisar de processar o corpo da resposta.

---

## 2.4 FTP

O FTP (*File Transfer Protocol*) é um dos protocolos mais antigos da internet. Permite navegar em sistemas de ficheiros remotos e transferir ficheiros em ambas as direcções. A sua maior fraqueza é a mesma do Telnet: transmite credenciais e dados em texto plano.

Por esta razão, o FTP anónimo ainda existe em servidores de distribuição pública de software, onde não há credenciais para proteger. Para qualquer uso autenticado, o FTP foi substituído pelo SFTP (que corre sobre SSH) e pelo SCP.

Uma sessão FTP típica para descarregar um ficheiro de um servidor anónimo:

```bash
$ ftp fileserver.exemplo.com
Connected to fileserver.exemplo.com.
Name: anonymous
Password: [endereço de email ou string vazia]

ftp> cd pub/releases
ftp> ls
ftp> lcd /tmp
ftp> get pacote-1.0.tar.gz
ftp> bye
```

Os comandos mais usados numa sessão FTP interactiva:

| Comando | Significado |
|---------|-------------|
| `ftp servidor` | Ligar ao servidor FTP |
| `anonymous` | Nome de utilizador para servidores públicos |
| `cd directorio` | Mudar de directório no servidor remoto |
| `ls` | Listar o directório remoto |
| `lcd Desktop` | Mudar directório local |
| `get ficheiro` | Descarregar ficheiro do servidor |
| `put ficheiro` | Enviar ficheiro para o servidor |
| `bye` | Terminar a sessão |

Para substituição moderna do FTP com segurança, use sempre SFTP ou SCP.

---

### sftp

O `sftp` oferece uma interface interactiva semelhante ao FTP, mas sobre um túnel SSH. Não requer um servidor FTP dedicado no destino: basta que o SSH esteja activo:

```bash
$ sftp carlos@192.168.1.10
Connected to 192.168.1.10.
sftp> ls
sftp> cd /var/log
sftp> get syslog
sftp> lcd /tmp
sftp> put configuracao.txt
sftp> bye
```

### rsync

O `rsync` é a ferramenta de eleição para sincronização de directórios e backups. A diferença fundamental em relação ao `scp` é que o `rsync` compara o conteúdo de origem e destino e transfere apenas as diferenças. Numa primeira sincronização, transfere tudo. Nas seguintes, transfere apenas o que mudou, tornando o processo muito mais eficiente para conjuntos de dados grandes.

```bash
# Sincronizar directório local para servidor remoto
$ rsync -avz /opt/app/ carlos@192.168.1.10:/opt/app/

# Sincronizar com delete (remove no destino o que não existe na origem)
$ rsync -avz --delete /opt/app/ carlos@192.168.1.10:/opt/app/

# Backup com preservação de permissões e links
$ rsync -avzP /dados/ carlos@backup-server:/backup/dados/

# Sincronizar sobre SSH com porta alternativa
$ rsync -avz -e 'ssh -p 2222' /dados/ carlos@192.168.1.10:/backup/

# Simulação sem transferir nada (dry run)
$ rsync -avz --dry-run /opt/app/ carlos@192.168.1.10:/opt/app/
```

As opções mais usadas do `rsync`:

| Opção | Efeito |
|-------|--------|
| `-a` | Modo arquivo: preserva permissões, timestamps, links simbólicos e proprietários |
| `-v` | Verbose: mostra os ficheiros sendo transferidos |
| `-z` | Comprime os dados durante a transferência |
| `-P` | Mostra progresso e permite retomar transferências interrompidas |
| `--delete` | Remove no destino ficheiros que não existem na origem |
| `--dry-run` | Simula a operação sem transferir nada |
| `--exclude` | Exclui padrões de ficheiros da sincronização |

O `rsync` é a base de muitas soluções de backup em Linux. Uma tarefa cron que corre `rsync` nocturnamente para um servidor de backup remoto é uma das implementações de backup mais simples e fiáveis disponíveis.

---

## 2.6 Análise de Tráfego com tcpdump

O `tcpdump` é uma ferramenta de captura e análise de pacotes de rede em tempo real. Requer privilégios de root porque precisa de aceder à interface de rede em modo promíscuo, lendo todos os pacotes que passam, não apenas os destinados à máquina local. É a ferramenta de diagnóstico de rede mais poderosa disponível na linha de comandos.

A forma mais simples captura todos os pacotes na interface activa:

```bash
$ sudo tcpdump -i enp0s31f6
```

Sem filtros, o output é avassalador. O valor real do `tcpdump` está nos filtros que permitem isolar exactamente o tráfego relevante:

```bash
# Capturar apenas tráfego na porta 80 (HTTP)
$ sudo tcpdump -i enp0s31f6 port 80

# Capturar tráfego de ou para um host específico
$ sudo tcpdump -i enp0s31f6 host 192.168.1.10

# Capturar tráfego entre dois hosts específicos
$ sudo tcpdump -i enp0s31f6 src 192.168.1.10 and dst 192.168.1.20

# Capturar apenas tráfego SSH
$ sudo tcpdump -i enp0s31f6 port 22

# Ver o conteúdo dos pacotes em ASCII (útil para protocolos de texto)
$ sudo tcpdump -i enp0s31f6 -A port 80

# Guardar captura para ficheiro (para análise posterior com Wireshark)
$ sudo tcpdump -i enp0s31f6 -w /tmp/captura.pcap

# Ler ficheiro de captura guardado
$ sudo tcpdump -r /tmp/captura.pcap

# Limitar o número de pacotes capturados
$ sudo tcpdump -i enp0s31f6 -c 100 port 443
```

### Casos de uso práticos

**Verificar se um serviço está a receber conexões:**

```bash
$ sudo tcpdump -i enp0s31f6 -n port 443 and tcp[tcpflags] & tcp-syn != 0
```
Este filtro mostra apenas os pacotes SYN (início de conexão TCP) na porta 443, o que confirma que clientes estão a tentar ligar ao HTTPS.

**Diagnosticar problemas de DNS:**

```bash
$ sudo tcpdump -i enp0s31f6 -n port 53
```
Mostra todas as queries e respostas DNS, útil para verificar se as consultas estão a sair e se chegam respostas.

**Confirmar que o SSH está a transmitir dados cifrados:**

```bash
$ sudo tcpdump -i enp0s31f6 -A port 22
```

O conteúdo deve ser ilegível. Se aparecer texto em claro numa sessão que deveria ser SSH, há algo muito errado.

> **O `tcpdump` em redes partilhadas pode capturar dados de outros utilizadores.** Usar `tcpdump` para interceptar tráfego que não é seu, mesmo numa rede que administra, pode ter implicações legais e de privacidade. Use-o estritamente para diagnóstico de problemas nos sistemas que administra.


## 4. Serviços de Rede Essenciais

Um servidor Linux raramente existe isolado. Faz parte de uma infraestrutura onde vários serviços comunicam entre si e servem clientes. E a maioria desses serviços depende de infraestrutura invisível: resolução de nomes para encontrar outros servidores, distribuição automática de endereços para que as máquinas entrem na rede sem configuração manual, sincronização de tempo para que os logs e certificados façam sentido, e um sistema centralizado de recolha de logs para saber o que se passa em toda a rede.

Estes serviços têm uma característica comum: quando funcionam, ninguém repara neles. Quando falham, tudo parece estar avariado ao mesmo tempo, e a causa raramente é óbvia. Esta secção cobre cada um deles com a profundidade necessária para os configurar e, mais importante, para os diagnosticar quando algo corre mal.

---

## 4.1 DNS: O Sistema de Nomes de Domínio

### O problema que o DNS resolve

Os endereços IP são eficientes para as máquinas e insuportáveis para os humanos. Ninguém memoriza `172.217.168.46` quando quer aceder ao Google, e mesmo que memorizasse, esse endereço pode mudar amanhã sem aviso. O DNS resolve este problema mapeando nomes legíveis para os endereços numéricos que a rede realmente usa.

Mas reduzir o DNS a uma tabela de tradução seria subestimá-lo. O DNS é também o mecanismo que permite localizar os servidores de email de um domínio, descobrir quais os servidores autoritativos de uma zona, verificar a autenticidade de remetentes de email, e publicar informação arbitrária associada a um domínio. É a infraestrutura sobre a qual quase tudo o resto assenta.

A consequência prática é directa: quando o DNS falha, a rede continua funcional mas parece completamente avariada. Os pacotes fluem normalmente, o gateway responde, mas nada funciona porque nenhum nome é resolvido. Aprender a distinguir "a rede está em baixo" de "o DNS está em baixo" é uma das competências mais úteis de diagnóstico que um administrador pode ter.

### Uma base de dados distribuída

O DNS é uma base de dados distribuída à escala planetária. Nenhum servidor conhece todos os nomes que existem. Em vez disso, a responsabilidade está delegada em hierarquia: cada organização mantém os registos dos seus próprios hosts, e os servidores cooperam entre si quando precisam de informação que não possuem.

A hierarquia funciona de cima para baixo. No topo estão os servidores raiz, que não conhecem nomes individuais mas sabem quais os servidores responsáveis por cada domínio de topo (`.com`, `.pt`, `.org`). Esses servidores conhecem os servidores responsáveis por cada domínio registado (`empresa.pt`, `google.com`). E esses conhecem os registos dos hosts específicos dentro do seu domínio.

Quando um cliente precisa de resolver `servidor.empresa.pt`, a sequência é a seguinte:

1. O cliente envia a query ao servidor DNS configurado no sistema, definido em `/etc/resolv.conf`.
2. Se esse servidor tiver a resposta em cache, devolve-a imediatamente e o processo termina aqui.
3. Se não tiver, consulta um servidor raiz, que responde indicando os servidores autoritativos de `.pt`.
4. Consulta um servidor de `.pt`, que indica os servidores autoritativos de `empresa.pt`.
5. Consulta um servidor de `empresa.pt`, que responde com o endereço IP de `servidor.empresa.pt`.
6. O servidor local guarda a resposta em cache durante o tempo definido pelo TTL do registo, e devolve-a ao cliente.

Este processo parece moroso, mas na prática resolve-se em milissegundos porque a esmagadora maioria das respostas já está em cache algures na cadeia. Um servidor DNS local bem dimensionado responde à maior parte das queries sem contactar ninguém.

### Servidores autoritativos e servidores recursivos

Existem dois papéis distintos que um servidor DNS pode desempenhar, e confundi-los é fonte de muitos mal-entendidos.

Um **servidor autoritativo** detém os dados oficiais de uma ou mais zonas. Quando alguém pergunta por um nome dentro dessas zonas, responde com autoridade porque os dados são seus. Um domínio deve ter pelo menos dois servidores autoritativos, idealmente em redes e localizações geográficas distintas, porque se todos ficarem inacessíveis o domínio deixa de existir do ponto de vista do resto da internet.

Um **servidor recursivo** não detém dados próprios. O seu trabalho é receber queries dos clientes e percorrer a hierarquia até obter uma resposta, guardando-a em cache para queries futuras. É este o tipo de servidor que se configura em `/etc/resolv.conf`.

Na prática, servidores autoritativos são normalmente configurados num modelo primário/secundário. O **servidor primário** (master) detém a cópia de referência dos dados. Os **servidores secundários** (slaves) copiam os dados do primário através de um mecanismo chamado transferência de zona (*zone transfer*), e respondem às queries com os mesmos dados. Assim, alterações feitas no primário propagam-se automaticamente.

### Registos de recurso

Os dados que compõem a base de dados DNS são chamados **registos de recurso** (*resource records*). Cada registo é uma linha que associa um nome a um tipo e a um valor. Os tipos que interessam a um administrador são estes:

| Tipo | Função |
|------|--------|
| **A** | Mapeia um hostname para um endereço IPv4 |
| **AAAA** | Mapeia um hostname para um endereço IPv6 |
| **CNAME** | Define um alias que aponta para outro nome |
| **MX** | Identifica o servidor de email de um domínio, com prioridade |
| **NS** | Identifica os servidores autoritativos de um domínio |
| **PTR** | Resolução inversa: mapeia um IP para um hostname |
| **TXT** | Texto arbitrário, usado para SPF, DKIM e verificação de propriedade |
| **SOA** | *Start of Authority*: metadados da zona (serial, refresh, expiry) |

Um exemplo de registos numa zona `empresa.pt`:

```
empresa.pt.        IN  SOA   ns1.empresa.pt. admin.empresa.pt. (
                              2025101501 ; serial
                              3600       ; refresh
                              600        ; retry
                              604800     ; expire
                              86400 )    ; minimum TTL

empresa.pt.        IN  NS    ns1.empresa.pt.
empresa.pt.        IN  NS    ns2.empresa.pt.
empresa.pt.        IN  MX    10 mail.empresa.pt.

ns1                IN  A     203.0.113.10
ns2                IN  A     203.0.113.11
www                IN  A     203.0.113.20
mail               IN  A     203.0.113.30
servidor           IN  A     192.168.1.50
ftp                IN  CNAME www.empresa.pt.
empresa.pt.        IN  TXT   "v=spf1 mx -all"
```

O campo **serial** no registo SOA é crítico em ambientes com servidores secundários: quando o serial aumenta, os secundários sabem que os dados mudaram e devem fazer uma nova transferência de zona. Esquecer de incrementar o serial após uma alteração é um erro clássico que resulta em servidores secundários a servir dados desactualizados indefinidamente.

### Resolução inversa

A resolução inversa mapeia um endereço IP de volta para um nome, usando registos PTR numa zona especial chamada `in-addr.arpa`. Para o endereço `203.0.113.20`, o nome consultado é `20.113.0.203.in-addr.arpa`, com os octetos invertidos.

É importante saber que a resolução inversa é frequentemente pouco fiável. Um endereço IP pode estar associado a vários hostnames, e o DNS não tem forma de saber qual deles é o "correcto". Além disso, o registo PTR tem de ser configurado manualmente pelo responsável pelo bloco de endereços, o que muitos administradores não fazem. Não se deve depender de resolução inversa para lógica crítica.

A excepção importante é o email: muitos servidores de email rejeitam mensagens de servidores sem registo PTR válido, por ser um indicador comum de spam. Se administra um servidor de email, garanta que o registo PTR do seu endereço está configurado.

### Configuração do cliente DNS

O ficheiro `/etc/resolv.conf` define quais os servidores DNS que o sistema consulta:

```
# /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
search empresa.pt
options timeout:2 attempts:3
```

A directiva `nameserver` define o endereço do servidor DNS. Podem listar-se até três; são consultados por ordem, e o segundo só é usado se o primeiro não responder dentro do timeout.

A directiva `search` define domínios que são acrescentados automaticamente a nomes curtos. Com `search empresa.pt` configurado, um pedido de resolução de `servidor` é expandido para `servidor.empresa.pt`. Isto permite usar nomes curtos dentro da rede da organização.

Em sistemas com NetworkManager, o `/etc/resolv.conf` é gerado automaticamente e qualquer edição manual será sobrescrita. As alterações fazem-se através do NetworkManager:

```bash
# Ver a configuração DNS actual de uma interface
$ nmcli device show enp0s31f6 | grep DNS

# Definir servidores DNS manualmente numa ligação
$ sudo nmcli connection modify "System enp0s31f6" ipv4.dns "8.8.8.8 1.1.1.1"
$ sudo nmcli connection modify "System enp0s31f6" ipv4.ignore-auto-dns yes
$ sudo nmcli connection up "System enp0s31f6"
```

### /etc/hosts e a ordem de resolução

Antes de consultar o DNS, o sistema verifica o ficheiro `/etc/hosts`, que contém mapeamentos estáticos locais:

```
# /etc/hosts
127.0.0.1   localhost localhost.localdomain
::1         localhost localhost.localdomain
192.168.1.50  servidor-app servidor-app.empresa.pt
192.168.1.51  servidor-db  servidor-db.empresa.pt
```

Este ficheiro é útil para ambientes de laboratório, para nomes que não devem depender do DNS, e para forçar temporariamente um nome a resolver para um endereço específico durante testes ou migrações.

A ordem pela qual o sistema consulta as diferentes fontes de resolução é definida em `/etc/nsswitch.conf`:

```
# /etc/nsswitch.conf
hosts:      files dns myhostname
```

A palavra `files` refere-se a `/etc/hosts`, e `dns` ao sistema DNS. A ordem importa: com esta configuração, `/etc/hosts` tem precedência sobre o DNS. Uma entrada em `/etc/hosts` sobrepõe-se sempre à resposta que o DNS daria, o que é útil mas também uma fonte comum de confusão quando alguém esquece uma entrada de teste no ficheiro.

---

## 4.2 Ferramentas de Consulta DNS

Quando a resolução de nomes falha, é preciso consultar o DNS directamente para descobrir onde está o problema. Existem três ferramentas principais, com propósitos ligeiramente diferentes.

### host

O `host` é a ferramenta mais simples, ideal para verificações rápidas:

```bash
# Resolver um hostname
$ host www.exemplo.com
www.exemplo.com has address 203.0.113.10
www.exemplo.com has IPv6 address 2001:db8::1

# Resolução inversa
$ host 203.0.113.10
10.113.0.203.in-addr.arpa domain name pointer www.exemplo.com.

# Consultar um tipo de registo específico
$ host -t MX empresa.pt
empresa.pt mail is handled by 10 mail.empresa.pt.

$ host -t NS empresa.pt
empresa.pt name server ns1.empresa.pt.
empresa.pt name server ns2.empresa.pt.

# Consultar um servidor DNS diferente
$ host www.exemplo.com 1.1.1.1
```

### nslookup

O `nslookup` funciona em modo directo ou interactivo. O modo interactivo é conveniente quando se quer fazer várias consultas seguidas com configurações diferentes:

```bash
$ nslookup
> server 8.8.8.8
Default server: 8.8.8.8

> set type=MX
> empresa.pt
empresa.pt  mail exchanger = 10 mail.empresa.pt.

> set type=A
> www.empresa.pt
Name:    www.empresa.pt
Address: 203.0.113.20

> exit
```

### dig

O `dig` é a ferramenta preferida dos administradores. Produz um output verboso que mostra exactamente o que foi perguntado, o que foi respondido, qual servidor respondeu e quanto tempo demorou:

```bash
$ dig www.exemplo.com

; <<>> DiG 9.16.23 <<>> www.exemplo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24601
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;www.exemplo.com.               IN      A

;; ANSWER SECTION:
www.exemplo.com.        300     IN      A       203.0.113.10

;; Query time: 12 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Wed Oct 15 14:23:11 UTC 2025
;; MSG SIZE  rcvd: 56
```

O campo `status` no cabeçalho é a primeira coisa a verificar. `NOERROR` significa que a query teve sucesso. `NXDOMAIN` significa que o nome não existe. `SERVFAIL` significa que o servidor teve um erro ao processar a query, muitas vezes por problemas de configuração ou DNSSEC.

O número `300` antes de `IN A` é o TTL: quantos segundos esta resposta pode ficar em cache antes de ser reconsultada.

Opções essenciais do `dig`:

```bash
# Output curto, apenas a resposta
$ dig +short www.exemplo.com
203.0.113.10

# Consultar tipo de registo específico
$ dig MX empresa.pt
$ dig NS empresa.pt
$ dig TXT empresa.pt
$ dig SOA empresa.pt

# Consultar um servidor DNS específico
$ dig @8.8.8.8 www.exemplo.com
$ dig @ns1.empresa.pt www.empresa.pt

# Resolução inversa
$ dig -x 203.0.113.10

# Traçar toda a cadeia de delegação desde os servidores raiz
$ dig +trace www.exemplo.com

# Ver apenas a secção de resposta, sem o resto
$ dig +noall +answer www.exemplo.com
```

O `dig +trace` merece destaque. Percorre a cadeia completa de resolução mostrando cada passo, desde os servidores raiz até à resposta autoritativa final. É a ferramenta certa para diagnosticar problemas de delegação: quando um domínio não resolve mas os registos parecem correctos, o `+trace` mostra exactamente em que ponto a cadeia se quebra.

### Fluxo de diagnóstico de problemas DNS

Quando um nome não resolve, a sequência de verificação é esta:

```bash
# 1. O servidor DNS configurado está acessível?
$ cat /etc/resolv.conf
$ ping -c 2 8.8.8.8

# 2. O servidor responde a queries?
$ dig @8.8.8.8 google.com +short

# 3. O nome específico resolve num servidor público conhecido?
$ dig @8.8.8.8 www.empresa.pt +short

# 4. Se resolve publicamente mas não localmente, o problema é o servidor local
$ dig www.empresa.pt

# 5. Se não resolve em lado nenhum, verificar a delegação
$ dig +trace www.empresa.pt

# 6. Confirmar que não há entrada conflituosa em /etc/hosts
$ grep empresa.pt /etc/hosts
```

---

## 4.3 DHCP: Configuração Automática de Rede

### O que o DHCP faz

Quando se liga um computador a uma rede, ele normalmente obtém um endereço IP, define a rota por defeito, e descobre quais os servidores DNS a usar, tudo sem qualquer intervenção humana. O DHCP (*Dynamic Host Configuration Protocol*) é o mecanismo por trás dessa aparente magia.

O protocolo permite que um cliente peça emprestado (*lease*) um conjunto de parâmetros de rede a um servidor central autorizado a distribuí-los. O modelo de empréstimo temporário é especialmente adequado a redes onde os dispositivos entram e saem constantemente: portáteis, telemóveis, máquinas virtuais efémeras.

Os parâmetros que um servidor DHCP pode distribuir incluem:

- Endereço IP e máscara de sub-rede
- Gateway por defeito
- Servidores DNS e domínio de pesquisa
- Servidores NTP para sincronização de tempo
- Servidores de log (syslog)
- Servidores TFTP para arranque por rede (PXE boot)

A lista completa tem dezenas de parâmetros definidos no RFC 2132, mas na prática só os primeiros quatro são usados com frequência.

### Como o DHCP funciona

A interacção entre cliente e servidor segue quatro passos, conhecidos pelo acrónimo DORA:

**Discover.** O cliente arranca sem qualquer configuração de rede. Não conhece o seu endereço nem o do servidor DHCP. Envia então um pacote broadcast para toda a rede local, essencialmente a perguntar "existe algum servidor DHCP aqui?".

**Offer.** Um servidor DHCP na rede recebe o broadcast e responde com uma oferta: um endereço IP disponível e os restantes parâmetros de configuração.

**Request.** O cliente responde aceitando formalmente a oferta. Este passo existe porque pode haver mais do que um servidor DHCP na rede, e o cliente tem de indicar qual das ofertas escolheu.

**Acknowledge.** O servidor confirma a atribuição, regista o lease na sua base de dados, e o cliente configura a interface com os parâmetros recebidos.

### O conceito de lease

Um endereço atribuído por DHCP não é permanente. É cedido por um período definido, o **lease time**. Quando metade do tempo de lease decorre, o cliente tenta renová-lo contactando directamente o servidor que lho atribuiu. Se a renovação for bem-sucedida, o contador reinicia. Se o cliente desaparecer sem renovar, o lease expira e o servidor pode reatribuir o endereço a outro cliente.

O tempo de lease é configurável e envolve um compromisso. Leases curtos permitem reciclar endereços rapidamente, o que é útil em redes com muitos dispositivos transitórios como uma rede WiFi de visitantes. Leases longos reduzem o tráfego de renovação e a carga no servidor, sendo adequados a redes estáveis de escritório. Valores típicos vão de algumas horas a vários dias.

O servidor é obrigado a manter registo dos endereços que atribuiu, e essa informação tem de sobreviver a reinícios do servidor, caso contrário o servidor poderia atribuir o mesmo endereço a duas máquinas diferentes.

### Configurar um servidor DHCP com ISC dhcpd

Em redes pequenas, o router faz normalmente de servidor DHCP e não é necessário configurar nada. Em redes maiores com múltiplas sub-redes, ou quando é necessário controlo fino sobre a atribuição de endereços, configura-se um servidor DHCP dedicado.

A implementação de referência é o `dhcpd` do ISC (*Internet Systems Consortium*), disponível em todas as distribuições Linux.

```bash
$ sudo dnf install dhcp-server -y
```

A configuração fica em `/etc/dhcp/dhcpd.conf`. Antes de escrever o ficheiro, é preciso reunir a seguinte informação: quais as sub-redes a gerir e que gamas de endereços distribuir, que atribuições estáticas fazer e para que endereços MAC, e quais os tempos de lease inicial e máximo.

Um exemplo completo:

```
# /etc/dhcp/dhcpd.conf

# Opções globais aplicadas a todas as sub-redes
option domain-name "empresa.pt";
option domain-name-servers 192.168.1.10, 8.8.8.8;
option ntp-servers 192.168.1.10;

default-lease-time 3600;      # 1 hora
max-lease-time 86400;         # 24 horas

# Este servidor é a autoridade DHCP oficial desta rede
authoritative;

# Sub-rede interna com distribuição de endereços
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
    option subnet-mask 255.255.255.0;
}

# Sub-rede DMZ declarada mas sem distribuição de endereços
subnet 203.0.113.0 netmask 255.255.255.0 {
}

# Reserva estática: esta máquina recebe sempre o mesmo endereço
host servidor-impressao {
    hardware ethernet 08:00:27:12:34:56;
    fixed-address 192.168.1.50;
}

host servidor-backup {
    hardware ethernet 08:00:27:ab:cd:ef;
    fixed-address 192.168.1.51;
}
```

Alguns pontos que merecem atenção neste ficheiro.

A directiva `authoritative` declara que este servidor é a autoridade DHCP oficial da rede. Sem ela, o servidor não envia respostas negativas a clientes que pedem endereços inválidos, o que atrasa a reconfiguração desses clientes.

Todas as sub-redes ligadas às interfaces do servidor têm de ser declaradas, mesmo as que não devem receber serviço DHCP. Por isso existe a declaração vazia da sub-rede `203.0.113.0`: sem ela, o `dhcpd` recusa arrancar.

As **reservas estáticas** (blocos `host`) associam um endereço MAC específico a um endereço IP fixo. A máquina continua a usar DHCP e a receber toda a configuração automaticamente, mas recebe sempre o mesmo endereço. Esta é a forma correcta de dar endereços fixos a servidores e impressoras: mantém a configuração centralizada em vez de espalhada por cada máquina.

> ⚠️ **A sintaxe do `dhcpd.conf` é intolerante.** Um ponto e vírgula em falta produz mensagens de erro crípticas que raramente apontam para a linha correcta. Verifique sempre a sintaxe antes de reiniciar o serviço.

```bash
# Verificar sintaxe sem arrancar o serviço
$ sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# Arrancar e activar o serviço
$ sudo systemctl enable --now dhcpd

# Abrir a porta no firewall
$ sudo firewall-cmd --add-service=dhcp --permanent
$ sudo firewall-cmd --reload
```

### Ver os leases activos

O servidor mantém a base de dados de leases num ficheiro de texto:

```bash
$ sudo cat /var/lib/dhcpd/dhcpd.leases

lease 192.168.1.105 {
  starts 3 2025/10/15 14:23:11;
  ends 3 2025/10/15 15:23:11;
  cltt 3 2025/10/15 14:23:11;
  binding state active;
  next binding state free;
  hardware ethernet 08:00:27:aa:bb:cc;
  client-hostname "portatil-carlos";
}
```

Para acompanhar a actividade do servidor em tempo real:

```bash
$ sudo journalctl -u dhcpd -f
```

### O agente de relay

Um pedido DHCP inicial é enviado por broadcast, e os broadcasts não atravessam routers. Isto significa que cada sub-rede física precisaria do seu próprio servidor DHCP, o que é impraticável em redes grandes.

O **agente de relay** resolve este problema. É um pequeno serviço que corre no router de cada sub-rede, escuta os broadcasts DHCP locais, e reencaminha-os por unicast para um servidor DHCP central noutra rede. As respostas fazem o caminho inverso. Assim, um único servidor DHCP pode servir dezenas de sub-redes.

```bash
$ sudo dnf install dhcp-relay -y
$ sudo systemctl edit --full dhcrelay
# Configurar o endereço do servidor DHCP central nos argumentos
$ sudo systemctl enable --now dhcrelay
```

### O lado do cliente

O cliente DHCP raramente precisa de configuração. Em CentOS com NetworkManager, basta a interface estar definida como automática:

```bash
# Ver a configuração obtida por DHCP
$ nmcli device show enp0s31f6

# Forçar renovação do lease
$ sudo nmcli connection down "System enp0s31f6"
$ sudo nmcli connection up "System enp0s31f6"

# Definir uma interface para usar DHCP
$ sudo nmcli connection modify "System enp0s31f6" ipv4.method auto
```

>  **Só deve existir um servidor DHCP activo por sub-rede.** Dois servidores a distribuir endereços da mesma gama produzem conflitos de endereçamento difíceis de diagnosticar, porque os sintomas são intermitentes e aparentemente aleatórios. Um servidor DHCP não autorizado numa rede (o chamado *rogue DHCP server*) é também um vector de ataque conhecido: permite ao atacante distribuir um gateway ou servidor DNS malicioso.

---

## 4.4 NTP e chronyd: Sincronização de Tempo

### Por que o tempo importa

O relógio de um servidor não é uma comodidade, é infraestrutura crítica. As consequências de um relógio desfasado atravessam todo o sistema.

Logs com timestamps incorrectos tornam impossível correlacionar eventos entre servidores durante um incidente. Se o servidor web regista um erro às 14:32 e o servidor de base de dados regista o erro correspondente às 14:47 porque o seu relógio está adiantado, reconstruir a sequência de eventos torna-se um exercício de adivinhação.

Certificados SSL/TLS têm validade temporal. Um servidor com o relógio significativamente errado rejeita certificados válidos ou aceita certificados expirados. Ligações HTTPS começam a falhar com erros que não fazem sentido.

Sistemas de autenticação como o Kerberos rejeitam tickets com mais de cinco minutos de diferença entre cliente e servidor, precisamente para evitar ataques de replay. Um relógio desfasado impede o login.

Bases de dados replicadas e sistemas de ficheiros distribuídos dependem de ordenação temporal para resolver conflitos. Relógios desalinhados corrompem essa ordenação.

Num servidor com relógio errado, a questão não é se algo vai falhar. É o quê, e quando é que alguém vai descobrir a causa real.

### O protocolo NTP e a hierarquia de stratum

O NTP (*Network Time Protocol*) organiza as fontes de tempo em níveis hierárquicos chamados **stratum**.

O **stratum 0** são os dispositivos de referência: relógios atómicos, receptores GPS, relógios de rádio. Não estão ligados à rede directamente.

O **stratum 1** são servidores ligados directamente a uma fonte de stratum 0. São as fontes mais precisas acessíveis pela rede.

O **stratum 2** sincroniza com servidores de stratum 1, o **stratum 3** com stratum 2, e assim sucessivamente até ao stratum 15. O stratum 16 indica um relógio não sincronizado.

Cada nível acrescenta uma pequena margem de erro, mas na prática qualquer stratum entre 2 e 4 oferece precisão largamente suficiente para um servidor.

### chronyd: o cliente NTP em CentOS

O `chronyd` é o daemon de sincronização padrão no CentOS Stream e em todas as distribuições baseadas em RHEL. Substituiu o `ntpd` tradicional por boas razões: converge muito mais rapidamente para a hora correcta, lida melhor com redes de qualidade variável, e funciona bem em sistemas que são frequentemente desligados ou suspensos, como máquinas virtuais e portáteis.

```bash
# Verificar se está instalado e activo
$ systemctl status chronyd

# Se necessário instalar
$ sudo dnf install chrony -y
$ sudo systemctl enable --now chronyd
```

### Configuração

O ficheiro de configuração é `/etc/chrony.conf`:

```bash
# /etc/chrony.conf

# Servidores NTP a usar
pool 2.centos.pool.ntp.org iburst

# Ou servidores específicos
# server ntp1.empresa.pt iburst
# server ntp2.empresa.pt iburst

# Ficheiro onde o chronyd regista a deriva do relógio local
driftfile /var/lib/chrony/drift

# Permitir ajuste abrupto nas 3 primeiras actualizações se o desvio for > 1s
makestep 1.0 3

# Sincronizar também o relógio de hardware (RTC)
rtcsync

# Registar no log ajustes superiores a 0.5 segundos
logchange 0.5

# Directório dos logs
logdir /var/log/chrony
```

A directiva `iburst` acelera drasticamente a sincronização inicial. Em vez de enviar um pacote por minuto, envia uma rajada de pacotes ao arrancar, convergindo em segundos em vez de minutos.

A directiva `makestep 1.0 3` merece explicação. Por defeito, o `chronyd` corrige o relógio gradualmente (*slewing*), acelerando ou abrandando ligeiramente a passagem do tempo até ao valor correcto. Isto evita saltos temporais que confundem aplicações. Mas se o desvio inicial for muito grande, o slewing demoraria horas. O `makestep` permite um salto abrupto (*step*), mas apenas nas três primeiras actualizações após o arranque. Depois disso, os ajustes voltam a ser sempre graduais.

### Verificar o estado

```bash
# Estado geral e qualidade da sincronização
$ chronyc tracking
Reference ID    : 51360401 (ntp1.exemplo.pt)
Stratum         : 3
Ref time (UTC)  : Wed Oct 15 14:30:00 2025
System time     : 0.000042318 seconds fast of NTP time
Last offset     : +0.000031234 seconds
RMS offset      : 0.000124891 seconds
Frequency       : 2.614 ppm fast
Skew            : 24.449 ppm
Root delay      : 0.012041217 seconds
Leap status     : Normal
```

Os campos mais importantes são o `Stratum` (deve ser baixo, tipicamente 2 a 4), o `System time` (o desvio actual, deve ser da ordem dos milissegundos) e o `Leap status` (deve ser `Normal`).

```bash
# Ver todas as fontes configuradas e o estado de cada uma
$ chronyc sources -v

MS Name/IP address         Stratum Poll Reach LastRx Last sample
^* ntp1.exemplo.pt               2   6   377    45  -0.031ms[+0.014ms]
^+ ntp2.exemplo.pt               2   6   377    43  +0.122ms[+0.107ms]
^- ntp3.exemplo.pt               3   7   377   112  -0.891ms[-0.905ms]
^? ntp4.exemplo.pt               0   6     0     -  +0.000ms[+0.000ms]
```

Os símbolos na primeira coluna indicam o estado de cada fonte:

| Símbolo | Significado |
|---------|-------------|
| `^*` | Fonte actualmente seleccionada para sincronização |
| `^+` | Fonte aceitável, candidata a substituir a principal |
| `^-` | Fonte rejeitada por divergir das restantes |
| `^?` | Fonte inacessível ou ainda sem dados suficientes |
| `^x` | Fonte considerada incorrecta (*falseticker*) |

A coluna `Reach` é um valor octal que representa as últimas 8 tentativas de contacto. O valor `377` significa que todas as 8 tiveram sucesso. Valores mais baixos indicam perda de pacotes ou um servidor intermitente.

```bash
# Forçar sincronização imediata
$ sudo chronyc makestep

# Estado integrado com systemd
$ timedatectl
               Local time: Wed 2025-10-15 14:30:00 WET
           Universal time: Wed 2025-10-15 14:30:00 UTC
                 RTC time: Wed 2025-10-15 14:30:00
                Time zone: Europe/Lisbon (WET, +0000)
System clock synchronized: yes
              NTP service: active

# Definir o fuso horário
$ sudo timedatectl set-timezone Europe/Lisbon
```

### chronyd como servidor NTP interno

Em infraestruturas com vários servidores, a boa prática é ter um ou dois servidores NTP internos que sincronizam com fontes externas, e ter todos os restantes hosts a sincronizar com esses servidores internos. Isto reduz o tráfego externo, garante que todos os hosts da rede têm exactamente a mesma hora, e mantém a sincronização funcional mesmo se a ligação à internet falhar.

Para transformar um host em servidor NTP, acrescenta-se ao `/etc/chrony.conf`:

```bash
# Permitir que a rede local sincronize com este servidor
allow 192.168.1.0/24

# Se a ligação externa falhar, continuar a servir tempo com base no relógio local
local stratum 10
```

A directiva `local stratum 10` é importante em ambientes isolados: sem ela, se o servidor perder acesso às fontes externas deixa de responder aos clientes, e toda a rede perde sincronização. Com ela, continua a servir tempo, marcado como stratum 10 para indicar que é uma fonte de baixa qualidade mas melhor do que nada.

```bash
$ sudo firewall-cmd --add-service=ntp --permanent
$ sudo firewall-cmd --reload
$ sudo systemctl restart chronyd

# No servidor, ver os clientes que estão a sincronizar
$ sudo chronyc clients
```

---

## 4.5 Servidor Web Apache

### Fundamentos: como funciona a web

Servir um site não é conceptualmente diferente de fornecer qualquer outro serviço de rede. Um daemon escuta ligações na porta TCP 80 (ou 443 para HTTPS), aceita pedidos de documentos, e transmite-os ao browser que os pediu. Grande parte desses documentos é hoje gerada dinamicamente por aplicações e bases de dados, mas isso é incidental ao protocolo subjacente.

### URLs, URIs e URNs

A forma como os recursos são identificados na web obedece a uma taxonomia definida pela Internet Society, e vale a pena conhecer a distinção.

Um **URL** (*Uniform Resource Locator*) descreve **como localizar** um recurso, indicando o mecanismo de acesso: `http://empresa.pt/index.html`.

Um **URN** (*Uniform Resource Name*) **identifica** um recurso sem dizer onde está nem como aceder-lhe: `urn:isbn:0-13-020601-6`.

Um **URI** (*Uniform Resource Identifier*) é o termo genérico que engloba ambos.

Na prática, quase tudo com que se trabalha são URLs, compostos por cinco elementos: protocolo, hostname, porta TCP (opcional), directório (opcional) e nome do ficheiro (opcional).

Os protocolos que podem aparecer num URL não se limitam ao HTTP:

| Protocolo | O que faz | Exemplo |
|-----------|-----------|---------|
| `file` | Acede a um ficheiro local | `file:///etc/syslog.conf` |
| `ftp` | Acede a um ficheiro remoto via FTP | `ftp://ftp.empresa.pt/pacote.tar.gz` |
| `http` | Acede a um recurso remoto via HTTP | `http://empresa.pt/index.html` |
| `https` | Acede a um recurso remoto via HTTP/SSL | `https://empresa.pt/encomenda.php` |
| `ldap` | Acede a serviços de directório LDAP | `ldap://ldap.empresa.pt:389/cn=Carlos` |
| `mailto` | Envia email para um endereço | `mailto:admin@empresa.pt` |

> *(imagem: Tabela 23.1 — URL protocols)*

### O protocolo HTTP na prática

O HTTP é um protocolo cliente/servidor sem estado. O cliente pede o conteúdo de um URL específico, e o servidor responde com dados ou com uma mensagem de erro. Depois disso, a ligação pode ser reutilizada ou fechada, mas o servidor não guarda memória do pedido anterior.

Como o HTTP é um protocolo de texto simples, pode-se falar com um servidor web directamente a partir do terminal. Isto é extremamente útil para diagnóstico:

```bash
$ telnet empresa.pt 80
Trying 203.0.113.20...
Connected to empresa.pt.
Escape character is '^]'.
GET / HTTP/1.1
Host: www.empresa.pt

HTTP/1.1 200 OK
Date: Wed, 15 Oct 2025 17:43:10 GMT
Server: Apache/2.4.53 (CentOS Stream)
Last-Modified: Wed, 15 Oct 2025 16:20:22 GMT
Content-Length: 7044
Content-Type: text/html

<!DOCTYPE html>
...
```

Neste diálogo, o cliente declarou que fala HTTP versão 1.1 e indicou qual o host virtual pretendido através do cabeçalho `Host`. O servidor respondeu com um código de estado (`200 OK`), a data, a identificação do software servidor, a data de modificação do ficheiro, o tamanho e o tipo de conteúdo. Uma linha em branco separa os cabeçalhos do corpo da resposta.

O cabeçalho `Host` é obrigatório em HTTP/1.1 e é ele que torna possível o virtual hosting: sem ele, um servidor com vários sites não saberia qual deles servir.

O equivalente moderno deste diagnóstico faz-se com `curl`:

```bash
# Ver apenas os cabeçalhos de resposta
$ curl -I https://empresa.pt

# Ver todo o diálogo, incluindo negociação TLS
$ curl -v https://empresa.pt

# Testar um virtual host específico num servidor pelo IP
$ curl -H "Host: www.empresa.pt" http://203.0.113.20
```

Este último comando é particularmente útil: permite testar um site num servidor antes de o DNS apontar para lá, o que é essencial durante migrações.

### Conteúdo dinâmico

Um servidor web não serve apenas ficheiros estáticos. Pode executar programas que geram conteúdo no momento do pedido. Existem três abordagens, com implicações de desempenho muito diferentes.

**CGI** (*Common Gateway Interface*) é a abordagem original. Não é uma linguagem, mas uma especificação de como o servidor web troca informação com programas externos. Para cada pedido, o servidor arranca um processo novo que executa o script e devolve o resultado. É simples e flexível (qualquer linguagem serve), mas arrancar um processo por cada pedido é catastrófico para o desempenho num servidor com tráfego significativo.

**Interpretadores embebidos** resolvem este problema integrando o interpretador da linguagem dentro do próprio servidor web, como módulo. Quando o servidor encontra um ficheiro com uma extensão conhecida, passa o conteúdo ao interpretador interno em vez de arrancar um processo externo. O ganho de desempenho é substancial.

| Linguagem | Módulo interpretador | Referência |
|-----------|---------------------|------------|
| Perl | `mod_perl` | perl.apache.org |
| Python | `mod_python` | modpython.org |
| PHP | `mod_php` (tradicional), PHP-FPM (moderno) | php.net |
| Ruby on Rails | Phusion Passenger (`mod_rails`) | modrails.com |

> *(imagem: Tabela 23.2 — Embedded scripting modules for the Apache web server)*

**FastCGI** é a terceira via. Mantém o interpretador a correr num processo separado e persistente, evitando o custo de arranque a cada pedido mas sem o acoplar ao servidor web. Tem a vantagem de permitir reiniciar a aplicação sem reiniciar o servidor web. Em ambientes modernos com PHP, o PHP-FPM (*FastCGI Process Manager*) é a abordagem preferida.

### Servidores de aplicação

Aplicações empresariais complexas podem precisar de mais funcionalidade do que um servidor HTTP oferece. Nestes casos, usa-se um servidor de aplicação, que corre a lógica de negócio e trabalha em conjunto com o Apache, tipicamente com o Apache a fazer de proxy inverso.

| Servidor | Tipo | Site |
|----------|------|------|
| Tomcat | Open source | tomcat.apache.org |
| Jetty | Open source | eclipse.org/jetty |
| GlassFish | Open source | glassfish.dev.java.net |
| JBoss | Open source | jboss.org |
| WebSphere | Comercial | ibm.com/websphere |
| WebLogic | Comercial | oracle.com/weblogic |

> *(imagem: Tabela 23.3 — Application servers)*

### Instalação em CentOS

```bash
$ sudo dnf install httpd -y

# Activar e iniciar
$ sudo systemctl enable --now httpd

# Abrir as portas no firewall
$ sudo firewall-cmd --add-service=http --permanent
$ sudo firewall-cmd --add-service=https --permanent
$ sudo firewall-cmd --reload

# Confirmar que responde
$ curl -I http://localhost
```

Os binários do Apache ficam em `/usr/sbin` e a configuração em `/etc/httpd`. O directório raiz de documentos por defeito é `/var/www/html/`.

### Estrutura de configuração

```
/etc/httpd/
├── conf/
│   └── httpd.conf           # Configuração principal
├── conf.d/
│   ├── welcome.conf         # Página de boas-vindas
│   ├── ssl.conf             # Configuração HTTPS
│   └── empresa.conf         # Virtual hosts (criados por si)
├── conf.modules.d/
│   └── *.conf               # Carregamento de módulos
└── logs -> /var/log/httpd   # Link para os logs
```

O `httpd.conf` está organizado em três blocos conceptuais. O primeiro define parâmetros globais: porta de escuta, gestão de processos, carregamento de módulos. O segundo configura o servidor por defeito, que responde a pedidos não capturados por nenhum virtual host: utilizador e grupo sob os quais o Apache corre (nunca root), o `DocumentRoot`, e as políticas de acesso. O terceiro define os virtual hosts.

Directivas globais que importa conhecer:

```apache
# Porta de escuta
Listen 80

# Utilizador e grupo sob os quais os processos filho correm
User apache
Group apache

# Email do administrador, mostrado em páginas de erro
ServerAdmin admin@empresa.pt

# Raiz de documentos por defeito
DocumentRoot "/var/www/html"

# Ficheiros de índice por ordem de preferência
DirectoryIndex index.html index.php

# Não revelar a versão do servidor nas respostas
ServerTokens Prod
ServerSignature Off
```

### Controlo de acesso a directórios

O Apache controla o acesso ao sistema de ficheiros através de blocos `<Directory>`. A configuração por defeito nega acesso a tudo e depois autoriza explicitamente apenas o necessário, que é a abordagem correcta:

```apache
# Negar acesso a todo o sistema de ficheiros por defeito
<Directory />
    AllowOverride none
    Require all denied
</Directory>

# Autorizar apenas o directório de documentos
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

A opção `Indexes` faz o Apache gerar automaticamente uma listagem de ficheiros quando não existe ficheiro de índice num directório. Em produção, isto deve ser desactivado: expõe a estrutura de ficheiros a quem visita o site.

A opção `FollowSymLinks` permite ao Apache seguir links simbólicos. É um risco de segurança: se alguém criar um link simbólico para `/etc` dentro do directório de documentos, o conteúdo de `/etc` passa a ser servido pela web. A alternativa mais segura é `SymLinksIfOwnerMatch`, que só segue links cujo dono coincide com o dono do alvo.

```apache
<Directory "/var/www/empresa">
    Options -Indexes +SymLinksIfOwnerMatch
    AllowOverride None
    Require all granted
</Directory>
```

### Virtual Hosts

O virtual hosting permite que um único servidor responda a vários domínios com conteúdos distintos. É a funcionalidade que torna possível alojar dezenas de sites numa só máquina.

Cria-se um ficheiro por site em `/etc/httpd/conf.d/`:

```apache
# /etc/httpd/conf.d/empresa.conf

<VirtualHost *:80>
    ServerName www.empresa.pt
    ServerAlias empresa.pt

    DocumentRoot /var/www/empresa

    ErrorLog /var/log/httpd/empresa-error.log
    CustomLog /var/log/httpd/empresa-access.log combined

    <Directory /var/www/empresa>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Preparar a estrutura de ficheiros:

```bash
$ sudo mkdir -p /var/www/empresa
$ echo "<h1>empresa.pt</h1>" | sudo tee /var/www/empresa/index.html
$ sudo chown -R apache:apache /var/www/empresa
$ sudo restorecon -Rv /var/www/empresa   # Corrigir contexto SELinux
```

O comando `restorecon` é essencial em CentOS: o SELinux exige que os ficheiros servidos pelo Apache tenham o contexto de segurança correcto. Sem isto, o Apache recebe "permission denied" mesmo com permissões de ficheiro correctas, o que é uma das fontes de confusão mais comuns em CentOS.

Verificar e aplicar:

```bash
# Verificar sintaxe antes de recarregar
$ sudo apachectl configtest
Syntax OK

# Recarregar sem interromper ligações activas
$ sudo systemctl reload httpd

# Listar os virtual hosts configurados
$ sudo apachectl -S
```

### Logs do Apache

O Apache mantém dois logs por site. O **access log** regista todos os pedidos recebidos. O **error log** regista erros de processamento e mensagens de diagnóstico.

```bash
# Acompanhar acessos em tempo real
$ sudo tail -f /var/log/httpd/access_log

# Acompanhar erros
$ sudo tail -f /var/log/httpd/error_log

# Contar pedidos por código de resposta HTTP
$ sudo awk '{print $9}' /var/log/httpd/access_log | sort | uniq -c | sort -rn

# Top 10 endereços IP com mais pedidos
$ sudo awk '{print $1}' /var/log/httpd/access_log | sort | uniq -c | sort -rn | head

# Ver apenas os pedidos que resultaram em erro 404
$ sudo awk '$9 == 404 {print $7}' /var/log/httpd/access_log | sort | uniq -c | sort -rn
```

Estes pipelines são construídos com as mesmas ferramentas do Capítulo 4, e mostram como o conhecimento de `awk`, `sort` e `uniq` se traduz directamente em capacidade de análise operacional.

### Comandos essenciais

```bash
# Verificar configuração
$ sudo apachectl configtest

# Listar módulos carregados
$ sudo apachectl -M

# Listar virtual hosts e a sua ordem de precedência
$ sudo apachectl -S

# Recarregar configuração sem cortar ligações
$ sudo systemctl reload httpd

# Reiniciar completamente
$ sudo systemctl restart httpd

# Ver processos do Apache
$ ps aux | grep httpd
```

### Escalabilidade e balanceamento de carga

Prever quantos pedidos um servidor suporta é difícil: depende do hardware, do sistema operativo, e sobretudo da natureza do site. Um servidor que serve apenas HTML estático tem um perfil de carga completamente diferente de um que faz consultas a bases de dados a cada pedido. Só a medição directa do site real no hardware real responde a essa pergunta.

Por isso, em vez de optimizar para um único servidor, a estratégia correcta é planear para escalabilidade horizontal. Existem três abordagens comuns:

**DNS round robin** é a mais simples. Associam-se vários endereços IP ao mesmo nome, e o servidor DNS devolve-os alternadamente. É primitivo mas funciona, e é usado até por grandes operadores. A limitação é não haver verificação de saúde: se um servidor falhar, o DNS continua a enviar clientes para lá.

**Balanceadores de hardware** são equipamentos dedicados que distribuem tráfego e verificam a saúde dos servidores. Rápidos e fiáveis, mas caros.

**Balanceadores de software** como o HAProxy ou o próprio Apache com `mod_proxy_balancer` oferecem a maior flexibilidade a custo zero, e são a escolha mais comum em infraestruturas modernas.

---

## 4.6 Email: Arquitectura e Configuração

### O sistema de email por dentro

Do ponto de vista do utilizador, o email é trivial: escreve-se uma mensagem, carrega-se em enviar, e segundos depois ela está na caixa de correio do destinatário do outro lado do mundo. A infraestrutura que torna isto possível é bastante mais complexa, e compreendê-la é o que permite a um administrador diagnosticar problemas em vez de encolher os ombros.

Um sistema de email é composto por cinco componentes com funções distintas:

O **MUA** (*Mail User Agent*) é o cliente que o utilizador usa para ler e escrever mensagens: Thunderbird, Outlook, ou uma interface web como o Gmail. O comando `mail` no terminal também é um MUA, e continua a ser útil em scripts.

O **MSA** (*Mail Submission Agent*) é o ponto de entrada das mensagens no sistema. Recebe o email do MUA, valida-o, faz limpeza dos cabeçalhos e injecta-o no sistema de transporte. Escuta tipicamente na porta 587 e exige autenticação, ao contrário do MTA que escuta na 25. Esta separação existe por causa do spam: permite distinguir claramente mensagens submetidas por utilizadores autenticados de mensagens que chegam de outros servidores.

O **MTA** (*Mail Transfer Agent*) é o motor do sistema. Transporta mensagens entre servidores. Quando alguém envia email para `destino@empresa.pt`, o MTA do remetente consulta o registo MX de `empresa.pt` no DNS, descobre qual o servidor responsável, liga-se a ele por SMTP e entrega a mensagem.

O **DA** (*Delivery Agent*) recebe a mensagem do MTA local e deposita-a na caixa de correio do utilizador no servidor.

O **AA** (*Access Agent*) permite ao MUA aceder às mensagens armazenadas no servidor, através dos protocolos IMAP ou POP3.

### IMAP e POP3

Os dois protocolos de acesso resolvem o mesmo problema de forma diferente.

O **POP3** assume um modelo em que todo o email é descarregado do servidor para o cliente. As mensagens podem ser apagadas do servidor após o download (e nesse caso deixam de estar nos backups) ou mantidas (e nesse caso o espaço no servidor cresce indefinidamente). É ineficiente em rede e inflexível para quem usa vários dispositivos.

O **IMAP** mantém as mensagens no servidor e sincroniza-as com o cliente. Descarrega uma mensagem de cada vez, permite navegar nos cabeçalhos sem descarregar os anexos, e gere pastas em vários dispositivos em simultâneo. Para praticamente todos os cenários modernos, é a escolha correcta.

Em ambos os casos, deve usar-se sempre a variante cifrada: **IMAPS** (porta 993) ou **POP3S** (porta 995). As versões não cifradas transmitem credenciais em texto plano.

### Anatomia de uma mensagem

Uma mensagem de email tem três partes distintas, e saber distingui-las é fundamental para diagnóstico.

O **envelope** determina para onde a mensagem é entregue e a quem deve ser devolvida se falhar. É invisível ao utilizador e não faz parte da mensagem propriamente dita: é usado internamente pelos MTAs. Normalmente coincide com os campos From e To dos cabeçalhos, mas nem sempre. Mensagens enviadas para listas de distribuição e mensagens forjadas por spammers têm frequentemente envelope e cabeçalhos divergentes.

Os **cabeçalhos** são pares propriedade/valor que registam informação sobre a mensagem: quando foi enviada, por que servidores passou, de quem é e para quem vai. Fazem parte da mensagem. Os clientes de email escondem a maioria deles por serem pouco interessantes para o utilizador comum, mas são a fonte primária de informação para diagnóstico.

O **corpo** é o conteúdo. Normalmente texto simples, ainda que esse texto represente frequentemente uma codificação de conteúdo binário através do standard MIME.

### Ler os cabeçalhos de uma mensagem

Dissecar cabeçalhos de email é uma competência essencial de administração. Quando um utilizador reporta que uma mensagem não chegou, ou que uma mensagem suspeita chegou, os cabeçalhos contêm a resposta.

Um exemplo simplificado:

```
Delivered-To: carlos@empresa.pt
Received: by mx.empresa.pt with SMTP id abc123;
        Wed, 15 Oct 2025 08:14:27 +0100
Received: from mail-relay.parceiro.com (mail-relay.parceiro.com [203.0.113.5])
        by mx.empresa.pt with ESMTP id def456;
        Wed, 15 Oct 2025 08:14:25 +0100
Received-SPF: pass (empresa.pt: domain of ana@parceiro.com designates
        203.0.113.5 as permitted sender)
Received: from workstation.parceiro.com (localhost [127.0.0.1])
        by mail-relay.parceiro.com (Postfix) with ESMTP id 789ghi;
        Wed, 15 Oct 2025 08:14:12 +0100
X-Virus-Scanned: amavisd-new at parceiro.com
Date: Wed, 15 Oct 2025 08:14:06 +0100
From: Ana Costa <ana@parceiro.com>
To: carlos@empresa.pt
Subject: Proposta de contrato
```

A leitura faz-se **de baixo para cima**, porque cada servidor que processa a mensagem acrescenta a sua linha `Received` no topo. Assim, a linha `Received` mais em baixo é a primeira do percurso, e a mais em cima é a última.

Neste exemplo, a mensagem saiu da workstation da Ana, passou pelo relay do domínio `parceiro.com` onde foi verificada por antivírus, e foi entregue ao servidor de `empresa.pt`. Os timestamps permitem calcular quanto tempo demorou cada salto.

A linha `Received-SPF: pass` indica que o servidor de origem estava autorizado a enviar email em nome do domínio `parceiro.com`. Um `fail` aqui é indicador de possível forja, embora também aconteça legitimamente em mensagens reencaminhadas.

> ⚠️ **Os cabeçalhos podem ser forjados.** Qualquer servidor no percurso pode inserir linhas `Received` falsas. Só as linhas acrescentadas pelos servidores sob o seu controlo são de confiança. Use esta informação com cautela ao investigar mensagens suspeitas.

### O protocolo SMTP

O SMTP (*Simple Mail Transfer Protocol*) é o protocolo usado em quase todas as transferências dentro do sistema de email: do MUA para o MSA, do MSA para o MTA, entre MTAs, e do MTA para o agente de entrega.

Porque tanto o formato das mensagens como o protocolo estão normalizados, servidores de fabricantes diferentes interoperam sem problema. O MTA de um lado não precisa de saber que software corre do outro lado, apenas que ambos falam SMTP.

O protocolo tem poucos comandos:

| Comando | Função |
|---------|--------|
| `HELO hostname` | Identifica o host que se liga (SMTP clássico) |
| `EHLO hostname` | Identifica o host que se liga (ESMTP, versão estendida) |
| `MAIL FROM:` | Inicia uma transacção, define o remetente do envelope |
| `RCPT TO:` | Define o destinatário do envelope |
| `DATA` | Inicia o corpo da mensagem |
| `QUIT` | Termina a sessão e fecha a ligação |
| `RSET` | Reinicia o estado da ligação |
| `VRFY` | Verifica se um endereço é válido |
| `HELP` | Mostra um resumo dos comandos suportados |

Praticamente tudo hoje usa ESMTP, a versão estendida. Uma sessão começa com `EHLO`; se o servidor do outro lado não perceber, o cliente recua para `HELO` e SMTP clássico.

Como o SMTP é um protocolo de texto, pode-se falar com um servidor de email directamente, o que é a forma mais directa de diagnosticar problemas de entrega:

```bash
$ telnet mail.empresa.pt 25
Trying 203.0.113.30...
Connected to mail.empresa.pt.
220 mail.empresa.pt ESMTP Postfix

EHLO teste.local
250-mail.empresa.pt
250-PIPELINING
250-SIZE 10240000
250-STARTTLS
250 8BITMIME

MAIL FROM:<admin@empresa.pt>
250 2.1.0 Ok

RCPT TO:<carlos@empresa.pt>
250 2.1.5 Ok

DATA
354 End data with <CR><LF>.<CR><LF>
Subject: Teste de entrega

Mensagem de teste enviada directamente por SMTP.
.
250 2.0.0 Ok: queued as 4A2B3C

QUIT
221 2.0.0 Bye
```

Os códigos numéricos das respostas seguem uma lógica simples. O primeiro dígito indica o tipo: **2xx** significa sucesso, **4xx** significa erro temporário (o remetente deve tentar mais tarde), e **5xx** significa erro permanente (a mensagem não será entregue). Um `450` significa "tente novamente"; um `550` significa "desista".

### Postfix: o MTA recomendado

O Postfix é o MTA padrão na maioria das distribuições Linux modernas. Foi criado por Wietse Venema com a segurança como prioridade absoluta, e o resultado fala por si: nunca foi encontrada uma vulnerabilidade explorável remotamente em nenhuma versão.

A arquitectura explica em parte esse historial. Ao contrário do sendmail, que é essencialmente um programa grande que faz tudo, o Postfix é composto por vários programas pequenos e cooperantes, cada um com uma função específica e privilégios mínimos. Comunicam entre si por sockets locais. A maioria pode correr em ambiente chroot, e nenhum é setuid.

O Postfix é também compatível com o sendmail ao nível da interface: o comando `/usr/sbin/sendmail` existe como wrapper, e os ficheiros de aliases e `.forward` têm o mesmo formato. Scripts existentes continuam a funcionar sem alteração.

### Os componentes do Postfix

Vale a pena conhecer os programas principais, porque os nomes aparecem nos logs e saber o que cada um faz acelera muito o diagnóstico.

Do lado da **recepção**: o `smtpd` recebe mensagens que chegam por SMTP e verifica se o cliente está autorizado a enviá-las. O `pickup` processa mensagens submetidas localmente através do comando `sendmail`. O `cleanup` acrescenta cabeçalhos em falta e reescreve endereços antes de a mensagem entrar na fila.

Do lado do **envio**: o `qmgr` (queue manager) decide para onde cada mensagem deve seguir e gere as filas. O `smtp` entrega mensagens a servidores remotos. O `local` entrega mensagens a caixas de correio locais, resolvendo aliases e ficheiros `.forward`. O `virtual` entrega a caixas de correio que não correspondem a contas locais do sistema.

### As filas do Postfix

O `qmgr` gere cinco filas, e saber o que significa cada uma é essencial para diagnosticar problemas de entrega:

| Fila | Conteúdo |
|------|----------|
| `incoming` | Mensagens que acabaram de chegar |
| `active` | Mensagens em processo de entrega |
| `deferred` | Mensagens cuja entrega falhou e serão tentadas de novo |
| `hold` | Mensagens retidas manualmente pelo administrador |
| `corrupt` | Mensagens que não podem ser lidas ou processadas |

A fila `deferred` é a que merece atenção regular. Quando uma entrega falha por erro temporário, a mensagem vai para lá e o Postfix tenta de novo com intervalos crescentes. Uma fila `deferred` que cresce constantemente é sinal de problema: pode ser um servidor de destino inacessível, um problema de DNS, ou o servidor ter sido colocado numa blacklist.

```bash
# Ver a fila de mensagens
$ mailq

# Equivalente
$ postqueue -p

# Ver detalhe de uma mensagem específica na fila
$ postcat -q 4A2B3C

# Forçar nova tentativa de entrega de toda a fila
$ sudo postqueue -f

# Apagar uma mensagem específica da fila
$ sudo postsuper -d 4A2B3C

# Apagar todas as mensagens deferred
$ sudo postsuper -d ALL deferred
```

### Instalação e configuração

```bash
$ sudo dnf install postfix -y
$ sudo systemctl enable --now postfix
```

O ficheiro de configuração principal é `/etc/postfix/main.cf`. Tem mais de 500 parâmetros possíveis, mas um servidor típico só precisa de definir uma dúzia. A recomendação do autor do Postfix é explícita: inclua apenas os parâmetros cujo valor difere do valor por defeito. Assim, se o valor por defeito mudar numa versão futura, a configuração acompanha automaticamente.

O ficheiro `master.cf` configura quais os programas do Postfix que correm e como. Na maioria dos casos não precisa de ser alterado.

### Configuração de null client

O cenário mais comum num servidor de aplicação não é receber email, é enviá-lo. Alertas de monitorização, relatórios de cron, notificações de aplicações. Para isto configura-se um **null client**: um servidor que não recebe email do exterior, mas encaminha todo o email que gera para um servidor de email central.

```bash
# /etc/postfix/main.cf (null client)

# Identidade do servidor
myhostname = servidor-app.empresa.pt
mydomain = empresa.pt

# Domínio acrescentado a endereços sem domínio
myorigin = $mydomain

# Nenhum domínio é local: não receber email para entrega local
mydestination =

# Encaminhar todo o email para o servidor central
# Os parênteses rectos evitam consulta MX e forçam consulta A
relayhost = [mail.empresa.pt]

# Escutar apenas em loopback: não aceitar ligações do exterior
inet_interfaces = loopback-only

# Não anunciar a versão do Postfix
smtpd_banner = $myhostname ESMTP
```

Os parênteses rectos em `relayhost = [mail.empresa.pt]` são significativos: instruem o Postfix a tratar o valor como um hostname (consulta de registo A) em vez de um domínio de email (consulta de registo MX). Sem eles, o Postfix procuraria o registo MX de `mail.empresa.pt`, que provavelmente não existe.

Aplicar e testar:

```bash
# Verificar a configuração
$ sudo postfix check

# Recarregar
$ sudo systemctl reload postfix

# Enviar um email de teste
$ echo "Corpo da mensagem de teste" | mail -s "Teste de envio" admin@empresa.pt

# Confirmar no log que saiu
$ sudo tail -f /var/log/maillog
```

### A ferramenta postconf

O `postconf` é a ferramenta central de gestão da configuração do Postfix:

```bash
# Ver todos os parâmetros com os valores actuais
$ postconf

# Ver um parâmetro específico
$ postconf mydestination

# Ver o valor por defeito de um parâmetro
$ postconf -d mydestination

# Ver apenas os parâmetros que diferem do valor por defeito
$ postconf -n

# Definir um parâmetro directamente
$ sudo postconf -e "relayhost = [mail.empresa.pt]"
```

O comando `postconf -n` é particularmente útil: mostra apenas o que foi efectivamente configurado, filtrando as centenas de parâmetros que estão nos valores por omissão. É a primeira coisa a executar quando se herda um servidor de email desconhecido.

### Aliases

O ficheiro `/etc/aliases` define redireccionamentos locais de email:

```
# /etc/aliases
postmaster:     root
root:           admin@empresa.pt
webmaster:      carlos, ana
alertas:        equipa-sistemas@empresa.pt
```

Este mecanismo é importante em servidores: o email dirigido a `root` (que inclui relatórios do cron e alertas do sistema) deve ser redireccionado para um endereço real que alguém lê.

Após editar o ficheiro, é necessário compilá-lo para o formato binário que o Postfix usa:

```bash
$ sudo newaliases
```

### Combate ao spam

Um servidor de email exposto à internet recebe volumes significativos de spam. As técnicas de defesa mais usadas são estas.

**Greylisting** rejeita temporariamente a primeira tentativa de entrega de qualquer remetente desconhecido, com um código 4xx que significa "tente mais tarde". Servidores legítimos voltam a tentar passados alguns minutos e a mensagem passa. A maioria dos sistemas de spam não tenta de novo, e a mensagem nunca chega. A eficácia desta técnica diminuiu à medida que os spammers melhoraram as suas implementações, mas continua útil em combinação com outras.

**Blacklists** (também chamadas RBLs, *Realtime Black Lists*) são listas de endereços IP conhecidos por enviar spam, publicadas através do DNS. O MTA consulta a lista antes de aceitar uma mensagem e rejeita as que vêm de endereços listados. O `spamhaus.org` é a mais conhecida.

**Whitelists** funcionam ao contrário: listas de domínios com boa reputação, usadas para reduzir falsos positivos, saltando algumas verificações para remetentes conhecidos como fiáveis.

**SpamAssassin** analisa o conteúdo da mensagem aplicando centenas de regras heurísticas, atribuindo pontos a cada característica suspeita. Mensagens acima de um limiar de pontuação são marcadas como spam.

**SPF** (*Sender Policy Framework*) é um registo TXT no DNS que declara quais os servidores autorizados a enviar email em nome de um domínio. O servidor receptor consulta este registo e pode rejeitar mensagens de servidores não autorizados. Um registo SPF típico:

```
empresa.pt.  IN  TXT  "v=spf1 mx a:relay.empresa.pt -all"
```

Isto declara que apenas os servidores MX do domínio e o `relay.empresa.pt` estão autorizados, e que tudo o resto deve ser rejeitado (`-all`).

**DKIM** (*DomainKeys Identified Mail*) assina criptograficamente as mensagens enviadas, permitindo ao receptor verificar que a mensagem não foi alterada em trânsito e que veio realmente do domínio que alega.

Configurar SPF e DKIM correctamente deixou de ser opcional: sem eles, os grandes fornecedores de email colocam as mensagens do seu domínio na pasta de spam ou rejeitam-nas por completo.

---

## 4.7 Centralização de Logs com rsyslog

### Por que centralizar

Gerir logs localmente em cada servidor cria três problemas sérios.

O primeiro é de segurança. Quando um servidor é comprometido, a primeira coisa que um atacante competente faz é apagar ou adulterar os logs locais para esconder o rasto. Logs enviados em tempo real para um servidor separado sobrevivem a essa limpeza.

O segundo é de eficiência operacional. Investigar um incidente que envolve vários servidores significa aceder a cada um deles individualmente, correlacionando timestamps à mão. Com logs centralizados, uma única pesquisa cobre toda a infraestrutura.

O terceiro é de retenção. Servidores individuais têm espaço em disco limitado e rodam os logs agressivamente. Um servidor de logs dedicado pode reter meses de histórico.

O `rsyslog` suporta envio de logs para servidores remotos nativamente, o que faz dele a base das arquitecturas de logging centralizado em Linux.

### Configurar o cliente

Em cada servidor que deve enviar logs, edita-se `/etc/rsyslog.conf` ou cria-se um ficheiro em `/etc/rsyslog.d/`:

```bash
# /etc/rsyslog.d/10-central.conf

# Enviar todos os logs para o servidor central via TCP
*.* @@192.168.1.200:514

# Alternativa: apenas logs de nível warning ou mais urgente
# *.warning @@192.168.1.200:514

# Enviar logs de autenticação para um servidor de segurança dedicado
# authpriv.* @@192.168.1.201:514
```

O duplo `@@` indica transporte TCP, que garante entrega. Um único `@` usaria UDP, mais rápido mas sem garantias: em caso de congestão de rede, mensagens perdem-se silenciosamente. Para logs de segurança, use sempre TCP.

### Configurar o servidor central

No servidor de logs, activa-se a recepção e define-se como organizar o que chega:

```bash
# /etc/rsyslog.conf (servidor central)

# Activar recepção via TCP na porta 514
module(load="imtcp")
input(type="imtcp" port="514")

# Template: organizar por hostname e por programa
template(name="LogsRemotos" type="string"
         string="/var/log/hosts/%HOSTNAME%/%PROGRAMNAME%.log")

# Aplicar o template a tudo o que vier de fora
if $fromhost-ip != '127.0.0.1' then {
    action(type="omfile" dynaFile="LogsRemotos")
    stop
}
```

Esta configuração produz uma estrutura de directórios organizada e navegável:

```
/var/log/hosts/
├── servidor-web-01/
│   ├── httpd.log
│   ├── sshd.log
│   └── CROND.log
├── servidor-db-01/
│   ├── mysqld.log
│   └── sshd.log
└── servidor-app-01/
    ├── sshd.log
    └── postfix.log
```

Abrir a porta e reiniciar:

```bash
$ sudo firewall-cmd --add-port=514/tcp --permanent
$ sudo firewall-cmd --reload
$ sudo systemctl restart rsyslog
```

### Verificar que funciona

```bash
# No cliente, gerar uma mensagem de teste
$ logger -t teste-central "Mensagem de verificação do log centralizado"

# No servidor central, confirmar que chegou
$ sudo grep "verificação" /var/log/hosts/*/teste-central.log

# Acompanhar em tempo real tudo o que chega
$ sudo tail -f /var/log/hosts/*/*.log
```

### Rotação dos logs centralizados

Um servidor central acumula volumes de dados consideráveis. É indispensável configurar a rotação:

```
# /etc/logrotate.d/logs-remotos
/var/log/hosts/*/*.log {
    daily
    rotate 90
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root root
    sharedscripts
    postrotate
        /usr/bin/systemctl reload rsyslog > /dev/null 2>&1 || true
    endscript
}
```

Esta configuração mantém 90 dias de histórico comprimido, o que é um ponto de partida razoável para a maioria das organizações. Os requisitos concretos dependem de políticas internas e de eventuais obrigações legais de retenção.

>  **rsyslog e journald em conjunto.** Em CentOS Stream, o `journald` é o ponto central de recolha local, e o `rsyslog` pode ser configurado para ler directamente do journal através do módulo `imjournal` e reencaminhar para o servidor central. Esta combinação aproveita a recolha estruturada do journald com a capacidade de transporte remoto do rsyslog, e é a arquitectura recomendada em sistemas RHEL modernos.
