# Kubernetes

### O que é?

-Kubernetes é uma aplicação desenvolvida pela Google para o controle de diversos containers que rodam aplicações

### Qual problema o Kubernetes resolve?

-A necessidade do controle de containers, já que eles são úteis para que rodemos microserviços 

### O que o Kubernetes oferece? 

- Alta confiabilidade quanto ao disponibilidade, ou seja, dificilmente haverão quedas
- Altamente escalável e alta performance
- Backup e restauração, se necessário

### Componentes

- Um "Node" é um servidor/computador e contêm "Pod's", que é a menor unidade do Kubernetes
- os Pods são abstrações de containers e rodam 1 aplicação apenas, usualmente
- Cada Pod possui um endereço IP que usa para comunicar com o outro Pod
- Se um Pod morre e ressurge outro em seu lugar um novo IP é criado para ele
- Para evitar transtornos causados por essas mudanças de IP, temos o "Service" que determina um IP estático para certo Pod,
    mesmo que ele morra
- Para que a aplicação fique visível no browser criamos o "External Service" e para nossas informaçõões não ficarem visíveis
    criamos o "Internal Service"
- Usualmente o "External Service" é do tipo protocoloSegurança://ipDoNode:porta, como https://124.89.101.2:8080, mas se quisermos
    um DNS para a URL usamos o componente "Ingress" que fará com que o request passe por lá antes do "Service"
- Normalmente, a comunicação entre dois Pods, como a comunicação entre uma aplicação e um banco de dados, é feita com comandos
    dentro da própria aplicação, logo, se a URL do banco de dados mudar, por exemplo, é necessário mudar a apliacação, o que é ruim. Para isso
    temos o componente "ConfigMap:" que conectamos à certo Pod e nos permite alterar certas configurações sem alterar a aplicação.
- Dados como username e senha não devem ser usados no ConfigMap por questões de segurança, para isso temos o componente "Secret"
    que é uma maneira encriptada
- Outro componente importante é o "Volumes" que resolve o problema da perda de dados quando um Pod morre. Isso porque associa um Pod 
    a um disco físico, seja na nuvem ou localmente. O kubernetes Cluster não tem tratamento de persistência de dados.
- Existem componentes que servem para a segurança do sistema como um todo, uma vez que a aplicação ou o banco de dados podem morrer
    e assim deixarem de ser acessíveis no browser. Para evitar isso, temos o "Deployment" para aplicações em que o estado não importa
    e o "StatefulSet" para aplicações como bancos de dados em que o estado importa. Esses componentes criam modelos ou réplicas
    (quantas quisermos) do Pod que desejarmos em outro Node, a partir do mesmo Service (IP e DNS). A diferença entre o Deployment
    e do StatefulSet é que o último fornece um controle de qual Pod do banco de dados está acessando os dados necessários, o que
    evita inconsistências de dados.

### Arquitetura do Kubernetes

- Cada Node tem vários Pods
- Cada Node roda 3 processos que devem estar instalados : Container Runtime (Docker. ou outro), Kubelet (Comunica com o Node e com
    o Container runtime e cria novos Pods) e, por fim, Kube Proxy, que auxilia no encaminhamento dos requests por meio do Service
    quando temos diferentes Nodes com réplicas. Isso auxilia na economia de recursos pois é um otimizador.
- A organização geral dos Nodes como o monitoramento da morte de Pods, e criação deles é feita pelo "Master Node" que possui 4 processos
    que roda em todos Master Node: API Server(Serve para fazer consultas ou pedir atualizações, e valida esses pedidos para enviar
    para outros processos que decidirão onde criar um Pod, por exemplo),  Scheduler (A partir de um pedido de novo Pod, o API server
    valida e o Scheduler decide onde é o melhor local para o Pod) ! QUEM CRIA DE FATO É O Kubelet. Controller Manager (Detecta mudanças
    de estado e tenta recuperar o estado deles a partir de um pedido para o Scheduler que envia para o Kubelet). Por fim temos o etcd 
    (é o cérebro do cluster porque armazena as mudanças feitas numa chave de valor, o que permite o Scheduler saber quais recursos
    estão livres e escolher onde criar o pod, por exemplo) !ETCD não armazena informações da aplicação e sim das alterações feitas 
    no cluster. Podemos ter mais de um Master node para diferentes intuitos.
- Um exemplo simples de Cluster seria um com 2 master nodes e 3 worker nodes, sendo que, apesar do serviço dos masters ser mais importante
    eles precisam de menos recusos como CPU, RAM e Armazenamento. Caso queiramos adicionar um Master ou um Worker, devemos conseguir
    um novo servidor vazio, instalar os processos referentes ao node(se for master precisamos do API Server etc, se for Worker precisamos
    do Kubelet etc.) e depois simplesmente adicionar ao cluster.

### Minikube e Kubectl

- Minikube é uma solução para gasto excessivo de recursos em um computador local, pois permite que, do nosso computador, a partir de uma 
máquina virtual, possamos ter um Node com tanto os processos do Worker quanto os do Master. 
- Kubectl são linhas de comando que permitem a comunicação com o Api Server (processo do node master que regula o que vai ser feito
    o que, mais tarde, é realizado pelos processos do Worker Node)

