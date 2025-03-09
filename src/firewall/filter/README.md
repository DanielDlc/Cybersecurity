# Testes de Firewall Filter no Kali Linux com iptables

## 1️⃣ Introdução ao iptables

O `iptables` é a ferramenta padrão para configurar firewalls no Linux. Ele permite definir **regras de filtragem de pacotes** para controlar o tráfego de entrada, saída e passagem pelo sistema.

### 🔹 **Por que utilizar o iptables?**
- Controle granular de tráfego na rede.
- Bloqueio de acessos não autorizados.
- Aplicação de políticas de segurança personalizadas.
- Implementação de regras para evitar ataques.

### **Visualizar Regras Atuais**
Para exibir as regras de firewall ativas no Kali Linux:
```bash
sudo iptables -L -v -n
```
📌 Isso exibe as regras ativas e o tráfego sendo permitido ou bloqueado.

### **Verificar Endereço IP**
```bash
ip -4 a
```
📌 Isso exibe informações sobre o endereço IP da máquina.

Saída esperada:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.66.3/24 brd 192.168.66.255 scope global dynamic noprefixroute eth0
       valid_lft 1905sec preferred_lft 1905sec
```

### **Zerar a Interface `eth1` caso necessário**
Se precisar remover a interface virtual criada:
```bash
sudo ip link delete eth1
```
📌 Isso remove a interface `eth1` e exibe a rede atual sem ela.

### **Verificar e Ativar o SSH**
Verificar se o SSH está rodando no Kali:
```bash
sudo systemctl status ssh
```
Ativar o SSH:
```bash
sudo systemctl start ssh
```
📌 Isso permite conexões SSH ao Kali.

---

## 2️⃣ Filtragem de Pacotes (`Filter`)
Os firewalls realizam filtragem de pacotes através de três canais principais:

### **1️⃣ Canal Input – Controla o tráfego que entra no Kali**
📌 **Objetivo:** Bloquear conexões SSH para impedir que outros dispositivos se conectem ao Kali.

✅ **Como permitir novamente conexões SSH:**
```bash
sudo iptables -D INPUT -p tcp --dport 22 -j DROP
```

🔹 **Teste:** Tente acessar o Kali de outro computador via SSH:
```bash
ssh user@192.168.66.3
```
📌 **Resultado esperado:** A conexão será permitida.

❌ **Comando para bloquear conexões SSH (porta 22):**
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```
📌 Agora, as conexões SSH serão bloqueadas.

---

### **2️⃣ Canal Output – Controla o tráfego originado do Kali**
📌 **Objetivo:** Bloquear o envio de pacotes ICMP (ping) para a internet.

❌ **Comando para impedir pings saindo do Kali:**
```bash
sudo iptables -A OUTPUT -p icmp --icmp-type echo-request -j DROP
```

🔹 **Teste:** No Kali, tente pingar um servidor externo:
```bash
ping -c 5 8.8.8.8
```
📌 **Resultado esperado:** O ping não será enviado.

✅ **Como permitir novamente pings de saída:**
```bash
sudo iptables -D OUTPUT -p icmp --icmp-type echo-request -j DROP
```

---

### **3️⃣ Canal Forward – Controla o tráfego que atravessa o Kali**
📌 **Objetivo:** Bloquear o acesso da rede interna a um site específico.

#### **1️⃣ Exibir a Rede Atual**
```bash
ip -4 a
```

#### **2️⃣ Criar uma Interface Virtual para Simular o Roteamento**
```bash
sudo ip link add name eth1 type dummy
sudo ip addr add 192.168.100.1/24 dev eth1
sudo ip link set eth1 up
```

#### **3️⃣ Habilitar o Encaminhamento de Pacotes**
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

#### **4️⃣ Testar o Acesso a um Site sem Bloqueio**
```bash
curl -I --max-time 5 http://example.com
```

#### **5️⃣ Bloquear o Acesso via FORWARD**
```bash
sudo iptables -A FORWARD -d example.com -j DROP
```

#### **6️⃣ Testar Acesso Novamente (Agora Bloqueado)**
```bash
curl -I --max-time 5 http://example.com
```

✅ **Para remover a regra e restaurar o acesso:**
```bash
sudo iptables -D FORWARD -d example.com -j DROP
```

#### **7️⃣ Remover a Interface Virtual e Exibir as Interfaces Atuais**
```bash
sudo ip link delete eth1
ip -4 a
```

---

## 3️⃣ Finalizando os Testes

### **🔍 Verificando regras ativas**
```bash
sudo iptables -L -v -n
```

### **🔄 Resetando todas as regras do firewall**
```bash
sudo iptables -F
sudo iptables -X
sudo iptables -Z
```
📌 **Isso limpa todas as regras e zera contadores.**

Agora, verifique se as regras foram apagadas:
```bash
sudo iptables -L -v -n
```
✅ **As chains devem aparecer vazias.**

---

## 4️⃣ Conclusão
O `iptables` insere as regras dentro do **Netfilter**, que roda dentro do **kernel Linux**.
Cada pacote que entra no sistema é comparado **sequencialmente** com as regras da chain correspondente (`INPUT`, `OUTPUT`, `FORWARD`).
O **Netfilter percorre a lista** até encontrar uma regra correspondente **ou chegar ao final** e aplicar a **policy padrão** (normalmente `ACCEPT`).

Este documento demonstra como utilizar `iptables` no Kali Linux para **controlar tráfego de entrada (Input), saída (Output) e passagem (Forward)**. Os testes ilustram o uso do firewall para garantir maior controle e segurança na rede.