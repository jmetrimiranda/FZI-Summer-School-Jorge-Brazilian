# Etapa 3 — Containers `casa` e `escola` validados ✔

*Validada em 30/07: ROS funcional nos dois containers, GUI testada, e uma descoberta sobre DDS que vale a página.*

## 1. Liberar a GUI (uma vez por sessão/boot)

```bash
xhost +local:docker
```

Esqueceu depois de um reboot e o RViz não abre? [É isso](troubleshooting.md#xhost-cada-boot).

## 2. Criar os containers (uma vez cada)

```bash
mkdir -p ~/rima_ws
docker run -it --net=host --ipc=host --privileged \
  -e DISPLAY -e QT_X11_NO_MITSHM=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/rima_ws:/root/rima_ws \
  --name casa osrf/ros:humble-desktop
```

Para o `escola`: mesmo comando, trocando `--name escola` e a imagem por `osrf/ros:kilted-desktop`.

## 3. Dia a dia

```bash
docker start -ai casa       # retomar (prompt vira root@... = você está dentro)
docker exec -it casa bash   # 2º/3º terminal no mesmo container
docker ps -a                # ver estados
```

!!! success "✅ Teste de aceite — ROS vivo em cada container"
    Terminal A (dentro do container):
    ```bash
    source /opt/ros/humble/setup.bash    # ou /opt/ros/kilted/ no escola
    ros2 run demo_nodes_cpp talker       # esperado: Publishing: 'Hello World: 1', 2, ...
    ```
    Terminal B (host → `docker exec -it casa bash`, depois source):
    ```bash
    ros2 run demo_nodes_cpp listener     # esperado: I heard: [Hello World: N]
    ```

!!! success "✅ Teste de aceite — GUI"
    ```bash
    docker exec -it casa bash -c "source /opt/ros/humble/setup.bash && rviz2"
    ```
    Esperado: a **janela do RViz abre na tela**. No log, `libGL error ... nouveau` seguido de `OpenGl version: 4.5` é **sucesso** — [entenda](troubleshooting.md#rviz-libgl).

??? failure "Deu errado?"
    - `Conflict. The container name ... already in use` → [o container já existe](troubleshooting.md#container-name-conflict)
    - `No executable found` → [typo ou pacote errado](troubleshooting.md#no-executable-found)
    - Listener mudo → [precisa do talker rodando em par](troubleshooting.md#listener-mudo)
    - `permission denied ... docker.sock` → [grupo/sessão](troubleshooting.md#docker-permission-denied)

## O fenômeno: Humble ↔ Kilted conversando sozinhos {#o-fenomeno}

Durante a validação, o listener rodando no `escola` (**Kilted**) começou a imprimir as mensagens do talker do `casa` (**Humble**) — versões diferentes de ROS, containers diferentes, sem nenhuma configuração de ponte.

Por quê: os dois usam `--net=host` (mesma pilha de rede), o mesmo `ROS_DOMAIN_ID` padrão (0) e o mesmo middleware default (Fast DDS). Para o DDS, eram dois processos vizinhos na mesma máquina — e DDS descobre vizinhos automaticamente.

**Lições práticas:**

1. Numa sala de evento com 20 notebooks na mesma rede, o DDS de todo mundo se mistura. Antídoto clássico: cada pessoa/equipe com seu `export ROS_DOMAIN_ID=N`.
2. É exatamente uma das dores que o **Zenoh** resolve: lá, o escopo de comunicação é definido por **routers conectados explicitamente**, não por descoberta multicast automática. Vivemos na prática o problema antes de aprender a solução — ver [Etapa 7](07-zenoh.md).
