# Gestão de Armazenamento e Arranque do Sistema

## 1. O Arranque do Sistema

### 1.1 Da alimentação ao prompt
Ligar um computador é um problema circular. Para carregar o sistema operativo do disco é preciso saber ler o disco, mas o código que sabe ler discos faz parte do sistema operativo que ainda não foi carregado. O sistema tem de se erguer pelos seus próprios atacadores, e é daí que vem o termo **bootstrapping**, hoje abreviado para *boot*.

A solução é uma cadeia de entregas sucessivas. Cada etapa tem apenas o conhecimento suficiente para localizar, carregar e ceder controlo à etapa seguinte, que é mais capaz do que ela. O firmware sabe ler alguns sectores do disco. Esses sectores contêm um gestor de arranque que sabe ler sistemas de ficheiros. O gestor de arranque carrega o kernel. O kernel monta o sistema de ficheiros raiz e arranca o primeiro processo de espaço de utilizador. Esse processo arranca tudo o resto.

O arranque é um período de vulnerabilidade especial. Erros de configuração, hardware em falta ou pouco fiável, e sistemas de ficheiros danificados podem todos impedir uma máquina de chegar ao fim do processo. É também uma das áreas mais delicadas de administrar, porque exige familiaridade com muitos outros aspectos do sistema em simultâneo.

Um arranque típico de um sistema Linux moderno atravessa seis fases:

1. **Firmware.** O BIOS ou UEFI inicializa o hardware essencial e localiza um dispositivo de arranque.
2. **Gestor de arranque.** O GRUB2 é carregado, apresenta o menu e carrega o kernel escolhido.
3. **Kernel.** O kernel descomprime-se, inicializa-se, detecta hardware e monta o sistema de ficheiros temporário initramfs.
4. **initramfs.** Carrega os controladores necessários para aceder ao sistema de ficheiros raiz real e monta-o.
5. **systemd.** O primeiro processo de espaço de utilizador arranca, activa unidades e serviços por ordem de dependência.
6. **Sistema operacional.** Serviços disponíveis, prompts de login activos.

O administrador tem pouco controlo interactivo sobre a maior parte destas fases. O que faz é alterar ficheiros de configuração que determinam o comportamento de cada uma, ou modificar os argumentos que uma fase passa à seguinte.

---

### 1.2 Firmware: BIOS e UEFI

Quando uma máquina é ligada, a primeira coisa que executa é código armazenado em memória não volátil na própria placa. Este código é o **firmware**, e tem duas gerações em uso.

#### BIOS

O **BIOS** (*Basic Input/Output System*) é a geração antiga, e é extremamente simples comparado com o firmware de estações de trabalho proprietárias. Inicializa o hardware básico, executa um teste de arranque, e depois procura um dispositivo de onde arrancar, seguindo uma ordem de preferência configurável.

Encontrado o dispositivo, o BIOS lê os primeiros 512 bytes, um segmento conhecido como **MBR** (*Master Boot Record*). Estes 512 bytes contêm três coisas: um pequeno programa de arranque com cerca de 446 bytes, a tabela de partições com 64 bytes, e uma assinatura de 2 bytes.

O programa dentro do MBR é minúsculo, e 446 bytes não chegam para nada de sofisticado. Tudo o que faz é localizar e carregar um programa maior, o gestor de arranque propriamente dito, que está noutro sítio do disco.

Quando se usa a tabela de partições moderna GPT num sistema com BIOS clássico, não existe o espaço tradicional a seguir ao MBR onde o segundo estágio do GRUB costumava caber. A solução é reservar uma pequena partição dedicada, a **partição biosboot** de 1 MiB, que serve exclusivamente para alojar esse código. Não tem sistema de ficheiros e nunca é montada. É por isso que ela aparecia na tabela de partições proposta no Capítulo 2 sem qualquer ponto de montagem associado.

#### UEFI

O **UEFI** (*Unified Extensible Firmware Interface*) substitui o BIOS em praticamente todo o hardware fabricado na última década, e resolve várias limitações da abordagem antiga.

O UEFI sabe ler sistemas de ficheiros, concretamente o FAT32. Isto significa que não precisa de um programa escondido em sectores brutos: os gestores de arranque são ficheiros normais, com nome e caminho, dentro de uma partição dedicada chamada **ESP** (*EFI System Partition*), montada tipicamente em `/boot/efi`.

O UEFI mantém também uma lista de entradas de arranque em memória não volátil da própria placa, que pode ser consultada e manipulada a partir do sistema operativo em funcionamento.

```bash
# Determinar se o sistema arrancou em modo UEFI ou BIOS
$ [ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"

# Listar as entradas de arranque registadas no firmware (apenas UEFI)
$ sudo efibootmgr -v
BootCurrent: 0001
Timeout: 1 seconds
BootOrder: 0001,0000
Boot0000* UEFI: Built-in EFI Shell
Boot0001* CentOS  HD(1,GPT,...)/File(\EFI\centos\shimx64.efi)

# Ver o conteúdo da partição ESP
$ ls /boot/efi/EFI/
BOOT  centos
```

O **Secure Boot** é uma funcionalidade do UEFI que verifica a assinatura criptográfica de cada componente antes de o executar, impedindo o carregamento de gestores de arranque ou kernels não assinados. É uma protecção real contra certas classes de ataque, mas também uma complicação quando é necessário carregar módulos de kernel compilados localmente, que precisam de ser assinados para funcionar.

---

### 1.3 GRUB2: o gestor de arranque

O **GRUB** (*GRand Unified Bootloader*) é o gestor de arranque padrão em praticamente todas as distribuições Linux. A sua função é escolher um kernel de uma lista previamente construída e carregá-lo com as opções que o administrador especificou.

A versão em uso hoje é o **GRUB2**, uma reescrita completa da versão original, agora chamada GRUB Legacy. A diferença mais visível para o administrador é a forma de configuração: no GRUB Legacy editava-se directamente o ficheiro de menu; no GRUB2 esse ficheiro é gerado automaticamente e nunca deve ser editado à mão.

#### Os ficheiros de configuração

Esta é a fonte de confusão mais comum com o GRUB2, e vale a pena ser explícito sobre a hierarquia:

| Ficheiro | Função |
|----------|--------|
| `/etc/default/grub` | Configurações principais. **É aqui que se editam as alterações.** |
| `/etc/grub.d/` | Scripts que geram as entradas do menu. Personalização avançada. |
| `/boot/grub2/grub.cfg` | Ficheiro gerado. **Nunca deve ser editado directamente.** |
| `/boot/efi/EFI/centos/grub.cfg` | O mesmo, em sistemas UEFI |

O `/etc/default/grub` de um CentOS típico:

```bash
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=cs/root rd.lvm.lv=cs/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
```

Os parâmetros mais relevantes:

`GRUB_TIMEOUT` define quantos segundos o menu fica visível antes de arrancar a entrada por defeito. O valor `0` arranca imediatamente sem mostrar o menu, e `-1` espera indefinidamente por input.

`GRUB_DEFAULT` define qual a entrada seleccionada por defeito. O valor `saved` significa "a última que arrancou com sucesso".

`GRUB_CMDLINE_LINUX` contém os parâmetros passados ao kernel em todas as entradas. É aqui que se acrescentam ou removem opções de arranque permanentes.

Após qualquer alteração, é obrigatório regenerar o ficheiro de menu:

```bash
# Sistemas BIOS
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Sistemas UEFI
$ sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```

#### O menu de arranque

Quando o GRUB apresenta o menu, existem três acções possíveis além de esperar:

Premir `e` sobre uma entrada abre um editor que permite alterar os parâmetros daquele arranque específico. As alterações **não são persistentes**: aplicam-se apenas a este arranque. Esta é a forma correcta de testar um parâmetro antes de o tornar permanente.

Premir `c` abre a linha de comandos do GRUB, um ambiente interactivo com completação de comandos e navegação de cursor. É a partir daqui que se pode arrancar um sistema cuja configuração está partida.

Premir `Esc` regressa ao menu a partir de qualquer um destes modos.

Os comandos disponíveis na linha de comandos do GRUB incluem:

| Comando | Significado |
|---------|-------------|
| `ls` | Lista dispositivos e partições reconhecidos |
| `set` | Define ou mostra variáveis do GRUB |
| `linux` | Carrega um kernel a partir do dispositivo raiz |
| `initrd` | Carrega a imagem initramfs correspondente |
| `boot` | Arranca o sistema com o kernel carregado |
| `search` | Procura ficheiros em todas as partições montáveis |
| `help` | Ajuda interactiva sobre um comando |
| `reboot` | Reinicia o sistema |

Um arranque manual completo a partir da linha de comandos do GRUB, útil quando o `grub.cfg` está corrompido:

```
grub> ls
(hd0) (hd0,gpt1) (hd0,gpt2) (hd0,gpt3)

grub> ls (hd0,gpt2)/
vmlinuz-5.14.0-162.el9.x86_64  initramfs-5.14.0-162.el9.x86_64.img

grub> set root=(hd0,gpt2)
grub> linux /vmlinuz-5.14.0-162.el9.x86_64 root=/dev/mapper/cs-root
grub> initrd /initramfs-5.14.0-162.el9.x86_64.img
grub> boot
```

#### Parâmetros de kernel

Os parâmetros passados ao kernel modificam o seu comportamento durante o arranque. Alguns dos mais úteis em administração:

| Parâmetro | Efeito |
|-----------|--------|
| `single` ou `s` | Arranca em modo de utilizador único |
| `systemd.unit=rescue.target` | Arranca no target de recuperação |
| `systemd.unit=emergency.target` | Arranca no target de emergência (mínimo) |
| `rd.break` | Interrompe o arranque dentro do initramfs |
| `init=/bin/bash` | Substitui o systemd por uma shell directa |
| `quiet` | Suprime as mensagens de arranque do kernel |
| `nomodeset` | Desactiva a definição de modo gráfico do kernel |
| `selinux=0` | Desactiva o SELinux para este arranque |

A ferramenta `grubby` permite manipular parâmetros de kernel sem editar ficheiros e sem regenerar a configuração manualmente:

```bash
# Ver a configuração do kernel actual
$ sudo grubby --info=DEFAULT

# Acrescentar um parâmetro a todos os kernels
$ sudo grubby --update-kernel=ALL --args="audit=1"

# Remover um parâmetro
$ sudo grubby --update-kernel=ALL --remove-args="quiet"

# Definir qual o kernel por defeito
$ sudo grubby --set-default /boot/vmlinuz-5.14.0-162.el9.x86_64
```

#### Múltiplos kernels

Ao contrário de outros pacotes de software, um kernel novo não substitui o anterior. É instalado ao lado dele, e ambos ficam disponíveis no menu do GRUB. Esta convenção existe por uma razão concreta: se uma actualização de kernel partir o sistema, é possível reiniciar e escolher a versão anterior sem recorrer a suportes de recuperação.

Com o tempo, o menu acumula várias versões. O sistema mantém por defeito as três ou quatro mais recentes e remove as mais antigas automaticamente.

```bash
# Listar os kernels instalados
$ rpm -q kernel

# Ver quantas versões o sistema mantém
$ grep installonly_limit /etc/dnf/dnf.conf
installonly_limit=3

# Ver o kernel em execução
$ uname -r
```

#### Proteger o GRUB com password

Sem protecção, qualquer pessoa com acesso físico à consola pode editar os parâmetros de arranque e obter uma shell de root sem qualquer autenticação. Definir uma password no GRUB fecha esta porta.

```bash
# Gerar o hash da password
$ grub2-mkpasswd-pbkdf2
```

Acrescenta-se depois a `/etc/grub.d/40_custom`:

```
set superusers="admin"
password_pbkdf2 admin grub.pbkdf2.sha512.10000.xxxxx...
```

E regenera-se a configuração. Este é um exemplo directo do compromisso discutido no Capítulo 7: a protecção é real, mas significa que a máquina deixa de arrancar sozinha se precisar de intervenção no menu, o que é problemático em servidores sem acesso físico fácil.

---

### 1.4 Kernel e initramfs

#### O que o kernel faz ao arrancar

Carregado pelo GRUB, o kernel começa por se descomprimir e inicializar as suas estruturas internas. Sonda o sistema para determinar quanta memória RAM está disponível, e reserva para si a porção de que precisa. Essa memória fica indisponível para processos de utilizador.

Segue-se a inventariação do hardware. O kernel percorre os barramentos do sistema, identifica os dispositivos presentes, e carrega os controladores correspondentes, normalmente como módulos independentes. É esta fase que produz a torrente de mensagens crípticas visível na consola durante o arranque, e que pode ser consultada depois:

```bash
$ dmesg | less
$ journalctl -k
```

#### Processos de kernel

Terminada a inicialização básica, o kernel cria um conjunto de processos que aparecem nas listagens do `ps` mas que não são processos normais. Não foram criados pelo mecanismo habitual de `fork`, e por isso são chamados processos espontâneos. Identificam-se pelos parênteses rectos à volta do nome:

```bash
$ ps aux | head
USER   PID %CPU %MEM  VSZ  RSS TTY STAT START TIME COMMAND
root     1  0.0  0.4 1234 5678 ?   Ss   09:15 0:02 /usr/lib/systemd/systemd
root     2  0.0  0.0    0    0 ?   S    09:15 0:00 [kthreadd]
root     3  0.0  0.0    0    0 ?   I<   09:15 0:00 [rcu_gp]
root    12  0.0  0.0    0    0 ?   S    09:15 0:00 [ksoftirqd/0]
root    98  0.0  0.0    0    0 ?   S    09:15 0:00 [kswapd0]
```

Alguns dos que aparecem com mais frequência:

