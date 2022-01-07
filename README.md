# Kubernetes

### O que é?

- Kubernetes é uma aplicação desenvolvida pela Google para o controle de diversos containers que rodam aplicações

### Qual problema o Kubernetes resolve?

- A necessidade do controle de containers, já que eles são úteis para que rodemos microserviços 

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
- A instalação do minikube ocorre a partir dos seguintes comandos no shell para a distro Ubuntu

~~~html
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
~~~

- O kubectl pode ser instalado com os comandos deste link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

### Comandos do Kubectl

- É importante entender que o Deployment comanda o ReplicaSet(réplicas)
- O ReplicaSet comanda o Pod
- O pod é uma abstração de um container
- Isso significa que temos a seguinte hierarquia: deployment > replicaset > pod, pois o deployment usa apenas o nome, o replicaset usa o nome + id e o pod usa nome + id + id próprio para se identificar(já que existem réplicas de um mesmo pod)
- Então, ao mostrarmos as replicaset's mostramos o nome + o id daquele pod e quantos deles temos.
- Com isso, podemos mexer diretamente e apenas no deployment para atualizar, selecionar quantidade de pods(réplicas) entre outras coisas, pois o deployment é a ideia geral.
~~~html

kubectl get deployment -> mostra deployments ( deployments são, aqui, a abstração de um pod)
kubectl get pod -> mostra pods existentes
kubectl get services -> mostra serviços existentes
kubectl create -h -> mostra comandos de criação existentes
kubectl create deployment NAME --image=image -> cria um pod, exemplo abaixo:
kubectl create deployment nginx-depl --image=nginx -> Como o deployment é uma abstração do pod, este comando cria um pod com a imagem do nginx baixado do dockerHub
kubectl get replicaset -> mostra quantas replicas temos dos pod's
kubectl edit deployment NAME -> edita o deployment, exemplo para nosso caso: nginx-depl
~~~

- Caso editemos o arquivo do deployment com o comando acima, por exemplo, mudando a versão da imagem, como não alteramos o nome do deployment ele continua intacto, mas o id do replicaSet muda e, consequentemente, o id do pod também, automaticamente, e simultaneamente, a replicaSet antiga morre (seus pods vão para a nova versão).

~~~html

kubectl logs [nome deployment-id replicaset-idPod] -> mostra registro/histórico 
kubectl exec -it [nome deployment-id replicaset-idPod] --bin/bash -> entra no terminal da aplicação
~~~

- Ao criarmos deployment, depois de passarmos o nome da imagem, podemos escolher algumas opções, mas se forem diversas, podemos criar um arquivo.yaml e pedirmos para executá-lo:

~~~html

kubectl apply -f arquivo.yaml

~~~
### O arquivo YAML de configuração do Kubernetes (REVISAR ESTA PARTE)

- O arquivo começa com o versão de api e o tipo da configuração, como, por exemplo, se é um deployment, um service, ou outro
- Depois disso,o arquivo é dividido em 3 partes: o Metadata(metadados, ou seja, dados que falam dos dados, como nomes etc), spec(especificações, como a quantidade de réplicas no deployment ou a porta no service, ou seja, são especificidades de cada tipo configurado) e o status, que são informações automaticamente geradas pelo kubernetes a partir do etcd (parte do master node que armazena dados)

Exemplo de Secret

~~~html

apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque            ->padrão
data:
  username:             -> devemos usar dados encriptados em base 64, o que pode ser feito no terminal com o comando 'echo -n 'valor' | base64'
  password: 
~~~

Se vamos referenciar o Secret no Deployment, devemos aplicar o secret antes para funcionar.



Exemplo de Deployment

~~~html
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
spec:
  replicas: 2
  selector:
  template:                     ->TEMPLETE:ESSA PARTE DEFINE A 'PLANTA' DE UM POD, QUAL IMAGEM, NOME, PORTA, ETC
    metadata:
      labels:                   ->LABELS E SELECTOR AJUDAM A FALAR QUAIS PODS ESTÃO ASSOCIADOS A ESSE DEPLOYMENT
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 8080
~~~

### Namespaces

