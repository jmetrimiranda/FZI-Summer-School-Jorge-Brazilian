# Etapa 8 — Internet no robô (NAT pelo notebook) 🔜

!!! warning "Planejada — será preenchida quando validarmos ao vivo"

**Objetivo:** o Jetson do robô acessando a internet através do Wi-Fi do notebook (para `git clone`/`pip` dentro do robô).

**Plano:** `nmcli con mod go2 ipv4.method shared ipv4.addresses 192.168.123.99/24` + `nmcli con up go2`; no Jetson: `sudo ip route replace default via 192.168.123.99` + DNS em `/etc/resolv.conf`. Aceite: `ping 8.8.8.8` e `ping google.com` respondendo **do Jetson**. Plano B: roteador de viagem pré-configurado com LAN `192.168.123.0/24` (IP `.1`, DHCP excluindo `.18/.99/.161`).
