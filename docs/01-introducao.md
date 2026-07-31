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

## 3. Unix e Linux: História e Evolução

Compreender o Linux exige compreender de onde veio, porque a sua história explica quase tudo o que o define hoje: a arquitectura, a filosofia, e sobretudo o facto de ser livre. A pergunta deixada na secção anterior, como é que um sistema tão poderoso veio a ser gratuito, só tem resposta através desta história.

Tudo começa com uma mensagem publicada por um estudante finlandês num grupo de discussão da internet, a 25 de Agosto de 1991:

> Olá a todos os que usam minix. Estou a fazer um sistema operativo (livre) (apenas um passatempo, não vai ser grande e profissional como o gnu) para clones AT 386(486). [...] Quaisquer sugestões são bem-vindas, mas não prometo que as implemente.
>
> Linus Torvalds

Aquilo que Torvalds descreveu modestamente como um passatempo viria a tornar-se um dos sistemas operativos mais importantes do mundo. Mas para entender o que ele estava a construir, e porque conseguiu construí-lo, é preciso recuar mais de duas décadas, até às origens do Unix.

### As origens do Unix nos Bell Labs

Praticamente todos os sistemas operativos modernos, com a notável excepção do Microsoft Windows, descendem do Unix, criado originalmente pela AT&T. Isto inclui o macOS da Apple e o próprio Linux. Para apreciar como um sistema livre pôde ser modelado a partir de um sistema proprietário da AT&T, é preciso compreender a cultura em que o Unix nasceu.

Nos anos 60 e 70, a AT&T era, sem rival, a companhia telefónica dos Estados Unidos. Esse monopólio dava-lhe o luxo de financiar investigação pura, e o centro dessa investigação eram os Bell Laboratories, em Nova Jérsia. Foi aí que, após o fracasso de um projecto chamado Multics por volta de 1969, dois funcionários, **Ken Thompson** e **Dennis Ritchie**, decidiram criar por conta própria um sistema operativo que oferecesse um ambiente melhor para desenvolver software.

O Unix não nasceu de uma necessidade de mercado, mas do desejo de ultrapassar os obstáculos que dificultavam a produção de programas. E nasceu num ambiente comunitário, de partilha livre de código, tanto dentro como fora dos Bell Labs. Esta cultura de colaboração é decisiva: foi ela que permitiu o desenvolvimento rápido de um Unix de alta qualidade, e foi também ela que, mais tarde, a AT&T teria dificuldade em travar.

### A filosofia Unix

O Unix assentou em alguns elementos fundamentais que sobrevivem, praticamente inalterados, no Linux de hoje. Reconhecê-los agora ajuda a compreender decisões de design que aparecem ao longo de todo este guia.

**O sistema de ficheiros hierárquico.** O Unix organizava os ficheiros numa estrutura de directórios e subdirectórios, uma árvore como a que o Capítulo 3 descreve em detalhe. Mais do que isso, simplificou o acesso a dispositivos complexos como discos e fitas representando-os também como ficheiros dentro dessa árvore. É a origem da filosofia "tudo é um ficheiro", já apresentada neste guia.

**A redirecção de entrada e saída e os pipes.** O Unix permitia direccionar a saída de um comando para um ficheiro, com o símbolo `>`, e mais tarde encadear comandos, ligando a saída de um à entrada do seguinte com o símbolo `|`. Um exemplo clássico:

```bash
    $ cat ficheiro1 ficheiro2 | sort | pr | lpr
```

Este comando junta dois ficheiros, ordena as linhas alfabeticamente, formata o texto para impressão, e envia o resultado para a impressora. Cada programa faz uma coisa e passa o resultado ao seguinte. Esta modularidade foi revolucionária: permitiu que muito código fosse desenvolvido por muitas pessoas diferentes, cada uma criando pequenas utilidades que se combinavam com as existentes. É a mesma lógica que o Capítulo 6 explora ao tratar de shell scripting.