- Namespaces são formas de organizar os recursos sendo um cluster dentro do cluster.
- Pomode visualizar os namespaces ativos com o comando 'kubectl get namespaces'
- Temos o kube-system que não devemos alterar, e trata de processos do sistema.
- Temos o kube-public, que contêm os dados de acesso público, como o configmap.
- Temos o kube-node-lease que contêm informações sobre a vida dos nodes, como a disponibilidade deste.
- Temos o default, que é usado para criar recursos no início caso não tenhamos criado outro namespace
- Podemos criar outros namespaces com o comando 'kubectl create namespace [nome]'
- Devemos usar diferentes namespaces para diferentes propósitos, como um apenas para a base de dados, por exemplo, ou para times diferentes, pois isso evita acúmulo e má organização, inclusive para quando times diferentes trabalham no mesmo projeto, já que isso pode gerar conflitos.
- É possível limitar o acesso e recursos a determinados grupos a apenas o namespace do projeto deles.
- É possivel limitar, por namespace, CPU, RAM e armazenamento.
- Apesar das vantagens, se em um namespace, temos um configmap que acessa um banco de dados, mesmo que desejemos um igual no em outro namespace, precisaremos criar um novo, pois não teremos acesso. O mesmo vale para um secret. Entretanto, recursos como o Service podem ser compartilhados entre namespaces.
- Alguns componentes como o Volume ou Node não podem ser criados dentro de um namespace. 
- Se quisermos aplicar uma configuração dentro de um determinado namespace diferente do default devemos fazer:

~~~html

kubectl apply -f arquivo.yaml --namespace=nomeNamespace

~~~

- Ou podemos colocar isso no 'metadata' do arquivo.yaml (mais recomendado por termos documentado)

### Ingress

- Quando temos o external service para acesso da aplicação pelo browser, a URL é do tipo  do tipo protocoloSegurança://ipDoNode:porta, como https://124.89.101.2:8080, como explicado, e o Ingress é um componente que auxilia na criação do DNS, ou seja, um nome, como https://my-app.com.
- Logo, no lugar do service, devemos criar um arquivo de configuração yaml do tipo Ingress, além de um ingress controller que regulará este processo.
- Para instalar o ingress controller devemos usar o comando 

~~~html

minikube addons enable ingress

~~~
- Nas specs do arquivo.yaml do Ingress devemos definir as regras (rules) de ingresso, além de podermos definir mais de um caminho(path) depois da / no fim do nome de domínio, como https://my-app.com/something.

### Helm

- Assim como no Linux temos o 'apt' para baixarmos pacotes, no Kubernetes temos o Helm, que nos auxilia a baixar arquivos.yaml de coisas comumente já feitas.
- Temos o helmHub ou podemos procurar com o comando "helm search [algo]'

### Volumes:

- Como o Kubernetes não trata a persistência de dados em caso de morte de um pod, por exemplo, de banco de dados, precisamos de um armazenamento que não dependa do ciclo de vida dos pods.
- Este armazenamento deve ser acessível em qualquer node.
- O armazenamento deve sobreviver mesmo que o cluster morra.
- Podemos definir o tipo persistentData no 'kind' do arquivo.yaml, mas ainda precisamos de um armazenamento físico para isso, seja em nuvem ou não. O armazenamento físico deve ser explicitado nas specs do documento.
- Não podemos colocar persistentData em algum namespace.
- Outro tipo (kind) definido no arquivo.yaml é o pvc(persistentData claim) que, quando um pod pedir um volume, vai tentar achar um volume com as espeficações definidas nele (como o tamanho definido no pvc) dentro do cluster)
- O pvc deve existir no mesmo namespace que o pod que usa ele.
- A storage class provê dinamicamente um persistent Volumes quando o PVC diz que necessita de uma com certa configuração, o que tira um pouco a tarefa maçante do administrador do Kubernetes de criar este persistent Volume manualmente toda vez, que é o que acontece geralmente. Também temos esse 'kind' no arquivo.yaml (StorageClass).

### StatefulSet

- Aplicações Stateful são aquelas com estado, ou seja, aquelas nas quais o estado importa, como bancos de dados 
- Para evitar inconsistências de dados, ao invés de usarmos deployment, usamos o statefulSet, outro 'kind' de arquivo.yaml 
- Enquanto os deployments usam como identificador do pod um hash aleatório, o statefulset usa algo do tipo $(statefulset name)-$(ordinal)