| Processo | Função |
|----------|--------|
| `kthreadd` | Cria e gere os restantes threads de kernel |
| `kswapd` | Move páginas de memória para swap quando a memória física escasseia |
| `ksoftirqd` | Trata interrupções de software que não puderam ser processadas imediatamente |
| `kjournald` | Escreve as actualizações do journal do sistema de ficheiros para disco |
| `khubd` | Configura dispositivos USB |

Destes, apenas o processo 1 é um processo de utilizador verdadeiro. Os restantes são partes do kernel apresentadas como processos por razões de escalonamento e arquitectura. Um administrador nunca precisa de interagir directamente com eles, mas convém reconhecê-los para não os confundir com processos anómalos.

#### O problema do initramfs

Existe aqui um problema de ovo e galinha que merece explicação, porque é a razão de ser de um componente que muitos administradores usam sem compreender.

Para montar o sistema de ficheiros raiz, o kernel precisa do controlador do dispositivo onde esse sistema de ficheiros reside. Se a raiz estiver num array RAID gerido por uma controladora de terceiros, ou num volume LVM, ou num disco NVMe, o kernel precisa do controlador adequado.

Mas existem centenas de controladores possíveis, e incluí-los todos no kernel produziria uma imagem enorme e desnecessariamente pesada para qualquer máquina concreta. A solução habitual é distribuí-los como módulos carregáveis. Só que módulos são ficheiros, e ficheiros vivem num sistema de ficheiros. O kernel precisa do módulo para montar o sistema de ficheiros onde o módulo está guardado.

O **initramfs** (*initial RAM filesystem*) resolve o impasse. É um arquivo comprimido que contém uma colecção mínima de módulos e utilitários, e que o gestor de arranque carrega para memória juntamente com o kernel. O kernel monta esse arquivo como sistema de ficheiros raiz temporário, executa o `init` que lá está, e esse init carrega os módulos necessários, monta o sistema de ficheiros raiz real, e transfere o controlo para o verdadeiro sistema.

Esta explicação fecha uma questão deixada em aberto no Capítulo 2: **porque é que a partição `/boot` fica fora do LVM**. O GRUB tem de conseguir ler o kernel e o initramfs antes de qualquer coisa relacionada com LVM estar disponível. Manter `/boot` numa partição simples garante que esses ficheiros são acessíveis com o mínimo de complexidade. O suporte a LVM entra depois, através dos módulos que o initramfs carrega.

#### Gerir o initramfs com dracut

Em CentOS, a ferramenta que constrói imagens initramfs é o `dracut`. Normalmente corre automaticamente quando um kernel é instalado ou actualizado, mas há situações em que é preciso invocá-la manualmente: depois de alterar a configuração de armazenamento, ou quando uma imagem fica corrompida.

```bash
# Ver as imagens existentes
$ ls -lh /boot/initramfs-*

# Inspeccionar o conteúdo de uma imagem
$ sudo lsinitrd /boot/initramfs-$(uname -r).img | less

# Regenerar a imagem do kernel actual
$ sudo dracut --force

# Regenerar para um kernel específico
$ sudo dracut --force /boot/initramfs-5.14.0-162.el9.x86_64.img 5.14.0-162.el9.x86_64

# Regenerar todas as imagens
$ sudo dracut --regenerate-all --force
```
---

### 1.5 systemd: targets e o fim dos runlevels

#### init: o processo número 1

Montado o sistema de ficheiros raiz, o kernel arranca o primeiro processo de espaço de utilizador. Este processo tem sempre o PID 1, é o antepassado de todos os processos de utilizador, e é ele que constrói o resto do sistema.

Historicamente este processo chamava-se `init` e seguia o modelo do System V. Em todas as distribuições Linux actuais, incluindo o CentOS, a implementação é o **systemd**.

#### O modelo antigo: runlevels

O modelo System V organizava o estado do sistema em **níveis de execução** (*runlevels*), cada um representando um conjunto de serviços que deviam estar a correr:

| Runlevel | Significado tradicional |
|----------|------------------------|
| 0 | Sistema completamente desligado |
| 1 | Modo de utilizador único (manutenção) |
| 2 | Multiutilizador sem rede |
| 3 | Multiutilizador completo com rede, sem interface gráfica |
| 4 | Não utilizado, disponível para uso local |
| 5 | Multiutilizador completo com interface gráfica |
| 6 | Reinício |

Os níveis 0 e 6 são especiais porque o sistema não pode permanecer neles: desliga-se ou reinicia como efeito lateral de lá entrar.

O ficheiro `/etc/inittab` definia o que fazer em cada nível, e o comando `telinit` permitia mudar de nível com o sistema em funcionamento. Cada serviço tinha um script em `/etc/init.d/` que aceitava os argumentos `start`, `stop` e `restart`. Ligações simbólicas em directórios como `/etc/rc3.d/` determinavam quais os scripts a executar em cada nível e por que ordem.

Este modelo funcionou durante décadas, mas acumulou limitações sérias:

**Desempenho.** Os scripts corriam sequencialmente, um de cada vez. Duas partes independentes do arranque não podiam correr em paralelo, mesmo em máquinas com vários núcleos ociosos.

**Gestão do sistema em execução.** Depois de um script arrancar um daemon, não existia forma fiável de saber que processos lhe pertenciam. Descobrir o PID de um serviço implicava consultar ficheiros `.pid` mantidos por convenção e nem sempre actualizados.

**Repetição de código.** Cada script continha grandes blocos de código genérico repetido, tornando difícil perceber o que cada um efectivamente fazia.

**Ausência de activação sob procura.** Praticamente tudo arrancava no boot, mesmo os serviços que só seriam usados esporadicamente.

#### systemd: unidades e dependências

O systemd resolve estes problemas mudando o modelo. Em vez de uma sequência de scripts, tem um conjunto de objectivos chamados **unidades** (*units*), cada um com as suas dependências declaradas.

Quando se activa uma unidade, o systemd activa primeiro as suas dependências, e depois trata dos detalhes da própria unidade. Como as dependências são declaradas explicitamente, o systemd pode activar em paralelo tudo o que não depende mutuamente, o que reduz drasticamente o tempo de arranque.

Existem vários **tipos de unidade**, e os mais relevantes no arranque são:

| Tipo | Extensão | Função |
|------|----------|--------|
| Service | `.service` | Controla daemons e serviços |
| Target | `.target` | Agrupa outras unidades, substitui os runlevels |
| Mount | `.mount` | Representa a montagem de um sistema de ficheiros |
| Socket | `.socket` | Representa um ponto de escuta de rede, permite activação sob procura |
| Timer | `.timer` | Activação por tempo, alternativa ao cron |
| Device | `.device` | Representa um dispositivo reconhecido pelo kernel |

#### Targets em vez de runlevels

Os **targets** substituem os runlevels, mas com uma diferença conceptual importante. Um runlevel era um estado numerado e exclusivo: o sistema estava no nível 3 ou no nível 5, nunca em ambos. Um target é apenas um agrupamento de unidades, e vários podem estar activos em simultâneo.

Por compatibilidade, existem targets que correspondem aos runlevels tradicionais:

| Target | Runlevel equivalente | Descrição |
|--------|---------------------|-----------|
| `poweroff.target` | 0 | Desligar |
| `rescue.target` | 1, s, single | Modo de recuperação, utilizador único |
| `multi-user.target` | 2, 3, 4 | Multiutilizador com rede, sem gráfico |
| `graphical.target` | 5 | Multiutilizador com interface gráfica |
| `reboot.target` | 6 | Reiniciar |
| `emergency.target` | — | Shell mínima, apenas raiz montada em leitura |

```bash
# Ver o target por defeito
$ systemctl get-default
multi-user.target

# Definir o target por defeito
$ sudo systemctl set-default multi-user.target

# Mudar de target com o sistema em funcionamento
$ sudo systemctl isolate graphical.target

# Ver os targets activos
$ systemctl list-units --type=target
```

Num servidor, o target por defeito deve ser `multi-user.target`. Arrancar em `graphical.target` num servidor consome memória e ciclos de CPU para nada, e aumenta a superfície de ataque com bibliotecas gráficas desnecessárias, como discutido no Capítulo 2.

#### Ficheiros de unidade

Os ficheiros de unidade vivem em dois locais com propósitos distintos:

`/usr/lib/systemd/system/` contém as unidades fornecidas pelos pacotes da distribuição. **Não deve ser editado**, porque as alterações são perdidas na próxima actualização do pacote.

`/etc/systemd/system/` contém as unidades e modificações locais. **É aqui que se fazem alterações**, e o que está aqui tem precedência sobre o directório anterior.

Esta regra é uma instância de um princípio geral em Linux: perante a escolha entre modificar algo em `/usr` ou em `/etc`, altera-se sempre em `/etc`.

Um ficheiro de unidade tem uma estrutura de secções em parênteses rectos com pares de opção e valor. Um exemplo de serviço:

```ini
[Unit]
Description=Servidor de aplicação da empresa
After=network-online.target
Wants=network-online.target
Requires=postgresql.service

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/aplicacao
ExecStart=/opt/aplicacao/bin/servidor --config /etc/aplicacao/config.yml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

A secção `[Unit]` descreve a unidade e as suas relações. `After` define ordem sem criar dependência: se ambas as unidades forem activadas, esta arranca depois. `Requires` cria dependência forte: se a unidade requerida falhar, esta também falha. `Wants` é uma dependência fraca: tenta activar a outra unidade mas prossegue se ela falhar.

A secção `[Service]` descreve como arrancar, recarregar e parar o serviço. A directiva `Restart=on-failure` faz o systemd reiniciar automaticamente o serviço se ele terminar com erro, algo que no modelo antigo exigia ferramentas externas.

A secção `[Install]` define onde a unidade se encaixa quando é activada com `systemctl enable`.

Após criar ou alterar um ficheiro de unidade, o systemd tem de reler a configuração:

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now aplicacao.service
```

#### Operar o systemd

O comando `systemctl` é a interface principal:

```bash
# Listar unidades activas
$ systemctl list-units

# Listar todas as unidades, incluindo inactivas
$ systemctl list-units --all

# Listar apenas serviços
$ systemctl list-units --type=service

# Ver que unidades falharam
$ systemctl --failed

# Estado detalhado de um serviço
$ systemctl status sshd.service

# Ver o conteúdo de um ficheiro de unidade
$ systemctl cat sshd.service

# Ver todas as propriedades de uma unidade
$ systemctl show sshd.service

# Ver as dependências de uma unidade
$ systemctl list-dependencies multi-user.target
```

O output do `systemctl status` é substancialmente mais informativo do que o equivalente no modelo antigo:

```bash
$ systemctl status sshd.service
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-10-15 09:15:41 WEST; 3 days ago
   Main PID: 1089 (sshd)
      Tasks: 1 (limit: 23456)
     Memory: 5.2M
        CPU: 1.842s
     CGroup: /system.slice/sshd.service
             └─1089 "sshd: /usr/sbin/sshd -D [listener]"

Oct 15 09:15:41 servidor sshd[1089]: Server listening on 0.0.0.0 port 22.
```

Além do estado, mostra o PID principal, os processos associados através do control group, o consumo de memória e CPU, e as últimas mensagens de log. O **cgroup** é o mecanismo que resolve o problema de rastreio do modelo antigo: todos os processos de um serviço ficam agrupados, e o systemd sabe sempre exactamente o que pertence a quê.

---

### 1.6 Personalização de mensagens do sistema

As mensagens que o sistema apresenta durante e após o arranque podem ser personalizadas. Além do valor estético, há aqui uma função prática e por vezes legal: identificar o sistema, avisar sobre as condições de acesso, e fornecer informação de contacto.

#### /etc/motd: mensagem do dia

O ficheiro `/etc/motd` (*message of the day*) é apresentado **depois** de um login bem-sucedido, tanto local como por SSH.

```bash
$ sudo vi /etc/motd
```

```
=========================================================
  SERVIDOR DE PRODUÇÃO — empresa.pt
=========================================================
  Este sistema é monitorizado e todas as sessões
  são registadas.

  Suporte: sistemas@empresa.pt | Ext. 4500
  Manutenção programada: domingos, 02:00-04:00
=========================================================
```

Em sistemas modernos existe também `/etc/motd.d/`, onde se podem colocar vários ficheiros que são concatenados, permitindo que diferentes pacotes ou scripts acrescentem informação sem colidir uns com os outros.

#### /etc/issue: banner antes do login

O `/etc/issue` é apresentado **antes** do prompt de login na consola local, e `/etc/issue.net` faz o mesmo para ligações de rede. Como aparecem antes da autenticação, são o local correcto para avisos legais de acesso.

Estes ficheiros suportam sequências de escape que são substituídas dinamicamente:

| Sequência | Valor |
|-----------|-------|
| `\n` | Nome do host |
| `\r` | Versão do kernel |
| `\m` | Arquitectura da máquina |
| `\d` | Data actual |
| `\t` | Hora actual |
| `\l` | Nome do terminal |

```
Sistema: \n (\l)
Kernel:  \r em \m

AVISO: O acesso não autorizado a este sistema é proibido
e será alvo de procedimento legal. Todas as actividades
são monitorizadas e registadas.
```

Para que o `/etc/issue.net` seja apresentado nas ligações SSH, é necessário activá-lo explicitamente em `/etc/ssh/sshd_config`:

```
Banner /etc/issue.net
```

#### Menu do GRUB

O texto e o aspecto do menu de arranque controlam-se através de `/etc/default/grub` e dos scripts em `/etc/grub.d/`. Para acrescentar uma entrada personalizada, edita-se `/etc/grub.d/40_custom` e regenera-se a configuração.

#### Ecrã de arranque

