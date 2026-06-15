#  Administração de Sistemas Linux

## 1.1 O Editor vi 
 
Aprender a linha de comandos do Linux tem algo em comum com aprender a tocar piano: não é uma coisa que se domina numa tarde. Exige prática,repetição e, acima de tudo, compreensão do instrumento. O editor `vi` (pronuncia-se *"vê-í"*) é um desses instrumentos fundamentais  parte do núcleo duro da tradição Unix. A sua interface é famosa por não ser amigável à primeira vista, mas quem observa um utilizador experiente aeditar ficheiros em `vi` assiste a algo próximo de virtuosismo: mãos que nunca abandonam o teclado, navegação fluida, edições cirúrgicas. Neste capítulo não nos tornamos virtuosos, mas saímos a saber tocar o equivalente às primeiras notas.
> **Nota:** Na maioria das distribuições Linux modernas, incluindo o CentOS, o `vi` invocado no terminal é na realidade o **Vim** (*Vi IMproved*), uma versão melhorada com funcionalidades adicionais como realce de sintaxe e desfazer multilevel. O comportamento base é idêntico ao `vi` original, por isso usaremos os dois termos de forma intercambiável ao longo desta secção.

---
### Por que razão aprender vi?
 
Numa era de editores gráficos intuitivos e editores de texto em modo texto como o `nano`, a questão é legítima. Há, no entanto, razões concretas para não ignorar o `vi`.
 
**Está sempre disponível.** Qualquer sistema Unix ou Linux desde um servidor remoto sem interface gráfica até uma instalação mínima de CentOS — tem `vi`. O `nano` é popular mas não é universal. A norma POSIX, que define os requisitos mínimos de compatibilidade entre sistemas Unix, exige explicitamente a presença do `vi`. Se alguma vez precisar de editar um ficheiro de configuração crítico numa máquina em estado degradado, o `vi` será provavelmente a única ferramenta disponível.
 
**É leve e rápido.** Não há tempo de carregamento, não há dependências gráficas, não há menus para navegar. Para um administrador de sistemas que edita dezenas de ficheiros de configuração por dia em servidores remotos via SSH, `vi` é a diferença entre eficiência e frustração. Um utilizador treinado nunca precisa de levantar os dedos do teclado durante toda a sessão de edição  cada ação, desde a navegação até ao corte de linhas, tem um atalho de teclado.
 
**É o que os administradores sérios usam.** Este motivo é menos técnico mas igualmente real: saber `vi` é uma marca de maturidade no ecossistema Linux. Não é obrigação, mas é uma vantagem.
---

### Iniciar e sair do vi
 
Para lançar o editor, basta invocar o comando `vi` no terminal. Em sessões de administração remotas (como as que estabelecemos no Capítulo 3 via SSH), este será o método padrão de edição de ficheiros:
 
```bash
$ vi
```
 
Deverá aparecer um ecrã semelhante ao seguinte:
 

<figure align="center">
  <img src="../assets/img/vi_init.png" alt="Ecra Inicial do Vi" width="600">
  <figcaption><b>img 1:</b> Ecra Inicial do Vi</figcaption>
</figure>


A primeira coisa a aprender  mesmo antes de escrever uma única letra é como **sair**. Esta é a brincadeira mais antiga da comunidade Linux: o número de pessoas que abriram o `vi` acidentalmente e não conseguiram sair é incalculável. Para sair, em modo de comando (que é o modo inicial, explicado a seguir), escreve-se:
 
```bash
:q
```
 
O símbolo `:` activa a linha de comandos do editor na parte inferior do ecrã, e `q` significa *quit*. Se o editor recusar sair — geralmente porque há alterações não guardadas  força-se a saída descartando as alterações:
 
```bash
:q!
```
 
O ponto de exclamação é a forma de dizer ao `vi` "tenho a certeza, sai mesmo assim".
 
> 💡 **Dica:** Se em algum momento se sentir "perdido" dentro do `vi`  teclas a produzir resultados inesperados, texto a aparecer em lugares errados  prima a tecla `Esc` duas vezes. Isso devolve-o ao modo de comando a partir de qualquer estado.
 
---

### O conceito fundamental: modos de edição
 
