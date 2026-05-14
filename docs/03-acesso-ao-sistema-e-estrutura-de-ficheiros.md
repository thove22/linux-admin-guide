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



