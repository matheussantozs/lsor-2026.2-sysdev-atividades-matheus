# Laboratório de Redes: Endereçamento IP e Conectividade

## 🎯 Objetivo
Estudos sobre endereço IP e utilização de comandos/aplicativos para diagnóstico de redes no Linux.

---

## 🛠️ Procedimento

Comandos executados em ordem cronológica:

1. `ip addr show`
2. `ip link`
3. `ip route`
4. `netplan status`
5. `ping -c 4 127.0.0.1`
6. `ping -c 4 10.0.2.2`
7. `ping -c 4 8.8.8.8`
8. `ping -c 4 google.com`

---

## 📝 Questões e Respostas

### **Pergunta 1**
Identificação dos parâmetros de rede do host:
* **Nome da Interface:** `enp0s3`
* **IP/Prefixo (IPv4):** `10.0.2.15/24`
* **Endereço IPv6:** `fd17:625c:f037:2:a00:27ff:fe81:4393/64`
* **Endereço MAC:** `08:00:27:81:43:93`
* **Gateway Padrão:** `10.0.2.2`

---

### **Pergunta 2**
**Por que o host não alcança o IP `10.0.2.15` no modo NAT?**  
O endereço `10.0.2.15` existe apenas dentro de uma rede virtual privada emulada pelo motor NAT do VirtualBox. O sistema operacional do *Host* não possui uma interface de rede configurada nessa sub-rede nem uma rota para ela, tornando esse IP inacessível diretamente para a placa física do *Host*.

---

### **Pergunta 3**
**Análise da execução do comando `ping`:**  
Não houve problema algum; todos os testes obtiveram resposta com sucesso.

---

### **Pergunta 4**
**Diferença funcional entre NAT e Bridge:**  
No modo **NAT**, a VM opera numa sub-rede privada e isolada (`10.0.2.15/24`), acedendo à rede externa através de um gateway virtual (`10.0.2.2`). Este funcionamento permite que a VM navegue na Internet, mas impede que o *Host* aceda diretamente à VM. Para obter comunicação bidirecional direta sem configurar o redirecionamento de portas, deve alterar-se a interface para o modo **Bridge**.
