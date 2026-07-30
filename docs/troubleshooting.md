# Troubleshooting

Todos os problemas abaixo **aconteceram de verdade** durante a preparação (30/07). Cada entrada: sintoma → causa → solução.

## Docker: permission denied no docker.sock {#docker-permission-denied}

**Sintoma:** `permission denied while trying to connect to the docker API at unix:///var/run/docker.sock`

**Causa:** seu usuário não está no grupo `docker` **na sessão atual**. Pegadinha: o `sudo usermod -aG docker $USER` grava no sistema, mas os grupos só são recarregados **no login da sessão gráfica** — fechar/abrir terminal não basta, e até um "Log out" mal-sucedido pode manter a sessão velha viva.

**Solução:** confirme o cadastro com `getent group docker` (seu nome deve aparecer) e **reinicie a máquina**. Depois, `groups` deve listar `docker`. Paliativo emergencial que não trava o trabalho: `alias docker='sudo docker'` (vale só naquele terminal).

## newgrp: command not found {#newgrp-not-found}

**Sintoma:** `Command 'newgrp' not found` (Ubuntu 26.04).

**Causa:** no 26.04 o `newgrp` mora no pacote `util-linux-extra`, que não vem por padrão.

**Solução:** `sudo apt install util-linux-extra`. E saiba o limite da ferramenta: `newgrp docker` ativa o grupo **só naquele shell** — para valer em todo lugar, reinicie a sessão.

## Perfil de rede não sobe: mismatching interface name {#iface-mudou}

**Sintoma:** `Error: Connection activation failed: No suitable device found ... (mismatching interface name)`

**Causa:** adaptadores USB-C→Ethernet podem gerar um **MAC novo a cada replug**; como o Linux nomeia a placa pelo MAC (`enx...`), o nome muda e o perfil amarrado ao nome antigo não encontra o dispositivo. Confirmado ao vivo: `enx00e04c361b96` → `enx0c3796dc8be5`.

**Solução:** desamarre o perfil do nome: `nmcli con mod go2 connection.interface-name ""` e `nmcli con up go2`. Efeito colateral aceito: o perfil passa a capturar qualquer ethernet — no evento, `nmcli con down go2` antes de usar rede cabeada com DHCP.

## Ping responde "Time to live exceeded" de um IP estranho {#ttl-exceeded}

**Sintoma:** `From 15.15.15.7 icmp_seq=1 Time to live exceeded` ao pingar `192.168.123.x`.

**Causa:** não existe rota local para a subrede do robô (a conexão `go2` não está ativa), então o pacote saiu pela rota padrão (Wi-Fi → operadora) e morreu em loop lá fora. O IP estranho é um roteador da operadora avisando.

**Solução:** `nmcli con up go2` e confira `ip a | grep 192.168.123.99`. O ping deve voltar com `ttl=64 time=0.4 ms`.

## Jetson (.18) não responde {#jetson-nao-responde}

**Sintoma:** `.161` pinga, `.18` não.

**Causa provável:** o Jetson do dock EDU demora ~1 min a mais para bootar que a controladora; ou o dock não está bem encaixado.

**Solução:** aguarde o boot completo e tente de novo; confira o encaixe do dock.

## docker run: Conflict, name already in use {#container-name-conflict}

**Sintoma:** `Conflict. The container name "/escola" is already in use by container ...`

**Causa:** `docker run` **cria** um container novo, e já existe um com esse nome (provavelmente de uma tentativa anterior). Não é erro grave — é o Docker te protegendo de duplicar.

**Solução:** use o que existe: `docker ps -a` para ver o estado; `docker start -ai escola` se `Exited`, `docker exec -it escola bash` se `Up`. Só se quiser recriar do zero: `docker rm escola` e então o `run`.

## ros2 run: No executable found {#no-executable-found}

**Sintoma:** `No executable found`

**Causa:** typo no nome do executável (aconteceu: `talke` em vez de `talker`) ou pacote errado.

**Solução:** confira o nome com `ros2 pkg executables demo_nodes_cpp`.

## ros2: command not found (no host) {#ros2-nao-encontrado-host}

**Sintoma:** `ros2: command not found` ou `source /opt/ros/...: No such file or directory` no prompt `jorgemetri@...`.