O **Plymouth** é o subsistema responsável pelo ecrã gráfico apresentado durante o arranque. Em servidores é frequentemente preferível desactivá-lo para ver as mensagens reais do sistema, o que ajuda no diagnóstico:

```bash
# Remover 'rhgb quiet' de GRUB_CMDLINE_LINUX em /etc/default/grub
$ sudo grubby --update-kernel=ALL --remove-args="rhgb quiet"
```

---

### 1.7 Diagnóstico de problemas de arranque

#### Analisar o arranque bem-sucedido

Mesmo quando o sistema arranca correctamente, vale a pena saber o que aconteceu e quanto tempo demorou cada parte.

```bash
# Tempo total e distribuição por fase
$ systemd-analyze
Startup finished in 1.203s (kernel) + 2.891s (initrd) + 8.432s (userspace) = 12.527s
multi-user.target reached after 8.412s in userspace

# Que unidades demoraram mais a arrancar
$ systemd-analyze blame
   4.123s NetworkManager-wait-online.service
   1.882s dnf-makecache.service
   0.923s firewalld.service
   0.412s sshd.service

# A cadeia crítica: o caminho que efectivamente determinou o tempo total
$ systemd-analyze critical-chain

# Gerar um gráfico SVG do arranque
$ systemd-analyze plot > /tmp/arranque.svg
```

O `blame` mostra o que demorou mais, mas o `critical-chain` é mais útil: uma unidade pode demorar muito e não atrasar o arranque se corre em paralelo com outras. Só o que está na cadeia crítica afecta o tempo total.

#### Logs de arranque

```bash
# Logs do arranque actual
$ journalctl -b

# Logs do arranque anterior
$ journalctl -b -1

# Listar todos os arranques registados
$ journalctl --list-boots

# Apenas mensagens do kernel
$ journalctl -k

# Apenas erros e mais graves
$ journalctl -b -p err

# Logs de uma unidade específica desde o arranque
$ journalctl -b -u sshd
```

Uma verificação particularmente útil, já vista no Capítulo 5: confirmar se o sistema encerrou correctamente da última vez.

```bash
$ journalctl -r -b -1 | head -20
```

Se as últimas mensagens mostram uma sequência ordenada de encerramento, o sistema foi desligado correctamente. Se terminam abruptamente sem qualquer indicação de shutdown, houve falha de energia, kernel panic ou reset forçado.

#### Targets de recuperação

Quando o sistema arranca mas não fica utilizável, existem dois modos degradados que permitem intervir.

O **rescue target** monta os sistemas de ficheiros locais e arranca um conjunto mínimo de serviços, apresentando uma shell de root. É o equivalente ao antigo modo de utilizador único.

O **emergency target** é ainda mais restrito: monta apenas a raiz e em modo de leitura, e não arranca serviço nenhum. É o último recurso quando nem o rescue funciona.

Para arrancar num destes modos, prime-se `e` sobre a entrada do menu do GRUB e acrescenta-se o parâmetro à linha que começa por `linux`:

```
systemd.unit=rescue.target
```

ou

```
systemd.unit=emergency.target
```

Premindo `Ctrl+X` arranca com o parâmetro acrescentado, sem o tornar permanente.

No modo de emergência, a raiz está montada apenas para leitura, e é impossível editar ficheiros de configuração. O primeiro comando a executar é normalmente:

```bash
# Remontar a raiz em modo de leitura e escrita
# mount -o remount,rw /
```

#### Recuperação da password de root

Este procedimento foi referido no Capítulo 5 mas o seu mecanismo só agora pode ser explicado adequadamente. O que acontece é o seguinte: interrompe-se o arranque **dentro do initramfs**, antes de o sistema real ter arrancado, e por isso antes de qualquer autenticação existir.

O passo a passo:

1. Reiniciar e premir `e` sobre a entrada do GRUB.
2. Na linha que começa por `linux`, acrescentar `rd.break` ao final.
3. Premir `Ctrl+X` para arrancar.

O sistema pára numa shell dentro do initramfs. Neste ponto, o sistema de ficheiros real está montado em `/sysroot`, em modo de leitura apenas.

```bash
# Remontar o sistema real em modo de escrita
switch_root:/# mount -o remount,rw /sysroot

# Mudar a raiz para o sistema real
switch_root:/# chroot /sysroot

# Alterar a password
sh-5.1# passwd root

# Marcar o sistema de ficheiros para relabelling do SELinux
sh-5.1# touch /.autorelabel

# Sair e reiniciar
sh-5.1# exit
switch_root:/# exit
```

O ficheiro `/.autorelabel` é essencial e frequentemente esquecido. Ao alterar `/etc/shadow` a partir do chroot, o contexto de segurança do SELinux do ficheiro fica incorrecto. Sem o relabelling, o sistema arranca mas o login falha, o que produz um problema aparentemente inexplicável. O relabelling completo demora alguns minutos no primeiro arranque seguinte.

Este procedimento demonstra também porque é que a protecção do GRUB com password importa: **quem tem acesso físico à consola tem acesso de root**, a menos que o menu esteja protegido.

#### Arranque a partir de suporte externo

Quando o sistema está demasiado partido para chegar sequer ao rescue target, a solução é arrancar a partir de um suporte externo: a imagem de instalação do CentOS em modo de recuperação, ou uma distribuição live.

Este método permite verificar e reparar sistemas de ficheiros, corrigir ficheiros críticos como `/etc/fstab` que impeçam o arranque, e restaurar a partir de cópias de segurança.

```bash
# Após arrancar do suporte externo, montar o sistema real
# mount /dev/mapper/cs-root /mnt/sysimage
# mount /dev/sda2 /mnt/sysimage/boot
# mount --bind /dev /mnt/sysimage/dev
# mount --bind /proc /mnt/sysimage/proc
# mount --bind /sys /mnt/sysimage/sys
# chroot /mnt/sysimage
```

A partir do `chroot`, o sistema comporta-se como se tivesse arrancado normalmente, e é possível reinstalar o GRUB, regenerar o initramfs, ou corrigir configurações.

#### Cenários comuns de falha

| Sintoma | Causa provável | Verificação |
|---------|---------------|-------------|
| GRUB não aparece | MBR ou ESP corrompidos | Arrancar de suporte externo, reinstalar GRUB |
| "Kernel panic: no init found" | initramfs corrompido ou raiz não encontrada | Arrancar kernel anterior, regenerar com `dracut` |
| Pára a montar sistemas de ficheiros | Entrada errada em `/etc/fstab` | Arrancar em emergency, corrigir o fstab |
| Arranca mas login falha sempre | Contexto SELinux incorrecto | `touch /.autorelabel` e reiniciar |
| Arranque muito lento | Serviço a esperar por timeout | `systemd-analyze blame` |

---

### 1.8 Encerramento do sistema

Encerrar um sistema Linux não é cortar a alimentação. Existe uma sequência ordenada que garante que os dados em cache são escritos para disco, os serviços terminam de forma limpa, e os sistemas de ficheiros ficam num estado consistente.

O procedimento, independentemente da implementação de init, é sempre este:

1. Todos os processos recebem um pedido para terminar de forma ordenada (sinal `TERM`).
2. Processos que não respondem dentro de um tempo limite recebem `KILL`.
3. Os sistemas de ficheiros são desmontados.
4. A raiz é remontada em modo de leitura apenas.
5. O kernel é instruído a desligar ou reiniciar.

```bash
# Desligar imediatamente
$ sudo shutdown -h now
$ sudo poweroff

# Reiniciar imediatamente
$ sudo shutdown -r now
$ sudo reboot

# Desligar dentro de 10 minutos, com aviso aos utilizadores
$ sudo shutdown -h +10 "Manutenção programada. Guardem o trabalho."

# Desligar a uma hora específica
$ sudo shutdown -h 23:00

# Cancelar um encerramento agendado
$ sudo shutdown -c
```

Quando se agenda um encerramento para o futuro, o comando `shutdown` cria o ficheiro `/run/nologin`. Enquanto esse ficheiro existir, o sistema recusa logins de todos os utilizadores excepto o root, evitando que alguém inicie trabalho que será interrompido em minutos.

O aviso enviado aos utilizadores com sessão activa usa o mecanismo `wall` visto no Capítulo 5.

## 2. Fundamentos de Armazenamento e Particionamento

## 2.1 Como o Linux vê o armazenamento

Antes de particionar um disco, é preciso perceber como o Linux organiza o armazenamento em camadas. Cada camada resolve um problema específico, e conhecê-las evita a confusão que surge quando os comandos mostram coisas que não parecem corresponder ao que se espera.

O ponto de partida é o **dispositivo de bloco**. Um disco inteiro é apresentado pelo kernel como um dispositivo de bloco, com um nome como `/dev/sda`. Chama-se dispositivo de bloco porque os dados são lidos e escritos em blocos de tamanho fixo, ao contrário dos dispositivos de caractere que lidam com fluxos de bytes.

Por cima do disco físico existe a **tabela de partições**, uma pequena área no início do disco que descreve como o espaço está dividido. Cada divisão é uma **partição**, e o kernel apresenta cada partição também como um dispositivo de bloco próprio, com nomes como `/dev/sda1` e `/dev/sda2`.

Dentro de uma partição vive o **sistema de ficheiros**, que é a base de dados que transforma um espaço bruto de blocos na hierarquia de ficheiros e directórios com que o utilizador interage. É esse sistema de ficheiros que se acede quando se corre `ls` ou `cd`.

Para chegar aos dados de um ficheiro, o kernel percorre estas camadas de cima para baixo: consulta a tabela de partições para localizar a partição correcta, procura na base de dados do sistema de ficheiros dessa partição a localização do ficheiro, e finalmente lê os blocos onde os dados estão.

Este percurso atravessa uma pilha de subsistemas dentro do kernel. Os processos de utilizador fazem pedidos através de chamadas de sistema, que passam pela camada do sistema de ficheiros, depois pela interface de dispositivo de bloco que mapeia as partições, e finalmente pelos controladores que falam com o hardware de armazenamento.

Existe ainda um caminho alternativo. É possível aceder ao dispositivo de bloco directamente, sem passar pelo sistema de ficheiros, através dos ficheiros de dispositivo em `/dev`. É este acesso directo que ferramentas como o `dd` ou o `fdisk` usam para manipular o disco ao nível dos blocos brutos, e é também o que as torna perigosas: escrevem directamente no disco sem a rede de segurança do sistema de ficheiros.

O **LVM** (*Logical Volume Manager*), que veremos na secção 4, insere-se entre a partição e o sistema de ficheiros, acrescentando uma camada de flexibilidade. Por agora, começamos pelo particionamento tradicional e directo.

---

## 2.2 Dispositivos de bloco e nomenclatura

Os nomes dos dispositivos de bloco seguem convenções que revelam o tipo de hardware e a sua ordem de detecção.

Os discos são nomeados por família. Discos SATA, SAS, SCSI e USB aparecem como `/dev/sda`, `/dev/sdb`, `/dev/sdc`, e assim por diante, onde a letra final indica a ordem pela qual o kernel os detectou. Discos NVMe, uma tecnologia mais recente e rápida, seguem um esquema diferente: `/dev/nvme0n1`, `/dev/nvme1n1`.

As partições acrescentam um número ao nome do disco:

| Dispositivo | Significado |
|-------------|-------------|
| `/dev/sda` | Primeiro disco SATA/SAS/USB (o disco inteiro) |
| `/dev/sda1` | Primeira partição do primeiro disco |
| `/dev/sda2` | Segunda partição do primeiro disco |
| `/dev/sdb1` | Primeira partição do segundo disco |
| `/dev/nvme0n1` | Primeiro disco NVMe |
| `/dev/nvme0n1p1` | Primeira partição do primeiro disco NVMe |

Note-se que os discos NVMe usam `p1` para separar o número da partição, porque o nome do disco já termina em número e `nvme0n11` seria ambíguo.

>  **A letra do dispositivo não é fixa.** O `/dev/sdb` de hoje pode ser o `/dev/sdc` amanhã, porque a atribuição depende da ordem de detecção. Acrescentar ou remover um disco, ou até uma alteração no tempo de arranque, pode reordenar as letras. Esta instabilidade é a razão pela qual, como veremos na secção 3, os sistemas de ficheiros nunca devem ser montados permanentemente pelo nome do dispositivo, mas sim pelo UUID.

Existem também interfaces especiais que representam armazenamento sem ser um disco físico simples. Os dispositivos `/dev/dm-*` e os nomes em `/dev/mapper/` correspondem ao **device mapper**, o subsistema que o LVM e o RAID usam. Se estes aparecerem no sistema, é sinal de que uma dessas tecnologias está em uso.

---

## 2.3 Tabelas de partições: MBR e GPT

A tabela de partições não tem nada de mágico. É apenas um bloco de dados no início do disco que descreve como os blocos estão divididos. Existem dois formatos em uso, e a diferença entre eles tem consequências práticas.

### MBR

O **MBR** (*Master Boot Record*) é o formato tradicional, que remonta aos primeiros PCs. Como visto na secção 1, os primeiros 512 bytes do disco contêm tanto o código de arranque como a tabela de partições, e é essa herança que impõe as suas limitações.

O MBR permite apenas **quatro partições primárias**. Para contornar este limite, uma das quatro pode ser designada como **partição estendida**, que funciona como um contentor para **partições lógicas**. Assim, um disco pode ter três partições primárias mais uma estendida que contém várias lógicas.

```
Disco com MBR:
├── sda1  (primária)
├── sda2  (primária)
├── sda3  (primária)
└── sda4  (estendida)
    ├── sda5  (lógica)
    ├── sda6  (lógica)
    └── sda7  (lógica)
```

As partições lógicas começam sempre a numeração no 5, independentemente de quantas primárias existam, porque os números 1 a 4 estão reservados às primárias.

