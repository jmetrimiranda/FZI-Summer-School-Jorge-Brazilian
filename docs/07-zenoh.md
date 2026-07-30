# Etapa 7 — Zenoh: o modo do evento 🔜

!!! warning "Planejada — será preenchida quando validarmos ao vivo. Comandos abaixo vêm da documentação oficial do rmw_zenoh, ainda não executados por nós."

**Objetivo:** dominar no container `escola` o fluxo que a FZI usa: Kilted + rmw_zenoh.

**O modelo mental** (diferente do DDS que vimos na [Etapa 3](03-containers.md#o-fenomeno)): nós **não** se descobrem sozinhos — cada máquina roda um **router** (`rmw_zenohd`), e máquinas se conectam ligando routers entre si, explicitamente.

**Plano (no `escola`):**

```bash
apt update && apt install -y ros-kilted-rmw-zenoh-cpp
source /opt/ros/kilted/setup.bash
export RMW_IMPLEMENTATION=rmw_zenoh_cpp
ros2 run rmw_zenoh_cpp rmw_zenohd          # terminal 1: o router (obrigatório!)
# terminais 2 e 3 (exec + source + export): talker / listener
```

Conectar ao router de outra máquina (porta padrão 7447):

```bash
ZENOH_CONFIG_OVERRIDE='connect/endpoints=["tcp/<IP_ROUTER>:7447"]' ros2 run rmw_zenoh_cpp rmw_zenohd
```

Ou um nó direto como cliente, sem router local:

```bash
ZENOH_CONFIG_OVERRIDE='mode="client";connect/endpoints=["tcp/<IP_ROUTER>:7447"]' ros2 run rviz2 rviz2
```

`ROS_DOMAIN_ID` continua valendo — use o mesmo em todas as pontas.
