#  Administração de Sistemas Linux

## 1. Edição e Manipulação de Texto no Servidor

### 1.1 O Editor vi 
 
Aprender a linha de comandos do Linux tem algo em comum com aprender a tocar piano: não é uma coisa que se domina numa tarde. Exige prática,repetição e, acima de tudo, compreensão do instrumento. O editor `vi` (pronuncia-se *"vê-í"*) é um desses instrumentos fundamentais  parte do núcleo duro da tradição Unix. A sua interface é famosa por não ser amigável à primeira vista, mas quem observa um utilizador experiente aeditar ficheiros em `vi` assiste a algo próximo de virtuosismo: mãos que nunca abandonam o teclado, navegação fluida, edições cirúrgicas. Neste capítulo não nos tornamos virtuosos, mas saímos a saber tocar o equivalente às primeiras notas.
> **Nota:** Na maioria das distribuições Linux modernas, incluindo o CentOS, o `vi` invocado no terminal é na realidade o **Vim** (*Vi IMproved*), uma versão melhorada com funcionalidades adicionais como realce de sintaxe e desfazer multilevel. O comportamento base é idêntico ao `vi` original, por isso usaremos os dois termos de forma intercambiável ao longo desta secção.

---
#### Por que razão aprender vi?
 
Numa era de editores gráficos intuitivos e editores de texto em modo texto como o `nano`, a questão é legítima. Há, no entanto, razões concretas para não ignorar o `vi`.
 
**Está sempre disponível.** Qualquer sistema Unix ou Linux desde um servidor remoto sem interface gráfica até uma instalação mínima de CentOS — tem `vi`. O `nano` é popular mas não é universal. A norma POSIX, que define os requisitos mínimos de compatibilidade entre sistemas Unix, exige explicitamente a presença do `vi`. Se alguma vez precisar de editar um ficheiro de configuração crítico numa máquina em estado degradado, o `vi` será provavelmente a única ferramenta disponível.
 
**É leve e rápido.** Não há tempo de carregamento, não há dependências gráficas, não há menus para navegar. Para um administrador de sistemas que edita dezenas de ficheiros de configuração por dia em servidores remotos via SSH, `vi` é a diferença entre eficiência e frustração. Um utilizador treinado nunca precisa de levantar os dedos do teclado durante toda a sessão de edição  cada ação, desde a navegação até ao corte de linhas, tem um atalho de teclado.
 
**É o que os administradores sérios usam.** Este motivo é menos técnico mas igualmente real: saber `vi` é uma marca de maturidade no ecossistema Linux. Não é obrigação, mas é uma vantagem.
---

#### Iniciar e sair do vi
 
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

#### O conceito fundamental: modos de edição
 
Esta é a segunda coisa mais importante a compreender sobre o `vi`, a seguir a saber como sair. O `vi` é um **editor modal**, o que significa que o mesmo teclado se comporta de formas radicalmente diferentes consoante o modo em que o editor se encontra.
 
A maioria dos editores modernos tem apenas um modo: tudo o que se escreve aparece no documento. O `vi` funciona de forma diferente, com dois modos principais:
 
- **Modo de comando** (*command mode*, também chamado *normal mode*): é o modo inicial. Cada tecla é interpretada como um comando de navegação ou manipulação. Premir `j` não escreve a letra "j"  move o cursor uma linha para baixo. Premir `d` não escreve "d"  inicia uma operação de eliminação. É neste modo que se navega, se apagam linhas, se copiam blocos de texto.
- **Modo de inserção** (*insert mode*): é o modo onde o texto é realmente escrito. O editor comporta-se como um editor convencional  as teclas produzem os caracteres correspondentes no documento.
A confusão inicial de quase todos os principiantes com `vi` vem exactamente daqui: entrar no editor, tentar escrever, e produzir um caos de comandos involuntários  porque o editor estava em modo de comando e interpretou cada tecla como uma instrução. Compreender este mecanismo antes de tocar no teclado evita essa frustração.
 
---

#### Criar e editar um ficheiro
 
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

##### Entrar em modo de inserção
 
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
##### Regressar ao modo de comando
 
Para voltar ao modo de comando (por exemplo, para guardar o ficheiro ou navegar), prima `Esc`. A indicação `-- INSERT --` desaparece da parte inferior do ecrã, confirmando que está novamente em modo de comando.
 
##### Guardar o trabalho
 
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
 
#### Navegar sem usar o rato
 
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

##### Prefixo numérico: multiplicar comandos
 
