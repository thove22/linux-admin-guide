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
 
> *(imagem: ecrã inicial do vi sem ficheiro aberto)*
 
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


