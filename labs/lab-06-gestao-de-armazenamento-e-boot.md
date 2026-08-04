# Laboratório 6 - Gestão de Armazenamento e Arranque do Sistema

## Objectivos

No fim deste laboratório, deverá ser capaz de:

- Identificar o modo de firmware (BIOS/UEFI) e o gestor de arranque, e diagnosticar o arranque com `systemd-analyze` e `journalctl`.
- Gerir o `systemd`: targets (runlevels), listagem e estado de serviços, arranque e recarga.
- Inspeccionar e particionar discos de bloco (`lsblk`, `blkid`, `fdisk`/`parted`).
- Criar sistemas de ficheiros, montá-los e torná-los persistentes com `/etc/fstab` e UUIDs.
- Gerir espaço de swap em partição.
- Construir e manipular volumes com LVM (PV, VG, LV, expansão, snapshots, remoção).
- Configurar RAID por software com `mdadm`.
- Operar ao nível de blocos com `dd` (backup do MBR/tabela de partições e imagens).
- Recuperar a password de root a partir do arranque.

## Pré-requisitos

- Ter lido o Capítulo 8.
- Uma máquina virtual com CentOS Stream.
- **Um disco virtual adicional** (1–2 GB) ligado à VM, que possa apagar por completo. Nos exercícios de particionamento é este o disco de trabalho — **nunca** o disco de sistema. Para os exercícios de RAID, ligue **dois** discos virtuais adicionais.
- Um utilizador comum com privilégios sudo.

---

## Exercícios

### Parte A  (Arranque e systemd)

#### Exercício 1 

1. Determine se o sistema arrancou em modo **UEFI** ou **BIOS** (verifique a existência de `/sys/firmware/efi`).
2. Confirme qual o sistema de init em uso (`ps -p 1 -o comm=` — deverá ser o `systemd`).

#### Exercício 2 

1. Veja o **target por defeito** (`systemctl get-default`) e o(s) target(s) activo(s) (`systemctl list-units --type=target`).
2. Mude temporariamente para o `multi-user.target` e volte ao `graphical.target` (se aplicável) com `systemctl isolate`.
3. Como se altera o target por defeito de forma permanente? A que "runlevels" antigos correspondem o `multi-user` e o `graphical`?

#### Exercício 3 

1. Liste os serviços **activos** (`systemctl list-units --type=service`) e os que **falharam** (`systemctl --failed`).
2. Mostre o estado detalhado do serviço `sshd` (`systemctl status sshd`).
3. **Reinicie** e depois **recarregue** um serviço (por exemplo o `sshd` ou o `chronyd`), e explique a diferença entre `restart` e `reload`.

#### Exercício 4 

1. Veja o tempo total de arranque e a distribuição por fase (`systemd-analyze`).
2. Descubra que unidades demoraram mais (`systemd-analyze blame`) e qual foi a cadeia crítica (`systemd-analyze critical-chain`).
3. Consulte os logs do arranque **actual** (`journalctl -b`) e do arranque **anterior** (`journalctl -b -1`), filtrando por erros (`-p err`).

### Parte B  (Inspecção e particionamento)

#### Exercício 5 

1. Liste os dispositivos de bloco com sistemas de ficheiros e UUIDs (`lsblk -f`) e identifique o disco de sistema e o **disco adicional** de trabalho.
2. Ligue (ou reinsira) o disco adicional e descubra o seu nome observando o kernel em tempo real (`journalctl -kf` ou `dmesg -w`).
3. Veja a tabela de partições actual do disco de trabalho (`sudo fdisk -l /dev/sdX`).

#### Exercício 6 

Sobre o **disco adicional** (confirme o nome primeiro!), com `fdisk` ou `parted`:

1. Apague todas as partições existentes e grave as alterações.
2. Crie três partições: **100 MB Linux**, **200 MB swap** (marque o tipo swap) e **500 MB para LVM** (tipo Linux LVM).
3. Grave e force o kernel a reler a tabela de partições (`sudo partprobe /dev/sdX`), confirmando com `lsblk`.

> Confirme o dispositivo com `lsblk` **antes** de gravar. Escrever a tabela de partições no disco errado destrói o sistema.

### Parte C  (Sistemas de ficheiros, montagem e swap)

#### Exercício 7 

1. Crie um sistema de ficheiros **ext4** na partição Linux de 100 MB (`sudo mkfs.ext4 /dev/sdX1`).
2. Crie o ponto de montagem `/mnt/mypart` e monte lá a partição.
3. Confirme com `df -h` e `lsblk`, e crie um ficheiro de teste dentro do ponto de montagem.

#### Exercício 8 

Torne a montagem persistente:

1. Obtenha o UUID da partição (`blkid /dev/sdX1`).
2. Acrescente a linha correspondente ao `/etc/fstab` (usando o **UUID**, não o nome do dispositivo).
3. Valide a configuração **sem reiniciar** (`sudo umount /mnt/mypart` seguido de `sudo mount -a`) e confirme que montou. Porque é que se usa o UUID em vez de `/dev/sdX1`?

#### Exercício 9 

1. Inicialize a partição de 200 MB como swap (`sudo mkswap /dev/sdX2`).
2. Active-a (`sudo swapon /dev/sdX2`) e confirme que o espaço de swap aumentou (`swapon --show` e `free -h`).
3. Desactive-a de novo com `sudo swapoff /dev/sdX2`.

### Parte D  (LVM)

#### Exercício 10 

Construa uma configuração LVM na partição de 500 MB:

1. Etiquete-a como PV (`sudo pvcreate /dev/sdX3`) e verifique com `pvs`.
2. Crie um volume group chamado `abc` (`sudo vgcreate abc /dev/sdX3`) e verifique com `vgs`.
3. Crie um volume lógico de **200 MB** chamado `data` (`sudo lvcreate -L 200M -n data abc`), formate-o e monte-o temporariamente em `/mnt/test`. Confirme com `lsblk` e `df -h`.

#### Exercício 11 

1. Aumente o volume lógico `data` de 200 MB para **300 MB**, redimensionando também o sistema de ficheiros num único comando (`sudo lvextend -r -L 300M /dev/abc/data`).
2. Confirme o novo tamanho com `lvs` e `df -h` — **sem desmontar**.

#### Exercício 12 

1. Crie um *snapshot* do volume `data` (`sudo lvcreate -s -L 50M -n data_snap /dev/abc/data`).
2. Monte o snapshot noutro ponto para inspeccionar o seu conteúdo (leitura).
3. Desmonte e remova o snapshot (`sudo lvremove /dev/abc/data_snap`). Para que servem os snapshots num backup consistente?

#### Exercício 13 

Desmonte e remova a configuração LVM de forma ordenada:

1. Desmonte `/mnt/test` e remova a respectiva linha do `/etc/fstab` (se a acrescentou).
2. Remova o volume lógico (`lvremove`), depois o volume group (`vgremove`) e por fim a etiqueta de PV (`pvremove`), devolvendo a partição a um estado normal.

### Parte E  (RAID e operações de blocos)

#### Exercício 14 

Com **dois discos virtuais adicionais**, crie um array RAID espelhado:

1. Crie um RAID 1 com os dois discos (`sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdX /dev/sdY`).
2. Acompanhe a construção (`cat /proc/mdstat`) e veja o detalhe (`sudo mdadm --detail /dev/md0`).
3. Crie um sistema de ficheiros no array, monte-o, e guarde a configuração (`mdadm --detail --scan >> /etc/mdadm.conf`).

#### Exercício 15 

Operações ao nível de blocos com o `dd` (com cuidado extremo):

1. Faça um backup do **MBR** do disco de trabalho — os primeiros 512 bytes (`sudo dd if=/dev/sdX of=~/mbr.bak bs=512 count=1`).
2. Crie uma **imagem comprimida** de uma partição pequena (`sudo dd if=/dev/sdX1 bs=4M | gzip > ~/part.img.gz`).
3. Explique porque é que o `dd` é chamado, meio a sério, *disk destroyer*, e que campo do comando é o mais perigoso.

> No `dd`, uma troca entre `if=` (origem) e `of=` (destino) sobrescreve o disco errado sem qualquer aviso. Verifique cada campo duas vezes antes de premir Enter.

### Parte F  (Recuperação e encerramento)

#### Exercício 16 

Recupere uma password de root "esquecida" (faça um *snapshot* da VM primeiro):

1. Reinicie, prima `e` sobre a entrada do GRUB e acrescente `rd.break` ao fim da linha `linux`. Arranque com `Ctrl+X`.
2. Na shell do initramfs, remonte a raiz em escrita (`mount -o remount,rw /sysroot`), mude para ela (`chroot /sysroot`) e defina a nova password (`passwd root`).
3. Marque o sistema para *relabelling* do SELinux (`touch /.autorelabel`), saia e reinicie. Porque é que o `/.autorelabel` é indispensável neste procedimento?

#### Exercício 17 

Termine o laboratório de forma limpa e segura:

1. Desmonte tudo o que montou (`umount`), desactive o swap adicional (`swapoff`), e remova as entradas de teste do `/etc/fstab`.
2. Agende um reinício com aviso aos utilizadores dentro de alguns minutos (`sudo shutdown -r +5 "Fim do laboratório"`), e depois cancele-o (`sudo shutdown -c`). Que comando desligaria a máquina imediatamente?

---

## Desafio 

Cenário: prepare um novo volume de dados resiliente e persistente para o servidor, do disco cru até à montagem automática no arranque.

1. Ligue à VM dois discos virtuais adicionais e confirme os seus nomes com `lsblk`.
2. Crie um array **RAID 1** com os dois discos e acompanhe a sincronização.
3. Sobre o array, construa uma pilha **LVM**: um PV sobre `/dev/md0`, um VG chamado `dados_vg`, e um LV chamado `web` que use metade do espaço.
4. Formate o LV com **XFS**, monte-o em `/srv/web`, e coloque lá um ficheiro de teste.
5. Torne a montagem **persistente** no `/etc/fstab` pelo **UUID**, e valide com `mount -a` (sem reiniciar).
6. Expanda o LV `web` para usar mais 100 MB **a quente** (`lvextend -r`), e confirme que `/srv/web` cresceu sem ser desmontado.
7. Produza um relatório (`~/relatorio_armazenamento.txt`) com o estado final: `lsblk -f`, `pvs`/`vgs`/`lvs`, `cat /proc/mdstat` e a linha correspondente do `/etc/fstab`.

Verifique cada passo à medida que avança. Se algo não funcionar como esperado, investigue antes de consultar a solução.