Uma das funcionalidades mais poderosas do `vi` é a possibilidade de prefixar qualquer comando de movimento com um número, multiplicando o seu efeito. Por exemplo:
 
- `5j` move o cursor **cinco linhas** para baixo
- `3w` avança **três palavras** para a frente
- `10G` vai directamente para a **linha 10** do ficheiro
Este mecanismo torna a navegação em ficheiros longos  como logs de sistema com milhares de linhas ou ficheiros de configuração extensos  substancialmente mais eficiente do que usar as setas repetidamente.
 
---
 
> **Sobre a nomenclatura:** a documentação oficial do Vim utiliza terminologia ligeiramente diferente da nomenclatura histórica do `vi`. O que aqui chamamos "modo de comando" é designado *normal mode* no Vim, e o que chamamos "comandos ex" (os que começam com `:`) são chamados *command mode*. Para os propósitos deste guia mantemos a nomenclatura mais intuitiva, mas é útil saber desta diferença caso consulte a documentação oficial.
 `
---

#### Edição de texto
 
A maior parte do trabalho de edição resume-se a um conjunto reduzido de operações: inserir texto, apagar texto, e mover texto de um sítio para outro através de corte e colagem. O `vi` suporta todas estas operações, à sua maneira. Suporta também uma forma limitada de desfazer alterações: estando em modo de comando, a tecla `u` desfaz a última alteração efectuada. Este comando vai ser útil à medida que experimentamos os comandos que se seguem.
 
##### Formas de entrar em modo de inserção
 
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

##### Abrir uma linha nova
 
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
 
##### Apagar texto
 
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

##### Cortar, copiar e colar
 
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

#### Pesquisa e substituição
 
O `vi` permite mover o cursor para qualquer posição do ficheiro com base numa pesquisa, e também executar substituições automáticas em todo o documento ou numa gama de linhas.
 
##### Pesquisar no ficheiro
 
Para pesquisar uma palavra ou frase, estando em modo de comando, prima `/`. Aparece um `/` na parte inferior do ecrã. Escreva o termo a pesquisar e prima `Enter`. O cursor salta para a próxima ocorrência desse termo no ficheiro.
 
```
/estado
```
 
Para repetir a pesquisa e avançar para a ocorrência seguinte, prima `n`. O mecanismo é idêntico ao que usámos no programa `less` no Capítulo 3.
 
O `vi` permite também pesquisas com expressões regulares, o que torna este mecanismo muito mais poderoso para padrões complexos. As expressões regulares são abordadas em profundidade num capítulo posterior.
 
##### Substituição global
 
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
 
#### Editar múltiplos ficheiros
 
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
 
##### Abrir um ficheiro adicional durante a sessão
 
Não é necessário fechar a sessão para abrir um novo ficheiro. O comando `:e` (*edit*) seguido do nome do ficheiro adiciona-o à sessão actual:
 
```
:e logs_recentes.txt
```
 
O novo ficheiro fica disponível como um buffer adicional, acessível através dos mesmos comandos `:bn`, `:bp` e `:buffer`.
 
##### Copiar conteúdo entre ficheiros
 
Com múltiplos ficheiros abertos, é possível copiar texto de um para outro usando os mesmos comandos de yank e paste que já conhecemos. O processo é directo: copiar o texto no ficheiro de origem com `yy` ou outro comando `y`, mudar para o buffer de destino com `:buffer`, e colar com `p` ou `P`.
 
##### Inserir o conteúdo completo de um ficheiro
 
Para inserir todo o conteúdo de um ficheiro externo na posição actual do cursor, usa-se o comando `:r` (*read*):
 
```
:r outro_ficheiro.txt
```
 
O conteúdo do ficheiro especificado é inserido imediatamente abaixo da linha onde o cursor se encontra. Isto é útil para, por exemplo, inserir um template de configuração num ficheiro que estamos a construir.
 
#### Guardar o trabalho: resumo dos métodos
 
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
 
## 1.2 sed — O Editor de Fluxo (Stream Editor)

O `vi` é a ferramenta certa quando precisamos de abrir um ficheiro, navegar pelo seu conteúdo e fazer edições de forma interactiva. Mas existe uma classe inteira de tarefas de administração onde abrir um editor interactivo seria a abordagem errada: quando precisamos de alterar a mesma linha em quarenta ficheiros de configuração, quando queremos extrair apenas as linhas de erro de um log com milhares de entradas, ou quando estamos a construir um script que deve modificar ficheiros automaticamente sem qualquer intervenção humana.

É precisamente para estas situações que existe o `sed` — o *stream editor*, ou editor de fluxo. Ao contrário do `vi`, o `sed` não é interactivo. Não se abre um ficheiro, não se navega, não se digita texto directamente. Em vez disso, passa-se uma instrução de edição como argumento na linha de comandos, e o `sed` aplica essa instrução ao texto linha por linha, escrevendo o resultado no *standard output*. O ficheiro original permanece intacto, a menos que se instruza explicitamente o contrário.

Esta diferença de filosofia é fundamental: o `vi` é um editor para humanos lerem e modificarem texto de forma deliberada; o `sed` é uma ferramenta para automatizar transformações de texto em escala, integrável em pipelines e scripts.

---

### Como o sed processa texto

Para usar o `sed` com confiança, é necessário compreender o ciclo que ele executa internamente para cada linha do ficheiro de entrada:

1. Lê uma linha do ficheiro de entrada e remove o caractere de nova linha no final.
2. Coloca essa linha num espaço de trabalho interno chamado **pattern space** (espaço de padrão).
3. Aplica sequencialmente todos os comandos que foram passados como argumento.
4. Escreve o conteúdo do *pattern space* no *standard output*, adicionando novamente o caractere de nova linha.
5. Limpa o *pattern space* e repete o processo para a linha seguinte.

Este ciclo continua até ao fim do ficheiro. O resultado é um fluxo de texto transformado que pode ser visualizado no terminal, redireccionado para um novo ficheiro, ou passado como entrada para outro comando através de uma pipeline.

Existe ainda um segundo espaço de armazenamento chamado **hold space** (espaço de retenção), que funciona como uma memória auxiliar persistente entre ciclos. O *hold space* não é apagado entre linhas, o que permite operações mais avançadas que envolvem guardar e recuperar conteúdo de linhas anteriores. Para a administração diária de sistemas, o *pattern space* é o que importa na maioria das situações.

---

### Sintaxe geral

A estrutura de um comando `sed` segue sempre a mesma lógica:

```bash
sed [opções] 'instrução' ficheiro
```

Ou, quando utilizado numa pipeline:

```bash
comando_anterior | sed [opções] 'instrução'
```

As opções mais importantes são:

| Opção | Efeito |
|-------|--------|
| `-n` | Suprime a saída automática. O `sed` só imprime o que for explicitamente instruído a imprimir com o comando `p`. |
| `-i` | Edita o ficheiro directamente (*in-place*), sem necessidade de redireccionamento. |
| `-i.bak` | Edita o ficheiro directamente mas guarda uma cópia de segurança com a extensão `.bak` antes de o modificar. |
| `-e` | Permite encadear múltiplas instruções num único comando. |

---

#### O comando de substituição: s

A operação mais utilizada do `sed` é a substituição, e a sua sintaxe é praticamente idêntica à que já vimos no `vi`:

```bash
sed 's/padrão/substituto/flags' ficheiro
```

Para tornar os exemplos concretos, vamos trabalhar com um ficheiro de configuração de rede típico de um servidor CentOS. Criamos o ficheiro com o seguinte conteúdo:

```bash
$ cat /etc/rede/servidor.conf
hostname=servidor-antigo
ip=10.0.0.5
porta=80
admin=root
estado=inactivo
```

##### Substituição básica

Para substituir a primeira ocorrência de um padrão em cada linha:

```bash
$ sed 's/inactivo/activo/' /etc/rede/servidor.conf
hostname=servidor-antigo
ip=10.0.0.5
porta=80
admin=root
estado=activo
```

O ficheiro original não foi alterado. O resultado foi apenas impresso no terminal. Para verificar:

```bash
$ cat /etc/rede/servidor.conf
hostname=servidor-antigo
ip=10.0.0.5
porta=80
admin=root
estado=inactivo
```

O conteúdo mantém-se igual. O `sed` processou e mostrou o resultado transformado, mas não tocou no ficheiro.

##### O flag global: g

Por defeito, o `sed` substitui apenas a **primeira** ocorrência do padrão em cada linha. Se uma linha contiver o padrão múltiplas vezes e quisermos substituir todas as ocorrências, é necessário adicionar o flag `g` (*global*):

```bash
$ echo "o servidor-antigo substituiu o servidor-antigo temporário" | sed 's/servidor-antigo/servidor-01/g'
o servidor-01 substituiu o servidor-01 temporário
```

Sem o `g`, apenas a primeira ocorrência seria substituída:

```bash
$ echo "o servidor-antigo substituiu o servidor-antigo temporário" | sed 's/servidor-antigo/servidor-01/'
o servidor-01 substituiu o servidor-antigo temporário
```

Este é um dos erros mais comuns com `sed`: esquecer o `g` e depois descobrir que algumas instâncias do padrão não foram alteradas.

##### Alterar delimitadores

O separador utilizado na instrução de substituição é por convenção a barra `/`, mas pode ser qualquer caractere. Isto é útil quando o padrão ou o substituto contém barras, como acontece com caminhos de ficheiros, evitando sequências de escape confusas:

```bash
# Com barras — requer escape para os / do caminho
$ sed 's/\/etc\/rede\//\/etc\/network\//g' ficheiro.conf