O MBR tem duas limitações que o tornam obsoleto para hardware moderno. Não suporta discos maiores que 2 TiB, porque usa endereços de 32 bits para localizar os blocos. E o esquema de partições primárias e estendidas é desajeitado. Por estas razões, o MBR foi substituído pelo GPT.

### GPT

O **GPT** (*GUID Partition Table*) é o formato moderno, associado ao firmware UEFI mas utilizável também com BIOS. Resolve as limitações do MBR de forma directa.

Suporta discos até tamanhos que na prática não têm limite relevante (na ordem dos zettabytes). Permite até 128 partições por defeito, todas em pé de igualdade, sem a distinção artificial entre primárias, estendidas e lógicas. E cada partição tem um identificador único global (GUID) e pode ter um nome descritivo.

O GPT guarda ainda uma cópia da tabela de partições no final do disco, além da cópia no início, o que oferece resistência a corrupção: se a tabela principal for danificada, a cópia de segurança permite recuperá-la.

### Qual usar

Para qualquer instalação nova em hardware moderno, GPT é a escolha correcta. A instalação do CentOS no Capítulo 2 usou GPT, e é por isso que a tabela de partições incluía a partição `biosboot`, necessária apenas quando se usa GPT com firmware em modo BIOS, como explicado na secção 1.

O MBR mantém relevância apenas em dois cenários: hardware muito antigo que não suporta GPT, e discos pequenos onde a compatibilidade com sistemas legados é necessária.

---

## 2.4 Inspeccionar o armazenamento

Antes de alterar seja o que for, é preciso saber o que existe. Vários comandos oferecem visões complementares do armazenamento do sistema.

### lsblk: a visão em árvore

O `lsblk` (*list block devices*) é o ponto de partida. Mostra todos os dispositivos de bloco numa estrutura em árvore que torna imediatamente visível a relação entre discos, partições e volumes:

```bash
$ lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda             8:0    0    50G  0 disk
├─sda1          8:1    0     1M  0 part
├─sda2          8:2    0     1G  0 part /boot
└─sda3          8:3    0    49G  0 part
  ├─cs-root   253:0    0    44G  0 lvm  /
  └─cs-swap   253:1    0     5G  0 lvm  [SWAP]
sdb             8:16   0   100G  0 disk
sr0            11:0    1  1024M  0 rom
```

Neste exemplo lê-se de imediato a estrutura: o disco `sda` de 50 GB tem três partições, sendo que a terceira contém volumes LVM montados na raiz e no swap. Existe um segundo disco `sdb` de 100 GB ainda sem partições. E `sr0` é a unidade óptica.

A coluna `TYPE` distingue `disk` (disco físico), `part` (partição) e `lvm` (volume lógico). A coluna `MOUNTPOINTS` mostra onde cada dispositivo está montado.

```bash
# Incluir os sistemas de ficheiros e UUIDs
$ lsblk -f

# Incluir informação sobre o modelo e número de série
$ lsblk -o NAME,SIZE,TYPE,FSTYPE,MODEL,SERIAL
```

### blkid: identificadores e tipos

O `blkid` (*block ID*) foca-se na identificação: mostra o UUID e o tipo de sistema de ficheiros de cada partição:

```bash
$ sudo blkid
/dev/sda2: UUID="a1b2c3d4-..." TYPE="xfs" PARTUUID="c9a5ebb0-01"
/dev/sda3: UUID="e5f6g7h8-..." TYPE="LVM2_member" PARTUUID="c9a5ebb0-02"
/dev/mapper/cs-root: UUID="i9j0k1l2-..." TYPE="xfs"
/dev/mapper/cs-swap: UUID="m3n4o5p6-..." TYPE="swap"
```

Este comando é essencial quando se prepara uma entrada no `/etc/fstab`, porque fornece o UUID exacto que se deve usar em vez do nome do dispositivo.

### fdisk -l e parted: a tabela de partições

Para ver a tabela de partições em detalhe:

```bash
$ sudo fdisk -l /dev/sda
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: A1B2C3D4-...

Device       Start       End   Sectors Size Type
/dev/sda1     2048      6143      4096   2M BIOS boot
/dev/sda2     6144   2103295   2097152   1G Linux filesystem
/dev/sda3  2103296 104857566 102754271  49G Linux LVM
```

O `parted` oferece uma alternativa que apresenta os tamanhos de forma mais legível:

```bash
$ sudo parted /dev/sda print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 53.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  3146kB  2097kB                      bios_grub
 2      3146kB  1077MB  1074MB  xfs
 3      1077MB  53.7GB  52.6GB                      lvm
```

> **Cuidado com as unidades.** O `fdisk -l` mostra tamanhos em sectores de 512 bytes, o que pode confundir: um valor pode parecer o dobro do tamanho real do disco. O `parted` mostra tamanhos aproximados numa unidade legível. Quando o valor exacto importa, o `fdisk` é mais fiável; quando importa a leitura rápida, o `parted` é mais claro.

---

## 2.5 Criar partições

Alterar a tabela de partições é uma operação de risco. Antes de começar, dois princípios são inegociáveis.

> **Alterar a tabela de partições pode tornar impossível recuperar os dados das partições afectadas.** Apagar ou redefinir uma partição pode apagar a localização do sistema de ficheiros que lá estava. Confirme sempre que tem uma cópia de segurança se o disco contém dados importantes.

> **Garanta que nenhuma partição do disco alvo está montada.** A maioria das distribuições monta automaticamente qualquer sistema de ficheiros detectado. Verifique com `lsblk` e desmonte tudo antes de continuar.

Existem três ferramentas principais para criar partições, e a escolha entre elas tem uma consequência importante.

### fdisk vs parted: uma diferença crítica

O **fdisk** e o **parted** funcionam de forma fundamentalmente diferente.

Com o `fdisk`, todas as alterações são feitas numa cópia em memória. Nada é escrito no disco até que se dê explicitamente o comando de gravação. Isto significa que se pode planear toda a nova tabela, rever, e desistir sem consequências se algo estiver errado.

Com o `parted`, as alterações são aplicadas ao disco à medida que os comandos são dados. Não há oportunidade de rever a tabela antes de a alterar.

Por esta razão, para criar e alterar partições, o `fdisk` é geralmente preferível: a natureza interactiva e a rede de segurança de não escrever nada até à confirmação tornam-no mais seguro. O `parted` é excelente para consulta e para scripts automatizados, onde o comportamento não-interactivo é uma vantagem.

Existe ainda o **gdisk**, que é ao `fdisk` o que este é para MBR mas especializado em GPT, e o **gparted**, uma interface gráfica sobre o `parted`.

### Criar partições com fdisk

O exemplo seguinte mostra a criação de duas partições num disco novo `/dev/sdb`: uma de 2 GB e outra ocupando o resto do espaço.

Primeiro, confirmar qual o disco e que não está montado:

```bash
$ lsblk
$ sudo fdisk /dev/sdb
```

O `fdisk` entra em modo interactivo. O comando `m` mostra a ajuda, e `p` imprime a tabela actual:

```
Command (m for help): p
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disklabel type: gpt
```

Criar a primeira partição com `n`:

```
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-209715199, default 2048): [Enter para aceitar]
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199): +2G

Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.
```

Os valores por defeito são quase sempre o que se pretende. O primeiro sector fica no valor por defeito (2048, que garante o alinhamento correcto, como veremos), e o tamanho é definido com a sintaxe `+2G`.

Criar a segunda partição, aceitando os valores por defeito para ocupar todo o espaço restante:

```
Command (m for help): n
Partition number (2-128, default 2): 2
First sector (...): [Enter]
Last sector (...): [Enter para usar todo o espaço restante]

Created a new partition 2 of type 'Linux filesystem'.
```

Rever antes de gravar com `p`:

```
Command (m for help): p
Device        Start       End   Sectors Size Type
/dev/sdb1      2048   4196351   4194304   2G Linux filesystem
/dev/sdb2   4196352 209715199 205518848  98G Linux filesystem
```

Se algo estiver errado, o comando `q` sai **sem gravar nada**. Se estiver tudo correcto, o comando `w` escreve a tabela no disco:

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Após gravar, é útil confirmar que o kernel releu a tabela:

```bash
$ lsblk /dev/sdb
$ sudo journalctl -k | tail -5
```

### O tipo de partição

Por defeito, o `fdisk` cria partições do tipo "Linux filesystem". Para certas utilizações é necessário mudar o tipo, o que se faz com o comando `t` dentro do `fdisk`. Os tipos mais relevantes:

| Tipo | Uso |
|------|-----|
| Linux filesystem | Partição de dados normal (por defeito) |
| Linux swap | Espaço de swap |
| Linux LVM | Partição que será um physical volume do LVM |
| Linux RAID | Partição que fará parte de um array RAID |
| EFI System | Partição ESP em sistemas UEFI |

Definir o tipo correcto não é obrigatório para o funcionamento, mas é uma boa prática: ajuda as ferramentas a identificar a função de cada partição e evita erros.

### Alinhamento de partições

Um detalhe que já não exige intervenção manual mas que importa compreender é o **alinhamento**. Nos discos SSD, os dados são lidos em blocos de tamanho fixo (páginas de 4096 ou 8192 bytes), e a leitura tem de começar num múltiplo desse tamanho. Se uma partição começar num ponto desalinhado, operações simples podem exigir duas leituras em vez de uma, degradando o desempenho.

As versões modernas das ferramentas de particionamento resolvem isto automaticamente, alinhando as partições em fronteiras de 1 MiB (o sector 2048). Este valor é um múltiplo de todos os tamanhos de página comuns, pelo que garante alinhamento óptimo sem cálculos. É por isso que o primeiro sector por defeito no `fdisk` é sempre 2048: aceitar o valor por defeito garante o alinhamento correcto.

### Forçar a releitura da tabela

Ocasionalmente, o kernel não relê a tabela de partições após uma alteração, normalmente porque alguma partição do disco ainda está em uso. Nesse caso, força-se a releitura:

```bash
$ sudo partprobe /dev/sdb
# ou
$ sudo blockdev --rereadpt /dev/sdb
```

Se mesmo assim o kernel não actualizar, a solução definitiva é reiniciar. Concluído o particionamento, as partições existem como dispositivos de bloco mas estão vazias. O passo seguinte, criar um sistema de ficheiros nelas, é o tema da secção 3.

## 3. Sistemas de Ficheiros, Montagem e Swap

Uma partição, por si só, não guarda ficheiros. É apenas um intervalo de blocos brutos no disco. Para que possa conter a hierarquia de ficheiros e directórios com que trabalhamos, é preciso instalar nela um **sistema de ficheiros**: a estrutura de dados que organiza os blocos, regista onde cada ficheiro começa e acaba, guarda as permissões, os donos e as datas, e mantém a árvore de directórios.

Esta secção cobre o ciclo completo: criar o sistema de ficheiros na partição, montá-lo para o tornar acessível, garantir que essa montagem persiste após reinícios, verificá-lo e repará-lo quando algo corre mal, e gerir o espaço de swap.

---

## 3.1 Criar sistemas de ficheiros: mkfs

### Escolher o sistema de ficheiros

O Linux suporta muitos sistemas de ficheiros, mas para a esmagadora maioria dos casos a escolha certa é o valor por defeito da distribuição. As ferramentas administrativas e a documentação assumem esse valor, e qualquer ganho de mudar para outro é marginal e dependente do contexto. Apenas três características são verdadeiramente não negociáveis: bom desempenho, tolerância a falhas e cortes de energia sem corrupção, e capacidade para discos e ficheiros do tamanho necessário. Os sistemas de ficheiros modernos por defeito já cobrem estas bases.

Vale a pena conhecer os principais:

**XFS** é o sistema de ficheiros por defeito no CentOS, RHEL e derivados desde o RHEL 7. É de alto desempenho, especialmente com ficheiros grandes e operações paralelas, e escala bem para volumes muito grandes. É o sistema de ficheiros com que se trabalha por defeito neste guia.

**ext4** é o padrão histórico do Linux e continua a ser o valor por defeito em distribuições como o Ubuntu. É a quarta iteração de uma linhagem que começou no ext2. O **ext3** acrescentou ao ext2 a funcionalidade de **journaling**, e o **ext4** acrescentou suporte para ficheiros maiores e para *extents*, que são intervalos contíguos de blocos em vez de blocos individuais.

**Btrfs** é um sistema de ficheiros mais recente, desenhado para escalar para além do ext4, com funcionalidades avançadas como snapshots ao nível do sistema de ficheiros.

**vfat** e **exfat** são os sistemas de ficheiros da Microsoft, usados por defeito na maioria dos suportes amovíveis como cartões SD e pens USB, precisamente por serem legíveis em praticamente todos os sistemas operativos.

### O conceito de journaling

O **journaling** é uma das razões pelas quais os sistemas de ficheiros modernos raramente corrompem após uma falha, e vale a pena compreender o mecanismo.

Sem journal, quando uma operação de escrita é interrompida a meio, por corte de energia por exemplo, o sistema de ficheiros pode ficar num estado inconsistente: um ficheiro que foi parcialmente escrito, uma entrada de directório que aponta para dados que nunca chegaram a ser gravados. Detectar e reparar estas inconsistências exigia percorrer toda a estrutura do disco, o que podia demorar horas.

Um sistema de ficheiros com journal reserva uma área onde regista o que vai fazer **antes** de o fazer. A operação é primeiro escrita no journal, depois é marcada como concluída com um registo de commit, e só então o sistema de ficheiros real é modificado. Se uma falha ocorrer a meio, o sistema recupera consultando o journal: refaz as operações que tinham commit e descarta as que não tinham. A verificação passa de horas para cerca de um segundo por sistema de ficheiros.

