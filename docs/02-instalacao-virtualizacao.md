# Instalação e Virtualização

### Descrevendo a Virtualização
Nos últimos 50 anos, tendências fundamentais mudaram a forma como os serviços de computação são fornecidos. O processamento em mainframes impulsionou as décadas de 60 e 70; os computadores pessoais e a tecnologia cliente/servidor dominaram as décadas de 80 e 90; e a Internet abrangeu a transição do século, continuando a evoluir até hoje. Atualmente, estamos no meio de mais uma tendência transformadora: a virtualização.

A virtualização é uma tecnologia disruptiva que quebra o status quo de como os computadores físicos são geridos, os serviços são entregues e os orçamentos são alocados. Para entender o seu impacto profundo no ambiente informático atual, é essencial compreender a sua evolução histórica.
### O que é virtualização
Na computação, o termo “virtual” refere-se à criação de uma representação lógica de um recurso físico. Em vez de utilizar diretamente um componente de hardware isolado, o sistema passa a trabalhar com uma abstração desse recurso, o que permite maior flexibilidade, eficiência e controlo.
Exemplos comuns incluem:
- VLANs (Redes Locais Virtuais): Melhoram o desempenho e a gestão da rede ao separá-la do hardware físico.
- SANs (Storage Area Networks): Oferecem maior flexibilidade e uso eficiente do armazenamento, abstraindo os discos físicos em objetos lógicos de fácil manipulação.
O foco principal, no entanto, é a virtualização de computadores inteiros. Na sua essência, funciona como a "realidade virtual" para programas de computador: cria um ambiente simulado que o sistema e as aplicações vivenciam como se fosse hardware físico real.
### O Monitor de Máquina Virtual (VMM) e as Regras de Popek e Goldberg
A primeira virtualização convencional ocorreu nos mainframes da IBM na década de 1960. Contudo, em 1974, Gerald J. Popek e Robert P. Goldberg formalizaram os requisitos da virtualização num artigo fundamental. Segundo a sua definição:
- Máquina Virtual (VM): Capaz de virtualizar todos os recursos de hardware (processadores, memória, armazenamento e rede).
- Monitor de Máquina Virtual (VMM) ou Hypervisor: O software que fornece e gere o ambiente onde as VMs operam.
Para um VMM ser considerado eficiente e cumprir a sua definição, deve exibir três propriedades cruciais:
1. Fidelidade (Fidelity): O ambiente criado para a VM tem de ser essencialmente idêntico à máquina física original.
2. Isolamento ou Segurança (Isolation/Safety): O VMM deve manter o controlo absoluto dos recursos do sistema, garantindo que as VMs não interferem umas com as outras.
3. Desempenho (Performance): A diferença de velocidade ou capacidade de processamento entre a VM e um equivalente físico deve ser mínima ou inexistente.
### O Papel Central do Hypervisor (Monitor de Máquina Virtual)
Na sua essência, um hypervisor é um árbitro de recursos. Historicamente conhecido como Monitor de Máquina Virtual (VMM), este software actuacomo a camada intermediária entre o hardware físico de um servidor e as máquinas virtuais (VMs) que nele operam. Sem um hypervisor, múltiplos sistemas operativos a tentar aceder simultaneamente aos mesmos recursos de hardware (como disco ou memória) resultariam em conflito e falha do sistema.
O hypervisor desempenha duas funções vitais, frequentemente ilustradas por analogias conceptuais:
1. O "Simulador de Realidade" (Abstração de Hardware): Tal como um simulador de realidade virtual engana os sentidos humanos, o hypervisor engana o sistema operativo convidado (Guest OS). Ele cria a ilusão de que a máquina virtual tem acesso exclusivo e directo a componentes físicos (CPU, RAM, Discos), quando, na verdade, está a interagir com fracções de recursos abstractos geridos pelo hypervisor.
2. O "Controlador de Tráfego" (Alocação de Recursos): O hypervisor intercepta todas as solicitações de Entrada/Saída (I/O) geradas pelas VMs. Quer seja um pedido de leitura no disco ou o envio de um pacote de rede, o hypervisor organiza, prioriza e encaminha esses pedidos para o hardware físico real de forma eficiente e justa, garantindo que nenhuma VM monopoliza o sistema.
#### Classificação de Arquitecturas: Type 1 vs. Type 2
Existem duas classes principais de hypervisors, categorizadas não pela sua capacidade, mas pela forma como são implementadas na pilha de hardware.
##### Hypervisors Tipo 1 (Bare-Metal)
Um hypervisor Tipo 1 é instalado directamente sobre o hardware físico do servidor, sem a necessidade de um sistema operativo subjacente.
- Desempenho: Por não ter camadas intermediárias, comunica directamente com o hardware, oferecendo o mais alto nível de eficiência e menor latência.
- Segurança e Isolamento: São inerentemente mais seguros. Como não dependem de um sistema operativo hospedeiro, a superfície de ataque é menor. Uma falha crítica numa máquina virtual fica estritamente contida nas fronteiras dessa VM, não afectando o hypervisor nem as outras máquinas alojadas.
-Exemplos: KVM (quando integrado nativamente), VMware ESXi, Proxmox.
##### Hypervisors Tipo 2 (Hosted)
Um hypervisor Tipo 2 funciona como uma aplicação instalada sobre um sistema operativo tradicional (como Windows, macOS ou Linux).
- Sobrecarga (Overhead): São menos eficientes. Cada pedido de hardware feito por uma VM tem de passar pelo hypervisor, que por sua vez o entrega ao sistema operativo hospedeiro, que finalmente interage com o hardware. Este processo adiciona latência.
- Confiabilidade: Possuem mais pontos de falha. Qualquer problema que afecte o sistema operativo hospedeiro (como uma actualização que exija reinicialização ou uma falha de segurança) impactará imediatamente todas as máquinas virtuais em execução.
- Casos de Uso: São ideais para ambientes de desenvolvimento, testes locais e utilizadores individuais.
- Exemplos: Oracle VM VirtualBox, VMware Workstation.
### Definição e Ontologia da Máquina Virtual (VM)
Uma Máquina Virtual é uma implementação de software de uma máquina (computador) que executa programas como uma máquina física. No modelo de arquitetura de sistemas, a VM representa a camada superior de abstração, operando sobre um Monitor de Máquina Virtual (VMM) ou Hypervisor, que por sua vez faz a interface com o hardware físico (Host). 
Fundamentalmente, a VM é caracterizada por dois estados de existência:
1. Estado Estático (Ficheiros de Dados): A VM é encapsulada como um conjunto de ficheiros num sistema de ficheiros. Os mais relevantes são o Ficheiro de Configuração (que enumera o hardware virtual, como vCPUs, capacidade de RAM e periféricos) e o Ficheiro de Disco Virtual (uma representação lógica de um dispositivo de armazenamento secundário).
2. Estado Dinâmico (Instanciação): Quando em execução, a VM é uma entidade ativa na memória RAM, cujas instruções são arbitradas pelo hypervisor.
### A Dualidade de Perspectiva: Abstração e Implementação
O sucesso da virtualização depende da manutenção de duas visões distintas e isoladas:
- Perspectiva Interna (Sistema Convidado): O Sistema Operativo Convidado (Guest OS) opera sob a ilusão de acesso exclusivo ao hardware. Através de controladores (drivers) genéricos, o Guest OS reconhece processadores, memória e controladores de rede como dispositivos físicos standard, ignorando a sua natureza lógica.
- Perspectiva Externa (Administrador/Host): Do ponto de vista do anfitrião, a VM é um processo isolado que consome recursos de um pool partilhado. Esta perspetiva permite a portabilidade, uma vez que a VM se torna independente das especificidades do hardware físico subjacente.
### A Expansão do Paradigma de Virtualização
Embora a virtualização de servidores seja a espinha dorsal das infraestruturas modernas, os princípios de abstracção expandiram-se para outras áreas da engenharia de sistemas:
- Virtualização de Desktops (VDI): Transfere o processamento e armazenamento do computador do utilizador final para servidores centrais. O utilizador acede ao seu ambiente através de thin clients, reduzindo drasticamente custos de hardware, consumo de energia e vulnerabilidades de segurança, uma vez que os dados nunca saem do centro de dados.
- Virtualização de Aplicações: Isola programas do sistema operativo subjacente, encapsulando os seus processos e dependências. Isto previne conflitos entre aplicações (ex: diferentes versões de bibliotecas) e facilita a distribuição em massa em ambientes corporativos.
- Contentores (Containers): Uma evolução tecnológica onde, em vez de virtualizar hardware para correr múltiplos sistemas operativos, virtualiza-se o próprio sistema operativo. Múltiplos ambientes partilham o mesmo Kernel, resultando num arranque quase instantâneo e num consumo de recursos mínimo, ideal para a arquitectura de microsserviços.
#### Virtualização como Motor do Cloud Computing
Há alguns anos, a Computação em Nuvem (Cloud Computing) era um conceito emergente; hoje, é o padrão da indústria. É imperativo compreender que a virtualização é o motor tecnológico que torna a Cloud possível.
Antes da virtualização, a gestão de centros de dados era um processo rígido, subutilizado e intensivo em mão-de-obra. A virtualização abstraiu a camada física, criando reservas de recursos (pools) de computação, rede e armazenamento altamente escaláveis e disponíveis. Este modelo permite o fornecimento dinâmico de infraestrutura como serviço (IaaS), simplificando a entrega de aplicações e permitindo que as organizações escalem as suas operações sob demanda, sem sacrificar a resiliência.
### A Importância Prática da Virtualização
Historicamente, os centros de dados debatiam-se com o modelo ineficiente de "um servidor, uma aplicação". Com o aumento da capacidade de processamento (impulsionado pela Lei de Moore), os servidores físicos acabavam por realizar cada vez menos trabalho em proporção à sua capacidade total, resultando num enorme desperdício de recursos.

A virtualização moderna resolveu este problema permitindo que vários sistemas operativos (workloads) corram no mesmo hardware em simultâneo, mantendo-se isolados. A primeira solução comercial para arquitetura x86 foi lançada pela VMware em 2001, seguida pela alternativa open-source Xen em 2003. Estes hypervisors instalam-se como uma camada de software diretamente sobre o hardware (bare-metal) ou sobre um sistema operativo anfitrião.

#### O Poder da Consolidação
O maior benefício da virtualização para os centros de dados foi a consolidação — a capacidade de condensar o trabalho de múltiplos servidores físicos em poucas máquinas altamente otimizadas.

- Rácio de Consolidação: É a métrica que define esta eficiência. Por exemplo, um servidor físico que aloje oito máquinas virtuais tem um rácio de consolidação de 8:1.

- Impacto: Mesmo um rácio conservador de 4:1 pode eliminar 75% dos servidores físicos de um centro de dados, resolvendo problemas críticos de espaço físico, consumo de energia e sobrecarga de gestão, exatamente quando a indústria mais precisava.