# Com outro delimitador — muito mais legível
$ sed 's|/etc/rede/|/etc/network/|g' ficheiro.conf
```

Ambos os comandos produzem o mesmo resultado. A segunda forma é claramente mais fácil de ler e de manter.

---

### Endereçamento: aplicar comandos a linhas específicas

Por defeito, os comandos do `sed` aplicam-se a todas as linhas do ficheiro. O **endereçamento** permite restringir a actuação a linhas específicas, usando número de linha ou padrão.

#### Por número de linha

Para aplicar uma substituição apenas na linha 2:

```bash
$ sed '2s/10.0.0.5/192.168.1.10/' /etc/rede/servidor.conf
hostname=servidor-antigo
ip=192.168.1.10
porta=80
admin=root
estado=inactivo
```

Para aplicar a um intervalo de linhas, usa-se a notação `início,fim`:

```bash
$ sed '1,3s/=/: /' /etc/rede/servidor.conf
hostname: servidor-antigo
ip: 10.0.0.5
porta: 80
admin=root
estado=inactivo
```

O caractere `$` representa a última linha do ficheiro, o que é útil quando não sabemos o número exacto de linhas:

```bash
$ sed '3,$s/=/: /' /etc/rede/servidor.conf
```

Este comando aplica a substituição da linha 3 até ao fim do ficheiro.

#### Por padrão

Em vez de números, pode-se usar um padrão para seleccionar as linhas onde o comando actua. O padrão é delimitado por barras:

```bash
$ sed '/admin/s/root/carlos/' /etc/rede/servidor.conf
hostname=servidor-antigo
ip=10.0.0.5
porta=80
admin=carlos
estado=inactivo
```

O `sed` aplicou a substituição apenas na linha que continha a palavra `admin`. Todas as outras permaneceram intactas.

---

##### Apagar linhas: d

O comando `d` (*delete*) elimina as linhas endereçadas do fluxo de saída. O ficheiro original não é modificado, mas as linhas eliminadas não aparecem no resultado.

Para remover uma linha específica pelo número:

```bash
$ sed '4d' /etc/rede/servidor.conf
hostname=servidor-antigo
ip=10.0.0.5
porta=80
estado=inactivo
```

A linha 4 (`admin=root`) foi removida da saída. Para remover todas as linhas que contêm um determinado padrão, como linhas de comentário (que em muitos ficheiros de configuração começam com `#`):

