# Instalação e Virtualização de Sistemas

## 1. A Evolução e o Impacto da Virtualização

Nos últimos 50 anos, tendências fundamentais mudaram a forma como os serviços de computação são fornecidos. O processamento em mainframes impulsionou as décadas de 60 e 70; os computadores pessoais e a arquitetura cliente/servidor dominaram as décadas de 80 e 90; e a Internet abrangeu a transição do século. Atualmente, o pilar de qualquer infraestrutura moderna assenta noutra tecnologia transformadora: a virtualização.

Historicamente, os centros de dados debatiam-se com o modelo ineficiente de **"um servidor, uma aplicação"**. Com o aumento exponencial da capacidade de processamento (impulsionado pela Lei de Moore), os servidores físicos acabavam por realizar cada vez menos trabalho em proporção à sua capacidade total, resultando num enorme desperdício de recursos.

A virtualização moderna resolveu este problema. Na sua essência, a computação **virtual** refere-se à criação de uma representação lógica de um recurso físico. Em vez de utilizar diretamente um componente de hardware isolado, o sistema passa a trabalhar com uma abstração.

Exemplos comuns incluem:
- **VLANs**: redes lógicas
- **SANs**: armazenamento lógico

O foco principal da administração de sistemas moderna, no entanto, é a virtualização de computadores inteiros, permitindo que múltiplos sistemas operativos (**workloads**) corram no mesmo hardware em simultâneo, mantendo-se rigorosamente isolados.

## 2. O Monitor de Máquina Virtual (VMM) e os Princípios Fundamentais

A primeira iteração de virtualização ocorreu nos mainframes da IBM na década de 1960. Contudo, em 1974, Gerald J. Popek e Robert P. Goldberg formalizaram os requisitos arquitetónicos da virtualização num artigo fundamental.

Segundo a sua definição, para um **Monitor de Máquina Virtual (VMM)** — ou **Hypervisor** — ser considerado eficiente, deve exibir três propriedades cruciais:

- **Fidelidade (Fidelity):** o ambiente criado para a VM tem de ser essencialmente idêntico à máquina física original.
- **Isolamento e Segurança (Isolation/Safety):** o VMM deve manter o controlo absoluto dos recursos do sistema, garantindo que as VMs não interferem umas com as outras nem acedem a áreas de memória não autorizadas.
- **Desempenho (Performance):** a diferença de capacidade de processamento entre a VM e um equivalente físico direto deve ser mínima.

Na sua essência, o hypervisor atua como um árbitro de recursos. Ele desempenha duas funções vitais:

- **Abstração de Hardware:** cria a ilusão de que o sistema operativo convidado (**Guest OS**) tem acesso exclusivo a componentes físicos (CPU, RAM, discos), interagindo com dispositivos virtuais.
- **Alocação de Recursos:** interceta, organiza e encaminha todas as solicitações de Entrada/Saída (**I/O**) geradas pelas VMs para o hardware real, de forma justa e eficiente.

## 3. Classificação de Arquiteturas: Type 1 vs. Type 2

Os hypervisors são categorizados pela forma como são implementados na pilha de hardware do sistema.

### Hypervisors Tipo 1 (Bare-Metal)

- Instalam-se diretamente sobre o hardware físico do servidor, sem a necessidade de um sistema operativo subjacente tradicional.
- **Desempenho:** oferecem o mais alto nível de eficiência e menor latência, comunicando diretamente com os componentes físicos.
- **Segurança:** superfície de ataque reduzida. Uma falha crítica numa máquina virtual fica estritamente contida nas fronteiras dessa VM.

**Exemplos:** KVM, VMware ESXi, Proxmox.

### Hypervisors Tipo 2 (Hosted)

- Funcionam como uma aplicação instalada sobre um sistema operativo tradicional (ex.: Windows, macOS, Linux desktop).
- **Sobrecarga (Overhead):** são menos eficientes. Cada pedido de hardware feito por uma VM tem de passar pelo hypervisor e, subsequentemente, pelo sistema operativo hospedeiro.
- **Casos de Uso:** ideais para ambientes de desenvolvimento local e testes à escala do utilizador individual.