**A portabilidade e a linguagem C.** Para que o Unix pudesse funcionar em máquinas diferentes, era necessária uma linguagem de programação de alto nível. Para esse fim, **Brian Kernighan** e **Dennis Ritchie** criaram a linguagem **C**, e em 1973 o Unix foi reescrito em C. Isto foi determinante: em vez de estar preso a um tipo de hardware, o Unix passou a poder ser adaptado a outras máquinas alterando apenas os controladores, sem tocar nos programas. Ainda hoje, a linguagem C é a principal usada para criar o kernel do Unix e do Linux.

Desta época sobrevive também o formato das **man pages**, a documentação em linha que continua a ser a forma primária de documentar comandos no Unix e no Linux, e que este guia recomenda consultar sempre que necessário.

### A comercialização e o nascimento do BSD

Antes de 1984, a AT&T estava proibida de vender sistemas informáticos, por causa do seu monopólio telefónico. Em consequência, o código-fonte do Unix era licenciado às universidades por um valor simbólico. Não existia um Unix comercial pronto a usar: recebia-se o código e compilava-se.

Foi a partir deste código que, na Universidade da Califórnia em Berkeley, nasceu a primeira grande variante do Unix, a **BSD** (*Berkeley Software Distribution*). Durante quase uma década, as versões da BSD e dos Bell Labs seguiram caminhos diferentes: a BSD manteve o espírito de partilha livre de código, enquanto a AT&T começou a orientar o Unix para a comercialização.

Com a divisão da AT&T em 1984, a empresa ficou finalmente livre para vender o Unix. Esta nova postura de propriedade começou a corroer o espírito de contribuição aberta. Surgiram processos judiciais para proteger o código e a marca registada Unix. E foi precisamente esta nova versão restritiva do Unix que, em 1984, deu origem à organização que abriria caminho ao Linux.

### O projecto GNU e a Free Software Foundation

Em 1984, **Richard Stallman** iniciou o projecto **GNU**, um nome recursivo que significa *GNU is Not UNIX*. Como projecto da **Free Software Foundation** (FSF), o GNU tinha um objectivo ambicioso: reescrever todo o sistema operativo Unix de raiz, de forma que pudesse ser distribuído livremente.

Reescrever milhões de linhas de código poderia parecer impossível para uma ou duas pessoas, mas ao distribuir o esforço por dezenas ou centenas de programadores, o projecto tornou-se viável. E aqui a filosofia Unix revelou-se uma vantagem decisiva: como o Unix era feito de peças pequenas com interfaces bem conhecidas, o trabalho de as recriar podia ser facilmente dividido. Mais ainda, como todos podiam ver o código produzido, código mal escrito era corrigido ou substituído rapidamente. Em alguns casos, o resultado era melhor do que o Unix original.

O projecto GNU produziu com sucesso milhares de utilitários. Mas falhou em produzir uma peça crítica: o **kernel**. As suas tentativas de construir um kernel livre não foram bem-sucedidas, e faltava assim a peça central para completar um sistema operativo inteiramente livre.

#### Software livre e a licença GPL

Para definir claramente como o software livre deveria ser tratado, o projecto GNU criou a **GPL** (*GNU Public License*), a licença mais conhecida do mundo do software livre e a que cobre o próprio kernel do Linux. Os seus princípios fundamentais são simples:

- O autor original mantém os direitos sobre o seu software.
- Qualquer pessoa pode usar, alterar e redistribuir o software livremente, mas tem de incluir o código-fonte, ou torná-lo facilmente acessível.
- Mesmo que alguém reempacote e revenda o software, o acordo original mantém-se, garantindo que todos os futuros destinatários têm a mesma liberdade de alterar o código.

Este último princípio é o que distingue a GPL: as melhorias que se fazem ao código têm de ser disponibilizadas aos outros. Assim, toda a comunidade beneficia do trabalho de cada um, tal como cada um beneficiou do trabalho anterior dos outros.

Com o tempo, o termo "software livre" foi sendo acompanhado pelo termo "código aberto" (*open source*). As duas designações refletem ênfases ligeiramente diferentes, a primeira preferida pela Free Software Foundation, a segunda pela Open Source Initiative, e há quem use a sigla FOSS (*Free and Open Source Software*) para abranger ambas.

### Linus constrói a peça que faltava

