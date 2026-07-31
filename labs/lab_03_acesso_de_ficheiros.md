# Laboratório 3 — Acesso ao Sistema e Estrutura de Ficheiros

## Objectivos

No fim deste laboratório, deverá ser capaz de:

- Aceder ao sistema por diferentes vias e transitar entre identidades com `su` e `sudo`.
- Navegar na árvore de directórios usando caminhos absolutos e relativos.
- Criar, copiar, mover e remover ficheiros e directórios com segurança.
- Usar wildcards para operar sobre conjuntos de ficheiros.
- Localizar ficheiros com `find` e `locate`.
- Criar e distinguir ligações físicas e simbólicas.
- Instalar, procurar e remover software com `dnf`.

## Pré-requisitos

- Ter lido o Capítulo 3.
- Uma máquina virtual com CentOS Stream. A maior parte dos exercícios corre em modo texto; os exercícios de acesso à consola beneficiam da máquina com interface gráfica.
- Um utilizador comum com privilégios sudo (criado durante a instalação, ver Capítulo 2).


---

## Exercícios

### Parte A  (Acesso e identidade)

#### Exercício 3.1 

Um servidor tem várias consolas virtuais independentes, um recurso útil quando uma sessão fica bloqueada. A partir do ambiente gráfico (ou da consola principal), mude para a segunda consola virtual, inicie sessão com a sua conta de utilizador, execute alguns comandos simples (`whoami`, `pwd`, `date`), termine a sessão, e regresse ao ambiente de partida.

#### Exercício 3.2 

Está a trabalhar com a sua conta normal e precisa de executar uma tarefa administrativa: consultar o ficheiro `/etc/shadow`, que só o root pode ler.

1. Tente ver o ficheiro com a sua conta normal (`cat /etc/shadow`). Observe o resultado.
2. Consulte o mesmo ficheiro usando `sudo`, sem mudar de identidade.
3. Agora abra uma sessão interactiva de root com o método correcto (`sudo -i`), confirme com `whoami` que é root, execute a consulta, e regresse à sua conta.
4. Reflita: porque é que `sudo -i` é preferível a `sudo su` do ponto de vista de auditoria? (A resposta está no Capítulo 3.)

#### Exercício 3.3 

Uma nova colega, a Ana, vai juntar-se à equipa. Prepare a conta dela.

1. Crie o utilizador `ana` (precisará de privilégios administrativos).
2. Defina uma password inicial para a conta dela.
3. Confirme que a conta foi criada, verificando a sua entrada em `/etc/passwd`.

### Parte B  (Navegação)

#### Exercício 3.4 

Sem usar o rato nem interface gráfica, apenas com a linha de comandos:

1. Descubra em que directório se encontra.
2. Navegue até `/etc/systemd` usando um caminho absoluto.
3. A partir daí, suba um nível usando um caminho relativo.
4. Salte directamente para o seu directório pessoal com um único comando curto.
5. Regresse ao directório onde estava imediatamente antes, com um único comando.

### Parte C  (Criar e manipular ficheiros)

Os exercícios seguintes constroem uma pequena estrutura de directórios. Fazem-se em sequência.

#### Exercício 3.5 

1. No seu directório pessoal, crie um directório chamado `projectos`.
2. Dentro de `projectos`, crie nove ficheiros vazios chamados `casa1`, `casa2`, e assim sucessivamente até `casa9`.
3. Imaginando que o directório contém muitos outros ficheiros, encontre um único argumento para o `ls` que liste apenas estes nove ficheiros.

#### Exercício 3.6

Crie a seguinte hierarquia de directórios de uma só vez: `~/projectos/casas/portas/`. Depois, crie os seguintes ficheiros vazios, praticando tanto caminhos absolutos como relativos a partir do seu directório pessoal:

- `~/projectos/casas/moradia.txt`
- `~/projectos/casas/portas/dobradica.txt`
- `~/projectos/exterior/vegetacao/paisagem.txt`

(Note que `exterior/vegetacao/` ainda não existe: terá de a criar.)

#### Exercício 3.7 

Copie os ficheiros `casa1` e `casa5` para o directório `~/projectos/casas/`.

#### Exercício 3.8 