Esta é a segunda coisa mais importante a compreender sobre o `vi`, a seguir a saber como sair. O `vi` é um **editor modal**, o que significa que o mesmo teclado se comporta de formas radicalmente diferentes consoante o modo em que o editor se encontra.
 
A maioria dos editores modernos tem apenas um modo: tudo o que se escreve aparece no documento. O `vi` funciona de forma diferente, com dois modos principais:
 
- **Modo de comando** (*command mode*, também chamado *normal mode*): é o modo inicial. Cada tecla é interpretada como um comando de navegação ou manipulação. Premir `j` não escreve a letra "j"  move o cursor uma linha para baixo. Premir `d` não escreve "d"  inicia uma operação de eliminação. É neste modo que se navega, se apagam linhas, se copiam blocos de texto.
- **Modo de inserção** (*insert mode*): é o modo onde o texto é realmente escrito. O editor comporta-se como um editor convencional  as teclas produzem os caracteres correspondentes no documento.
A confusão inicial de quase todos os principiantes com `vi` vem exactamente daqui: entrar no editor, tentar escrever, e produzir um caos de comandos involuntários  porque o editor estava em modo de comando e interpretou cada tecla como uma instrução. Compreender este mecanismo antes de tocar no teclado evita essa frustração.
 
---

### Criar e editar um ficheiro
 
Vamos criar um ficheiro novo passando o nome desejado como argumento ao `vi`. Se o ficheiro não existir, o editor cria-o; se já existir, abre-o para edição.
 
```bash
$ vi servidor_notas.txt
```
 
O ecrã deverá mostrar algo como:
 

<figure align="center">
  <img src="../assets/img/vi_empty.png" alt="Ecra inicial do Vi" width="600">
  <figcaption><b>img 1:</b> ecra inicial do vi </figcaption>
</figure>

 
As linhas com o símbolo `~` (til) à esquerda indicam linhas vazias  linhas que não existem no ficheiro. É a forma do `vi` mostrar que o documento está vazio.
 
**Ainda não escreva nada.** O editor está em modo de comando.

#### Entrar em modo de inserção
 
Para começar a escrever texto, é necessário entrar no modo de inserção. O método mais directo é premir a tecla `i` (*insert*). Após o fazer, deverá aparecer na parte inferior do ecrã a indicação:
 
```
-- INSERT --
```
 
Agora sim, o editor aceita texto normal. Experimente escrever:
 
```
hostname: servidor-01
ip: 192.168.1.10
admin: carlos
estado: producao
```
#### Regressar ao modo de comando
 
Para voltar ao modo de comando (por exemplo, para guardar o ficheiro ou navegar), prima `Esc`. A indicação `-- INSERT --` desaparece da parte inferior do ecrã, confirmando que está novamente em modo de comando.
 
#### Guardar o trabalho
 
Guardar é feito através de um comando ex: uma linha de instrução activada pelo símbolo `:` em modo de comando. Para escrever (guardar) o ficheiro:
 
```
:w
```
 
O editor confirmará com uma mensagem na parte inferior do ecrã semelhante a:
 
```
"servidor_notas.txt" [New] 4L, 67C written
```
 
Onde `4L` indica 4 linhas escritas e `67C` indica 67 caracteres. Para guardar e sair numa única instrução:

```
:wq
```
 
Ou, em alternativa, usando o atalho equivalente em modo de comando:
 
```
ZZ
```
 
---
 
### Navegar sem usar o rato
 
Em modo de comando, o `vi` oferece um sistema completo de navegação por teclado. As setas de direcção funcionam, mas existe uma razão histórica  e prática  para aprender as teclas alternativas.

Quando o `vi` foi criado, muitos terminais de vídeo não tinham teclas de seta. Os programadores que o desenvolveram mapearam a navegação nas teclas `h`, `j`, `k`, `l`  posicionadas sob a mão direita numa disposição QWERTY  de forma a nunca precisar de mover as mãos para navegar. Para quem passa horas num terminal, isto traduz-se em velocidade real.
 
A tabela seguinte detalha as teclas de movimento disponíveis em modo de comando:
 