Estava montado o cenário. O projecto GNU tinha construído quase todo um sistema operativo livre, mas faltava-lhe o kernel. A BSD tinha um sistema quase completo, mas em 1992 foi atingida por um processo judicial da AT&T que, embora viesse a ser abandonado, gerou incerteza suficiente para travar o seu ímpeto. Muitos começaram a procurar uma alternativa livre. O momento estava maduro para um estudante finlandês que trabalhava no seu próprio kernel.

**Linus Torvalds** começou a trabalhar no Linux em 1991, enquanto estudante na Universidade de Helsínquia. Queria um kernel semelhante ao Unix para poder usar em casa o mesmo tipo de sistema que usava na universidade. Usava então o Minix, mas queria ir além dos seus limites.

Ainda que Torvalds tenha inicialmente afirmado que o Linux fora escrito para o processador 386 e provavelmente não seria portável, outros insistiram numa abordagem mais portável e contribuíram para ela. Em Outubro de 1991, a versão 0.02 já tinha grande parte do código original reescrito em C, o que abriu caminho à sua adaptação a outras máquinas.

O kernel Linux era a última peça, e a mais importante, para completar um sistema operativo inteiramente livre ao abrigo da GPL. Foi por isso que, quando se começaram a montar distribuições juntando o kernel de Torvalds aos utilitários do GNU, o nome que ficou foi "Linux". Algumas distribuições, como o Debian, chamam-se a si próprias **GNU/Linux**, reconhecendo o contributo essencial do projecto GNU, uma questão sobre a qual alguns membros do projecto GNU insistem com razão.

### Unix e Linux: a relação e a questão da certificação

Compreendida a história, a relação entre Unix e Linux torna-se clara, e desfaz uma confusão comum.

O Linux **não contém código do Unix original**. Foi escrito de raiz, recriando o comportamento do Unix a partir das suas interfaces públicas e dos padrões que as descreviam, nomeadamente o **POSIX** (*Portable Operating System Interface*) e a definição de interface do UNIX System V. Curiosamente, o próprio Torvalds pediu uma cópia do padrão POSIX nos primeiros tempos do projecto. Provavelmente ninguém na AT&T esperava que alguém conseguisse escrever um clone do Unix a partir dessas interfaces, sem usar uma única linha do código original.

Daí a distinção precisa: o Linux não é Unix, é um sistema **semelhante ao Unix** (*Unix-like*). "Unix" é hoje uma marca registada, gerida pelo The Open Group, e um sistema só pode chamar-se oficialmente "Unix" se passar por um processo de certificação dispendioso. O Linux nunca foi certificado dessa forma, e não precisa de o ser: reflecte uma combinação de conformidade com os padrões POSIX, System V e BSD, sem carregar o rótulo formal.

### Do desenvolvimento à actualidade

Hoje, o desenvolvimento do Linux é coordenado pela **Linux Foundation**, uma organização sem fins lucrativos que emprega o próprio Linus Torvalds e cuja lista de patrocinadores é um retrato das maiores empresas de tecnologia do mundo: IBM, Red Hat, SUSE, Oracle, Intel, Google e muitas outras. A sua função é proteger e acelerar o crescimento do Linux, fornecendo proteção legal e padrões de desenvolvimento.

O facto de gigantes tecnológicos, muitos deles concorrentes ferozes entre si, financiarem em conjunto o desenvolvimento de um sistema operativo livre é, talvez, a melhor prova de que o modelo colaborativo que começou nos Bell Labs há mais de meio século não só funcionou, como se tornou parte essencial da infraestrutura tecnológica mundial.

Compreendida esta história, resta perceber como é que este kernel único dá origem à variedade de sistemas Linux que existem, e qual deles é o objecto deste guia. É o tema da secção seguinte, sobre distribuições.

## 4. Distribuições Linux

### O conceito de distribuição

Como vimos, o Linux é, tecnicamente, apenas o kernel. Ter código-fonte espalhado pela internet, pronto a ser compilado, funcionava bem para técnicos experientes, mas os utilizadores comuns precisavam de uma forma mais simples de montar um sistema Linux funcional. Foi para responder a essa necessidade que surgiram as **distribuições**.

