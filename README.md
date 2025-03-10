# vpn
# Configuración de VPN Sitio a Sitio con IPsec en Cisco Packet Tracer

## 📌 1. Topología de Red

**Dispositivos:**
- 2 Routers (**Cisco 2911** o **Cisco 1941**)
- 2 Switches (**Cisco 3560** o **Cisco 2960**)
- 2 PCs

**Conexiones:**
1. `PC0 → Switch0 → Router0 (G0/0 o F0/0)`
2. `PC1 → Switch1 → Router1 (G0/0 o F0/0)`
3. `Router0 (F0/1) ↔ Router1 (F0/1) [Crossover]`

**Direcciones IP:**

| Dispositivo  | Interfaz         | IP            | Máscara         | Descripción    |
|-------------|----------------|--------------|----------------|--------------|
| PC0         | NIC            | 192.168.10.2  | 255.255.255.0  | Red local 1  |
| Router0     | G0/0 o F0/0     | 192.168.10.1  | 255.255.255.0  | Gateway PC0  |
| Router0     | F0/1           | 10.0.0.1     | 255.255.255.252 | Enlace VPN   |
| Router1     | F0/1           | 10.0.0.2     | 255.255.255.252 | Enlace VPN   |
| Router1     | G0/0 o F0/0     | 192.168.20.1  | 255.255.255.0  | Gateway PC1  |
| PC1         | NIC            | 192.168.20.2  | 255.255.255.0  | Red local 2  |

---

## ⚙️ 2. Configuración de los Routers

### Router0
```bash
enable
configure terminal
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface FastEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

crypto isakmp policy 10
 authentication pre-share
 hash sha
 encryption aes 256
 group 2
 lifetime 86400
exit

crypto isakmp key toor address 10.0.0.2

crypto ipsec transform-set TSET esp-aes esp-sha-hmac
exit

access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255

crypto map CMAP 10 ipsec-isakmp
 set peer 10.0.0.2
 match address 101
 set transform-set TSET
exit

interface FastEthernet0/1
 crypto map CMAP
exit

do wr
```

### Router1
```bash
enable
configure terminal
interface GigabitEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

interface FastEthernet0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

crypto isakmp policy 10
 authentication pre-share
 hash sha
 encryption aes 256
 group 2
 lifetime 86400
exit

crypto isakmp key toor address 10.0.0.1

crypto ipsec transform-set TSET esp-aes esp-sha-hmac
exit

access-list 101 permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255

crypto map CMAP 10 ipsec-isakmp
 set peer 10.0.0.1
 match address 101
 set transform-set TSET
exit

interface FastEthernet0/1
 crypto map CMAP
exit

do wr
```

---

## ✅ 3. Verificación de la VPN

### Verificar ISAKMP
```bash
show crypto isakmp sa
```
🔹 Debe mostrar **QM_IDLE** si la VPN está activa.

### Verificar IPsec
```bash
show crypto ipsec sa
```
🔹 Debe mostrar paquetes **cifrados** y **descifrados**.

### Probar conectividad
📌 Desde **PC0 (192.168.10.2)** prueba **ping** a **PC1 (192.168.20.2)**:
```bash
ping 192.168.20.2
```
✔️ **Si responde, la VPN está funcionando correctamente.**