| Tecla | Movimento |
|-------|-----------|
| `l` ou seta direita | Um caractere para a direita |
| `h` ou seta esquerda | Um caractere para a esquerda |
| `j` ou seta baixo | Uma linha para baixo |
| `k` ou seta cima | Uma linha para cima |
| `0` (zero) | Início da linha actual |
| `^` | Primeiro caractere não-espaço da linha actual |
| `$` | Fim da linha actual |
| `w` | Início da próxima palavra (incluindo pontuação) |
| `W` | Início da próxima palavra (ignorando pontuação) |
| `b` | Início da palavra anterior (incluindo pontuação) |
| `B` | Início da palavra anterior (ignorando pontuação) |
| `Ctrl-F` ou `Page Down` | Avançar uma página |
| `Ctrl-B` ou `Page Up` | Recuar uma página |
| `G` | Última linha do ficheiro |
| `nG` | Ir para a linha número `n` (ex: `5G` vai para a linha 5) |
| `1G` | Primeira linha do ficheiro |

#### Prefixo numérico: multiplicar comandos
 
Uma das funcionalidades mais poderosas do `vi` é a possibilidade de prefixar qualquer comando de movimento com um número, multiplicando o seu efeito. Por exemplo:
 
- `5j` move o cursor **cinco linhas** para baixo
- `3w` avança **três palavras** para a frente
- `10G` vai directamente para a **linha 10** do ficheiro
Este mecanismo torna a navegação em ficheiros longos  como logs de sistema com milhares de linhas ou ficheiros de configuração extensos  substancialmente mais eficiente do que usar as setas repetidamente.
 
---
 
