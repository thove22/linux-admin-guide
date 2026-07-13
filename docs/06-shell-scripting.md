# Capítulo 6 — Automação com Shell Scripting
 
## 1. O Kernel, a Shell e o Terminal
 
### 1.1 O papel do kernel
 
O kernel é o núcleo do sistema operativo Linux. É o componente que arranca primeiro quando o sistema liga, que gere directamente o hardware  CPU, memória, discos, interfaces de rede — e que serve de árbitro entre todos os programas que precisam de aceder a esses recursos. Nenhum programa de utilizador comunica directamente com o hardware: tudo passa pelo kernel, através de um mecanismo controlado chamado chamadas de sistema (*system calls*).
 
Um sistema Linux completo é composto por várias camadas: o kernel em baixo, as bibliotecas e utilitários GNU em cima, e no topo um shell que une tudo e o expõe ao utilizador de forma utilizável. Sem o kernel, nada corre. Sem o shell, o utilizador não tem forma prática de interagir com o que o kernel oferece.
 
### 1.2 O que é o shell
 
O shell é um interpretador de comandos. Recebe texto digitado pelo utilizador (ou lido de um ficheiro), interpreta-o e traduz-o em instruções que o sistema operativo executa. É a camada de ligação entre o ser humano e o kernel.
 
Quando se escreve `ls -l /etc` num terminal, é o shell que analisa essa linha, identifica que `ls` é um programa externo, constrói os argumentos correctos e pede ao kernel que execute o programa. O kernel cria um processo, o programa corre, escreve para o stdout, e o shell apresenta o resultado.
 
O shell não é apenas um executor de comandos isolados. Tem a sua própria linguagem de programação, com variáveis, condicionais, ciclos, funções e substituições. É esta capacidade que transforma o shell numa ferramenta de automação poderosa.
 
### 1.3 Os diferentes tipos de shell
 
Existem várias shells disponíveis em sistemas Linux, cada uma com características próprias:
 
- **Bash** (*Bourne Again SHell*) é o shell por defeito na maioria das distribuições Linux, incluindo o CentOS. É retro-compatível com o shell `sh` original do UNIX e incorpora funcionalidades úteis do Korn Shell e do C Shell. É o shell que cobre este capítulo.
 
- **sh** (*Bourne Shell*) é o shell original do UNIX, desenvolvida por Stephen Bourne nos anos 70. O Bash é essencialmente uma extensão moderna do `sh`. Muitos scripts de sistema são escritos em `sh` puro para garantir portabilidade máxima entre sistemas Unix e Linux.
 
- **zsh** (*Z Shell*) é umo shell moderna com funcionalidades avançadas de auto-completação, temas e personalização. É o shell por defeito no macOS desde 2019 e é popular entre utilizadores que passam muito tempo no terminal.
 
- **ksh** (*Korn Shell*) foi desenvolvida na Bell Labs e é comum em sistemas AIX e Solaris. Introduziu várias funcionalidades que mais tarde foram incorporadas no Bash.
 
- **csh** e **tcsh** (*C Shell* e *TENEX C Shell*) têm uma sintaxe inspirada na linguagem C. São menos comuns em scripting de sistema mas ainda aparecem em ambientes Unix mais antigos.
 
Para saber qual o shell activa na sessão actual:
 
```bash
$ echo $SHELL
/bin/bash
 
$ echo $0
bash
```
 
Para listar todas as shells disponíveis no sistema:
 
```bash
$ cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/zsh
/usr/bin/zsh
```
 
---

## 2. Anatomia de um Script Bash
 
### 2.1 O shebang e a estrutura básica
 
Um script shell é um ficheiro de texto que contém uma sequência de comandos que a shell executa linha a linha, exactamente como se fossem digitados manualmente no terminal. A única diferença é que estão guardados num ficheiro e podem ser executados repetidamente.
 
A primeira linha de qualquer script deve ser o **shebang**, que indica ao kernel qual o interpretador a usar:
 
```bash
#!/bin/bash
```
 
O `#!` é o shebang em si. O que se segue é o caminho absoluto para o interpretador. Quando o kernel encontra um ficheiro executável que começa com `#!`, lê o caminho que se segue e usa esse programa para interpretar o restante do ficheiro.
 
Uma alternativa comum é usar `env` para localizar o interpretador através do `PATH`, o que é mais portável entre sistemas:
 
```bash
#!/usr/bin/env bash
```
 
Após o shebang, qualquer linha que comece com `#` é um comentário e é ignorada pela shell. Os comentários são essenciais para scripts que serão lidos por outros ou revisitados meses depois:
 
