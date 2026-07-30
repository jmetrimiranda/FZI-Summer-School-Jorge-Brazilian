# RimA Summer School — Go2 Prep

Guia vivo da preparação de um **Unitree Go2 EDU** para a **RimA Summer School** (FZI Research Center / KIT, Karlsruhe, 03–07.08.2026). Tudo aqui foi **executado e validado de verdade** num Alienware com Ubuntu 26.04 — inclusive os erros, que viraram a página de [Troubleshooting](troubleshooting.md).

!!! info "O contexto que define tudo"
    - Os robôs do evento (Spot e Go2s) rodam **ROS 2 Kilted + Zenoh (rmw_zenoh)**, e a organização fornece um container pronto para falar com eles.
    - **Kilted não existe para Ubuntu 26.04** (só 24.04) → nada de ROS no host: **tudo via Docker**.
    - O Go2 "de casa" fala **CycloneDDS 0.10.2** nativo → ambiente separado (Humble) para ele.

## Como usar este guia

Cada etapa termina com um bloco **✅ Teste de aceite**: rode e compare com a saída esperada. Se não bater, cada teste tem um expansível **"Deu errado?"** apontando para a solução na página de [Troubleshooting](troubleshooting.md). Só avance com o teste verde — foi assim que esta preparação inteira foi feita.

## Progresso

| Etapa | Status |
|---|---|
| 0 — Estratégia e decisões | ✔ 30/07 |
| 1 — Rede cabeada + SSH + reconhecimento do Jetson | ✔ 30/07 |
| 2 — Docker instalado + imagens | ✔ 30/07 |
| 3 — Containers `casa` (Humble) e `escola` (Kilted) validados | ✔ 30/07 |
| 4 — unitree_ros2: o robô no `ros2 topic list` do notebook | ✔ 30/07 |
| 5 — SDK2: mover o robô por código | ✔ 30/07 |
| 6 — Sensores: LiDAR no RViz + câmera | ✔ 30/07 |
| 7 — Zenoh (modo escola) | ✔ 30/07 |
| 8 — Internet no robô (NAT) | ✔ 30/07 |
| 9 — Firmware OTA + backup do Jetson | 🔜 próxima |
| 10 — Ensaio geral cronometrado + kit | 🔜 |
| Apêndice — ponte sem fio via Zenoh | ⏸ estacionada 30/07 (pós-viagem) |

**Progresso geral: ~90%.** Técnica completa — e o robô com internet via NAT. Restam as duas etapas de logística: firmware/backup e kit + ensaio geral.

## Chegada e estudo

- **[Guia de Chegada — o Dia D](guia-chegada.md)** — o que deste site você vai consultar LÁ, na ordem dos acontecimentos.
- **[Nav2 essencial](estudo-nav2.md)** — a teoria (6 peças + behavior trees) e a prática (`BasicNavigator`) para acompanhar as aulas.
- **[Flashcards Anki](flashcards.md)** — 58 cartões por tópico, incluindo modo "digite o comando".

## Ambiente de referência

- Notebook: Alienware 16 Area-51, **Ubuntu 26.04 LTS** (Resolute Raccoon), Wi-Fi `wlp132s0f0`, adaptador USB-C→Ethernet (nome muda a cada replug — ver [Etapa 1](01-rede-go2.md)).
- Robô: **Unitree Go2 EDU** — controladora `192.168.123.161`, Jetson do dock `192.168.123.18` (Ubuntu 20.04 aarch64, ROS 2 Foxy embarcado).
- Docker `docker.io` 29.1.3; imagens `osrf/ros:humble-desktop` e `osrf/ros:kilted-desktop`.