Copie recursivamente o directório `/usr/share/doc/` (ou um subdirectório à sua escolha, se preferir algo mais pequeno) para `~/projectos/`, mantendo as datas e permissões originais dos ficheiros. Que opção do `cp` preserva esses atributos?

#### Exercício 3.9 

Liste recursivamente todo o conteúdo de `~/projectos/`, encaminhando o resultado para o `less`, de modo a poder percorrer a listagem página a página.

#### Exercício 3.10 

Remova os ficheiros `casa6`, `casa7` e `casa8` de uma só vez, sem que o sistema peça confirmação para cada um.

#### Exercício 3.11 

Mova os ficheiros `casa3` e `casa4` para o directório `~/projectos/casas/portas/`.

#### Exercício 3.12 

Remova o directório `~/projectos/casas/portas/` juntamente com tudo o que contém.

> Este exercício usa `rm -r`. Confirme o caminho **antes** de premir Enter. Este é exactamente o tipo de comando que, com um erro de digitação, causa estragos irreversíveis.

### Parte D (Wildcards)

#### Exercício 3.13 

Ainda dentro de `~/projectos/`, e usando wildcards:

1. Liste todos os ficheiros cujo nome começa por `casa`.
2. Liste apenas `casa1`, `casa2` e `casa9` num único comando, usando um conjunto entre parênteses rectos.
3. Antes de apagar seja o que for, use `ls` com um padrão para **prever** que ficheiros seriam afectados por um `rm casa?`. Explique porque este hábito de pré-visualização é importante.

### Parte E (Localizar ficheiros)

#### Exercício 3.14 

1. Use o `find` para localizar o ficheiro de configuração `sshd_config` a partir da raiz do sistema.
2. Use o `find` para encontrar todos os ficheiros terminados em `.conf` dentro de `/etc`. (Atenção à protecção do wildcard.)
3. Instale o `locate` se necessário, actualize o seu índice, e use-o para encontrar o mesmo `sshd_config`. Compare a rapidez das duas abordagens.
4. Explique, com base no que fez, quando faz sentido usar `find` e quando faz sentido usar `locate`.

### Parte F (Ligações)

#### Exercício 3.15 

1. Crie um ficheiro `~/projectos/original.txt` com algum texto lá dentro.
2. Crie uma ligação física para ele chamada `~/projectos/ligacao_fisica.txt`.
3. Crie uma ligação simbólica para ele chamada `~/projectos/ligacao_simbolica.txt`.
4. Use `ls -li` para observar os números de inode dos três ficheiros. Quais partilham o mesmo inode? Porquê?
5. Apague o `original.txt`. Verifique o que acontece ao conteúdo acessível através da ligação física e através da ligação simbólica. Explique a diferença.

### Parte G (Gestão de pacotes)

#### Exercício 3.16 

1. Procure nos repositórios um pacote relacionado com a ferramenta `tree` (que mostra directórios em árvore).
2. Instale-o.
3. Use o `tree` para visualizar a estrutura de `~/projectos/` que construiu ao longo deste laboratório.
4. Descubra a que pacote pertence o comando `tree`, usando o `rpm`.
5. Remova o pacote.

---

## Desafio 

1. Crie a estrutura `~/trabalho/scripts/`, `~/trabalho/dados/` e `~/trabalho/logs/` num único comando.
2. Dentro de `~/trabalho/dados/`, crie dez ficheiros `registo01.log` a `registo10.log`.
3. Usando um único comando com wildcards, liste apenas os registos de `registo01` a `registo05`.
4. Encontre, em todo o sistema, todos os ficheiros de configuração terminados em `.conf` que tenham sido modificados nos últimos 7 dias, e guarde essa lista num ficheiro `~/trabalho/dados/conf_recentes.txt`.
5. Crie uma ligação simbólica em `~/trabalho/logs/` que aponte para o directório real dos logs do sistema (`/var/log`), de forma a poder aceder-lhe comodamente a partir da sua área de trabalho.
6. Instale a ferramenta `tree` (se ainda não estiver instalada) e produza uma listagem em árvore de todo o `~/trabalho/`, guardando o resultado em `~/trabalho/estrutura.txt`.

Verifique cada passo à medida que avança. Se algo não funcionar como esperado, investigue antes de consultar a solução.