```bash
#!/bin/bash
#
# Script: verificar_disco.sh
# Descrição: Verifica o uso de espaço em disco e alerta se acima do limite
# Autor: Carlos Silva
# Data: 2025-10-15
# Uso: ./verificar_disco.sh [limite_percentagem]
```
 
### 2.2 Criar e executar um script
 
Para criar um script funcional, são necessários dois passos além de escrever o código: guardar o ficheiro e definir as permissões de execução.
 
```bash
# Criar o ficheiro
$ vi primeiro_script.sh
 
# Definir permissões: leitura e execução para o dono
$ chmod 700 primeiro_script.sh
 
# Executar
$ ./primeiro_script.sh
```
 
A permissão de leitura é tão necessária quanto a de execução: a shell precisa de ler o ficheiro para o interpretar. `chmod 700` dá ao dono leitura, escrita e execução, sem dar acesso a mais ninguém, o que é adequado para scripts de administração.
 
Um exemplo mínimo e funcional:
 
```bash
#!/bin/bash
# primeiro_script.sh — verificar se o sistema está acessível
 
echo "Sistema: $(hostname)"
echo "Data e hora: $(date)"
echo "Utilizador actual: $(whoami)"
echo "Uptime:"
uptime
```
 
### 2.3 Variáveis
 
As variáveis em Bash armazenam valores que podem ser referenciados e modificados ao longo do script. A atribuição não usa espaços à volta do `=`:
 
```bash
#!/bin/bash
 
# Atribuição — sem espaços à volta do =
SERVIDOR="servidor-01"
PORTA=8080
LIMITE=85
 
# Referência com $
echo "Servidor: $SERVIDOR"
echo "Porta: $PORTA"
 
# Aspas duplas permitem expansão de variáveis
echo "O limite é: ${LIMITE}%"
```
 
A convenção é usar maiúsculas para variáveis de âmbito global ou de configuração, e minúsculas para variáveis locais dentro de funções. As chavetas `${}` são opcionais mas tornam o código mais legível, especialmente quando a variável está adjacente a outros caracteres: `${SERVIDOR}_backup` é mais claro que `$SERVIDOR_backup` (que seria interpretado como a variável `$SERVIDOR_backup`).
 
#### Aspas simples vs aspas duplas
 
Este é um dos pontos de maior confusão em Bash. A regra é directa:
 
**Aspas duplas** permitem expansão de variáveis e substituição de comandos. Tudo o resto é tratado como texto literal:
 
```bash
$ NOME="Carlos"
$ echo "Olá, $NOME"
Olá, Carlos
```
 
**Aspas simples** tratam tudo literalmente. Nenhuma substituição é feita:
 
```bash
$ echo 'Olá, $NOME'
Olá, $NOME
```
 
Esta distinção é crítica em scripts que constroem comandos ou strings dinamicamente. Como regra geral, use sempre aspas duplas à volta de variáveis para evitar problemas com espaços e caracteres especiais:
 
```bash
# Sem aspas — pode falhar se $FICHEIRO tiver espaços no nome
cp $FICHEIRO /backup/
 
# Com aspas duplas — seguro
cp "$FICHEIRO" /backup/
```
 
#### Variáveis de ambiente
 
O sistema pré-define um conjunto de variáveis de ambiente disponíveis em qualquer script:
 
```bash
echo $HOME      # Directório home do utilizador
echo $USER      # Nome do utilizador actual
echo $PATH      # Lista de directórios de procura de executáveis
echo $PWD       # Directório de trabalho actual
echo $HOSTNAME  # Nome do servidor
echo $SHELL     # Shell em uso
```
 
### 2.4 Parâmetros especiais
 
Quando um script é chamado com argumentos, estes ficam disponíveis em variáveis especiais:
 
```bash
#!/bin/bash
# Uso: ./gerir_servico.sh nginx restart
 
echo "Nome do script: $0"
echo "Primeiro argumento: $1"
echo "Segundo argumento: $2"
echo "Todos os argumentos: $@"
echo "Número de argumentos: $#"
echo "PID deste script: $$"
```
 
Se o script for chamado com `./gerir_servico.sh nginx restart`, o output será:
 
```
Nome do script: ./gerir_servico.sh
Primeiro argumento: nginx
Segundo argumento: restart
Todos os argumentos: nginx restart
Número de argumentos: 2
PID deste script: 4821
```
 
A variável `$@` é especialmente útil para passar todos os argumentos recebidos para outro comando dentro do script, sem ter de os enumerar individualmente.
 
O comando `shift` remove o primeiro argumento e desloca os restantes:
 
```bash
#!/bin/bash
echo "Primeiro: $1"
shift
echo "Agora primeiro (era segundo): $1"
```
 