**Causa:** **por design** deste setup, não existe ROS no host — só nos containers.

**Solução:** entre no container primeiro (`docker start -ai casa` ou `docker exec -it casa bash`); o prompt vira `root@...` e aí sim `source` + `ros2` funcionam.

## RViz: libGL error / nouveau failed {#rviz-libgl}

**Sintoma:** `libGL error: failed to load driver: nouveau` (repetido) ao abrir o `rviz2` no container.

**Causa:** o container tenta usar a GPU NVIDIA, mas não tem o driver proprietário dentro dele.

**Solução:** nenhuma necessária — repare na linha seguinte `OpenGl version: 4.5 (GLSL 4.5)`: o RViz caiu para **renderização por software (llvmpipe)** e abre normalmente. Só se ficar lento demais com nuvens grandes vale instalar o `nvidia-container-toolkit` (fora do escopo por ora). `XDG_RUNTIME_DIR not set` e `Stereo is NOT SUPPORTED` são ruído normal.

## Listener não imprime nada {#listener-mudo}

**Sintoma:** `ros2 run demo_nodes_cpp listener` fica em silêncio.

**Causa:** o teste é um **par** — sem um talker publicando (no mesmo domain), não há o que ouvir.

**Solução:** suba o talker em outro terminal do container e confira `echo $ROS_DOMAIN_ID` igual nos dois lados.

## ros2 node list vazio no Jetson (mas topic list cheio) {#node-list-vazio}

**Sintoma:** `ros2 node list` não retorna nada, mas `ros2 topic list -t` mostra ~120 tópicos.

**Causa:** os serviços nativos da Unitree são participantes DDS "puros" que publicam nos tópicos sem se registrar como nós ROS completos no grafo.

**Solução:** nenhuma — é o comportamento esperado. **A verdade está nos tópicos.**

## O que significa Exited (127) no docker ps {#exit-127}

**Sintoma:** container listado como `Exited (127)`.

**Causa:** o último comando executado dentro dele não existia (ex.: rodar `docker` dentro do container). 127 = "command not found". Inofensivo.

**Solução:** `docker start -ai NOME` e siga o baile.

## RViz não abre depois de um reboot {#xhost-cada-boot}

**Sintoma:** GUI funcionava ontem; hoje `rviz2` não abre janela (erro de display/authorization).

**Causa:** o `xhost +local:docker` vale **por sessão** — reboot/relogin apaga a permissão.

**Solução:** rode `xhost +local:docker` no host de novo. Está no checklist de início de sessão por isso.

## Adaptador sumiu do nmcli device status {#adaptador-sumiu}

**Sintoma:** a interface `enx...` não aparece na lista — nem como `disconnected`. (E o `nmcli device status` é justamente o primeiro comando do diagnóstico de rede.)

**Causa:** o adaptador USB-C foi desplugado (ou mal encaixado) — para o sistema, a placa deixou de existir.

**Solução:** replugar adaptador e cabo; aguardar a linha `enx... ethernet connected go2` voltar (o perfil reativa sozinho); confirmar com os pings em `.161` e `.18`. Lembre: ao replugar, o nome/MAC pode mudar — irrelevante para nós, pois o perfil não é amarrado ao nome e o `setup.sh` acha a interface pelo IP.

## topic list vazio com CycloneDDS {#topic-list-vazio-cyclone}

**Sintoma:** dentro do container `casa`, `ros2 topic list` não mostra os tópicos do robô.

**Checklist na ordem:**

1. `source /root/rima_ws/unitree_ros2/setup.sh` foi rodado **neste** terminal? A linha `IF_ROBO=` imprimiu um nome real?
2. No **host**: `nmcli device status` mostra a conexão `go2` ativa? Os pings em `.161`/`.18` respondem?
3. Cache do daemon: `ros2 daemon stop` e repita `ros2 topic list` (ele renasce sozinho).

## IF_ROBO vazio no setup.sh {#if-robo-vazio}

**Sintoma:** o `setup.sh` imprime o AVISO e `IF_ROBO=` sem nada.

**Causa:** nenhuma interface carrega o IP `192.168.123.99` — a conexão `go2` não está de pé no host (adaptador fora, cabo solto, ou perfil down).

