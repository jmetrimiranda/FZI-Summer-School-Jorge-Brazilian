# Etapa 7 — Zenoh: o modo do evento ✔

*Validada em 30/07 — talker/listener via rmw_zenoh (fase 1) e nó em `mode="client"` conectado direto ao router (fase 3, o drill exato do evento).*

## 1. O que é o Zenoh — e por que a FZI o escolheu

**Zenoh** é um protocolo moderno de pub/sub (projeto Eclipse) desenhado para redes *reais*: Wi-Fi, redes grandes, até internet. **rmw_zenoh** é a camada que faz o ROS 2 falar Zenoh em vez de DDS.

A diferença filosófica em relação ao mundo das etapas 3–6:

| | **DDS** (o que dominamos em casa) | **Zenoh** (o que o evento usa) |
|---|---|---|
| Descoberta | Automática, por **multicast** na rede local | **Nada se descobre sozinho**: um **router** (`rmw_zenohd`) intermedia |
| Como entrar na rede | Estar na mesma subrede/domain basta | **Apontar** explicitamente para um router (TCP) |
| Configuração típica | "qual interface" (`CYCLONEDDS_URI`) | "qual endereço" (`connect/endpoints=["tcp/IP:7447"]`) |
| Em Wi-Fi / sala cheia | Multicast sofre; todos se misturam (vimos os containers se acharem sozinhos na [Etapa 3](03-containers.md#o-fenomeno)) | TCP explícito: estável e com escopo controlado |
| Sintoma clássico de erro | Interface errada → `topic list` vazio | **Router não está rodando / endpoint errado** → `topic list` vazio |

## 2. O que validamos (saídas reais)

**O trio sagrado — em TODO terminal Zenoh:**

```bash
source /opt/ros/kilted/setup.bash
export RMW_IMPLEMENTATION=rmw_zenoh_cpp
# no evento, soma-se: export ROS_DOMAIN_ID=N
```

**O router (1 por máquina) — Terminal 1, fica rodando:**

```bash
ros2 run rmw_zenoh_cpp rmw_zenohd
pgrep -af rmw_zenohd      # confirma: 2 processos (wrapper + binário)
```

**Fase 1 — pub/sub via Zenoh** (Terminais 2 e 3, cada um com o trio):

```bash
ros2 run demo_nodes_cpp talker
ros2 run demo_nodes_cpp listener
```

!!! success "✅ Teste de aceite — validado"
    `I heard: [Hello World: 88 ... 103]` fluindo — primeiro par de nós conversando por Zenoh.

!!! info "Observação de graça que os logs entregaram"
    O talker foi morto no 103 e reiniciado (contagem voltou a 1) — e o listener **nem piscou**: recebeu `103` e, segundos depois, `1, 2...`. Nós vêm e vão; a assinatura registrada permanece, e o novo publisher é casado a ela automaticamente.

**Fase 3 — o drill do evento: nó em modo *client*, direto no router** (sem sessão peer local):

```bash
ZENOH_CONFIG_OVERRIDE='mode="client";connect/endpoints=["tcp/127.0.0.1:7447"]' ros2 run demo_nodes_cpp listener
```

Dissecando: `mode="client"` = este nó não participa da malha par-a-par; vira cliente puro de um router. `connect/endpoints` = onde está o router (aqui `127.0.0.1` simulando; **no evento, o IP do robô/servidor da FZI**). `7447` = porta padrão do Zenoh.

!!! success "✅ Teste de aceite — validado"
    `I heard: [Hello World: 15 ... 22]` — o nó entrou pelo router e ouviu o talker.

## 3. Experimento opcional — matar o router (teoria a verificar)

*Não executado na validação — fica como exercício de 2 minutos.* Pelo desenho do rmw_zenoh, o router serve à **descoberta/liveliness**; os **dados** fluem par-a-par entre as sessões já apresentadas. Previsão testável: matando o router (Ctrl+C no Terminal 1), (a) o par talker/listener **existente continua** trocando mensagens; (b) um **listener novo fica ilhado** (não ouve nada) até o router voltar. Rode e confira — anotando o resultado aqui.

## 4. O Dia D — segunda-feira, passo a passo

O cenário: você chega, conecta no Wi-Fi deles (eduroam/convidados), e os robôs da FZI estão na mesma rede rodando Kilted + Zenoh (com um container fornecido por eles — [como usar](02-docker-basico.md#imagem-do-evento)).

1. **As duas perguntas de ouro** para a organização — são o bilhete de entrada:
   - *"Qual o endpoint do router Zenoh?"* (um IP, talvez `tcp/IP:7447`)
   - *"Qual `ROS_DOMAIN_ID` vocês usam?"*
2. **Em cada terminal**: o trio sagrado (`source` + `RMW_IMPLEMENTATION` + `ROS_DOMAIN_ID`). Esquecer o export num terminal = nó "mudo" — o erro nº 1.
3. **Duas formas de entrar na rede deles:**
   - **A (recomendada — vários terminais, uma configuração):** suba o SEU router local ponteado no deles; todos os seus nós usam o router local normalmente:
     ```bash
     ZENOH_CONFIG_OVERRIDE='connect/endpoints=["tcp/IP_DA_FZI:7447"]' ros2 run rmw_zenoh_cpp rmw_zenohd
     ```
   - **B (rápida — um nó só):** client mode direto, o que validamos na fase 3, trocando `127.0.0.1` pelo IP deles.
4. **Aceite no local:** `ros2 topic list` mostrando os tópicos do robô deles; `ros2 topic echo` em um deles.
5. **Se vier vazio**, o [checklist Zenoh](troubleshooting.md#zenoh-topic-vazio).

**O que mudou em relação a casa:** lá o DDS multicast achava seu Go2 sozinho pelo cabo (a configuração era *interface*). No evento, nada é achado sozinho — **você aponta para um endereço**. E o container da FZI provavelmente já vem com isso tudo embutido; conhecer o que há por baixo é o que permite *consertar* quando algo falhar.

## 5. Diagnóstico rápido Zenoh

```bash
pgrep -af rmw_zenohd          # router vivo?
echo $RMW_IMPLEMENTATION      # rmw_zenoh_cpp NESTE terminal?
echo $ROS_DOMAIN_ID           # igual ao da outra ponta?
ros2 topic list               # vazio? -> checklist no troubleshooting
```

## 6. FAQ — as perguntas que ficaram depois da validação

**"Zenoh só funciona em Wi-Fi?"** Não — Zenoh é **agnóstico ao meio**: roda TCP sobre qualquer rede IP (cabo, Wi-Fi, até internet). A nossa própria validação nem saiu da máquina (`127.0.0.1`). A associação com Wi-Fi existe porque *no evento* o meio será Wi-Fi; se te derem um cabo lá, o mesmo comando funciona — só muda o IP.

**"Preciso converter as etapas 4–6 para Zenoh?"** Não. O **seu** Go2 fala DDS — as etapas 4–6 continuam exatamente como estão para ele. Zenoh é o idioma dos robôs **da FZI**. São dois mundos, e você agora domina os dois; a tabela do §1 mostra o contraste.

**"O que exatamente eu digito de diferente na segunda-feira?"** O trio sagrado em cada terminal + o endpoint deles no lugar do `127.0.0.1` (§4). Todo o resto — `ros2 topic list/echo`, RViz, seus nós Python — é **idêntico** ao que você já usa.

**"E aquela ponte para mover o MEU Go2 via Zenoh?"** Exercício opcional de intuição — nada do evento depende dele. Fica registrado como projeto pós-viagem.