> **Sobre a nomenclatura:** a documentação oficial do Vim utiliza terminologia ligeiramente diferente da nomenclatura histórica do `vi`. O que aqui chamamos "modo de comando" é designado *normal mode* no Vim, e o que chamamos "comandos ex" (os que começam com `:`) são chamados *command mode*. Para os propósitos deste guia mantemos a nomenclatura mais intuitiva, mas é útil saber desta diferença caso consulte a documentação oficial.
 `
---

### Edição de texto
 
A maior parte do trabalho de edição resume-se a um conjunto reduzido de operações: inserir texto, apagar texto, e mover texto de um sítio para outro através de corte e colagem. O `vi` suporta todas estas operações, à sua maneira. Suporta também uma forma limitada de desfazer alterações: estando em modo de comando, a tecla `u` desfaz a última alteração efectuada. Este comando vai ser útil à medida que experimentamos os comandos que se seguem.
 
#### Formas de entrar em modo de inserção
 
Já conhecemos o comando `i` para inserir texto na posição actual do cursor. Mas o `vi` oferece outras formas de entrar em modo de inserção, cada uma adequada a uma situação diferente.
 
Imagine que temos o nosso ficheiro `servidor_notas.txt` com o seguinte conteúdo:
 
```
hostname: servidor-01
ip: 192.168.1.10
admin: carlos
estado: producao
```
 
Se quiséssemos acrescentar texto ao fim da primeira linha, o comando `i` não seria suficiente porque não nos deixa posicionar o cursor depois do último caractere da linha. Para isso existe o comando `a` (*append*). Ao mover o cursor para o fim da linha e premir `a`, o cursor avança um caractere além do fim da linha e o editor entra em modo de inserção, permitindo acrescentar texto a seguir.
 
Mas como quase sempre queremos acrescentar texto ao fim de uma linha, o `vi` tem um atalho ainda mais directo: o comando `A` (maiúsculo). Independentemente de onde o cursor estiver na linha, `A` move-o imediatamente para o final e entra em modo de inserção. Por exemplo, posicionando o cursor no início da primeira linha e premindo `A`, podemos acrescentar informação directamente:
 
```
hostname: servidor-01 [actualizado]
```
 
Prima `Esc` para regressar ao modo de comando.

#### Abrir uma linha nova
 
Outra forma de inserir texto é "abrir" uma linha nova entre duas linhas existentes. O `vi` tem dois comandos para isso:
 
| Comando | Abre uma linha |
|---------|----------------|
| `o` | Abaixo da linha actual |
| `O` | Acima da linha actual |
 
Por exemplo, com o cursor posicionado na segunda linha (`ip: 192.168.1.10`), premir `o` cria uma linha em branco imediatamente abaixo e entra em modo de inserção. Podemos então escrever:
 
```
gateway: 192.168.1.1
```

Prima `Esc` para sair do modo de inserção. Se quisermos desfazer esta adição, basta premir `u` em modo de comando.
 
O comando `O` (maiúsculo) faz o mesmo mas abre a linha acima da posição actual do cursor.
 
#### Apagar texto
 
O `vi` oferece vários comandos para apagar texto, todos organizados em torno de dois caracteres base.
 
O comando `x` apaga o caractere na posição actual do cursor. Pode ser precedido de um número para apagar vários caracteres de uma vez: `3x` apaga o caractere actual e os dois seguintes.
 
O comando `d` é mais versátil. Combinado com qualquer comando de movimento, elimina o texto desde a posição actual até ao destino desse movimento. A tabela seguinte mostra os exemplos mais úteis:

| Comando | O que apaga |
|---------|-------------|
| `x` | O caractere actual |
| `3x` | O caractere actual e os dois seguintes |
| `dd` | A linha actual completa |
| `5dd` | A linha actual e as quatro seguintes |
| `dW` | Da posição actual até ao início da próxima palavra |
| `d$` | Da posição actual até ao fim da linha |
| `d0` | Da posição actual até ao início da linha |
| `d^` | Da posição actual até ao primeiro caractere não-espaço da linha |
| `dG` | Da linha actual até ao fim do ficheiro |
| `d20G` | Da linha actual até à vigésima linha do ficheiro |

Note-se que o `vi` original suporta apenas um nível de desfazer. O Vim (que é o que está instalado no CentOS) suporta múltiplos níveis, o que permite desfazer uma sequência de alterações premindo `u` repetidamente.

#### Cortar, copiar e colar
 
O comando `d` não apaga apenas texto — corta-o. Cada vez que se usa `d`, o texto eliminado é copiado para um buffer interno (equivalente à área de transferência). O comando `p` cola o conteúdo desse buffer após o cursor, e `P` (maiúsculo) cola antes do cursor.
 
O comando `y` (*yank*) copia texto sem o eliminar, usando a mesma lógica combinatória do `d`. A tabela seguinte resume os comandos de cópia disponíveis:


| Comando | O que copia |
|---------|-------------|
| `yy` | A linha actual completa |
| `5yy` | A linha actual e as quatro seguintes |
| `yW` | Da posição actual até ao início da próxima palavra |
| `y$` | Da posição actual até ao fim da linha |
| `y0` | Da posição actual até ao início da linha |
| `y^` | Da posição actual até ao primeiro caractere não-espaço da linha |
| `yG` | Da linha actual até ao fim do ficheiro |
| `y20G` | Da linha actual até à vigésima linha do ficheiro |

Para experimentar: com o cursor na primeira linha do ficheiro, primir `yy` copia essa linha. Mover para a última linha com `G` e premir `p` cola a linha copiada imediatamente abaixo. Premir `P` colaria acima. O comando `u` desfaz a colagem.

### Pesquisa e substituição
 
O `vi` permite mover o cursor para qualquer posição do ficheiro com base numa pesquisa, e também executar substituições automáticas em todo o documento ou numa gama de linhas.
 
#### Pesquisar no ficheiro
 
Para pesquisar uma palavra ou frase, estando em modo de comando, prima `/`. Aparece um `/` na parte inferior do ecrã. Escreva o termo a pesquisar e prima `Enter`. O cursor salta para a próxima ocorrência desse termo no ficheiro.
 
```
/estado
```
 
Para repetir a pesquisa e avançar para a ocorrência seguinte, prima `n`. O mecanismo é idêntico ao que usámos no programa `less` no Capítulo 3.
 
O `vi` permite também pesquisas com expressões regulares, o que torna este mecanismo muito mais poderoso para padrões complexos. As expressões regulares são abordadas em profundidade num capítulo posterior.
 
#### Substituição global
 
Para substituir texto em todo o ficheiro ou numa gama de linhas, o `vi` usa um comando ex com a seguinte estrutura:
 
```
:%s/texto_antigo/texto_novo/g
```
 
Por exemplo, para substituir todas as ocorrências de `producao` por `producção` no ficheiro inteiro:
 
```
:%s/producao/producao/g
```
 
A tabela seguinte explica cada componente deste comando:
 
| Elemento | Significado |
|----------|-------------|
| `:` | Inicia um comando ex |
| `%` | Define o âmbito como todo o ficheiro (equivalente a `1,$`, ou seja, da primeira à última linha). Se omitido, a substituição aplica-se apenas à linha actual |
| `s` | Especifica a operação de substituição (*substitution*) |
| `/producao/producao/` | Define o padrão a procurar e o texto de substituição |
| `g` | Aplica a substituição a todas as ocorrências em cada linha. Se omitido, substitui apenas a primeira ocorrência por linha |
  
É também possível pedir confirmação antes de cada substituição, acrescentando `c` no final:
 
```
:%s/producao/producao/gc
```
 
O editor para em cada ocorrência e apresenta a pergunta:
 
```
replace with producao (y/n/a/q/l/^E/^Y)?
```
 
As opções disponíveis são `y` para confirmar, `n` para saltar, `a` para substituir todas as restantes sem mais confirmação, `q` para cancelar a operação e `l` para substituir esta ocorrência e sair.
 
### Editar múltiplos ficheiros
 
É comum precisar de trabalhar em mais de um ficheiro ao mesmo tempo, seja para fazer alterações em vários ficheiros relacionados, seja para copiar conteúdo de um para outro. O `vi` permite abrir vários ficheiros numa única sessão, passando-os como argumentos no arranque:
 
```bash
$ vi servidor_notas.txt interfaces.txt
```
 
O editor abre com o primeiro ficheiro visível no ecrã. Para navegar entre os ficheiros abertos, usam-se os seguintes comandos ex:
 
```
:bn        (buffer next — avança para o ficheiro seguinte)
:bp        (buffer previous — recua para o ficheiro anterior)
```
 
O `vi` impede a mudança de ficheiro se houver alterações não guardadas no ficheiro actual. Para forçar a mudança descartando as alterações, acrescenta-se `!`:
 
```
:bn!
```
 
Para ver a lista de todos os ficheiros abertos na sessão actual:
 
```
:buffers
```
 
O editor apresenta uma lista na parte inferior do ecrã com os índices de cada buffer:
 
```
  1 %a   "servidor_notas.txt"         line 1
  2      "interfaces.txt"             line 0
