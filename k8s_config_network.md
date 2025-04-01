sudo vim /etc/netplan/50-cloud-init.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:  # Substitua pelo nome da sua interface
      dhcp4: no
      addresses:
        - 192.168.0.100/24  # Defina seu IP fixo
      gateway4: 192.168.0.1  # IP do seu roteador/gateway
      nameservers:
        addresses:
          - 8.8.8.8  # Servidor DNS do Google
          - 1.1.1.1  # Servidor DNS da Cloudflare

========================================================

sudo netplan apply
