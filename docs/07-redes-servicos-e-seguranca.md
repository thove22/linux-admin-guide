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

## 2.2 Telnet: o Predecessor que não Deve ser Usado

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

## 2.3 wget e curl: Transferência e Automação HTTP

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

## 2.4 FTP: Transferência Clássica e as suas Limitações

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

### sftp: FTP seguro sobre SSH

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

### rsync: sincronização eficiente entre sistemas

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

:
