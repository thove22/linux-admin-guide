# Laboratório 4 - Automação com Shell Scripting

## Objectivos

No fim deste laboratório, deverá ser capaz de:

- Escrever, tornar executável e correr scripts Bash com shebang e comentários adequados.
- Usar variáveis, aspas, variáveis de ambiente e substituição de comandos correctamente.
- Ler parâmetros posicionais e entrada interactiva do utilizador.
- Controlar o fluxo com condicionais (`if`/`elif`, `case`) e os operadores de teste.
- Repetir tarefas com ciclos (`for`, `while`, `until`) e controlá-los com `break`/`continue`.
- Organizar código com funções, argumentos, valores de retorno e âmbito `local`.
- Tornar scripts robustos com códigos de saída, `set -euo pipefail` e mensagens de erro em stderr.
- Aumentar a produtividade no terminal com aliases permanentes e gestão do histórico.

## Pré-requisitos

- Ter lido o Capítulo 6.
- Uma máquina virtual com CentOS Stream. Todos os exercícios correm em modo texto, pelo que servem tanto a máquina sem interface gráfica como a máquina com interface gráfica (através de um terminal).
- Um utilizador comum com privilégios sudo (criado durante a instalação, ver Capítulo 2).
- Um editor de texto à sua escolha (`vi`/Vim, do Capítulo 5, é suficiente).


---

## Exercícios

### Parte A  (Primeiros scripts)

#### Exercício 1 

Todo o administrador tem uma pasta pessoal de scripts. Vamos criar o primeiro.

1. Crie o directório `~/bin` (se ainda não existir) e, lá dentro, um script chamado `myownscript`.
2. Comece o ficheiro com o shebang `#!/bin/bash` e inclua comentários a descrever o que o script faz.
3. Faça o script imprimir, lendo a informação em tempo real (com substituição de comandos), algo como:
   ```
   Hoje é Sat Dec 10 15:45:04 WEST 2011.
   Está em /home/joe e o seu host é abc.example.com.
   ```
   (use `date`, `pwd`/`$PWD` e `hostname`).
4. Torne-o executável (`chmod 700`) e corra-o com `./myownscript`.

#### Exercício 2 

1. Num script, defina `NOME="Carlos"` e uma variável numérica `PORTA=8080` (atenção: **sem** espaços à volta do `=`).
2. Imprima `Olá, $NOME` uma vez com **aspas duplas** e outra com **aspas simples**. Explique a diferença no resultado.
3. Mostre a utilidade das chavetas imprimindo `${NOME}_backup` e compare com o que aconteceria com `$NOME_backup`.
4. Imprima três variáveis de ambiente pré-definidas: `$HOME`, `$USER` e `$PATH`.

### Parte B  (Parâmetros e entrada do utilizador)

#### Exercício 3 

1. Crie um script que leia três parâmetros posicionais e os atribua a variáveis `ONE`, `TWO` e `THREE`.
2. Faça-o imprimir exactamente neste formato:
   ```
   Há X parâmetros que incluem Y.
   O primeiro é A, o segundo é B, o terceiro é C.
   ```
   substituindo `X` pelo número de parâmetros (`$#`), `Y` por todos os parâmetros (`$@`), e `A`/`B`/`C` pelo conteúdo de `ONE`/`TWO`/`THREE`.
3. Teste com três argumentos à sua escolha.

#### Exercício 4

1. Crie um script que obtenha a percentagem de uso da raiz `/` .
2. Com `if`/`elif`/`else` e **operadores aritméticos** (`-ge`), imprima:
   - `CRÍTICO` se o uso for ≥ 90 e termine com `exit 2`;
   - `AVISO` se for ≥ 75 e termine com `exit 1`;
   - `OK` caso contrário e termine com `exit 0`.
3. Corra o script e confirme o código de saída com `echo $?`.

#### Exercício 5 

Antes de operar sobre um ficheiro, um bom script verifica se ele existe e está acessível.

1. Crie um script que receba um caminho como `$1`.
2. Usando os testes de ficheiro, verifique e reporte: se **existe** (`-e`), se é um **ficheiro regular** (`-f`) ou um **directório** (`-d`), e se tem permissão de **leitura** (`-r`).
3. Usando a forma compacta `&&`/`||` (sem `if`), imprima numa só linha "legível" ou "não legível" para o caminho recebido.

### Parte D  (Ciclos)

#### Exercício 6

1. Crie um script que percorra a lista `sshd crond rsyslog` num ciclo `for`.
2. Para cada serviço, use `systemctl is-active --quiet` dentro de um `if` e imprima se está `activo` ou `INACTIVO`.
3. Em seguida, crie um segundo ciclo `for` que percorra os ficheiros `/var/log/*.log` (glob) e imprima o tamanho de cada um (`du -sh`). Lembre-se de saltar entradas que não sejam ficheiros com `[ -f "$F" ] || continue`.

### Parte E  (Funções)

#### Exercício 7

1. Crie um script com uma função `registar_log` que receba um **nível** e uma **mensagem** e imprima algo como `[2025-08-03 14:22:01] [AVISO] texto` (use `date` dentro da função).
2. Declare as variáveis internas da função com `local`.
3. Chame a função várias vezes com níveis diferentes (`INFO`, `AVISO`, `ERRO`) a partir da lógica principal.

#### Exercício 8 

1. Escreva uma função `obter_ip_principal` que imprima o IP principal da máquina (dica: `ip route get 1.1.1.1 | awk '{print $7; exit}'`).
2. Capture o resultado numa variável com substituição de comandos (`MEU_IP=$(obter_ip_principal)`) e imprima-o.
3. Explique a diferença entre `return` (que define o código de saída) e "devolver" texto por stdout.

### Parte G  (Scripts de administração aplicados)

#### Exercício 9

1. Escreva um script `painel.sh` que apresente, de forma legível e organizada num ecrã, um resumo do estado do sistema com pelo menos: **CPU** (load average / `uptime`), **memória** (`free -h`), e **disco** (`df -h`).
2. Use funções e cabeçalhos (`echo "=== CPU ==="`) para tornar a saída fácil de ler.
3. Inclua no topo a data e o hostname.

#### Exercício 10 

1. Escreva `backup.sh` que receba um directório de origem como `$1` e produza um arquivo comprimido com data no nome (por exemplo `backup_$(date +%F).tar.gz`) num destino definido. Valide que a origem existe antes de arquivar.
2. Escreva `restore.sh` que receba o arquivo `.tar.gz` como `$1` e o extraia para um destino indicado, validando que o arquivo existe.
3. Teste o par: faça backup de uma pasta de teste, apague-a, e restaure-a. Confirme que o conteúdo foi recuperado.

### Parte H  (Produtividade no terminal)

#### Exercício 11 

1. Adicione ao seu `~/.bashrc` alguns aliases úteis: `ll='ls -lh'`, `..='cd ..'`, e os aliases de segurança `rm='rm -i'`, `cp='cp -i'`, `mv='mv -i'`.
2. Recarregue o ficheiro com `source ~/.bashrc` e teste os aliases.
3. Explique por que razão estes aliases **não** estão disponíveis dentro de um script, e o que se deve usar em vez deles.

---

## Desafio 

Cenário: construa um **painel de administração interactivo** — um único script que reúne tudo o que aprendeu.

1. Escreva um script `admin.sh` que apresente um **menu** repetido (num ciclo `while`) com as seguintes opções:
   1. Ver os 5 processos que mais CPU consomem.
   2. Verificar o uso de disco e alertar se acima de um limite.
   3. Verificar o estado de uma lista de serviços.
   4. Fazer backup de um directório indicado pelo utilizador.
   5. Testar a conectividade de rede (ver ponto 3 abaixo).
   0. Sair.
2. Implemente cada opção como uma **função** separada, e use um `case` para encaminhar a escolha do utilizador (lida com `read`). Valide escolhas inválidas.
3. A função de conectividade deve testar, com `ping -c1`, pelo menos o gateway e um servidor de DNS público (por exemplo `8.8.8.8`), e reportar para cada um se está `acessível` ou `INACESSÍVEL`. (Se quiser ir mais longe, faça-a registar os problemas num ficheiro de log com data.)
4. Comece o script com `set -euo pipefail` e mensagens de erro em stderr, e garanta que o menu só termina quando o utilizador escolhe `0`.

Verifique cada passo à medida que avança. Se algo não funcionar como esperado, investigue antes de consultar a solução.