### Estruturas fundamentais

Independentemente do tipo, os sistemas de ficheiros partilham alguns conceitos herdados da tradição UNIX.

O **inode** é uma entrada de tabela que guarda toda a informação sobre um ficheiro, excepto o nome: permissões, dono, grupo, datas, tamanho, e a localização dos blocos de dados. O nome do ficheiro está guardado na entrada de directório, que aponta para o inode. É por isso que uma hard link não cria um novo inode: cria apenas uma nova entrada de directório que aponta para o inode existente. Cada inode tem um número, visível com `ls -i`.

O **superbloco** é o registo que descreve as características do próprio sistema de ficheiros: o tamanho dos blocos, a localização das tabelas de inodes, o mapa de blocos livres. É informação tão crítica que o `mkfs` cria várias cópias de segurança espalhadas pelo disco, para o caso de a original ser danificada.

### O comando mkfs

O `mkfs` (*make filesystem*) cria um sistema de ficheiros numa partição. A forma geral especifica o tipo e o dispositivo:

```bash
# Criar um sistema de ficheiros XFS (padrão no CentOS)
$ sudo mkfs -t xfs /dev/sdb1

# Criar um sistema de ficheiros ext4
$ sudo mkfs -t ext4 /dev/sdb1

# Sintaxe alternativa com o comando específico
$ sudo mkfs.xfs /dev/sdb1
$ sudo mkfs.ext4 /dev/sdb1
```

Na prática, o `mkfs` é apenas um invólucro que chama o programa específico de cada sistema de ficheiros. Quando se corre `mkfs -t xfs`, o que executa é o `mkfs.xfs`. Para o XFS, este por sua vez é o `mkfs.xfs`; para o ext4, é uma ligação para o `mke2fs`.

O `mkfs` determina automaticamente o número de blocos do dispositivo e define valores por defeito razoáveis. A não ser que se saiba exactamente o que se está a fazer, esses valores não devem ser alterados.

> **Criar um sistema de ficheiros destrói todos os dados existentes na partição.** O `mkfs` deve ser executado apenas uma vez por partição nova, ou quando se quer deliberadamente apagar o conteúdo. Correr `mkfs` sobre um sistema de ficheiros existente apaga irreversivelmente os dados que lá estavam. Confirme sempre o nome do dispositivo antes de premir Enter.

---

## 3.2 Montar e desmontar

Criar o sistema de ficheiros não o torna acessível. Para que os processos o possam usar, tem de ser **montado**: ligado a um directório da hierarquia do sistema, chamado **ponto de montagem**.

O ponto de montagem é um directório normal. Depois de um sistema de ficheiros ser montado sobre ele, o conteúdo original desse directório fica oculto e é substituído pelo conteúdo do sistema de ficheiros montado. Por convenção, `/mnt` é usado para montagens temporárias e `/media` para suportes amovíveis, mas o ponto de montagem pode ser qualquer directório.

### Montar

```bash
# Criar o directório que servirá de ponto de montagem
$ sudo mkdir /mnt/dados

# Montar a partição nesse ponto
$ sudo mount /dev/sdb1 /mnt/dados

# O tipo é normalmente detectado automaticamente, mas pode ser especificado
$ sudo mount -t xfs /dev/sdb1 /mnt/dados
```

Após a montagem, tudo o que for escrito em `/mnt/dados` é gravado na partição `/dev/sdb1`.

### Ver o que está montado

```bash
# Listar todos os sistemas de ficheiros montados
$ mount

# Versão mais legível, em árvore
$ findmnt

# Ver o espaço usado e disponível
$ df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/cs-root    44G  8.2G   36G  19% /
/dev/sda2            1014M  247M  768M  25% /boot
/dev/sdb1              98G  1.1G   97G   2% /mnt/dados
```

O comando `df -h` (*disk free*, com a opção `-h` de *human-readable*) é a forma habitual de verificar o espaço. Cada linha mostra o tamanho total, o usado, o disponível, a percentagem de utilização e o ponto de montagem.

### Desmontar

```bash
# Desmontar pelo ponto de montagem
$ sudo umount /mnt/dados

# Ou pelo dispositivo
$ sudo umount /dev/sdb1
```

Note-se que o comando é `umount`, sem o "n", uma peculiaridade histórica do UNIX.

> **Não se pode desmontar um sistema de ficheiros que esteja em uso.** Se algum processo tiver um ficheiro aberto nesse sistema de ficheiros, ou se a shell actual estiver dentro dele, o `umount` falha com "target is busy". Para descobrir o que está a usar o sistema de ficheiros:

```bash
# Ver que processos estão a usar o ponto de montagem
$ sudo lsof /mnt/dados
$ sudo fuser -m /mnt/dados

# Sair do directório se for esse o problema
$ cd /
```

---

## 3.3 UUIDs e montagem persistente com /etc/fstab

### O problema dos nomes de dispositivo

As montagens feitas com o comando `mount` são temporárias: desaparecem no reinício. Para que um sistema de ficheiros seja montado automaticamente no arranque, tem de ser registado no ficheiro `/etc/fstab`.

Mas há um problema, já antecipado na secção 2. Os nomes de dispositivo como `/dev/sdb1` não são estáveis. Dependem da ordem pela qual o kernel detecta os discos, e essa ordem pode mudar quando se acrescenta ou remove hardware. Uma entrada em `/etc/fstab` que referencie `/dev/sdb1` pode, após adicionar um disco novo, passar a montar o disco errado, ou falhar por completo, impedindo o arranque do sistema.

A solução é identificar os sistemas de ficheiros pelo seu **UUID** (*Universally Unique Identifier*), um número de série único gerado no momento da criação do sistema de ficheiros. Ao contrário do nome do dispositivo, o UUID acompanha o sistema de ficheiros independentemente da ordem de detecção.

```bash
# Ver os UUIDs de todos os sistemas de ficheiros
$ sudo blkid
/dev/sda2: UUID="a1b2c3d4-..." TYPE="xfs"
/dev/sdb1: UUID="f5e6d7c8-9012-3456-abcd-ef1234567890" TYPE="xfs"
```

### A estrutura do /etc/fstab

O ficheiro `/etc/fstab` (*filesystem table*) tem uma linha por sistema de ficheiros, com seis campos:

```
# <dispositivo>              <ponto montagem>  <tipo>  <opções>       <dump> <fsck>
UUID=a1b2c3d4-...            /                 xfs     defaults        0      0
UUID=e5f6g7h8-...            /boot             xfs     defaults        0      0
UUID=f5e6d7c8-...            /mnt/dados        xfs     defaults        0      0
/dev/mapper/cs-swap         swap              swap    defaults        0      0
```

O significado de cada campo:

| Campo | Função |
|-------|--------|
| Dispositivo | O que montar, identificado por UUID (preferível), LABEL ou nome |
| Ponto de montagem | Onde montar. Para swap, usa-se a palavra `swap` |
| Tipo | O sistema de ficheiros: `xfs`, `ext4`, `swap`, `nfs` |
| Opções | Opções de montagem separadas por vírgulas |
| Dump | Usado por ferramentas de backup antigas. Normalmente `0` |
| fsck | Ordem de verificação no arranque. `1` para a raiz, `2` para as outras, `0` para não verificar |

O último campo, a ordem de `fsck`, determina que sistemas de ficheiros são verificados no arranque e por que ordem. A raiz leva `1` para ser verificada primeiro; os restantes sistemas de ficheiros locais levam `2`; e sistemas de ficheiros que não devem ser verificados, como swap ou montagens de rede, levam `0`.

### Opções de montagem comuns

O campo de opções controla como o sistema de ficheiros é montado:

| Opção | Efeito |
|-------|--------|
| `defaults` | Conjunto padrão razoável (rw, suid, dev, exec, auto, async) |
| `ro` / `rw` | Montar em leitura apenas / leitura e escrita |
| `noatime` | Não actualizar a data de último acesso (melhora o desempenho) |
| `nofail` | Não impedir o arranque se o dispositivo não existir |
| `noexec` | Impedir a execução de binários a partir deste sistema de ficheiros |
| `nosuid` | Ignorar bits setuid e setgid |
| `_netdev` | Aguardar pela rede antes de montar (para sistemas de ficheiros de rede) |

A opção `nofail` merece destaque para discos secundários e amovíveis: sem ela, se o disco não estiver presente no arranque, o systemd espera indefinidamente e o sistema não arranca. Com ela, o arranque prossegue mesmo que o disco falte.

### Adicionar uma entrada permanente

O procedimento completo para tornar uma montagem permanente:

```bash
# 1. Obter o UUID do sistema de ficheiros
$ sudo blkid /dev/sdb1
/dev/sdb1: UUID="f5e6d7c8-9012-3456-abcd-ef1234567890" TYPE="xfs"

# 2. Criar o ponto de montagem
$ sudo mkdir -p /mnt/dados

# 3. Acrescentar a linha ao /etc/fstab
$ echo 'UUID=f5e6d7c8-9012-3456-abcd-ef1234567890 /mnt/dados xfs defaults 0 2' | sudo tee -a /etc/fstab

# 4. Validar a configuração SEM reiniciar
$ sudo mount -a
```

> **Valide sempre o /etc/fstab com `mount -a` antes de reiniciar.** Como visto na secção 1, um erro no fstab é uma das formas mais fáceis de impedir o arranque de um servidor. O comando `mount -a` tenta montar todas as entradas do fstab que ainda não estão montadas. Se não devolver erros, o arranque também não terá problemas de montagem. Se devolver, corrija antes de reiniciar, enquanto ainda tem acesso ao sistema.

---

## 3.4 Verificação e reparação: fsck e xfs_repair

Sistemas de ficheiros podem tornar-se inconsistentes após falhas de energia, problemas de hardware ou erros do kernel. As ferramentas de verificação detectam e reparam estas inconsistências, mas a ferramenta correcta depende do tipo de sistema de ficheiros, e este é um ponto onde muitos administradores tropeçam.

### fsck para a família ext

O `fsck` (*filesystem check*) é a ferramenta tradicional, e funciona para os sistemas de ficheiros da família ext (ext2, ext3, ext4).

```bash
# O sistema de ficheiros TEM de estar desmontado
$ sudo umount /dev/sdb1

# Verificar e reparar interactivamente
$ sudo fsck /dev/sdb1

# Reparar automaticamente sem perguntar (responder sim a tudo)
$ sudo fsck -y /dev/sdb1

# Forçar verificação mesmo que o sistema pareça limpo
$ sudo fsck -f /dev/sdb1
```

> **Nunca corra `fsck` num sistema de ficheiros montado em modo de escrita.** Fazê-lo pode corromper irreversivelmente o sistema de ficheiros, porque o `fsck` assume que tem controlo exclusivo sobre a estrutura. Desmonte sempre primeiro, ou use o modo de recuperação visto na secção 1 para verificar a própria raiz.

Quando o `fsck` encontra ficheiros cujo directório pai não consegue determinar, coloca-os no directório `lost+found` na raiz de cada sistema de ficheiros. Como o nome do ficheiro estava guardado apenas no directório pai perdido, estes ficheiros recebem o número do inode como nome. O inode preserva o UID do dono, o que facilita devolvê-los. **Este directório não deve ser apagado.**

### xfs_repair para XFS

Aqui está o ponto crítico que a documentação genérica frequentemente ignora: **o `fsck` não repara sistemas de ficheiros XFS.** Como o CentOS usa XFS por defeito, esta é a situação que um administrador de CentOS efectivamente enfrenta.

Se correr `fsck` num sistema de ficheiros XFS, ele não faz praticamente nada: existe um `fsck.xfs` mas é essencialmente um programa vazio que retorna sucesso sem verificar nada. A verificação real do XFS acontece automaticamente quando o sistema de ficheiros é montado, através da recuperação do seu journal interno. Para reparação manual, a ferramenta é o `xfs_repair`.

```bash
# O sistema de ficheiros TEM de estar desmontado
$ sudo umount /dev/sdb1

# Verificar sem reparar (dry run)
$ sudo xfs_repair -n /dev/sdb1

# Reparar
$ sudo xfs_repair /dev/sdb1
```

Existe uma diferença importante no fluxo de trabalho do XFS. Se o journal do XFS estiver "sujo" (com transações não concluídas de uma falha), o `xfs_repair` recusa-se a prosseguir e pede que o sistema de ficheiros seja primeiro montado e desmontado para o journal ser reproduzido:

```bash
# Deixar o journal ser reproduzido montando e desmontando
$ sudo mount /dev/sdb1 /mnt/dados
$ sudo umount /dev/sdb1

# Agora o xfs_repair pode prosseguir
$ sudo xfs_repair /dev/sdb1
```

Em último recurso, quando o journal está tão danificado que impede a montagem, pode forçar-se o `xfs_repair` a descartá-lo com a opção `-L`. Isto deve ser usado apenas como último recurso, porque descartar o journal pode significar perder as transações que ele continha:

```bash
$ sudo xfs_repair -L /dev/sdb1
```

### Resumo: que ferramenta usar

| Sistema de ficheiros | Ferramenta de reparação |
|---------------------|------------------------|
| ext2, ext3, ext4 | `fsck` ou `e2fsck` |
| XFS | `xfs_repair` |
| Btrfs | `btrfs check` |
| vfat/exfat | `fsck.vfat` / `fsck.exfat` |

A regra prática para o CentOS: para a raiz e restantes sistemas de ficheiros, que são XFS, a ferramenta é o `xfs_repair`. O `fsck` só é relevante se existirem partições ext no sistema.

---

## 3.5 Gestão de swap

### O que é o swap