Uma distribuição Linux é o conjunto de tudo o que é necessário para criar um sistema operativo completo e utilizável, mais os procedimentos para o instalar e pôr a funcionar. Antes de o kernel ser útil, é preciso reunir muito mais: os comandos básicos (os utilitários do GNU), os serviços que se querem oferecer (como acesso remoto ou um servidor web), possivelmente uma interface gráfica e aplicações, e uma forma de instalar tudo isso no disco.

Uma distribuição faz precisamente essa reunião. Pega no kernel, junta-lhe o software escolhido, empacota tudo de forma coerente, e fornece um instalador e ferramentas de gestão. É por isso que existem muitas distribuições diferentes: todas partilham o mesmo kernel Linux, mas diferem no software que incluem, na filosofia que seguem, e no público a que se destinam.

### As grandes famílias

Ao longo dos anos surgiram centenas de distribuições, muitas para necessidades específicas. Mas duas tornaram-se a base de onde descendem quase todas as outras: **Red Hat** e **Debian**.

**A família Red Hat.** Quando o Red Hat Linux apareceu no final dos anos 90, tornou-se rapidamente a distribuição mais popular, por várias razões que ainda hoje são relevantes. Introduziu o formato de pacotes **RPM**, que, ao contrário dos simples arquivos comprimidos, guardava informação sobre cada pacote (versão, autor, ficheiros de configuração, dependências) numa base de dados local, tornando fácil descobrir o que estava instalado, actualizá-lo ou removê-lo. Este é o mesmo sistema RPM que o Capítulo 3 detalha. Trouxe também um instalador simples e ferramentas gráficas de administração. Desta família descendem o Red Hat Enterprise Linux, o Fedora, o CentOS, o Rocky Linux, o AlmaLinux e o Oracle Linux, entre outros. É a família a que pertence o sistema usado neste guia.

**A família Debian.** O Debian foi, tal como o Red Hat, uma distribuição pioneira que se destacou na gestão de pacotes, usando o formato **deb** e as suas próprias ferramentas. Ganhou reputação de grande estabilidade. Dele descendem mais de uma centena de distribuições, sendo a mais bem-sucedida o **Ubuntu**, que acrescentou ao Debian um instalador gráfico simples e ferramentas fáceis de usar, focando-se em trazer novos utilizadores para o Linux.

Existem outras famílias e distribuições independentes de nota, como o **SUSE** (de origem alemã, forte no mercado empresarial europeu) e o **Arch Linux** (orientado para utilizadores avançados que querem controlo total sobre o seu sistema).

### Como escolher uma distribuição

Não existe a "melhor" distribuição em abstracto; existe a mais adequada a um propósito. Um utilizador doméstico que quer facilidade pode preferir o Ubuntu. Um programador que quer a tecnologia mais recente pode escolher o Fedora. Uma empresa que precisa de estabilidade e suporte comercial opta pelo Red Hat Enterprise Linux.

Para quem aprende administração de sistemas com vista a uma carreira, a escolha deve orientar-se pelo mercado de trabalho. E aqui, a família Red Hat, e em particular o RHEL e os sistemas compatíveis com ele, domina o segmento empresarial, o que torna essas competências as mais procuradas. É essa a razão pela qual este guia se baseia no CentOS Stream, um sistema da família Red Hat, como veremos de seguida.

## 5. CentOS, RHEL e o Ecossistema Enterprise Linux

Esta secção fecha o capítulo situando com precisão o sistema sobre o qual todo o guia assenta. É um tema que exige cuidado, porque o panorama mudou significativamente nos últimos anos, e muita documentação e muitos guias na internet ainda descrevem uma realidade que já não existe.

### A linhagem RHEL

O **Red Hat Enterprise Linux** (RHEL) é a distribuição Linux empresarial de referência. Enquanto outras distribuições se focavam no ambiente de trabalho ou nas pequenas empresas, o RHEL concentrou-se nas funcionalidades exigidas por aplicações críticas de grandes empresas e governos: sistemas capazes de processar transacções das maiores bolsas financeiras do mundo, de funcionar em clusters e como anfitriões de virtualização.

