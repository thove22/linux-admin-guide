# Soluções — Laboratório 3: Acesso ao Sistema e Estrutura de Ficheiros

Este documento contém as soluções explicadas do Laboratório 3. Cada solução mostra os comandos, o resultado esperado, e uma explicação do que se aprende. Onde a saída de um comando é, ela própria, a lição, incluem-se capturas de ecrã.

> 💡 **Antes de consultar estas soluções, tente resolver os exercícios sozinho.** O valor da prática está no processo de descobrir a resposta. Use as soluções para compreender o que fez, ou para desbloquear quando estiver realmente preso, não para copiar.

---

## Parte A — Acesso e identidade

### Solução Exercício 2

```bash
# 1. Com a conta normal: acesso negado
$ cat /etc/shadow
cat: /etc/shadow: Permission denied

# 2. Com sudo, sem mudar de identidade
$ sudo cat /etc/shadow
[sudo] password for thoveserv:
root:$y$j9T$daS02nfDatQ3F6714...::0:99999:7:::
bin:*:20186:0:99999:7:::
...
thoveserv:$y$j9T$/n46aLbVqZnEd5f...:0:99999:7:::

# 3. Sessão interactiva de root
$ sudo -i
# whoami
root
#
```

O resultado destes passos é visível na captura seguinte:

> *(imagem: saída dos comandos `cat`, `sudo cat` e `sudo -i` sobre `/etc/shadow`)*

![Transição de identidade com sudo](../../assets/img/sol_02_1-4.png)

##### 4. **Observações sobre a saída:**

- O primeiro `cat /etc/shadow` devolve `Permission denied`: um utilizador comum não pode ler este ficheiro, que contém as passwords cifradas de todas as contas.
- Com `sudo cat`, o conteúdo aparece. Cada linha corresponde a uma conta; o segundo campo é a password cifrada (contas de sistema mostram `*` ou `!`, que significam que não têm login por password).
- Após `sudo -i`, o prompt muda de `thoveserv@localhost:~$` para `root@localhost:~#`. O símbolo `#` no final do prompt é a convenção que indica uma sessão de root, e o `whoami` confirma-o.
---

### Solução Exercício 3

```bash
# 1. Criar o utilizador
$ sudo useradd Anna

# 2. Definir a password inicial
$ sudo passwd Anna
New password:
BAD PASSWORD: The password fails the dictionary check - it is too simplistic/systematic
Retype new password:
passwd: password updated successfully

# 3. Confirmar a criação da conta
$ sudo cat /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
...
thoveserv:x:1000:1000:thoveserv:/home/thoveserv:/bin/bash
Anna:x:1001:1001::/home/Anna:/bin/bash
```

O resultado destes passos é visível na captura seguinte:

> *(imagem: criação da conta Anna com `useradd`, `passwd` e confirmação em `/etc/passwd`)*

![Criação de conta de utilizador](../../assets/img/sol_03_1-3.png)

##### 4. **Observações sobre a saída:**

- O `useradd Anna` cria a conta em silêncio, sem produzir qualquer mensagem quando tem sucesso. A ausência de saída é, em Unix, sinal de que correu bem.
- O `passwd Anna` avisa `BAD PASSWORD: ... too simplistic/systematic` porque a password escolhida falhou a verificação de qualidade do sistema (o módulo `pam_pwquality`). Note-se que, por se estar a executar como root via `sudo`, o aviso **não** impede a definição: o root pode impor uma password fraca, e a mensagem seguinte confirma `password updated successfully`. Um utilizador comum a alterar a sua própria password seria obrigado a escolher outra.
- No `/etc/passwd`, a nova conta aparece na última linha: `Anna:x:1001:1001::/home/Anna:/bin/bash`. Os campos, separados por `:`, são o nome (`Anna`), o `x` que indica que a password está no `/etc/shadow`, o UID (`1001`), o GID (`1001`), o campo de comentário (vazio), o directório pessoal (`/home/Anna`) e a shell (`/bin/bash`). O UID 1001 segue-se ao 1000 do primeiro utilizador criado na instalação.

> 💡 Por convenção, os nomes de utilizador em Linux escrevem-se em minúsculas (`anna` em vez de `Anna`). O sistema aceita maiúsculas, mas a prática habitual, e a que evita confusões em scripts e permissões, é usar apenas minúsculas.

---

## Parte B — Navegação

### Solução Exercício 4

```bash
# 1. Descobrir o directório actual
$ pwd
/home/thoveserv

# 2. Navegar para /etc/systemd com caminho absoluto
$ cd /etc/systemd/

# 3. Subir um nível com caminho relativo
$ cd ..

# 4. Saltar directamente para o directório pessoal
$ cd

# 5. Regressar ao directório anterior
$ cd -
/etc
```

O resultado destes passos é visível na captura seguinte:

> *(imagem: navegação com `pwd`, caminhos absolutos e relativos, `cd` e `cd -`)*

![Navegação na árvore de directórios](../../assets/img/sol_03_4.png)

##### 6. **Observações sobre a saída:**

- O `pwd` confirma o ponto de partida, `/home/thoveserv`, o directório pessoal do utilizador.
- O `cd /etc/systemd/` usa um **caminho absoluto** (começa por `/`): funciona a partir de qualquer ponto do sistema.
- O `cd ..` usa um **caminho relativo**: os dois pontos (`..`) referem-se ao directório pai, subindo de `/etc/systemd` para `/etc`. O prompt actualiza-se em conformidade.
- O `cd` sem qualquer argumento salta directamente para o directório pessoal, equivalente a `cd ~`. Repare que o prompt volta a mostrar `~`.
- O `cd -` regressa ao directório onde se estava imediatamente antes (`/etc`) e imprime esse caminho. É o atalho ideal para alternar entre dois directórios sem digitar o caminho completo.

---
