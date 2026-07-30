# Etapa 4 — unitree_ros2: o robô no notebook ✔

*Validada em 30/07 — o notebook lendo a telemetria viva do Go2 via ROS 2, de dentro do container `casa`. LiDAR chegando a **15.34 Hz** com desvio de 0.8 ms.*

!!! info "Como isso funciona (a corrente inteira)"
    Container **não é** máquina virtual: é um processo do próprio Linux, no mesmo kernel — e com `--net=host` ele usa **as mesmas interfaces de rede do host**. A cadeia: cabo → interface do host → (`--net=host`) a mesma interface dentro do container → CycloneDDS 0.10.x compilado (o "dialeto" exato do robô) → `CYCLONEDDS_URI` apontando a descoberta para essa interface → robô e container se descobrem como participantes DDS na subrede `192.168.123.x` → tópicos fluem.

## 1. Ambiente limpo + dependências + clones

O build do cyclonedds exige um shell **sem** ROS carregado. O terminal do `docker start -ai` já nasce com ROS (o entrypoint da imagem faz source automático); o de `docker exec` nasce limpo — use ele:

```bash
docker start casa            # liga em segundo plano
docker exec -it casa bash    # shell limpo (NÃO dê source!)
printenv | grep -E 'AMENT|CMAKE_PREFIX'
```

!!! success "✅ Teste de aceite — ambiente limpo"
    Saída **vazia** no `printenv` acima (dentro do container).

```bash
apt update && apt install -y iproute2 ros-humble-rmw-cyclonedds-cpp ros-humble-rosidl-generator-dds-idl libyaml-cpp-dev python3-colcon-common-extensions nano
cd /root/rima_ws
git clone https://github.com/unitreerobotics/unitree_ros2
cd unitree_ros2/cyclonedds_ws/src
git clone https://github.com/ros2/rmw_cyclonedds -b humble
git clone https://github.com/eclipse-cyclonedds/cyclonedds -b releases/0.10.x
cd ..
```

Na imagem `humble-desktop`, as dependências ROS já vinham instaladas ("0 newly installed") — só o `iproute2` (comando `ip`) foi novidade.

## 2. Os dois builds, na ordem crítica

```bash
colcon build --packages-select cyclonedds     # SEM source do ROS antes!
```

!!! success "✅ Teste de aceite — build 1"
    `Summary: 1 package finished` (validado em ~4 s).

```bash
source /opt/ros/humble/setup.bash
colcon build
```

!!! success "✅ Teste de aceite — build 2"
    `Summary: 5 packages finished` — `cyclonedds`, `rmw_cyclonedds_cpp`, `unitree_api`, `unitree_go`, `unitree_hg`, nenhum `failed`.

## 3. setup.sh com interface dinâmica

Versão nossa, melhorada em relação à do repositório: humble, caminhos do nosso workspace e **descoberta automática da interface pelo IP** — resolve de vez o adaptador USB-C que troca de nome/MAC a cada replug:

```bash
cat > /root/rima_ws/unitree_ros2/setup.sh << 'FIM'
#!/bin/bash
echo "Setup unitree ros2 (humble + interface dinamica)"
source /opt/ros/humble/setup.bash
source /root/rima_ws/unitree_ros2/cyclonedds_ws/install/setup.bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
IF_ROBO=$(ip -4 -o addr show | awk '/192.168.123.99/{print $2}')
if [ -z "$IF_ROBO" ]; then echo "AVISO: nenhuma interface com 192.168.123.99 - conexao go2 ativa no host?"; fi
export CYCLONEDDS_URI='<CycloneDDS><Domain><General><Interfaces><NetworkInterface name="'$IF_ROBO'" priority="default" multicast="default" /></Interfaces></General></Domain></CycloneDDS>'
echo "IF_ROBO=$IF_ROBO"
FIM
source /root/rima_ws/unitree_ros2/setup.sh
```

**Regra permanente:** todo terminal novo que for falar com o robô começa com `source /root/rima_ws/unitree_ros2/setup.sh`.

!!! success "✅ Teste de aceite — interface encontrada"
    A linha `IF_ROBO=enx...` (ou `wlp...`, se via roteador) impressa com um nome real.

??? failure "Deu errado?"
    - `IF_ROBO=` vazio → [conexão go2 não está ativa no host](troubleshooting.md#if-robo-vazio)
    - Adaptador nem aparece no `nmcli device status` → [foi desplugado](troubleshooting.md#adaptador-sumiu)

## 4. O teste-troféu

```bash
ros2 topic list                             # os tópicos do ROBÔ, no notebook
ros2 topic echo /sportmodestate --no-arr    # telemetria viva (Ctrl+C p/ sair)
ros2 topic hz /utlidar/cloud                # taxa do LiDAR
```

!!! success "✅ Teste de aceite — validado com estes números"
    `/lowstate`, `/sportmodestate`, `/utlidar/cloud`... na lista; o echo mostrando `body_height: ~0.311`, IMU `temperature: 79` (normal — o chip roda quente); e o hz cravado: `average rate: 15.34 — std dev: 0.0008s`.

??? failure "Deu errado?"
    - `topic list` vazio → [checklist do CycloneDDS](troubleshooting.md#topic-list-vazio-cyclone)

## E se não for cabo? (roteador / Wi-Fi)

O DDS não liga para o meio físico — liga para estar **na mesma subrede IP**. Com o roteador de viagem como switch (robô numa porta LAN, notebook em outra ou no Wi-Fi dele), LAN configurada como `192.168.123.0/24`, **nada muda no software** — e como o `setup.sh` acha a interface *pelo IP*, ele encontra sozinho até a `wlp...` do Wi-Fi. Ressalva: descoberta multicast sobre Wi-Fi é menos confiável que cabo — cabo segue sendo o caminho de menor erro. No evento, o Zenoh da FZI conecta por TCP a um router e nem depende de multicast ([Etapa 7](07-zenoh.md)).
