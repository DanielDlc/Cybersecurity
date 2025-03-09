# Cybersecurity

UFCD 9195

# Testes de Firewall no Kali Linux com iptables

## 1️⃣ Introdução ao iptables

O `iptables` é a ferramenta padrão para configurar firewalls no Linux. Ele permite definir **regras de filtragem de pacotes** para controlar o tráfego de entrada, saída e passagem pelo sistema.

### 🔹 **Por que utilizar o iptables?**
- Controle granular de tráfego na rede.
- Bloqueio de acessos não autorizados.
- Aplicação de políticas de segurança personalizadas.
- Implementação de regras para evitar ataques.

Para visualizar as regras atuais no firewall do Kali, utilize:
```bash
sudo iptables -L -v -n
```
📌 **Isso exibe as regras ativas e o tráfego sendo permitido ou bloqueado.**

---

## 2️⃣ Filtragem de Pacotes (Filter)
Os firewalls realizam filtragem de pacotes através de três canais principais:

### **1️⃣ Canal Input – Controla o tráfego que entra no Kali**

📌 **Objetivo:** Bloquear conexões SSH para impedir que outros dispositivos se conectem ao Kali.

🔹 **Comando para bloquear conexões SSH (porta 22):**
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

🔹 **Teste:** Tente acessar o Kali de outro computador via SSH:
```bash
ssh user@192.168.66.3
```
📌 **Resultado esperado:** A conexão será rejeitada.

✅ **Como permitir novamente conexões SSH:**
```bash
sudo iptables -D INPUT -p tcp --dport 22 -j DROP
```

---

### **2️⃣ Canal Output – Controla o tráfego originado do Kali**

📌 **Objetivo:** Bloquear o envio de pacotes ICMP (ping) para a internet.

🔹 **Comando para impedir pings saindo do Kali:**
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

🔹 **Comando para impedir acesso a `example.com`:**
```bash
sudo iptables -A FORWARD -d example.com -j DROP
```

🔹 **Teste:** Em outro dispositivo na rede, tente acessar o site:
```bash
curl -I http://example.com
```
📌 **Resultado esperado:** O site não carregará.

✅ **Como liberar o acesso novamente:**
```bash
sudo iptables -D FORWARD -d example.com -j DROP
```

---

## 3️⃣ Finalizando os Testes

### 🔍 **Verificando regras ativas**
Para conferir todas as regras que estão ativas no firewall:
```bash
sudo iptables -L -v -n
```

### 🔄 **Removendo todas as regras do firewall**
Caso precise restaurar o firewall para o estado inicial, execute:
```bash
sudo iptables -F
```
📌 **Isso limpa todas as regras e permite todo o tráfego novamente.**

---

## 4️⃣ Conclusão
Este documento demonstra como utilizar `iptables` no Kali Linux para **controlar tráfego de entrada (Input), saída (Output) e passagem (Forward)**. Os testes ilustram o uso do firewall para garantir maior controle e segurança na rede.