O modelo de negócio da Red Hat não assenta na venda do software em si, que é livre, mas na venda de **subscrições**: acesso a suporte técnico, actualizações certificadas, garantia de compatibilidade com hardware e software de terceiros, e formação. Em 2012, a Red Hat tornou-se a primeira empresa de software de código aberto a ultrapassar mil milhões de dólares de receita anual, precisamente com base neste modelo. Hoje pertence à IBM.

Para compreender o CentOS, é preciso primeiro compreender três sistemas relacionados que compõem a linhagem RHEL:

O **Fedora** é a distribuição livre e de vanguarda, patrocinada pela Red Hat, onde as novas tecnologias são testadas antes de serem consideradas para o RHEL. É o campo de ensaios: instável por natureza, mas onde o futuro do RHEL é experimentado.

O **RHEL** é o produto comercial, estável e suportado, construído a partir daquilo que amadureceu no Fedora.

O **CentOS** era, tradicionalmente, uma reconstrução livre do RHEL. E é aqui que a história se complica.

### CentOS Linux e CentOS Stream: a mudança de 2020

Durante muitos anos, o **CentOS Linux** foi exactamente aquilo que a maioria dos guias ainda hoje descreve: uma reconstrução comunitária do RHEL. A Red Hat publicava o código-fonte do RHEL, e o projecto CentOS recompilava-o, removendo as marcas registadas da Red Hat, para produzir um sistema **funcionalmente idêntico ao RHEL mas gratuito**. Como era estável, gratuito e essencialmente igual ao RHEL, o CentOS tornou-se a escolha por defeito de inúmeras empresas de alojamento web e organizações que queriam a robustez do RHEL sem o custo da subscrição. Tinha um ciclo de vida longo, de dez anos, como o RHEL.

Em Dezembro de 2020, a Red Hat anunciou uma mudança que abalou a comunidade: o **CentOS Linux seria descontinuado**, e o foco passaria para o **CentOS Stream**. Esta não foi uma simples mudança de nome, mas uma inversão fundamental do papel do CentOS.

A diferença é a seguinte. O CentOS Linux era **downstream** (a jusante) do RHEL: vinha *depois* do RHEL, reconstruindo uma versão já lançada e estável. O CentOS Stream é **upstream** (a montante) do RHEL: vem *antes*, sendo o ramo de desenvolvimento onde as próximas versões do RHEL são preparadas. Situa-se agora entre o Fedora e o RHEL.

```
Fedora  →  CentOS Stream  →  RHEL
(vanguarda)  (desenvolvimento)  (estável, comercial)
```

Na prática, isto significa que o CentOS Stream deixou de ser uma cópia estável do RHEL para passar a ser uma pré-visualização contínua da *próxima* versão do RHEL. Os pacotes são actualizados de forma contínua, à medida que ficam prontos, sem versões menores fixas, e o ciclo de vida encurtou de dez para cinco anos.

A reacção da comunidade foi de forte descontentamento. O anúncio incluiu ainda o encurtamento do ciclo de vida do CentOS Linux 8, que estava previsto durar até 2029 e passou a terminar no final de 2021. O CentOS Linux 7, a última versão do modelo antigo, chegou ao fim de vida em Junho de 2024.

### Rocky Linux e AlmaLinux

O descontentamento com o fim do CentOS Linux criou uma lacuna evidente: milhares de organizações queriam continuar a ter uma reconstrução gratuita, estável e downstream do RHEL, exactamente o que o CentOS Linux era e o CentOS Stream deixou de ser. Em resposta, surgiram quase de imediato duas novas distribuições para ocupar esse lugar.

O **Rocky Linux** foi criado por Gregory Kurtzer, um dos fundadores originais do próprio projecto CentOS. O nome é uma homenagem a Rocky McGaugh, outro co-fundador do CentOS já falecido. É gerido pela Rocky Enterprise Software Foundation, e a sua primeira versão estável saiu em 2021.

O **AlmaLinux** foi criado pela empresa CloudLinux e é gerido pela AlmaLinux OS Foundation, uma organização sem fins lucrativos. A sua primeira versão estável saiu também em 2021.

