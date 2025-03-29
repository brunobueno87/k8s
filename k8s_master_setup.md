# Documentação dos Comandos para Configurar um Node Master no Kubernetes

OBS; Estou partindo do ponto que você esta com uma maquina com os pre-rec minimos como descrito na doc;

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin

## Descrição dos Comandos

### 1. Configurações Iniciais

#### **Desativar o firewall**
```bash
sudo ufw disable
```
Desativa o firewall UFW para evitar bloqueios de comunicação entre os componentes do cluster.

#### **Desabilitar o swap**
```bash
sudo swapoff -a
```
Desativa o uso de swap, pois o Kubernetes requer que o swap esteja desabilitado para funcionamento adequado.

#### **Comentar a entrada de swap no arquivo fstab**
```bash
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
Impede que o swap seja reativado automaticamente ao reiniciar o sistema.

#### **Habilitar o encaminhamento de pacotes IPv4**
```bash
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```
Habilita o encaminhamento de pacotes IPv4 para permitir comunicação entre pods.

### 2. Configurar módulos do kernel

#### **Carregar módulos necessários**
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```
Configura e carrega os módulos `overlay` e `br_netfilter` para suportar redes em contêineres.

#### **Aplicar configurações de sistema**
```bash
sudo sysctl --system
```
Aplica todas as configurações do kernel configuradas nos arquivos de configuração.

### 3. Instalar ferramentas do Kubernetes

#### **Preparar o ambiente**
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
```
Atualiza os repositórios e instala dependências necessárias.

#### **Adicionar chave pública do Kubernetes**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
Baixa e adiciona a chave de assinatura do repositório do Kubernetes.

#### **Adicionar repositório do Kubernetes**
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Adiciona o repositório oficial do Kubernetes.

#### **Instalar componentes do Kubernetes**
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
Instala o kubelet, kubeadm e kubectl e impede que sejam atualizados automaticamente.

#### **Habilitar e iniciar o kubelet**
```bash
sudo systemctl enable --now kubelet
```
Habilita o kubelet para iniciar automaticamente no boot.

### 4. Instalar e configurar o Containerd

#### **Preparar o ambiente do Docker**
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
```
Instala dependências para configurar o Docker.

#### **Adicionar chave pública do Docker**
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Adiciona a chave de assinatura do repositório do Docker.

#### **Adicionar repositório do Docker**
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Adiciona o repositório oficial do Docker.

#### **Instalar o containerd**
```bash
sudo apt-get update && sudo apt-get install -y containerd.io
```
Instala o runtime containerd.

#### **Configurar o containerd**
```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```
Gera o arquivo de configuração padrão, ajusta para usar cgroups gerenciados pelo systemd e reinicia o containerd.

#### **Validar e habilitar o containerd**
```bash
sudo systemctl status containerd
sudo systemctl enable --now kubelet
```
Verifica o status do containerd e habilita o kubelet.

### 5. Inicializar o Cluster Kubernetes

#### **Iniciar o cluster**
```bash
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=192.168.0.28
```
Inicializa o cluster Kubernetes especificando a rede de pods e o endereço do servidor API.

#### **Configurar acesso ao cluster**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Configura o acesso ao cluster copiando o arquivo de configuração e ajustando permissões.

#### **Validar a configuração do cluster**
```bash
kubectl config view
kubectl get nodes
kubectl get pods -n kube-system
```
Exibe a configuração atual, os nodes do cluster e os pods em execução no namespace `kube-system`.

#### **Monitorar os pods do sistema**
```bash
watch -n1 kubectl get pods -n kube-system
```
Atualiza a cada segundo a lista de pods do namespace `kube-system`.

### 6. Configurar Rede do Cluster

#### **Aplicar plugin de rede Weave**
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
Instala o plugin de rede Weave Net para comunicação entre pods.

### 7. Melhorias e Ajustes Finais

#### **Habilitar autocomplete do kubectl**
```bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> /home/ubuntu/.bashrc
```
Habilita o autocomplete para o comando `kubectl` no Bash.

#### **Configurar encaminhamento de pacotes persistente**
```bash
sudo nano /etc/sysctl.conf
```
Adicione ou edite a linha:
```plaintext
net.ipv4.ip_forward = 1
```

#### **Aplicar configurações e reiniciar**
```bash
sudo sysctl -p
```
Aplica as alterações do arquivo de configuração e reinicia o servidor.

---

## Notas Finais
Após reiniciar, valide se o CoreDNS inicializa corretamente e todos os componentes do cluster estão funcionando como esperado.


