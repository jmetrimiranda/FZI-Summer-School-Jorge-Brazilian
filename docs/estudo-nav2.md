# Estudo — Nav2 essencial (leitura de véspera)

*Por que isto: a escola gira em torno de robôs andando em ambientes domésticos com 3D mapping, semântica, ROS 2 e behavior trees — e Nav2 é O stack de navegação do ROS 2, o vocabulário das aulas de terça/quarta.*

## O mapa mental em 6 peças

Pensa numa pessoa atravessando uma casa desconhecida:

1. **Mapa** — a planta da casa (no 2D clássico: *occupancy grid*, células livres/ocupadas; na quarta-feira o tema é a versão **3D/volumétrica** disso).
2. **Localização** — "onde EU estou no mapa?" (clássico: **AMCL**, filtro de partículas: milhares de palpites que convergem conforme o laser bate com o mapa; alternativa: SLAM constrói o mapa enquanto localiza).
3. **Costmaps** — o mapa "com medo": cada obstáculo ganha uma auréola de custo (**inflação**) para o robô planejar com folga. São dois: **global** (a casa toda, atualização lenta) e **local** (a janela ao redor do robô, atualização rápida com os sensores ao vivo).
4. **Planner** — desenha o **caminho global** no costmap global (A*/Smac/NavFn: "vá por este corredor").
5. **Controller** — segue o caminho **localmente**, gerando velocidades a cada instante e desviando do que aparecer (DWB, RPP).
6. **Behaviors/Recoveries** — o plano B quando trava: limpar costmap, girar no lugar, dar ré, esperar.

E quem rege a orquestra é o **BT Navigator**: uma *behavior tree* decide quando planejar, quando seguir, quando recuperar.

## Behavior trees em 3 palavras

- **Ação** — uma folha que faz algo (planejar, seguir, girar) e devolve sucesso/falha/rodando.
- **Sequência (→)** — executa os filhos em ordem; **falha se um falhar** ("planeje E siga").
- **Fallback (?)** — tenta o próximo filho **quando um falha** ("siga; se não der, recupere; se não der, desista").

A árvore padrão do Nav2 lê-se assim: *"[planeje o caminho → siga o caminho]; se qualquer parte falhar → [limpe o costmap → gire → dê ré] e tente de novo"*. Quando o professor mostrar um XML de BT, você vai reconhecer esses três blocos.

Uma frase sobre **lifecycle nodes**: os nós do Nav2 têm estados (configurar → ativar) — por isso existe um "bringup" que liga tudo na ordem certa.

## A interface que você vai usar na prática

Navegar = mandar uma **action** `NavigateToPose` (goal: uma `PoseStamped` no frame `map`; feedback: distância restante; resultado: chegou/falhou). Reconhecimento no terminal (funciona contra qualquer stack Nav2 ativo, como o da escola):

```bash
ros2 action list
ros2 interface show nav2_msgs/action/NavigateToPose
```

## Prática 1 — o jeito de apontar com o mouse

No RViz com Nav2 ativo: botão **"Nav2 Goal"** (ou "2D Goal Pose") → **clica no destino e ARRASTA** para dar a orientação final. O robô planeja e vai. É o primeiro contato clássico.

## Prática 2 — o jeito de apontar com código

O `nav2_simple_commander` embrulha a action numa classe amigável — este é o esqueleto que você provavelmente usará nos exercícios:

```python
import rclpy
from geometry_msgs.msg import PoseStamped
from nav2_simple_commander.robot_navigator import BasicNavigator, TaskResult

rclpy.init()
nav = BasicNavigator()
nav.waitUntilNav2Active()            # espera o stack (lifecycle) ficar ativo

goal = PoseStamped()
goal.header.frame_id = 'map'         # destino no frame do MAPA
goal.header.stamp = nav.get_clock().now().to_msg()
goal.pose.position.x = 2.0
goal.pose.position.y = 0.5
goal.pose.orientation.w = 1.0        # orientacao final (quaternion)

nav.goToPose(goal)
while not nav.isTaskComplete():
    fb = nav.getFeedback()
    print(f"faltam {fb.distance_remaining:.2f} m")

print("sucesso?", nav.getResult() == TaskResult.SUCCEEDED)
```

Lendo: `waitUntilNav2Active` respeita o lifecycle; o goal é uma pose **no mapa** (o TF da [Etapa 6](06-sensores.md#2-por-que-a-nuvem-parece-torta-frames-e-tf-o-conceito) fechando o círculo: `map → odom → base`); o laço de feedback é o mesmo padrão de "acompanhar uma tarefa longa" que você usou a semana toda.

## Se quiser simular em casa (opcional, pesado)

No container `escola`: `apt install ros-kilted-navigation2 ros-kilted-nav2-bringup` + um simulador (TurtleBot em Gazebo). É o tutorial "Getting Started" de **docs.nav2.org** — vale como projeto, não como véspera de viagem.

## O fio que liga tudo à escola

Segunda você conhece os robôs; terça, os *software stacks* (Nav2 estará lá dentro); quarta, a navegação 3D — os **costmaps/mapas volumétricos** desta página em versão gente-grande; e behavior trees aparecem descritos no próprio convite do evento. Chegar com estas 6 peças + 3 palavras na cabeça transforma as palestras de "chuva de siglas" em revisão.