**Solução:** no host, `nmcli con up go2` (replugando antes, se preciso) e re-rode o `source` no container.

## SportClient devolve ret diferente de 0 {#ret-nao-zero}

**Sintoma:** `StandUp ret: 3xxx` (ou outro número ≠ 0) e/ou o robô não se mexe.

**Causas prováveis, na ordem de checagem:**

1. **Rede/descoberta:** o teste de leitura (`leitura_estado.py`) ainda imprime dados? Se não, o problema é rede ([checklist](troubleshooting.md#topic-list-vazio-cyclone)), não a API.
2. **Timeout:** demorou exatamente o valor do `SetTimeout` para falhar? O serviço não respondeu — robô ocupado ou interface errada no `ChannelFactoryInitialize`.
3. **Modo de operação:** o robô pode estar num modo que recusa o comando (ex.: deitado demais para `StandUp` direto → tente `RecoveryStand()`; ou o modo de movimento atual — o serviço `motion_switcher`, visível nos tópicos `/api/motion_switcher/*`, gerencia isso).
4. Não repita em loop cego: anote o número do ret, o estado físico do robô, e investigue.

## O .bashrc do container agora carrega o ROS sozinho {#bashrc-ros-automatico}

**Contexto:** foi adicionado `source /opt/ros/humble/setup.bash` ao `~/.bashrc` do container `casa`. Conveniente — todo `docker exec` já nasce com ROS — **mas** mata o truque do "shell limpo" usado para compilar o cyclonedds na [Etapa 4](04-unitree-ros2.md).

**Se um dia precisar rebuildar o cyclonedds do zero:** comente essa linha (`nano ~/.bashrc` dentro do container), abra um `docker exec` novo, confirme `printenv | grep AMENT` vazio, compile, e descomente.

## ModuleNotFoundError: No module named 'unitree_go' {#modulenotfound-unitree-go}

**Sintoma:** um script Python com `from unitree_go.msg import ...` morre na importação.

**Causa:** `unitree_go` não vem do pip nem do ROS de fábrica — é o pacote de mensagens **gerado pelo colcon** na Etapa 4, vivendo no overlay (`cyclonedds_ws/install/`). O terminal atual não carregou o overlay (o `.bashrc` do container só carrega o Humble base). Detalhe pedagógico: recriar o arquivo não muda nada — o erro está no **ambiente**, não no código.

**Solução:** `source /root/rima_ws/unitree_ros2/setup.sh` (a regra permanente) e rode de novo.

## Traceback BrokenPipeError depois de um grep -m1 {#broken-pipe-grep}

**Sintoma:** `ros2 topic echo ... | grep -m1 algo` imprime o resultado certo **e** um traceback `BrokenPipeError: [Errno 32]`.

**Causa:** é o comando funcionando. O `grep -m1` fecha o pipe assim que acha a primeira ocorrência; o `ros2 topic echo`, ao tentar continuar escrevendo, leva "broken pipe" do kernel e morre com traceback dramático. O grep matou o echo *de propósito*.

**Solução:** nenhuma — leia a linha do resultado e ignore o obituário.

## /frontvideostream: "Aguardando frames..." + erros serdata {#frontvideostream-mudo}

**Sintoma:** o assinante rclpy nunca recebe callback e o terminal enche de `serdata.cpp: invalid data size` / `unable initialize generic sequence` repetidos.

**Diagnóstico (fechado em 30/07):** os erros vêm em rajada = **cada mensagem está chegando** e sendo rejeitada na desserialização — o `videohub` publica normalmente. Sanidade que isola o problema: `/sportmodestate`, do mesmo pacote `unitree_go`, desserializa perfeitamente → pilha e build corretos; a divergência é específica do `Go2FrontVideoData`. No Jetson, `ros2 interface show` responde `Unknown package` (o Foxy embarcado é baunilha; os tipos vivem embutidos nos binários dos serviços — o `topic list -t` só funciona porque o *nome* do tipo viaja na metadata de descoberta do DDS). Aresta conhecida no ecossistema (issue #102 do unitree_sdk2_python relata o mesmo quadro).

**Solução:** usar os caminhos que funcionam — `VideoClient.GetImageSample()` (validado, 1080p) ou o stream GStreamer. Estudo de caso completo na [Etapa 6](06-sensores.md).
