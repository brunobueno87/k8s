# Documentação dos Comandos para Configurar um Node Master no Kubernetes

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
sudo echo 1 | /proc/sys/net/ipv4/ip_forward
```
Habilita o encaminhamento de pacotes IPv4 para permitir comunicação entre pods.


#### **Caso queira deixar a mudança permanentemente no seu worker
```bash
sudo vim /etc/sysctl.conf
```

E adicione ou edite, se já existir:
```bash
net.ipv4.ip_forward=1
```

Depois aplica sem reiniciar com:
```bash
sudo sysctl -p
```

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

### 5. Fazer o Join do node master do Cluster Kubernetes

#### **Join no node master do o cluster**
```bash
sudo kubeadm join 192.168.0.101:6443 --token 8k5bex.fqbaekqoi136l4re  --discovery-token-ca-cert-hash sha256:94713c894bd6442122f5757cc6d9a869bc19c53c98f8abef8c3132a2c7036670 
```