O **swap** é espaço em disco usado como extensão da memória RAM. Quando a memória física escasseia, o kernel move páginas de memória menos usadas para o swap, libertando RAM para o que está activo. Ao contrário de um sistema de ficheiros, o swap não guarda ficheiros: o kernel mantém o seu próprio mapeamento simplificado entre páginas de memória e blocos de swap.

O swap pode residir numa partição dedicada ou num ficheiro. A partição dedicada é ligeiramente mais eficiente e é a abordagem tradicional; o ficheiro de swap é mais flexível e útil quando é preciso acrescentar swap rapidamente sem reparticionar.

Sobre a quantidade de swap, a regra tradicional é uma quantidade igual à RAM para sistemas com pouca memória, reduzindo a proporção à medida que a RAM aumenta. Foi esta a lógica por trás da decisão tomada no Capítulo 2, onde se atribuíram 4 GiB de swap a uma máquina com 4 GiB de RAM. Convém lembrar, no entanto, que a melhor opção de todas é não precisar de swap: se um sistema recorre constantemente ao swap, a solução real é acrescentar RAM, porque o disco é ordens de grandeza mais lento que a memória.

### Criar swap numa partição

O processo tem um paralelo directo com a criação de um sistema de ficheiros: onde se usa `mkfs` e `mount`, usa-se `mkswap` e `swapon`.

```bash
# 1. Marcar o tipo de partição como swap no fdisk (comando t, tipo "Linux swap")

# 2. Inicializar a partição como swap
$ sudo mkswap /dev/sdb2
Setting up swapspace version 1, size = 4 GiB
UUID=1a2b3c4d-...

# 3. Activar o swap
$ sudo swapon /dev/sdb2

# 4. Verificar
$ sudo swapon --show
NAME       TYPE      SIZE USED PRIO
/dev/sdb2  partition   4G   0B   -2

$ free -h
              total        used        free      shared  buff/cache   available
Mem:          3.7Gi       1.2Gi       1.8Gi       12Mi       700Mi       2.3Gi
Swap:         4.0Gi          0B       4.0Gi
```

### Criar swap num ficheiro

Quando não há partição disponível, um ficheiro de swap resolve o problema:

```bash
# 1. Criar o ficheiro (4 GiB neste exemplo)
$ sudo dd if=/dev/zero of=/swapfile bs=1M count=4096
# ou, mais rápido:
$ sudo fallocate -l 4G /swapfile

# 2. Definir permissões restritas (obrigatório)
$ sudo chmod 600 /swapfile

# 3. Inicializar como swap
$ sudo mkswap /swapfile

# 4. Activar
$ sudo swapon /swapfile
```

As permissões `600` são obrigatórias: o `mkswap` recusa-se a usar um ficheiro de swap legível por outros utilizadores, porque conteria dados de memória potencialmente sensíveis.

### Tornar o swap permanente

Tal como os sistemas de ficheiros, o swap tem de ser registado no `/etc/fstab` para ser activado no arranque:

```
# Swap em partição, por UUID
UUID=1a2b3c4d-...    swap    swap    defaults    0    0

# Swap em ficheiro
/swapfile            swap    swap    defaults    0    0
```

Após acrescentar a entrada, activar tudo sem reiniciar:

```bash
$ sudo swapon -a
```

### Desactivar swap

```bash
# Desactivar um swap específico
$ sudo swapoff /dev/sdb2

# Desactivar todo o swap
$ sudo swapoff -a
```
---

## 4. LVM: Gestão de Volumes Lógicos

## 4.1 O problema que o LVM resolve

Imagine-se o seguinte cenário, familiar a qualquer administrador. Cria-se uma partição de 50 GB para uma aplicação, calculando generosamente. Seis meses depois, descobre-se que a aplicação usa apenas 10 GB, mas a partição ao lado, que guarda os dados dos utilizadores, está cheia. Com particionamento tradicional, não há saída fácil: as fronteiras das partições estão fixas no disco, e redimensioná-las é uma operação arriscada que frequentemente exige apagar e recriar.

O **LVM** (*Logical Volume Manager*) resolve este problema introduzindo uma camada de abstracção entre os discos físicos e os sistemas de ficheiros. Em vez de criar sistemas de ficheiros directamente sobre partições rígidas, o LVM agrupa o espaço de armazenamento num reservatório flexível de onde se aloca conforme a necessidade, e essa alocação pode ser ajustada dinamicamente, muitas vezes sem sequer desmontar o sistema de ficheiros.

Foi esta flexibilidade que justificou a escolha do LVM na instalação do Capítulo 2. As operações que o LVM torna possíveis incluem redimensionar volumes a quente, mover volumes entre discos físicos sem interrupção, tirar snapshots, e agregar vários discos num único espaço contínuo.

---

## 4.2 Arquitectura: PV, VG e LV

O LVM organiza-se em três camadas, e compreender a relação entre elas é a chave para tudo o resto.

O **PV** (*Physical Volume*) é a base. É um disco ou partição que recebeu uma etiqueta LVM, tornando-o utilizável pelo sistema. Um PV pode ser um disco inteiro, uma partição, ou até um array RAID.

O **VG** (*Volume Group*) é o reservatório. Agrupa um ou mais PVs num único conjunto de espaço. É como juntar vários baldes de água num único tanque: deixa de importar de que balde veio cada litro. O espaço total do VG é a soma do espaço de todos os PVs que o compõem.

O **LV** (*Logical Volume*) é o que se usa. É uma fatia alocada a partir do VG, que se comporta como uma partição: cria-se nele um sistema de ficheiros e monta-se. Mas ao contrário de uma partição, um LV pode ser redimensionado, movido e duplicado com facilidade.

A relação lê-se de baixo para cima: vários discos físicos tornam-se PVs, os PVs juntam-se num VG, e do VG cortam-se os LVs que efectivamente se usam.

```
Discos/partições físicas   →   PV   →   VG   →   LV   →   sistema de ficheiros
   /dev/sdb, /dev/sdc         (etiqueta) (reservatório) (fatia)      (xfs, ext4)
```

Uma nota sobre nomenclatura. Os comandos do LVM começam com um prefixo que indica a camada em que operam: comandos `pv*` manipulam physical volumes, comandos `vg*` manipulam volume groups, e comandos `lv*` manipulam logical volumes. Saber isto torna os dezenas de comandos do LVM imediatamente legíveis.

O VG é subdividido internamente em unidades de alocação chamadas **PE** (*Physical Extents*), tipicamente de 4 MiB. Um LV é na prática um conjunto de PEs. Este detalhe raramente exige atenção directa, mas aparece no output dos comandos de diagnóstico.

---

## 4.3 Construir uma configuração LVM do zero

O exemplo seguinte constrói uma configuração LVM completa a partir de dois discos novos, `/dev/sdb` e `/dev/sdc`, criando um volume group que os agrega e um volume lógico para dados.

### Passo 1: criar os physical volumes

```bash
# Etiquetar os discos como PVs
$ sudo pvcreate /dev/sdb /dev/sdc
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.

# Verificar
$ sudo pvs
  PV         VG   Fmt  Attr PSize    PFree
  /dev/sdb        lvm2 ---  100.00g 100.00g
  /dev/sdc        lvm2 ---  100.00g 100.00g

# Ver detalhe completo
$ sudo pvdisplay
```

Note-se que se usaram os discos inteiros. Também é possível criar PVs sobre partições (`/dev/sdb1`), o que é preferível quando o disco tem outras utilizações, mas para discos dedicados ao LVM usar o disco inteiro é comum.

### Passo 2: criar o volume group

```bash
# Criar um VG chamado "dados_vg" agregando os dois PVs
$ sudo vgcreate dados_vg /dev/sdb /dev/sdc
  Volume group "dados_vg" successfully created

# Verificar
$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  dados_vg    2   0   0 wz--n- 199.99g 199.99g

# Ver detalhe completo
$ sudo vgdisplay dados_vg
```

O VG `dados_vg` tem agora praticamente 200 GB, a soma dos dois discos, disponíveis como um único reservatório.

### Passo 3: criar o volume lógico

```bash
# Criar um LV de 50 GB chamado "web"
$ sudo lvcreate -L 50G -n web dados_vg
  Logical volume "web" created.

# Criar um LV que use todo o espaço livre restante
$ sudo lvcreate -l 100%FREE -n arquivo dados_vg

# Verificar
$ sudo lvs
  LV       VG        Attr       LSize
  web      dados_vg  -wi-a-----  50.00g
  arquivo  dados_vg  -wi-a----- 149.99g
```

A opção `-L` especifica um tamanho absoluto; a opção `-l 100%FREE` usa todo o espaço disponível. O LV fica acessível através de um dispositivo em `/dev/dados_vg/web` ou, equivalentemente, `/dev/mapper/dados_vg-web`.

### Passo 4: criar o sistema de ficheiros e montar

A partir daqui, o LV comporta-se como uma partição normal:

```bash
# Criar o sistema de ficheiros
$ sudo mkfs -t xfs /dev/dados_vg/web

# Montar
$ sudo mkdir -p /mnt/web
$ sudo mount /dev/dados_vg/web /mnt/web
```

Para tornar a montagem permanente, acrescenta-se ao `/etc/fstab` usando o UUID, exactamente como na secção 3:

```bash
$ sudo blkid /dev/dados_vg/web
# Acrescentar a linha correspondente ao fstab
```

---

## 4.4 Expandir volumes a quente

A funcionalidade mais valiosa do LVM no dia-a-dia é a capacidade de aumentar um volume que está a ficar cheio, frequentemente sem sequer o desmontar. É a resposta directa ao problema que abriu esta secção.

Suponha-se que `/mnt/web` está a ficar sem espaço e precisa de mais 30 GB.

### Passo 1: confirmar que há espaço no VG

```bash
$ sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  dados_vg    2   2   0 wz--n- 199.99g  99.99g
```

Há praticamente 100 GB livres no VG, mais que suficiente.

### Passo 2: aumentar o volume lógico

```bash
# Acrescentar 30 GB ao LV
$ sudo lvextend -L +30G /dev/dados_vg/web
  Size of logical volume dados_vg/web changed from 50.00 GiB to 80.00 GiB.
  Logical volume dados_vg/web successfully resized.
```

A opção `-L +30G` acrescenta 30 GB ao tamanho actual. Também se pode usar `-l +100%FREE` para consumir todo o espaço livre do VG.

### Passo 3: aumentar o sistema de ficheiros

Aumentar o LV apenas alarga o contentor. O sistema de ficheiros dentro dele continua com o tamanho antigo e tem de ser expandido separadamente. Aqui, o comando depende do tipo de sistema de ficheiros.

Para **XFS** (o padrão do CentOS), usa-se o `xfs_growfs`, e crucialmente, **o sistema de ficheiros tem de estar montado**:

```bash
# O XFS cresce montado; recebe o ponto de montagem, não o dispositivo
$ sudo xfs_growfs /mnt/web
```

Para **ext4**, usa-se o `resize2fs`:

```bash
$ sudo resize2fs /dev/dados_vg/web
```

Esta é uma diferença conceptual importante entre os dois sistemas de ficheiros: o XFS só pode crescer montado e nunca pode encolher; o ext4 pode ser redimensionado em ambas as direcções, mas encolher exige que esteja desmontado.

Confirmar o resultado:

```bash
$ df -h /mnt/web
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/dados_vg-web     80G  1.1G   79G   2% /mnt/web
```

Todo este processo, do `lvextend` ao `xfs_growfs`, aconteceu com o sistema de ficheiros montado e em uso. Nenhuma interrupção de serviço, nenhum reinício. É esta a promessa do LVM cumprida na prática.

### Aumentar o VG primeiro, se necessário

Se o VG não tiver espaço livre suficiente, primeiro acrescenta-se-lhe um novo disco:

```bash
# Etiquetar o novo disco
$ sudo pvcreate /dev/sdd

# Acrescentá-lo ao VG existente
$ sudo vgextend dados_vg /dev/sdd

# Agora há mais espaço para expandir os LVs
$ sudo vgs
```

Este encadeamento, acrescentar um disco ao VG e depois estender o LV, permite crescer o armazenamento de um servidor indefinidamente, simplesmente adicionando discos.

---

## 4.5 Snapshots

Um **snapshot** é uma imagem congelada de um volume lógico num instante preciso. A sua utilidade principal é permitir uma cópia de segurança consistente: em vez de fazer backup de um sistema de ficheiros que está a ser modificado durante o processo, tira-se um snapshot instantâneo e faz-se o backup a partir dele, com a garantia de que reflecte um único momento coerente.

O LVM implementa snapshots com uma técnica chamada *copy-on-write*. No momento da criação, o snapshot não copia dados: apenas aponta para os mesmos blocos do volume original. Quando um bloco do original é modificado, a versão antiga é primeiro copiada para o espaço do snapshot. Assim, o snapshot preserva a imagem do momento em que foi criado, e só ocupa espaço proporcional à quantidade de alterações feitas depois.

```bash
# Criar um snapshot de 10 GB do volume web
$ sudo lvcreate -L 10G -s -n web_snap /dev/dados_vg/web
  Logical volume "web_snap" created.
```

A opção `-s` indica que é um snapshot, e o volume de origem é especificado no final. O tamanho de 10 GB não é o tamanho do snapshot: é o espaço reservado para guardar os blocos que forem modificados no original enquanto o snapshot existir.

Este ponto é a armadilha principal dos snapshots do LVM.

> **Um snapshot que fica sem espaço corrompe-se irreversivelmente.** O espaço reservado ao snapshot enche-se à medida que o volume original é modificado. Se se esgotar, o LVM deixa de conseguir manter a imagem coerente e o snapshot torna-se inutilizável. Por isso, os snapshots do LVM devem ser de curta duração (criados, usados para o backup, e removidos) ou ter espaço reservado próximo do tamanho do volume de origem. Não são um mecanismo de versionamento permanente.

