# Compreensão do Ecossistema Linux

## 1. O Que é um Sistema Operativo

Antes de estudar o Linux, é necessário compreender o que é, na sua essência, um sistema operativo. Este é o alicerce sobre o qual assenta tudo o resto neste guia, e ignorá-lo seria começar a construir sem fundações.

### O papel do sistema operativo

Um computador é, no fundo, um conjunto de componentes físicos: um processador que executa instruções, memória que guarda dados temporariamente, discos que armazenam informação de forma permanente, e periféricos como teclados, ecrãs e interfaces de rede. Estes componentes, por si só, não sabem cooperar. O processador não sabe intrinsecamente como ler um ficheiro de um disco, e um programa não sabe como enviar dados para a placa de rede.

O **sistema operativo** é o software que resolve este problema. Actua como intermediário entre o hardware e os programas que o utilizador quer executar. Quando uma aplicação precisa de gravar um ficheiro, não comunica directamente com o disco: pede ao sistema operativo, que trata dos detalhes físicos da operação. Esta mediação liberta os programadores de terem de conhecer os pormenores de cada peça de hardware, e permite que o mesmo programa funcione em máquinas com componentes diferentes.

As responsabilidades de um sistema operativo agrupam-se em algumas áreas fundamentais. Gere os **processos**, decidindo qual programa usa o processador em cada momento e por quanto tempo. Gere a **memória**, atribuindo a cada programa o espaço de que necessita e impedindo que um interfira com outro. Gere o **armazenamento**, organizando os dados em ficheiros e directórios. Gere os **dispositivos**, traduzindo pedidos genéricos em comandos específicos para cada peça de hardware. E gere o **acesso**, garantindo que apenas utilizadores autorizados realizam determinadas operações, um tema que atravessa todo este guia.

### O kernel e o espaço de utilizador

Dentro de um sistema operativo, existe uma separação arquitectural fundamental que importa compreender desde já, porque reaparece em vários capítulos deste guia.

O **kernel** (núcleo) é o coração do sistema operativo. É a parte que interage directamente com o hardware e que detém autoridade total sobre a máquina. É o kernel que acede fisicamente à memória, que controla os dispositivos, e que arbitra qual processo é executado em cada instante. Por operar com privilégios totais, o kernel é uma zona protegida: os programas comuns não lhe acedem directamente.

O **espaço de utilizador** (*user space*) é onde correm todos os outros programas: as aplicações, os utilitários, a linha de comandos, os serviços. Estes programas não têm acesso directo ao hardware. Quando precisam de algo que exige privilégios, como ler um ficheiro ou abrir uma ligação de rede, fazem um pedido formal ao kernel através de um mecanismo chamado **chamada de sistema** (*system call*).

Esta separação existe por uma razão de segurança e estabilidade. Se qualquer programa pudesse aceder directamente ao hardware, um erro numa aplicação trivial poderia corromper a memória de outra ou bloquear a máquina inteira. Ao confinar o acesso privilegiado ao kernel, o sistema garante que os programas ficam isolados uns dos outros e que o hardware é acedido de forma controlada. É esta a fronteira que explica, mais adiante neste guia, por que razão certas operações exigem privilégios elevados e por que o processo de arranque carrega o kernel antes de tudo o resto.

### Onde o Linux se encaixa

Feita esta distinção, é possível esclarecer uma confusão comum. Tecnicamente, **"Linux" é o nome do kernel**, não do sistema operativo completo. Foi o kernel que Linus Torvalds criou em 1991, e é o kernel que continua a ser desenvolvido sob esse nome.

Um kernel sozinho, contudo, não é utilizável. Precisa de um conjunto de programas em espaço de utilizador que lhe dêem forma: uma linha de comandos, utilitários para manipular ficheiros, bibliotecas, ferramentas de configuração. Quando se junta o kernel Linux a esse conjunto de software, obtém-se um sistema operativo completo e funcional. É a este conjunto que, no uso corrente, se chama simplesmente "Linux", ainda que a designação mais rigorosa seja frequentemente **GNU/Linux**, por reconhecer o contributo essencial do software do projecto GNU, como veremos na secção sobre a história do sistema.

Para os efeitos práticos deste guia, quando falarmos em "Linux" referimo-nos ao sistema operativo completo: o kernel e todo o ecossistema de programas que o rodeia e que torna possível administrar um servidor.
