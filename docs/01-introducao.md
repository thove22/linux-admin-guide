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

## 2. Linux no Panorama Tecnológico Actual

O Linux é um dos avanços tecnológicos mais importantes das últimas décadas. Para além do seu impacto no crescimento da internet e do seu papel como tecnologia que viabiliza uma vasta gama de dispositivos, o desenvolvimento do Linux tornou-se um modelo de como projectos colaborativos podem ultrapassar aquilo que indivíduos ou empresas isoladas conseguem alcançar sozinhos.

### A omnipresença invisível do Linux

A percepção comum de que o Linux é um sistema de nicho, usado apenas por entusiastas, não podia estar mais afastada da realidade. O Linux é, na prática, a infraestrutura invisível sobre a qual assenta grande parte da computação moderna. A maioria das pessoas usa Linux todos os dias sem o saber, porque não o encontra num ecrã de computador pessoal, mas sim nas máquinas que sustentam os serviços de que dependem.

Esta é uma distinção que vale a pena interiorizar antes de começar a estudar administração de sistemas: aprender Linux não é aprender um sistema marginal, é aprender a operar a tecnologia que faz funcionar a internet, a computação empresarial e uma parte substancial dos dispositivos que nos rodeiam.

### Servidores e a internet

É no domínio dos servidores que o Linux verdadeiramente domina. A maioria dos sítios da internet cujos servidores são identificáveis corre sobre Linux, e entre os sítios de maior tráfego do mundo essa proporção é ainda mais expressiva.

As razões são práticas. O Linux é estável, podendo funcionar durante anos sem reinício. É eficiente, aproveitando bem os recursos do hardware. É seguro, com um modelo de permissões robusto e correcções de segurança rápidas. E é livre, o que elimina custos de licenciamento que, à escala de milhares de servidores, seriam proibitivos.

Muitas das maiores empresas tecnológicas construíram a sua infraestrutura sobre Linux. A Google corre a sua tecnologia de pesquisa em incontáveis servidores Linux. Uma configuração clássica de serviço web é a chamada *stack* LAMP (Linux, servidor web Apache, base de dados MySQL e linguagem de scripting PHP), toda ela composta por software de código aberto, e que este guia aborda em vários capítulos.

### Cloud, contentores e supercomputação

A ascensão da computação em nuvem consolidou ainda mais o domínio do Linux. A esmagadora maioria das cargas de trabalho nas grandes plataformas de nuvem pública, como a AWS, a Azure e a Google Cloud, corre sobre Linux. As tecnologias de contentores e de orquestração que definem a arquitectura de software moderna nasceram e vivem no ecossistema Linux.

No extremo superior do desempenho, o domínio é absoluto: a totalidade dos 500 supercomputadores mais rápidos do mundo corre Linux, e assim tem sido de forma ininterrupta há vários anos. Quando o objectivo é extrair o máximo desempenho de hardware de ponta, não há alternativa séria ao Linux, precisamente por permitir a personalização ao nível do kernel que essas máquinas exigem.

### Android e dispositivos móveis

Talvez o exemplo mais surpreendente da omnipresença do Linux esteja no bolso de milhares de milhões de pessoas. O **Android**, o sistema operativo móvel mais usado do mundo, é construído sobre o kernel Linux. Cada telemóvel Android é, na sua base, um sistema Linux, ainda que a camada visível ao utilizador seja bastante diferente da de um servidor.

Isto significa que o Linux é, por larga margem, o sistema operativo mais implantado do planeta, não pela via do computador pessoal, mas pela via do dispositivo móvel.

### Sistemas embebidos e da Internet das Coisas

Para além dos telemóveis, o Linux está presente numa infinidade de dispositivos que raramente associamos a um computador: routers domésticos, televisores inteligentes, sistemas de navegação automóvel, equipamento industrial, e uma boa parte dos dispositivos da Internet das Coisas (IoT). A sua adaptabilidade, o facto de ser livre, e a possibilidade de o ajustar a hardware limitado tornam-no a escolha natural para estes sistemas embebidos.

### O ambiente de trabalho (desktop)

É no computador pessoal que a presença do Linux é menor. No ambiente de trabalho, os sistemas da Microsoft e da Apple continuam largamente dominantes, e a quota do Linux, embora em crescimento lento e constante, permanece modesta. Isto não reflecte uma limitação técnica, mas sim factores históricos de mercado, pré-instalação de outros sistemas nos computadores vendidos, e hábitos dos utilizadores. Para quem administra sistemas, no entanto, é no servidor, e não no desktop, que o Linux importa verdadeiramente.

### O que distingue o Linux: software livre e de código aberto

Compreender por que o Linux se tornou tão dominante exige compreender aquilo que o distingue fundamentalmente dos sistemas da Microsoft e da Apple. Estes últimos são sistemas **proprietários**, o que tem implicações concretas:

- Não é possível ver o código usado para criar o sistema operativo.
- Não é possível alterar o sistema aos seus níveis mais básicos, nem usá-lo como base para construir outro sistema.
- Não é possível inspeccionar o código para encontrar erros, estudar vulnerabilidades de segurança, ou simplesmente aprender o que o código faz.
- Pode não ser possível integrar software próprio no sistema se os seus criadores não expuserem as interfaces necessárias.

O Linux é o oposto: é **software livre e de código aberto** (*open source*). O seu código está disponível para qualquer pessoa ver, estudar, modificar e redistribuir.

Alguém poderá objectar: "que me importa isso? Não sou programador, não quero ver nem alterar como o meu sistema é construído." A resposta é que o facto de outros poderem usar software livre como bem entendem foi precisamente o motor que impulsionou o crescimento explosivo da internet, dos telemóveis, de dispositivos especializados e de centenas de empresas tecnológicas. O software livre reduziu drasticamente os custos da computação e permitiu uma explosão de inovação. As empresas que hoje assentam a sua infraestrutura em Linux precisam, cada vez mais, de pessoas com as competências para operar esses sistemas, e é para essas competências que este guia contribui.

Fica, porém, uma pergunta por responder: como é que um sistema tão poderoso e flexível veio também a ser livre? Para o compreender, é preciso conhecer a sua origem, e a história peculiar do movimento do software livre que conduziu ao Linux, tema da secção seguinte.
