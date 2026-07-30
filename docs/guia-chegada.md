# Guia de Chegada — o Dia D na FZI

*A resposta curta para "o que deste site eu vou consultar LÁ?". Na ordem em que os momentos acontecem.*

## Antes de sair do alojamento

- [ ] Mochila conforme a [Etapa 10](10-ensaio-kit.md): notebook + carregador, adaptador USB-ethernet (+ reserva), 2 cabos, adaptador de tomada EU, powerbank, cheatsheet impresso.
- [ ] Documento de identidade — o **guest agreement se assina presencialmente na segunda**.

## Ao chegar (Room "New York", 09:00)

1. **Assinar o guest agreement** e conectar no **Wi-Fi** (eduroam se tiver; senão, pedir a rede de convidados). Aceite: navegador abrindo qualquer site.
2. **Pegar o container deles** → seguir [Curso Docker § "usando a imagem que o evento fornecer"](02-docker-basico.md#imagem-do-evento) — cobre os dois formatos (`docker pull NOME` ou `docker load -i arquivo.tar`) e o nosso `docker run` padrão. Se eles derem um `docker run` próprio, **use o deles** (a página explica como completá-lo se faltar a ponte X11).
3. **As duas perguntas de ouro** (em inglês, prontas):
   - *"What is the Zenoh router endpoint we should connect to?"*
   - *"Which `ROS_DOMAIN_ID` are we using?"*

## Para conectar nos robôs deles

4. Em **cada terminal**, o trio sagrado; depois, forma A (seu router ponteado) ou B (client mode) → tudo na [Etapa 7 § Dia D](07-zenoh.md#4-o-dia-d-segunda-feira-passo-a-passo).
5. **Aceite:** `ros2 topic list` mostrando os tópicos dos robôs deles + um `ros2 topic echo` vivo.
6. Vai abrir RViz? **`xhost +local:docker` no host primeiro** — [o esquecimento clássico](troubleshooting.md#xhost-cada-boot).

## Se te derem um cabo direto num Go2 deles

O perfil `go2` **já existe no seu notebook**: basta plugar e `nmcli con up go2` ([Etapa 1 §1](01-rede-go2.md) — os IPs internos `.161/.18` são os mesmos em qualquer Go2). E o reflexo inverso: **antes** de plugar o notebook numa tomada de rede da FZI (DHCP), `nmcli con down go2` ([Etapa 1 §4](01-rede-go2.md#4-nota-para-o-evento)).

## O que deste site NÃO se aplica lá

| Seção | Por quê |
|---|---|
| Etapas 4–6 (unitree_ros2, SDK, sensores) | São do **seu** Go2 via DDS — não rode contra os robôs deles sem instrução da equipe |
| Etapa 8 (NAT) | Os robôs deles já têm a infraestrutura da FZI |
| App Unitree Go / OTA | Robô dos outros não se atualiza 🙂 |

## Se algo falhar

Direto ao [Troubleshooting](troubleshooting.md) — os atalhos mais prováveis lá: [Zenoh topic list vazio](troubleshooting.md#zenoh-topic-vazio), [docker permission denied](troubleshooting.md#docker-permission-denied), [xhost](troubleshooting.md#xhost-cada-boot), e a [taxonomia de erros do pip](troubleshooting.md#pip-sem-wheel) se instalar algo na hora.