### 2.5 Substituição de comandos
 
A substituição de comandos permite usar o output de um comando como valor numa variável ou como argumento noutro comando. A sintaxe moderna usa `$()`:
 
```bash
#!/bin/bash
 
DATA=$(date '+%Y-%m-%d')
UTILIZADOR=$(whoami)
ESPACO_LIVRE=$(df -h / | awk 'NR==2 {print $4}')
 
echo "Relatório gerado em: $DATA"
echo "Gerado por: $UTILIZADOR"
echo "Espaço livre em /: $ESPACO_LIVRE"
```
 
A sintaxe alternativa com backticks `` ` ` `` é equivalente mas menos legível, especialmente quando aninhada:
 
```bash
# Com backticks (mais antigo, menos legível)
DATA=`date '+%Y-%m-%d'`
 
# Com $() (moderno, preferível)
DATA=$(date '+%Y-%m-%d')
```
 
### 2.6 Códigos de saída
 
Todo o programa que termina em Linux deixa um **código de saída** para o processo que o chamou. Por convenção: `0` significa sucesso, e qualquer valor diferente de zero indica algum tipo de erro ou condição anómala.
 
A variável especial `$?` guarda o código de saída do último comando executado:
 
```bash
$ ls /etc > /dev/null
$ echo $?
0
 
$ ls /directorio_inexistente > /dev/null
ls: cannot access '/directorio_inexistente': No such file or directory
$ echo $?
2
```
 
Nos scripts, é boa prática terminar com um código de saída explícito:
 
```bash
#!/bin/bash
 
if [ ! -f "/etc/nginx/nginx.conf" ]; then
    echo "ERRO: ficheiro de configuração do nginx não encontrado" >&2
    exit 1
fi
 
echo "Configuração encontrada."
exit 0
```
 
A redireccão `>&2` envia a mensagem de erro para o stderr em vez do stdout, o que é a prática correcta para mensagens de diagnóstico.
 
### 2.7 Protecção com set
 
Scripts de produção devem começar com estas opções de protecção logo após o shebang:
 
```bash
#!/bin/bash
set -euo pipefail
```
 
Cada opção tem um efeito específico:
 
`-e` faz o script terminar imediatamente se qualquer comando retornar um código de saída diferente de zero. Sem isto, o script continua a executar comandos após um erro, o que pode causar danos em cascata.
 
`-u` trata variáveis não definidas como um erro. Sem isto, uma variável com typo simplesmente expande para uma string vazia, o que pode produzir comportamentos inesperados e difíceis de diagnosticar.
 
`-o pipefail` garante que uma falha em qualquer parte de uma pipeline faz a pipeline inteira falhar. Sem isto, `comando_falha | outro_comando` retornaria 0 se `outro_comando` terminasse com sucesso, mesmo que `comando_falha` tivesse falhado.
 
---

## 3. Controlo de Fluxo: Condicionais
 
### 3.1 O comando test e os operadores de comparação
 
Antes de escrever condicionais, é necessário perceber como a shell avalia condições. O comando `[` (também chamado `test`) é o mecanismo central de avaliação. É literalmente um programa que recebe uma expressão e retorna 0 (verdadeiro) ou 1 (falso):
 
```bash
$ [ -f /etc/passwd ] && echo "existe" || echo "não existe"
existe
```
 
#### Testes de ficheiros
 
| Operador | Testa se |
|----------|----------|
| `-e ficheiro` | O ficheiro existe (qualquer tipo) |
| `-f ficheiro` | É um ficheiro regular |
| `-d ficheiro` | É um directório |
| `-h ficheiro` | É um link simbólico |
| `-s ficheiro` | Existe e não está vazio |
| `-r ficheiro` | Tem permissão de leitura |
| `-w ficheiro` | Tem permissão de escrita |
| `-x ficheiro` | Tem permissão de execução |
 
#### Testes de strings
 
| Operador | Testa se |
|----------|----------|
| `str1 = str2` | As strings são iguais |
| `str1 != str2` | As strings são diferentes |
| `-z str` | A string está vazia |
| `-n str` | A string não está vazia |
 
#### Testes aritméticos
 
Para comparações numéricas, usa-se operadores específicos em vez do `=`, porque `=` compara strings e `01` seria diferente de `1` como string mas igual como número:
 
| Operador | Retorna verdadeiro quando o primeiro argumento é |
|----------|--------------------------------------------------|
| `-eq` | igual ao segundo |
| `-ne` | diferente do segundo |
| `-lt` | menor que o segundo |
| `-gt` | maior que o segundo |
| `-le` | menor ou igual ao segundo |
| `-ge` | maior ou igual ao segundo |
 
