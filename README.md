# conjur-oss-k3s

Este repositório contém um playbook para a instalação do **Conjur OSS** em um ambiente K3s.

O **Conjur OSS (Open Source)** é uma solução de gerenciamento de segredos e controle de acesso projetada para ambientes DevOps, contêineres e nuvens. Ele ajuda a proteger credenciais, chaves de API e outros segredos sensíveis, garantindo que apenas identidades autorizadas possam acessá-los.

---

## Pré-requisitos

1. Atualize os pacotes do sistema:
   ```bash
   sudo apt update
   ```
2. Caso não tenha o ansible é necessário instalar:
   ```bash
    sudo apt install ansible unzip git -y
   ```
4. Instale as dependências necessárias:
   ```bash
    sudo apt install ansible unzip git python3-kubernetes -y
   ```
5. O **MetalLB** é o balanceador de carga necessário para fornecer suporte a IPs estáticos em clusters Kubernetes em ambientes bare-metal. Consulte a documentação oficial: [Documentação Oficial do MetalLB](https://metallb.io)

## Instalação do MetalLB

* Certifique-se de que o K3s está instalado e funcionando.

1. Aplique o manifesto do MetalLB
```bash
 kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/manifests/metallb.yaml
```

**Configure o Pool de IPs:** Crie um arquivo YAML para definir a faixa de IPs que o MetalLB usará:
```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: my-ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.33.200-192.168.33.210  # Substitua pelo intervalo de IPs da sua rede

---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: my-l2-advertisement
  namespace: metallb-system
spec: {}
```
**Descubra sua faixa de IPs:** Use o comando **ip addr**, que mostrará algo como **192.168.100.10/24**. Substitua o intervalo no arquivo acima por algo adequado à sua rede.

## Executando o Playbook

Após configurar os pré-requisitos, execute o playbook com o seguinte comando:
```bash
 ansible-playbook install-conjur-oss.yaml --ask-become-pass
```
## Comandos Úteis para Validação
1. Verificar os pods no namespace:
```bash
 kubctl get pods -n conjur-oss
NAME                                         READY   STATUS    RESTARTS  AGE
conjur-cluster-conjur-oss-6654b47b44-skp9f   2/2     Running   0         18h
conjur-cluster-postgres-0                    1/1     Running   0         18h
```
2. Verificar o Ingress:
```bash
 kubctl get ingress -n conjur-oss
NAME             CLASS     HOSTS        ADDRESS          PORTS     AGE
conjur-ingress   traefik   conjur.poc   192.168.33.200   80, 443   14h
```
3. Certifique-se de que o DNS foi adicionado no arquivo /etc/hosts:
```bash
 cat /etc/hosts
127.0.0.1 localhost
192.168.33.200 conjur.poc
```
** Esse IP deve corresponder ao obtido com:
```bash
 kubectl get ingress conjur-ingress -n conjur-oss
NAME             CLASS     HOSTS        ADDRESS          PORTS     AGE
conjur-ingress   traefik   conjur.poc   192.168.33.200   80, 443   14h
```
Com isso, seu ambiente estará pronto para usar o Conjur OSS! 🚀