```bash
$ sed '/^#/d' ficheiro.conf
```

O símbolo `^` é uma âncora de expressão regular que significa "início da linha". Portanto, `/^#/` significa "linhas que começam com `#`". Esta é uma das operações mais úteis para limpar ficheiros de configuração e ver apenas as directivas activas, sem o ruído dos comentários.

Para remover linhas em branco:

```bash
$ sed '/^$/d' ficheiro.conf
```

`/^$/` significa "linhas onde o início é imediatamente seguido pelo fim", ou seja, linhas vazias.

---

##### Imprimir linhas selectivamente: p com -n

O comando `p` (*print*) imprime o conteúdo do *pattern space*. Sozinho, produz linhas duplicadas porque o `sed` já imprime cada linha por defeito. A sua utilidade real surge em combinação com a opção `-n`, que suprime a saída automática:

```bash
$ sed -n '2,3p' /etc/rede/servidor.conf
ip=10.0.0.5
porta=80
```

Este comando imprime apenas as linhas 2 e 3. Com padrão:

```bash
$ sed -n '/estado/p' /etc/rede/servidor.conf
estado=inactivo
```

A combinação `-n` com `p` transforma o `sed` numa ferramenta de extracção precisa, semelhante ao `grep` mas com controlo adicional sobre intervalos de linhas.

---

##### Edição directa de ficheiros: -i

Até agora, todos os exemplos produziram saída no terminal sem modificar os ficheiros originais. Quando queremos que as alterações sejam escritas directamente no ficheiro, usa-se a opção `-i` (*in-place*):