Ambos são reconstruções gratuitas, estáveis e compatíveis ao nível binário com o RHEL, com ciclos de vida longos de cerca de dez anos. Ambos oferecem ferramentas para migrar sistemas CentOS existentes. São, para todos os efeitos práticos, os sucessores directos daquilo que o CentOS Linux costumava ser, e são hoje escolhas muito populares para produção.

### Implicações práticas para o administrador

O que significa tudo isto para quem aprende administração de sistemas hoje?

Em primeiro lugar, e mais importante: **as competências são transferíveis entre todos estes sistemas.** RHEL, CentOS Stream, Rocky Linux, AlmaLinux e Oracle Linux partilham a mesma base, o mesmo gestor de pacotes (dnf/rpm), a mesma estrutura de ficheiros, as mesmas ferramentas de administração, o mesmo systemd, o mesmo firewalld, o mesmo SELinux. Aprender a administrar um deles é aprender a administrar todos. Tudo o que este guia ensina aplica-se igualmente a qualquer um.

Em segundo lugar, a escolha entre eles depende do contexto:

- **RHEL** é para ambientes de produção empresarial que precisam de suporte comercial certificado e podem pagar a subscrição.
- **Rocky Linux** ou **AlmaLinux** são a escolha para quem quer um sistema gratuito, estável e compatível com o RHEL em produção, ocupando o papel que o CentOS Linux tinha.
- **CentOS Stream** é adequado para quem quer acompanhar o desenvolvimento do RHEL, testar funcionalidades futuras, ou aprender, sendo mais recente que os clones mas menos estável que eles.
- **Fedora** é para quem quer a tecnologia mais recente e não precisa de estabilidade a longo prazo.

Este guia usa o **CentOS Stream** como referência. As razões são pedagógicas: é livre, é directamente ligado ao RHEL, e as competências adquiridas transferem-se sem alteração para o RHEL comercial, para o Rocky Linux, para o AlmaLinux, ou para qualquer sistema da família. Quando um comando ou um conceito for específico do CentOS, será assinalado, mas na prática quase tudo neste guia é comum a toda a família Red Hat.

## 6. Oportunidades Profissionais com Linux

Chegados ao fim deste capítulo introdutório, vale a pena olhar para a frente. Aprender a administrar sistemas Linux não é apenas um exercício técnico: é uma competência com valor concreto no mercado de trabalho, e compreender esse contexto ajuda a manter a motivação ao longo dos capítulos mais exigentes que se seguem.

### O baixo custo de começar

Uma das características mais notáveis do mundo do software livre é o baixíssimo custo de entrada. Para desenvolver uma ideia, criar um projecto tecnológico, ou simplesmente aprender, os custos concretos resumem-se hoje a um computador e uma ligação à internet. O Linux e milhares de pacotes de software estão disponíveis gratuitamente para quem quiser construir alguma coisa.

Muitas das maiores empresas tecnológicas do mundo começaram exactamente assim, com pouco mais do que uma ideia e ferramentas de código aberto. E o mundo do software livre traz consigo algo que o software proprietário não oferece: comunidades de programadores, administradores e utilizadores dispostos a partilhar conhecimento e a ajudar. Qualquer projecto de código aberto está permanentemente à procura de pessoas para escrever código, testar software ou redigir documentação, e é uma forma de aprender rodeado de quem já domina a matéria.

### A procura por competências em Linux

A adopção do Linux no mundo empresarial não pára de crescer, e com ela a procura por profissionais capazes de administrar esses sistemas. Como vimos ao longo deste capítulo, o Linux domina os servidores, a nuvem e a supercomputação, e essa omnipresença traduz-se directamente em oportunidades de emprego.

Os dados de mercado confirmam a tendência. As ofertas de emprego relacionadas com Linux têm crescido de forma consistente, e os salários reflectem a procura: um administrador de sistemas Linux nos mercados mais desenvolvidos aufere valores que vão desde o patamar de entrada até bem acima dos seis dígitos, em dólares, para engenheiros seniores com certificações. Uma das preocupações recorrentes das empresas que adoptam Linux não é sequer o custo ou a compatibilidade, mas a dificuldade em encontrar pessoas com as competências necessárias para suportar esses sistemas.

