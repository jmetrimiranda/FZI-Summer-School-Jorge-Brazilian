# Etapa 1 — Rede cabeada com o Go2 ✔

*Validada em 30/07. Objetivo: notebook e robô conversando por cabo, com SSH sem senha no Jetson.*

## Mapa de IPs (subrede fixa do Go2)

| IP | Quem |
|---|---|
| `192.168.123.161` | Controladora do robô |
| `192.168.123.18` | Jetson do dock EDU |
| `192.168.123.99` | Seu notebook (você define) |

## 1. Perfil de rede no notebook

O perfil é criado **sem amarrar ao nome da interface** — o adaptador USB-C gera um MAC novo a cada replug, e o Linux nomeia a placa pelo MAC (`enx` + MAC). Amarrar ao nome quebra no replug seguinte (aconteceu: `enx00e04c361b96` virou `enx0c3796dc8be5`).

```bash
nmcli con add type ethernet con-name go2 ipv4.method manual ipv4.addresses 192.168.123.99/24
nmcli con up go2      # com o cabo plugado no robô ligado
```

Se o perfil já existia amarrado a um nome antigo:

```bash
nmcli con mod go2 connection.interface-name ""
nmcli con up go2
```

!!! success "✅ Teste de aceite — enxergar o robô"
    ```bash
    ip a | grep 192.168.123.99   # o IP aparece numa interface enx...
    ping -c3 192.168.123.161     # esperado: respostas ~0.4 ms
    ping -c3 192.168.123.18      # esperado: respostas ~0.4 ms
    ```

??? failure "Deu errado?"
    - `No suitable device found ... mismatching interface name` → [o nome da interface mudou](troubleshooting.md#iface-mudou)
    - `From <IP estranho> Time to live exceeded` → [o pacote saiu pela internet, não pelo cabo](troubleshooting.md#ttl-exceeded)
    - `.18` não responde mas `.161` sim → [Jetson ainda bootando / dock](troubleshooting.md#jetson-nao-responde)

## 2. SSH no Jetson + chave (nunca mais digitar senha)

```bash
ssh unitree@192.168.123.18    # senha: 123 — aceite o fingerprint com "yes"
exit                          # volte ao notebook
ssh-keygen -t ed25519         # Enter em tudo (se ainda não tem chave)
ssh-copy-id unitree@192.168.123.18
```

O aviso `WARNING: connection is not using a post-quantum key exchange` é cosmético — ignore.

!!! success "✅ Teste de aceite — login sem senha"
    ```bash
    ssh unitree@192.168.123.18
    ```
    Esperado: cair direto no banner do Ubuntu 20.04, **sem pedir senha**, terminando no seletor `ros:foxy(1) noetic(2) ?`.

## 3. Reconhecimento do Jetson (o que existe lá dentro)

O Jetson roda Ubuntu 20.04 (aarch64/tegra) com **ROS 2 Foxy** e ROS 1 Noetic. No login, responda `1` (foxy). Depois:

```bash
source /opt/ros/foxy/setup.bash
ros2 topic list -t                        # ~120 tópicos nativos do robô
ros2 topic echo /sportmodestate --no-arr  # --no-arr esconde os arrays gigantes
```

Descobertas úteis: `/lowstate`, `/sportmodestate`, `/wirelesscontroller`, `/frontvideostream`, `/utlidar/cloud`, `/utlidar/imu`, `/utlidar/height_map`, `/utlidar/robot_odom` e `/uslam/*` (SLAM embarcado).

`ros2 node list` **vazio é normal** — [entenda o porquê](troubleshooting.md#node-list-vazio).

!!! danger "NÃO rode `apt upgrade` no Jetson"
    O banner oferece 242 updates. É isca: kernel tegra + stack Unitree podem quebrar a dias do evento. Atualização de robô, só a oficial via app Unitree Go (OTA).

## 4. Nota para o evento

O perfil `go2` captura **qualquer** ethernet. Antes de plugar o notebook numa tomada de rede da FZI (DHCP), rode `nmcli con down go2`; para voltar ao robô, `nmcli con up go2`.
