# Curso rápido — Docker (o mínimo que você precisa) ✔

*Instalação validada em 30/07 no Ubuntu 26.04. Esta página é a base conceitual de todo o resto do guia.*

## Os 3 conceitos

**Imagem** = a receita congelada (SO + programas instalados). `osrf/ros:humble-desktop` é uma imagem: Ubuntu 22.04 com ROS 2 Humble já dentro. Você **baixa** imagens do Docker Hub com `docker pull`.

**Container** = uma instância viva criada a partir de uma imagem — como objeto vs. classe. Nossos containers `casa` e `escola` nasceram das imagens acima e **guardam o que você instala dentro deles** entre paradas.

**Volume** (`-v pasta_host:pasta_container`) = uma pasta compartilhada entre host e container. Usamos `-v ~/rima_ws:/root/rima_ws`: o código escrito ali sobrevive até se o container for destruído.

## run vs. start vs. exec (o trio que confunde todo mundo)

| Comando | O que faz | Quando usar |
|---|---|---|
| `docker run ... --name X imagem` | **Cria** um container novo a partir da imagem | Uma vez por container |
| `docker start -ai X` | **Liga** um container existente que estava `Exited` | Toda vez que voltar ao trabalho |
| `docker exec -it X bash` | Abre **mais um terminal** dentro de um container que já está `Up` | 2º, 3º, 4º terminal |

Rodar `docker run` de novo com o mesmo `--name` dá o erro `Conflict: name already in use` — [é sinal de que o container já existe](troubleshooting.md#container-name-conflict); use `start`/`exec`.

## As flags do nosso comando padrão, explicadas

```bash
docker run -it --net=host --ipc=host --privileged \
  -e DISPLAY -e QT_X11_NO_MITSHM=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/rima_ws:/root/rima_ws \
  --name casa osrf/ros:humble-desktop
```

- `-it` → terminal interativo.
- `--net=host` → **ver abaixo**, é a flag mais importante.
- `-e DISPLAY` + `-v /tmp/.X11-unix...` → ponte gráfica: janelas (RViz) abrem na sua tela. Requer `xhost +local:docker` no host, **uma vez por sessão**.
- `-e QT_X11_NO_MITSHM=1` → evita um crash clássico de apps Qt em container.
- `-v ~/rima_ws:/root/rima_ws` → seu workspace persistente.
- `--privileged` → acesso a dispositivos do host (útil para GPU/USB).

### Por que --net=host {#por-que-nethost}

Sem essa flag, o container vive numa rede interna isolada (a bridge `docker0`) e **não enxerga** a interface do robô nem consegue fazer descoberta DDS/Zenoh direito. Com `--net=host`, o container usa as interfaces reais do notebook — para a rede, ele **é** o notebook. Consequência que observamos ao vivo: dois containers com `--net=host` se enxergam via DDS como se fossem processos vizinhos ([a história completa](03-containers.md#o-fenomeno)).

## Ciclo de vida e códigos de saída

`docker ps -a` mostra todos os containers e seus estados: `Up` (rodando), `Exited (0)` (saiu normal, ex.: você digitou `exit`), `Exited (127)` ([último comando não existia](troubleshooting.md#exit-127) — inofensivo), `Exited (130)` (Ctrl+C).

## A regra do prompt

!!! tip "Onde estou?"
    `jorgemetri@...:~$` → **host** → é aqui que vivem os comandos `docker ...`
    `root@...:/#` → **dentro do container** → é aqui que vivem `source` e `ros2 ...`
    Rodar `docker` dentro do container dá `command not found`; rodar `ros2` no host também — [por design](troubleshooting.md#ros2-nao-encontrado-host).

## Limpeza (quando precisar)

```bash
docker ps -a            # o que existe
docker rm NOME          # apagar um container parado (o que estiver fora de -v se perde!)
docker images           # imagens baixadas
docker system df        # quanto disco o Docker usa
```

## Instalação no Ubuntu 26.04 (como foi feita, validada)

```bash
sudo apt update && sudo apt install -y docker.io
sudo usermod -aG docker $USER
sudo apt install -y util-linux-extra   # 26.04: newgrp vem em pacote separado!
sudo reboot                            # grupos só recarregam no login da sessão
```

Depois do reboot:

```bash
docker pull osrf/ros:kilted-desktop
docker pull osrf/ros:humble-desktop
```

!!! success "✅ Teste de aceite — Docker funcional sem sudo"
    ```bash
    groups                       # "docker" tem que estar na lista
    docker run --rm hello-world  # esperado: "Hello from Docker!"
    docker images                # esperado: as duas imagens osrf/ros listadas
    ```

??? failure "Deu errado?"
    - `permission denied ... docker.sock` → [grupo não carregou na sessão](troubleshooting.md#docker-permission-denied)
    - `newgrp: command not found` → [pegadinha do 26.04](troubleshooting.md#newgrp-not-found)

## Usando a imagem que o evento fornecer {#imagem-do-evento}

No dia, a FZI vai entregar uma imagem própria para falar com os robôs deles. **O padrão que você já domina não muda — muda só o nome no final.** Há dois formatos possíveis de entrega:

**Formato A — registro (Docker Hub ou registry deles):** eles passam um nome, tipo `fzi/rima-summer-school:latest`:

```bash
docker pull NOME_DA_IMAGEM_DELES
docker run -it --net=host --ipc=host --privileged \
  -e DISPLAY -e QT_X11_NO_MITSHM=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v ~/rima_ws:/root/rima_ws \
  --name fzi NOME_DA_IMAGEM_DELES
```

**Formato B — arquivo (pendrive/rede local, comum em eventos):** eles passam um `.tar`:

```bash
docker load -i imagem-da-fzi.tar
docker images            # descubra aqui o nome que ela ganhou
docker run ... --name fzi NOME_QUE_APARECEU
```

!!! success "✅ Teste de aceite"
    `docker images` listando a imagem deles, e o `docker run` te deixando num prompt `root@...` dentro dela.

!!! tip "Se a FZI der um comando `docker run` próprio, use o deles"
    A imagem deles pode esperar flags específicas. O que você leva desta página é a fluência para **entender** o comando que te passarem — e completá-lo (ex.: adicionar a ponte X11) se faltar algo.