Por outras palavras: as competências que este guia ensina são, elas próprias, um activo profissional.

### Como se ganha dinheiro com software livre

Uma dúvida natural para quem chega ao Linux é: se o software é livre e gratuito, como é que há empresas que ganham dinheiro com ele? A resposta revela modelos de negócio que se afastam da simples venda de licenças.

O modelo mais importante é o das **subscrições**. A Red Hat, como vimos, não vende o RHEL como um produto avulso, mas sim subscrições que dão acesso a suporte técnico garantido, actualizações certificadas, ferramentas de gestão e a base de conhecimento da empresa. Paga-se não pelo software, que é livre, mas pela garantia e pela tranquilidade que uma grande organização precisa quando corre aplicações críticas.

Existem outros modelos. A **formação e certificação** é um mercado significativo, ajudando profissionais a comprovar as suas competências. As **recompensas** (*bounties*) permitem a quem precisa de uma funcionalidade específica pagar para que ela seja desenvolvida com prioridade, mantendo-se o resultado sob licença livre. E muitos projectos sustentam-se de **donativos** de indivíduos e empresas que dependem do seu código.

### Certificação: RHCSA e RHCE

Para quem procura emprego como profissional de Linux, a certificação é frequentemente um requisito ou, no mínimo, uma preferência dos empregadores. No ecossistema Red Hat, que é o deste guia, as duas certificações de referência são a RHCSA e a RHCE.

A **RHCSA** (*Red Hat Certified System Administrator*, exame EX200) é a certificação fundamental. Cobre as competências essenciais de administração de sistemas, e é a porta de entrada. A **RHCE** (*Red Hat Certified Engineer*, exame EX294) é a certificação avançada, e tem hoje uma característica importante: está inteiramente focada em automação com **Ansible**, refletindo a procura da indústria por competências de infraestrutura como código. A RHCSA é pré-requisito para a RHCE.

O que distingue estas certificações, e as torna respeitadas pelos empregadores, é o facto de serem **práticas**. Não há perguntas de escolha múltipla. O candidato senta-se perante um sistema real e tem de executar tarefas concretas, sob limite de tempo, exactamente como faria no trabalho. É avaliado pelos resultados que consegue obter. Isto torna-as difíceis de contornar e uma prova genuína de capacidade.

O ponto mais relevante para quem usa este guia é o seguinte: **os tópicos do exame RHCSA correspondem, quase na íntegra, à matéria coberta ao longo destes capítulos.** Uma olhadela aos objectivos oficiais do exame torna-o evidente:

| Objectivo do exame RHCSA | Onde é tratado neste guia |
|--------------------------|---------------------------|
| Ferramentas essenciais: shell, redirecções, gestão de ficheiros e permissões, man pages | Capítulos 3 e 4 |
| Operar sistemas: arranque, modos de recuperação, serviços, logs | Capítulos 5 e 8 |
| Configurar armazenamento local: partições, LVM, swap | Capítulo 8 |
| Criar e configurar sistemas de ficheiros, montagem, NFS | Capítulo 8 |
| Implementar e manter sistemas: rede, cron, pacotes, kernel | Capítulos 3, 5 e 7 |
| Gerir utilizadores e grupos, autenticação por LDAP | Capítulos 5 e 7 |
| Gerir segurança: firewall e SELinux | Capítulo 7 |

De forma semelhante, os tópicos da RHCE, que se concentram na configuração e segurança de servidores (Apache, DNS, NFS, correio, SSH, NTP, tudo protegido com firewall e SELinux), são precisamente os serviços que o Capítulo 7 aborda em detalhe.

Isto não é coincidência. Este guia foi construído em torno das competências que definem um administrador de sistemas Linux competente, e essas são as mesmas competências que as certificações procuram validar. Ainda que o objectivo imediato não seja obter uma certificação, dominar a matéria destes capítulos coloca o leitor no caminho certo, tanto para o exame como, mais importante, para o trabalho real que a certificação representa.
