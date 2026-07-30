# Etapa 4 — unitree_ros2: o robô no notebook 🔜

!!! warning "Planejada — será preenchida quando validarmos ao vivo"

**Objetivo:** rodar `ros2 topic list` **dentro do container `casa`** e ver os tópicos do robô (`/lowstate`, `/sportmodestate`, `/utlidar/cloud`...) — ou seja, o notebook lendo o Go2 via ROS 2.

**Plano (no container `casa`):**

1. Instalar dependências: `ros-humble-rmw-cyclonedds-cpp`, `ros-humble-rosidl-generator-dds-idl`, `libyaml-cpp-dev`, `git`.
2. Clonar `unitreerobotics/unitree_ros2` em `/root/rima_ws`; dentro de `cyclonedds_ws/src`, clonar `rmw_cyclonedds -b humble` e `cyclonedds -b releases/0.10.x`.
3. **Ponto crítico:** compilar o cyclonedds **sem** ter dado `source` no ROS antes (`colcon build --packages-select cyclonedds`); só depois `source /opt/ros/humble/setup.bash` e `colcon build` do resto.
4. Ajustar o `setup.sh` do repo: humble + caminho do workspace + **interface dinâmica** (o nome muda a cada replug):
   ```bash
   export IF_ROBO=$(ip -4 -o addr show | awk '/192.168.123.99/{print $2}')
   ```
5. Teste de aceite planejado: `ros2 topic list` mostrando os tópicos do robô + `ros2 topic hz /utlidar/cloud` com taxa estável.