> *(imagem: Tabela 11-3 — Arithmetic Comparison Operators)*
 
### 3.2 if / then / elif / else / fi
 
A estrutura `if` executa um bloco de código se uma condição for verdadeira:
 
```bash
#!/bin/bash
# verificar_servico.sh — verifica se um serviço está activo
 
SERVICO="$1"
 
if [ -z "$SERVICO" ]; then
    echo "Uso: $0 <nome_do_servico>" >&2
    exit 1
fi
 
if systemctl is-active --quiet "$SERVICO"; then
    echo "$SERVICO está a correr."
else
    echo "$SERVICO está parado. A tentar reiniciar..."
    systemctl start "$SERVICO"
    if systemctl is-active --quiet "$SERVICO"; then
        echo "$SERVICO reiniciado com sucesso."
    else
        echo "ERRO: Não foi possível reiniciar $SERVICO." >&2
        exit 1
    fi
fi
```
 
Este script illustra vários pontos importantes. A variável `$1` é sempre colocada em aspas duplas para lidar com o caso em que é vazia. A condição `-z "$SERVICO"` verifica exactamente essa situação. O `if` pode usar qualquer comando como condição, não apenas `[`: `systemctl is-active` retorna 0 se o serviço está activo, o que é exactamente o que `if` precisa.
 
Quando há múltiplas condições alternativas, usa-se `elif`:
 
```bash
#!/bin/bash
 
USO=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
 
if [ "$USO" -ge 90 ]; then
    echo "CRÍTICO: disco a $USO% de capacidade"
    exit 2
elif [ "$USO" -ge 75 ]; then
    echo "AVISO: disco a $USO% de capacidade"
    exit 1
else
    echo "OK: disco a $USO% de capacidade"
    exit 0
fi
```
 
#### Operadores lógicos && e ||
 
Para condições compostas, podem usar-se os operadores `&&` (AND) e `||` (OR):
 
```bash
# Verdadeiro se ambas as condições forem verdadeiras
if [ -f "$FICHEIRO" ] && [ -r "$FICHEIRO" ]; then
    cat "$FICHEIRO"
fi
 
# Verdadeiro se pelo menos uma condição for verdadeira
if [ "$OPCAO" = "sim" ] || [ "$OPCAO" = "s" ]; then
    echo "Confirmado."
fi
```
 
Os operadores `&&` e `||` também funcionam directamente na linha de comandos sem `if`, como forma compacta de escrever condicionais simples:
 
```bash
# Executar backup apenas se o directório existir
[ -d /dados ] && tar -czf /backup/dados.tar.gz /dados
 
# Criar directório se não existir
[ -d /var/log/app ] || mkdir -p /var/log/app
```
 
### 3.3 A estrutura case
 
O `case` é mais adequado do que `if/elif` quando há muitos valores possíveis para a mesma variável. É especialmente limpo para scripts que aceitam argumentos como "start", "stop", "restart":
 
```bash
#!/bin/bash
# gerir_servico.sh — script de gestão de serviços
 
SERVICO="$1"
ACCAO="$2"
 
if [ $# -ne 2 ]; then
    echo "Uso: $0 <servico> <start|stop|restart|status>" >&2
    exit 1
fi
 
case "$ACCAO" in
    start)
        echo "A iniciar $SERVICO..."
        systemctl start "$SERVICO"
        ;;
    stop)
        echo "A parar $SERVICO..."
        systemctl stop "$SERVICO"
        ;;
    restart)
        echo "A reiniciar $SERVICO..."
        systemctl restart "$SERVICO"
        ;;
    status)
        systemctl status "$SERVICO"
        ;;
    reload)
        echo "A recarregar configuração de $SERVICO..."
        systemctl reload "$SERVICO"
        ;;
    *)
        echo "Acção desconhecida: $ACCAO" >&2
        echo "Acções válidas: start, stop, restart, status, reload" >&2
        exit 1
        ;;
esac
```
 
A estrutura `case` funciona da seguinte forma: a variável após `case` é comparada com cada padrão antes do `)`. Quando há correspondência, os comandos até `;;` são executados e o `case` termina, saltando para `esac`. O padrão `*` funciona como caso por defeito, apanhando tudo o que não correspondeu aos padrões anteriores.
 
Os padrões suportam wildcards e alternativas com `|`:
 
```bash
case "$RESPOSTA" in
    sim|s|yes|y|1)
        echo "Confirmado."
        ;;
    nao|n|no|0)
        echo "Cancelado."
        ;;
    *)
        echo "Resposta inválida."
        ;;
esac
```
 
---