**Exemplos:** Oracle VM VirtualBox, VMware Workstation.

## 4. A Stack de Engenharia: KVM e Virt-Manager

Embora soluções como a VMware e o Xen tenham sido pioneiras na arquitetura x86, o mercado de virtualização expandiu-se com alternativas robustas orientadas ao ecossistema open-source. Em ambientes de engenharia assentes puramente em Linux, a eficiência e a integração profunda com o sistema operativo são imperativas.

É neste contexto que se destaca o **KVM (Kernel-based Virtual Machine)**. Inicialmente desenvolvido pela Qumranet (adquirida pela Red Hat em 2008), o KVM representa uma abordagem arquitetónica distinta: em vez de construir um hypervisor de raiz, o KVM é entregue como um módulo do kernel Linux.

Ao carregar este módulo, o próprio kernel Linux é convertido num hypervisor **Bare-Metal (Tipo 1)**. Isto traz vantagens técnicas massivas:

- **Reaproveitamento de Código:** o KVM herda e utiliza as funcionalidades avançadas já existentes no Linux, como o escalonador de processos (**scheduler**) e a gestão avançada de memória, sem necessidade de duplicar estas funções.
- **Adoção e Escalabilidade:** devido à sua performance e integração nativa, o KVM tornou-se a direção de futuro para a infraestrutura empresarial e é a espinha dorsal de soluções de computação em nuvem (**cloud computing**) abertas, como o OpenStack.

### A Interface de Gestão: Virt-Manager

Para interagir com o KVM e o daemon libvirt sem depender exclusivamente de extensas linhas de comandos QEMU, utiliza-se o **Virtual Machine Manager (virt-manager)**.

Esta ferramenta gráfica atua como um painel de controlo central, permitindo provisionar, monitorizar e configurar máquinas virtuais (como instâncias CentOS) com um controlo granular sobre o hardware virtualizado, perfeitamente integrado em ambientes desktop Linux modernos de alta performance.

## 5. Anatomia e Gestão da Máquina Virtual (VM)

No topo desta hierarquia (**Hardware -> Hypervisor -> VM**) encontra-se a própria Máquina Virtual. Fundamentalmente, a VM é caracterizada por dois estados de existência:

- **Estado Estático (Ficheiros de Dados):** do ponto de vista do sistema anfitrião, a VM é apenas um conjunto de ficheiros. Os cruciais são o ficheiro de configuração (que enumera o hardware virtual, como a capacidade de RAM) e o ficheiro de disco virtual (uma representação lógica do armazenamento, ex.: `.qcow2`).
- **Estado Dinâmico (Instanciação):** quando em execução, é uma entidade ativa na RAM.

### Mecanismos de Gestão de Recursos

A gestão da CPU numa VM recorre a **vCPUs**. Uma vCPU não é um processador físico dedicado, mas sim uma fatia de tempo de execução (**time-slice**). O hypervisor (neste caso, o KVM) utiliza os seus algoritmos para distribuir os ciclos de processamento dos núcleos físicos entre as várias VMs, permitindo taxas de consolidação elevadas.

### Funcionalidades Avançadas de Ficheiros

O encapsulamento lógico da VM introduz capacidades de administração de sistemas incomparáveis face ao hardware físico tradicional:

- **Clonagem:** replicação dos ficheiros de uma VM para criar uma instância independente, reduzindo o aprovisionamento de semanas para minutos.
- **Templates:** VMs preconfiguradas e seladas, utilizadas como "moldes" para implantar arquiteturas de software padronizadas.
- **Snapshots (Pontos de Restauro):** congelamento do estado exato do sistema (memória e disco). As operações de escrita subsequentes são redirecionadas para um disco delta. Em caso de falha crítica durante uma atualização do CentOS, por exemplo, o administrador pode reverter a VM instantaneamente para o estado seguro anterior.
- **OVF (Open Virtualization Format):** norma aberta que empacota a VM, permitindo a sua exportação e importação entre hypervisors de diferentes fornecedores.
