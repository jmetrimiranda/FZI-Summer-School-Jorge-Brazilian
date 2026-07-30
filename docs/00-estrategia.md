# Etapa 0 — Estratégia

Antes de qualquer comando, as decisões de arquitetura — cada uma derivada de um fato verificado.

## Fatos → decisões

| Fato | Decisão |
|---|---|
| O evento usa **ROS 2 Kilted + Zenoh**; a organização fornece um container pronto | Dominar **Docker** é pré-requisito, não luxo |
| Kilted só tem pacotes oficiais para **Ubuntu 24.04**; o notebook roda 26.04 | **Nenhum ROS instalado no host** — tudo em containers |
| O Go2 fala **CycloneDDS 0.10.2** nativamente; Zenoh **não** enxerga DDS | Dois ambientes separados |

## Os dois containers

| Container | Imagem | Para quê |
|---|---|---|
| `escola` | `osrf/ros:kilted-desktop` | Treinar o stack do evento (Zenoh, router, drills) |
| `casa` | `osrf/ros:humble-desktop` | Falar com o **seu** Go2 (unitree_ros2 + CycloneDDS) |

## Convenções usadas no guia inteiro

- **`<IF_ROBO>`** = interface ethernet ligada ao Go2. No Alienware, o adaptador USB-C **troca de nome/MAC a cada replug** — nunca escreva o nome fixo; com a conexão `go2` ativa, descubra assim:

```bash
IF_ROBO=$(ip -4 -o addr show | awk '/192.168.123.99/{print $2}')
echo $IF_ROBO
```

- **Regra do prompt** (a habilidade mais usada da semana): `jorgemetri@...` = **host** → comandos `docker ...`; `root@...` = **dentro do container** → comandos `ros2 ...`.
- Containers **sempre** com `--net=host` (o porquê está no [curso de Docker](02-docker-basico.md#por-que-nethost)).