```bash
$ sed -i 's/servidor-antigo/servidor-01/g' /etc/rede/servidor.conf
```

Após este comando, o ficheiro foi modificado permanentemente. Esta é uma operação irreversível, portanto a prática recomendada em administração de sistemas é sempre testar o comando sem o `-i` primeiro, verificar que o resultado é o esperado, e só depois aplicar com `-i`.

Para situações em que o risco é elevado, como editar ficheiros de configuração de serviços críticos, a opção `-i.bak` cria automaticamente uma cópia de segurança antes de modificar o ficheiro:

```bash
$ sed -i.bak 's/servidor-antigo/servidor-01/g' /etc/rede/servidor.conf
```

Após este comando existem dois ficheiros: `servidor.conf` com o conteúdo modificado, e `servidor.conf.bak` com o conteúdo original. Se algo correr mal, a recuperação é imediata.

> ⚠️ **Aviso:** A opção `-i` sobrescreve o ficheiro sem confirmação. Nunca a use directamente sobre ficheiros críticos do sistema sem antes testar o comando e garantir que existe uma cópia de segurança.

---

##### Encadear múltiplos comandos: -e

A opção `-e` permite aplicar várias instruções de edição em sequência, numa única invocação do `sed`. Cada `-e` introduz uma instrução adicional:

```bash
$ sed -e 's/servidor-antigo/servidor-01/g' \
      -e 's/inactivo/activo/' \
      -e 's/porta=80/porta=443/' \
      /etc/rede/servidor.conf
hostname=servidor-01
ip=10.0.0.5
porta=443
admin=root
estado=activo
```

As três substituições são aplicadas em sequência, linha a linha, num único passo sobre o ficheiro. Isto é equivalente a passar o ficheiro três vezes por `sed` encadeados numa pipeline, mas mais eficiente.

---

### sed em pipelines

Sendo uma ferramenta de fluxo, o `sed` integra-se naturalmente nas pipelines que aprendemos no Capítulo 4. Pode receber dados de outros comandos e passar o resultado transformado para o comando seguinte:

```bash
$ cat /var/log/messages | sed '/^#/d' | sed '/^$/d' | grep "ERROR"
```

Ou de forma mais concisa, usando `-e`:

```bash
$ cat /var/log/messages | sed -e '/^#/d' -e '/^$/d' | grep "ERROR"
```

Este pipeline remove linhas de comentário e linhas vazias do log antes de procurar por erros, reduzindo o ruído e tornando a análise mais directa.

---

### Caso prático: actualizar um endereço IP em ficheiros de configuração

Um cenário real que ilustra bem o poder do `sed` em administração de sistemas: o servidor de base de dados foi migrado de `10.0.1.5` para `10.0.1.50`, e esse endereço está referenciado em vários ficheiros de configuração. Em vez de abrir cada ficheiro manualmente com o `vi`, um único comando resolve o problema:

```bash
$ sed -i.bak 's/10\.0\.1\.5/10.0.1.50/g' /etc/app/db.conf /etc/app/cache.conf /etc/cron.d/backup
```

Note-se a diferença entre o padrão (`10\.0\.1\.5`) e o substituto (`10.0.1.50`): no padrão, os pontos são escapados com `\.` porque na linguagem de expressões regulares o ponto é um caractere especial que corresponde a qualquer caractere. Sem o escape, o padrão `10.0.1.5` corresponderia a cadeias como `10X0Y1Z5`, o que não é o pretendido. No texto de substituição, o ponto é tratado literalmente e não precisa de escape.

---

#### sed vs vi: quando usar cada um

Estas duas ferramentas não são concorrentes, são complementares. A escolha entre elas depende do contexto:

| Situação | Ferramenta adequada |
|----------|-------------------|
| Editar um ficheiro de configuração manualmente | `vi` |
| Substituir um valor em 20 ficheiros de uma vez | `sed` |
| Navegar por um ficheiro longo à procura de algo | `vi` |
| Remover comentários de um ficheiro num script | `sed` |
| Reestruturar o conteúdo de um ficheiro com lógica complexa | `vi` |
| Transformar a saída de um comando antes de a guardar | `sed` em pipeline |

> 💡 **Para aprofundar:** O `sed` suporta expressões regulares completas nos seus padrões, o que multiplica significativamente o seu poder expressivo. As expressões regulares são abordadas em detalhe no Capítulo 6, que cobre automação com shell scripting. Depois de as dominar, os comandos `sed` que hoje parecem complexos tornam-se naturais.