O uso típico é fazer o backup e remover o snapshot logo de seguida:

```bash
# Montar o snapshot para ler o seu conteúdo
$ sudo mkdir /mnt/snap
$ sudo mount -o ro,nouuid /dev/dados_vg/web_snap /mnt/snap

# Fazer o backup a partir do snapshot
$ sudo tar -czf /backup/web-$(date +%F).tar.gz -C /mnt/snap .

# Desmontar e remover o snapshot
$ sudo umount /mnt/snap
$ sudo lvremove /dev/dados_vg/web_snap
```

A opção `nouuid` na montagem é necessária no XFS porque o snapshot tem o mesmo UUID do original, e o XFS recusa montar dois sistemas de ficheiros com o mesmo UUID sem ela.

Verificar o estado de um snapshot:

```bash
$ sudo lvs
  LV       VG        Attr       LSize  Origin Data%
  web      dados_vg  owi-aos--- 80.00g
  web_snap dados_vg  swi-a-s--- 10.00g web    12.45
```

A coluna `Data%` mostra quanto do espaço do snapshot já foi consumido. Quando se aproxima dos 100%, o snapshot está prestes a corromper-se.

---

## 4.6 Reduzir e remover volumes

### Reduzir um volume

Reduzir um volume é mais delicado do que aumentá-lo, e a ordem das operações inverte-se. Ao aumentar, alarga-se primeiro o contentor (LV) e depois o conteúdo (sistema de ficheiros). Ao reduzir, encolhe-se primeiro o conteúdo e só depois o contentor, caso contrário o LV ficaria mais pequeno que o sistema de ficheiros e os dados seriam truncados.

>**O XFS não pode ser reduzido.** Esta é uma limitação de design do XFS: cresce mas nunca encolhe. Se precisar de reduzir um volume XFS, a única via é fazer backup dos dados, recriar o volume menor, criar um novo sistema de ficheiros e restaurar. Só os sistemas de ficheiros ext podem ser reduzidos.

Para um volume **ext4**, o processo é:

```bash
# 1. Desmontar
$ sudo umount /mnt/web

# 2. Verificar o sistema de ficheiros (obrigatório antes de redimensionar)
$ sudo e2fsck -f /dev/dados_vg/web

# 3. Reduzir o sistema de ficheiros para 40 GB
$ sudo resize2fs /dev/dados_vg/web 40G

# 4. Reduzir o volume lógico para o mesmo tamanho
$ sudo lvreduce -L 40G /dev/dados_vg/web

# 5. Remontar
$ sudo mount /dev/dados_vg/web /mnt/web
```

A ordem é crítica: o sistema de ficheiros é reduzido para 40 GB **antes** de o LV. Inverter estes passos destrói dados.

### Remover volumes

Para desmontar e remover permanentemente um volume lógico:

```bash
# Desmontar primeiro
$ sudo umount /mnt/web

# Remover a entrada correspondente do /etc/fstab

# Remover o volume lógico
$ sudo lvremove /dev/dados_vg/web
Do you really want to remove active logical volume dados_vg/web? [y/n]: y
  Logical volume "web" successfully removed
```

Para remover as camadas superiores, quando já não são necessárias:

```bash
# Remover o volume group (remove também todos os LVs que contém)
$ sudo vgremove dados_vg

# Remover a etiqueta LVM dos discos, devolvendo-os a discos normais
$ sudo pvremove /dev/sdb /dev/sdc
```

## 5. RAID: Redundância e Desempenho

## 5.1 O problema que o RAID resolve

Um disco físico vai falhar. Não é uma questão de "se", mas de "quando". Discos são componentes mecânicos ou electrónicos com uma vida útil finita, e a sua falha é uma das causas mais comuns de perda de dados e de indisponibilidade em servidores. As cópias de segurança protegem contra a perda de dados, mas não contra o tempo de paragem: restaurar um servidor a partir de backups pode demorar horas ou dias, durante os quais o serviço está indisponível.

O **RAID** (*Redundant Array of Inexpensive Disks*) aborda este problema distribuindo ou replicando os dados por vários discos. Consoante a configuração, o RAID pode reduzir a zero o tempo de paragem associado a uma falha de disco, e em muitos casos aumentar também o desempenho. A ideia central é que um conjunto de discos passe a comportar-se como uma única unidade lógica, mais fiável ou mais rápida do que qualquer disco individual.

É importante ser claro sobre uma coisa desde o início: **o RAID não substitui as cópias de segurança.** O RAID protege contra a falha física de um disco, mas não contra a eliminação acidental de ficheiros, corrupção de dados, ataques de ransomware ou desastres que afectem toda a máquina. Um administrador que confie apenas no RAID e negligencie os backups está a pedir problemas.

---

## 5.2 RAID por hardware e por software

Existem duas formas de implementar RAID.

O **RAID por hardware** usa uma controladora dedicada que apresenta ao sistema operativo um conjunto de discos como se fosse um único disco composto. O sistema operativo nem sequer sabe que existe RAID: vê apenas um disco.

O **RAID por software** é implementado pelo próprio sistema operativo, que lê e escreve em vários discos de acordo com as regras do RAID. No Linux, isto faz-se com o `mdadm`.

Poderia parecer que o RAID por hardware é sempre superior, mas a realidade é mais matizada. Como o gargalo de desempenho num sistema RAID são quase sempre os próprios discos, não há razão para assumir que uma implementação por hardware seja mais rápida. O RAID por hardware predominou no passado por duas razões: a falta de suporte no sistema operativo, e a capacidade de algumas controladoras guardarem escritas em memória não volátil, o que melhora o desempenho e protege contra certos problemas de corrupção.

Mas há uma armadilha importante. Muitas das "placas RAID" vendidas para PCs não têm memória não volátil nenhuma: são apenas interfaces SATA com algum software de RAID embutido. As implementações de RAID nas motherboards de PC caem nesta categoria. Nestes casos, é preferível usar o RAID por software do Linux.

Há ainda um risco menos óbvio do RAID por hardware que vale a pena considerar. Se a controladora RAID falhar, pode levar consigo o acesso a todos os discos, mesmo que os discos em si estejam intactos, porque os dados foram escritos num formato específico daquela controladora. Substituir uma controladora avariada por um modelo diferente pode tornar os dados ilegíveis. O RAID por software não tem este problema: os discos podem ser movidos para qualquer máquina Linux e o array reconstruído.

---

## 5.3 Os níveis de RAID e os seus compromissos

O RAID faz essencialmente duas coisas. Pode melhorar o desempenho distribuindo os dados por vários discos (*striping*), permitindo que vários discos trabalhem em simultâneo para servir um único fluxo de dados. E pode replicar dados por vários discos, reduzindo o risco associado à falha de um disco individual.

A replicação assume duas formas. No **espelhamento** (*mirroring*), os blocos de dados são reproduzidos bit a bit em vários discos. Nos **esquemas de paridade**, um ou mais discos contêm uma soma de verificação (checksum) dos blocos dos restantes discos, que permite reconstruir os dados perdidos. O espelhamento é mais rápido mas consome mais espaço; os esquemas de paridade são mais eficientes em espaço mas têm desempenho inferior.

O RAID descreve-se tradicionalmente em "níveis", mas o termo é enganador: níveis mais altos não são necessariamente melhores. São apenas configurações diferentes, e usa-se a que servir a necessidade.

### JBOD (linear)

O JBOD (*Just a Bunch Of Disks*) não é sequer um nível de RAID verdadeiro, mas quase todas as controladoras o implementam. Concatena os endereços de vários discos para criar um único disco virtual maior. Não oferece nem redundância nem ganho de desempenho. Hoje, esta funcionalidade obtém-se melhor com um gestor de volumes lógicos como o LVM, visto na secção anterior.

### RAID 0 (striping)

O **RAID 0** existe estritamente para aumentar o desempenho. Combina dois ou mais discos do mesmo tamanho, mas em vez de os empilhar um a seguir ao outro, distribui os dados alternadamente entre eles. Leituras e escritas sequenciais são assim espalhadas por vários discos, reduzindo os tempos de acesso.

O preço é a fiabilidade. O RAID 0 é **menos fiável** do que discos separados: se qualquer um dos discos falhar, todos os dados do array se perdem, porque cada ficheiro está espalhado por todos os discos. Um array de dois discos tem aproximadamente o dobro da taxa de falha anual de um disco individual.

```
RAID 0 — striping por 2 discos
Disco 1:  [bloco a] [bloco c] [bloco e]
Disco 2:  [bloco b] [bloco d] [bloco f]
Capacidade útil: 100% (soma dos discos)
Tolerância a falhas: NENHUMA
```

Usa-se RAID 0 apenas quando o desempenho é crítico e os dados são descartáveis ou replicados noutro sítio: cache, ficheiros temporários de processamento, dados que podem ser regenerados.

### RAID 1 (mirroring)

O **RAID 1** é o espelhamento. As escritas são duplicadas para dois ou mais discos em simultâneo. Isto torna as escritas ligeiramente mais lentas do que num disco único, mas oferece velocidade de leitura comparável ao RAID 0, porque as leituras podem ser distribuídas pelos vários discos com a mesma informação.

A vantagem é a redundância directa: se um disco falhar, o outro tem uma cópia completa e o sistema continua a funcionar sem interrupção. O custo é o espaço: dois discos de 1 TB em RAID 1 oferecem apenas 1 TB de capacidade útil, porque tudo é guardado em duplicado.

```
RAID 1 — mirroring de 2 discos
Disco 1:  [bloco a] [bloco b] [bloco c]
Disco 2:  [bloco a] [bloco b] [bloco c]   (cópia idêntica)
Capacidade útil: 50%
Tolerância a falhas: 1 disco
```

O RAID 1 é a escolha comum para os discos do sistema operativo, onde a fiabilidade importa mais do que a capacidade.

### RAID 5 (striping com paridade)

O **RAID 5** procura um equilíbrio entre desempenho, capacidade e redundância. Distribui os dados por três ou mais discos como o RAID 0, mas reserva o equivalente a um disco para informação de paridade, distribuída por todos os discos. Se um disco falhar, os dados que continha podem ser reconstruídos a partir da paridade dos restantes.

O custo em espaço é apenas de um disco, independentemente do número total: num array de cinco discos de 1 TB, a capacidade útil é de 4 TB. Isto torna o RAID 5 muito mais eficiente que o espelhamento para arrays grandes.

```
RAID 5 — striping com paridade distribuída, 3 discos
Disco 1:  [bloco a] [bloco c] [paridade]
Disco 2:  [bloco b] [paridade] [bloco e]
Disco 3:  [paridade] [bloco d] [bloco f]
Capacidade útil: (N-1)/N
Tolerância a falhas: 1 disco
```

A desvantagem é o desempenho de escrita: cada escrita exige recalcular e actualizar a paridade, o que introduz uma penalização. O RAID 5 também sofre de uma vulnerabilidade conhecida como *write hole*, em que uma falha de energia a meio de uma escrita pode deixar os dados e a paridade dessincronizados.

### RAID 6 (dupla paridade)

O **RAID 6** é como o RAID 5 mas com dois blocos de paridade em vez de um, tolerando a falha de **dois** discos em simultâneo. Isto responde a um problema real: em arrays grandes, a reconstrução após a falha de um disco pode demorar horas, e durante esse período um segundo disco pode falhar. O RAID 6 protege contra esse cenário, ao custo de sacrificar dois discos de capacidade e de um desempenho de escrita ainda menor.

### RAID 10 (espelho de stripes)

O **RAID 10** (também escrito 1+0) combina os dois mundos: espelha conjuntos de discos em stripe. Obtém o desempenho do striping e a redundância do espelhamento em simultâneo. O custo é o mesmo do RAID 1, metade da capacidade, mas oferece melhor desempenho que o RAID 5 e reconstrução mais rápida após falha. É a escolha comum para bases de dados e cargas de trabalho exigentes onde tanto o desempenho como a fiabilidade importam.

### Comparação

| Nível | Mínimo de discos | Capacidade útil | Tolerância a falhas | Uso típico |
|-------|-----------------|-----------------|--------------------|--------------------|
| RAID 0 | 2 | 100% | Nenhuma | Dados descartáveis, desempenho puro |
| RAID 1 | 2 | 50% | 1 disco | Discos de sistema |
| RAID 5 | 3 | (N-1)/N | 1 disco | Armazenamento geral, bom equilíbrio |
| RAID 6 | 4 | (N-2)/N | 2 discos | Arrays grandes |
| RAID 10 | 4 | 50% | 1 por espelho | Bases de dados, alto desempenho |

---

## 5.4 RAID por software com mdadm

O `mdadm` (*multiple device administration*) é a ferramenta de RAID por software do Linux. Cria e gere arrays que aparecem como dispositivos `/dev/md0`, `/dev/md1`, e assim por diante.

```bash
$ sudo dnf install mdadm -y
```

### Criar um array

O exemplo seguinte cria um array RAID 5 com três discos.

```bash
# Criar o array RAID 5 com 3 discos
$ sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
mdadm: array /dev/md0 started.

# Acompanhar a construção inicial do array
$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdd[3] sdc[1] sdb[0]
      209584128 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      [====>................]  recovery = 21.4% (finish=8.2min speed=95000K/sec)
```

A construção inicial (sincronização) demora, mas o array já é utilizável durante esse processo. O indicador `[UUU]` mostra que os três discos estão presentes e saudáveis (`U` de *up*); um `_` indicaria um disco em falta.

### Criar o sistema de ficheiros e montar

O array comporta-se como um dispositivo de bloco normal:

```bash
$ sudo mkfs -t xfs /dev/md0
$ sudo mkdir -p /mnt/raid
$ sudo mount /dev/md0 /mnt/raid
```

Em muitos cenários, o RAID e o LVM são combinados: cria-se o array com `mdadm`, e depois usa-se `/dev/md0` como physical volume do LVM, obtendo tanto a redundância do RAID como a flexibilidade do LVM.

### Tornar o array permanente

A configuração do array tem de ser guardada para sobreviver a reinícios:

```bash
# Guardar a configuração do array
$ sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf

# Regenerar o initramfs para incluir a configuração (ver secção 1)
$ sudo dracut --force

# Acrescentar a montagem ao /etc/fstab pelo UUID, como na secção 3
```

---

## 5.5 Monitorização e substituição de discos

A redundância do RAID só tem valor se as falhas forem detectadas e corrigidas. Um array RAID 5 que perdeu um disco continua a funcionar, mas perdeu a sua redundância: uma segunda falha destrói tudo. Detectar a primeira falha rapidamente é, portanto, crítico.

### Ver o estado do array

```bash
# Estado resumido de todos os arrays
$ cat /proc/mdstat

# Detalhe completo de um array
$ sudo mdadm --detail /dev/md0
/dev/md0:
        Version : 1.2
     Raid Level : raid5
     Array Size : 209584128 (199.87 GiB)
   Raid Devices : 3
  Total Devices : 3
          State : clean
 Active Devices : 3
Working Devices : 3
 Failed Devices : 0

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       3       8       48        2      active sync   /dev/sdd
```

O campo `State` deve indicar `clean`. Um estado `degraded` significa que um disco falhou e o array está a funcionar sem redundância.

### Monitorização automática

Deixar a verificação do estado para inspecção manual é insuficiente: uma falha pode passar despercebida durante dias. O `mdadm` pode monitorizar os arrays e enviar um email quando algo corre mal:

```bash
# Configurar o destinatário dos alertas em /etc/mdadm.conf
MAILADDR admin@empresa.pt

# Activar o serviço de monitorização
$ sudo systemctl enable --now mdmonitor
```

Este alerta, combinado com o servidor de email configurado no Capítulo 7, garante que a falha de um disco chega ao conhecimento do administrador imediatamente.

### Substituir um disco avariado

Quando um disco falha, o processo de substituição faz-se com o array em funcionamento, sem interrupção de serviço.

```bash
# 1. Marcar o disco como falhado (se ainda não o estiver automaticamente)
$ sudo mdadm --manage /dev/md0 --fail /dev/sdc

# 2. Remover o disco do array
$ sudo mdadm --manage /dev/md0 --remove /dev/sdc

# 3. Substituir fisicamente o disco avariado pelo novo

# 4. Adicionar o novo disco ao array
$ sudo mdadm --manage /dev/md0 --add /dev/sdc

# 5. Acompanhar a reconstrução
$ cat /proc/mdstat
md0 : active raid5 sdc[4] sdd[3] sdb[0]
      209584128 blocks super 1.2 level 5 [3/2] [UU_]
      [=========>...........]  recovery = 47.8% (finish=4.2min speed=98000K/sec)
```

Durante a reconstrução, o array volta a estar completo assim que o processo termina. O indicador passa de `[UU_]` (um disco em falta) de volta a `[UUU]` (todos presentes).

> **A reconstrução é o momento de maior risco de um array RAID.** Durante a reconstrução, todos os discos restantes são lidos intensivamente para recalcular os dados do disco substituído. Se um segundo disco estiver no limite da vida útil, este esforço pode precipitar a sua falha. É por isso que o RAID 6, que tolera duas falhas, é preferido em arrays grandes onde a reconstrução demora horas. E é mais uma razão pela qual o RAID não dispensa backups.

---

## 6. Operações ao Nível de Blocos com dd
 
## 6.1 O que é o dd e por que é perigoso
 
O `dd` é uma ferramenta de cópia de dados ao nível dos blocos brutos. Ao contrário de ferramentas como `cp` ou `rsync`, que copiam ficheiros e compreendem a estrutura do sistema de ficheiros, o `dd` não sabe nem quer saber o que é um ficheiro: copia bytes, um bloco de cada vez, de uma origem para um destino, exactamente como os encontra.
 
Esta indiferença ao conteúdo é o que torna o `dd` tão poderoso e tão perigoso. Poderoso, porque pode copiar qualquer coisa: um disco inteiro incluindo a tabela de partições e o gestor de arranque, uma partição, o registo de arranque, ou uma imagem ISO para uma pen USB. Perigoso, porque escreve exactamente onde lhe mandam, sem verificações, sem confirmação, e sem forma de desfazer. Uma troca entre a origem e o destino, ou uma letra de dispositivo errada, apaga irreversivelmente um disco inteiro num instante.
 
Por esta razão, o `dd` é tratado numa secção própria, separado das outras ferramentas de armazenamento. Merece a atenção isolada que se dá a um instrumento afiado.
 
---
 
## 6.2 Sintaxe
 
A sintaxe do `dd` é distinta da maioria dos comandos Unix. Usa pares de `opção=valor` em vez de flags:
 
```bash
dd if=<origem> of=<destino> bs=<tamanho_bloco> [opções]
```
 
Os parâmetros principais:
 
| Parâmetro | Significado |
|-----------|-------------|
| `if=` | *input file*: a origem de onde ler |
| `of=` | *output file*: o destino para onde escrever |
| `bs=` | *block size*: tamanho de cada bloco lido e escrito |
| `count=` | número de blocos a copiar (limita a cópia) |
| `status=progress` | mostra o progresso durante a operação |
| `conv=` | conversões e opções de comportamento |
 
O parâmetro `bs` afecta o desempenho. Valores pequenos como o defeito de 512 bytes são lentos para cópias grandes; um valor de `4M` (4 mebibytes) é um bom equilíbrio para a maioria das operações de disco.
 
>  **Os parâmetros `if` e `of` são a origem de quase todos os desastres com o `dd`.** `if` é de onde se lê, `of` é para onde se escreve. Trocá-los, ou enganar-se na letra do dispositivo, sobrescreve o alvo errado. Antes de premir Enter num comando `dd` que escreve para um dispositivo, verifique o comando duas vezes, e confirme os dispositivos com `lsblk` imediatamente antes.
 
O `status=progress` merece ser sempre incluído em operações longas. Sem ele, o `dd` não mostra qualquer indicação de progresso e parece estar bloqueado, quando na verdade pode estar a meio de uma cópia de horas.
 
---
 
## 6.3 Clonagem de discos e partições
 
### Clonar um disco inteiro
 
Copiar um disco inteiro para outro, incluindo tabela de partições, gestores de arranque e todos os dados:
 
```bash
# Desmontar ambos os discos primeiro
$ sudo umount /dev/sdb* 2>/dev/null
 
# Clonar sda para sdb na íntegra
$ sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress conv=fsync
```
 
A opção `conv=fsync` garante que todos os dados são efectivamente escritos no disco antes de o `dd` terminar, evitando perda de dados se a energia falhar logo após a cópia.
 
>  **O disco de destino tem de ter pelo menos o mesmo tamanho do de origem.** E todo o conteúdo do destino é apagado. Confirme que `/dev/sdb` é realmente o disco que pretende sobrescrever, e não, por exemplo, o disco do sistema.
 
### Clonar uma partição
 
```bash
$ sudo dd if=/dev/sda1 of=/dev/sdb1 bs=4M status=progress conv=fsync
```
 
### Criar uma imagem de disco num ficheiro
 
Em vez de clonar directamente para outro disco, é frequentemente mais útil criar uma imagem num ficheiro, que pode ser guardada, comprimida e restaurada mais tarde:
 
```bash
# Criar uma imagem do disco
$ sudo dd if=/dev/sda of=/backup/sda.img bs=4M status=progress
 
# Criar uma imagem comprimida, poupando espaço
$ sudo dd if=/dev/sda bs=4M | gzip > /backup/sda.img.gz
 
# Restaurar a partir da imagem
$ sudo dd if=/backup/sda.img of=/dev/sda bs=4M status=progress
 
# Restaurar a partir de uma imagem comprimida
$ gunzip -c /backup/sda.img.gz | sudo dd of=/dev/sda bs=4M status=progress
```
 
### Clonar entre máquinas pela rede
 
Combinando o `dd` com o SSH, é possível clonar um disco directamente para outra máquina, sem armazenamento intermédio:
 
```bash
# Clonar um disco local para um disco na máquina remota
$ sudo dd if=/dev/sda bs=4M | ssh admin@servidor-backup "sudo dd of=/dev/sdb bs=4M"
 
# Guardar uma imagem comprimida num servidor remoto
$ sudo dd if=/dev/sda bs=4M | gzip | ssh admin@backup "cat > /backup/sda-$(date +%F).img.gz"
```
 
---
 
## 6.4 Backup da tabela de partições e do registo de arranque
 
Uma das utilizações mais valiosas do `dd` é preservar as estruturas de arranque de um disco, que ocupam pouco espaço mas cuja perda impede o sistema de arrancar.
 
### Discos MBR
 
Como visto na secção 1, o MBR ocupa os primeiros 512 bytes do disco, contendo o código de arranque e a tabela de partições. Fazer uma cópia dele é trivial:
 
```bash
# Backup do MBR (512 bytes)
$ sudo dd if=/dev/sda of=/backup/mbr.img bs=512 count=1
 
# Restaurar o MBR
$ sudo dd if=/backup/mbr.img of=/dev/sda bs=512 count=1
```
 
O `count=1` limita a cópia a um único bloco de 512 bytes, exactamente o tamanho do MBR.
 
### Discos GPT
 
Aqui há uma diferença importante. O truque do `dd` com 512 bytes aplica-se apenas a discos MBR. Os discos GPT, usados em sistemas UEFI modernos e na instalação do Capítulo 2, guardam a tabela de partições de forma diferente, com uma cópia no início e outra no fim do disco. Para estes, a ferramenta correcta não é o `dd`, mas o `sgdisk` do pacote `gdisk`:
 
```bash
# Backup da tabela de partições GPT
$ sudo sgdisk --backup=/backup/gpt-sda.bin /dev/sda
 
# Restaurar a tabela de partições GPT
$ sudo sgdisk --load-backup=/backup/gpt-sda.bin /dev/sda
```
 
Usar o truque do MBR num disco GPT preservaria apenas parte da estrutura e daria uma falsa sensação de segurança. Convém saber qual o esquema em uso, o que se verifica com `parted /dev/sda print` ou `fdisk -l`.
 
---
 
## 6.5 Outras utilizações
 
### Criar ficheiros de tamanho conhecido
 
O `dd` é útil para criar ficheiros de teste ou reservar espaço, lendo da fonte especial `/dev/zero`, que produz bytes nulos indefinidamente:
 
```bash
# Criar um ficheiro de 100 MB cheio de zeros
$ dd if=/dev/zero of=/tmp/teste.img bs=1M count=100
 
# Criar um ficheiro de swap, como visto na secção 3
$ sudo dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
```
 
### Criar suportes de arranque
 
Escrever uma imagem ISO para uma pen USB, criando um suporte de instalação arrancável:
 
```bash
$ sudo dd if=/caminho/centos-stream.iso of=/dev/sdX bs=4M status=progress && sync
```
 
>  **Confirme que `/dev/sdX` é a pen USB e não um disco do sistema.** Este é um dos erros mais catastróficos e mais comuns com o `dd`: escrever uma ISO sobre o disco do sistema em vez da pen. Execute `lsblk` imediatamente antes para confirmar o dispositivo, e nunca use um nome de dispositivo de memória em vez de o verificar.
 
### Recuperar dados de discos danificados
 
O `dd` normal pára ao encontrar um erro de leitura. Para discos em falha, existe uma variante mais robusta, o `ddrescue`, concebida especificamente para recuperar o máximo de dados possível de um disco moribundo:
 
```bash
$ sudo dnf install ddrescue -y
 
# Recuperar um disco danificado para uma imagem, com log de progresso
$ sudo ddrescue /dev/sda /backup/recuperacao.img /backup/recuperacao.log
```
 
O `ddrescue` tenta primeiro ler as zonas boas rapidamente, e só depois insiste nas zonas problemáticas, mantendo um log que lhe permite retomar de onde parou. É a ferramenta certa quando um disco está a falhar e cada leitura conta.
 
---
 
## 6.6 Precauções ao usar dd
 
O `dd` não tem rede de segurança. Não pede confirmação, não avisa antes de sobrescrever, e não há forma de desfazer. Alguns hábitos que evitam desastres:
 
Verifique sempre os dispositivos com `lsblk` imediatamente antes de correr o comando. A letra de um disco pode ter mudado desde a última vez que verificou.
 
Leia o comando da direita para a esquerda antes de o executar, confirmando que `of=` aponta para o destino correcto. O `of` é o que vai ser destruído.
 
Nunca corra `dd` em dispositivos montados quando o objectivo é clonar. Desmonte primeiro para garantir consistência.
 
Inclua `status=progress` em qualquer operação longa, para saber que está a progredir e não bloqueada.
 
Para operações críticas, considere primeiro fazer um ensaio com um ficheiro de destino em vez de um dispositivo, para confirmar que a lógica do comando está correcta.
 
> **O `dd` é frequentemente apelidado de *disk destroyer* pela comunidade, meio a brincar, meio a sério.** O nome verdadeiro vem de *data duplicator*, mas a alcunha capta uma verdade: é uma das ferramentas mais úteis e mais impiedosas do Linux. Usado com atenção, resolve problemas que nenhuma outra ferramenta resolve. Usado com pressa, transforma um problema pequeno num desastre. A diferença está inteiramente na atenção de quem o executa.
 