Press ENTER or type command to continue
```
 
Para saltar directamente para um buffer pelo seu número:
 
```
:buffer 2
```
 
#### Abrir um ficheiro adicional durante a sessão
 
Não é necessário fechar a sessão para abrir um novo ficheiro. O comando `:e` (*edit*) seguido do nome do ficheiro adiciona-o à sessão actual:
 
```
:e logs_recentes.txt
```
 
O novo ficheiro fica disponível como um buffer adicional, acessível através dos mesmos comandos `:bn`, `:bp` e `:buffer`.
 
#### Copiar conteúdo entre ficheiros
 
Com múltiplos ficheiros abertos, é possível copiar texto de um para outro usando os mesmos comandos de yank e paste que já conhecemos. O processo é directo: copiar o texto no ficheiro de origem com `yy` ou outro comando `y`, mudar para o buffer de destino com `:buffer`, e colar com `p` ou `P`.
 
#### Inserir o conteúdo completo de um ficheiro
 
Para inserir todo o conteúdo de um ficheiro externo na posição actual do cursor, usa-se o comando `:r` (*read*):
 
```
:r outro_ficheiro.txt
```
 
O conteúdo do ficheiro especificado é inserido imediatamente abaixo da linha onde o cursor se encontra. Isto é útil para, por exemplo, inserir um template de configuração num ficheiro que estamos a construir.
 
### Guardar o trabalho: resumo dos métodos
 
Ao longo desta secção usámos `:w` para guardar, mas o `vi` oferece várias variantes que convém conhecer:
 
| Comando | Acção |
|---------|-------|
| `:w` | Guarda o ficheiro sem sair |
| `:wq` | Guarda e sai |
| `ZZ` | Guarda e sai (atalho em modo de comando) |
| `:q!` | Sai sem guardar, descartando alterações |
| `:w nome_alternativo.txt` | Guarda uma cópia com outro nome (equivalente a "Guardar como") |
 
O comando `:w` aceita um nome de ficheiro opcional. Isto permite guardar uma versão alternativa do ficheiro que está a ser editado sem abandonar o ficheiro original. Por exemplo, se estivemos a editar `servidor_notas.txt` e queremos guardar uma cópia de segurança antes de continuar:
 
```
:w servidor_notas_backup.txt
```
 
O ficheiro original continua aberto e activo na sessão. A cópia fica guardada em disco com o novo nome.
 

