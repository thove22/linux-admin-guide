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

