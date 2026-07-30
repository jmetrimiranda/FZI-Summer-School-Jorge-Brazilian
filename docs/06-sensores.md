# Etapa 6 — Sensores: LiDAR no RViz + câmera ✔

*Validada em 30/07 — nuvem do L1 renderizando no RViz (`Status: Ok`) e câmera frontal via **VideoClient do SDK (1920×1080!)**. A tentativa de assinar `/frontvideostream` via rclpy revelou uma aresta conhecida do ecossistema — documentada abaixo como estudo de caso.*

## 1. A nuvem do LiDAR no RViz

No container `casa` (GUI liberada com `xhost +local:docker` no host desta sessão):

```bash
docker exec -it casa bash
source /root/rima_ws/unitree_ros2/setup.sh          # a regra permanente!
ros2 topic echo /utlidar/cloud --no-arr | grep -m1 frame_id
```

Resultado validado: **`frame_id: utlidar_lidar`**. (O traceback `BrokenPipeError` depois disso é [sucesso barulhento](troubleshooting.md#broken-pipe-grep), não erro.)

```bash
rviz2
```

Na janela: **Global Options → Fixed Frame** = `utlidar_lidar` → **Add → By topic → /utlidar/cloud → PointCloud2** → **Size (m)** = `0.03`. Navegação: botão esquerdo gira, scroll aproxima, botão do meio arrasta.

!!! success "✅ Teste de aceite — validado"
    Display `PointCloud2 → Status: Ok`, o ambiente aparecendo em pontos 3D coloridos por intensidade — e o teste definitivo: passar o braço na frente do robô e vê-lo nascer na nuvem.

!!! tip "Decay Time"
    `0` = só a última nuvem; `2` = acumula 2 s (bom para visão "ao vivo"); `999` = escaneia o ambiente inteiro parado — denso, mas borra com o micro-balanço do robô em pé.

## 2. Por que a nuvem parece "torta" (frames e TF — o conceito)

Observação real da validação: o chão aparece **em diagonal** em relação ao grid. Não é defeito — é geometria de referenciais:

1. O L1 é **montado fisicamente inclinado** na cabeça do Go2 (para ver o chão onde vai pisar);
2. `/utlidar/cloud` entrega os pontos **no sistema de coordenadas do sensor** (`utlidar_lidar`) — o "para cima" desse sistema acompanha a inclinação;
3. O grid do RViz vive no plano XY do **Fixed Frame** — que, aqui, é o próprio sensor. Logo, **o quarto está reto; a régua é que está torta.**

O aviso laranja `No tf data` conta a causa: ninguém está publicando a árvore **TF** (as transformações `utlidar_lidar → body → odom`). Num sistema completo, com TF, você escolheria `odom` como Fixed Frame e o RViz rotacionaria cada nuvem para o referencial do mundo — chão plano. É exatamente o tema da quarta-feira na summer school (navegação 3D em mapas volumétricos).

**Experimento de 30 s que prova a teoria:** o robô também publica `/utlidar/cloud_base` (a nuvem já transformada para o referencial do corpo — estava na [lista do Jetson](01-rede-go2.md)). Descubra o `frame_id` dela, aponte o Fixed Frame para ele e compare a inclinação.

## 3. Câmera — caminho A: GStreamer (multicast H.264, no host)

O robô transmite H.264 por **multicast UDP** em `230.1.1.1:1720`. Pipeline oficial:

```bash
sudo apt install -y gstreamer1.0-tools gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-libav
IF_ROBO=$(ip -4 -o addr show | awk '/192.168.123.99/{print $2}')
gst-launch-1.0 udpsrc address=230.1.1.1 port=1720 multicast-iface=$IF_ROBO ! application/x-rtp,media=video,encoding-name=H264 ! rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! autovideosink
```

Lendo o pipeline como uma linha de montagem: `udpsrc` escuta o grupo multicast na interface do robô → `rtph264depay` tira o vídeo de dentro dos pacotes RTP → `h264parse` organiza o fluxo → `avdec_h264` descomprime → `videoconvert` ajusta o formato de pixel → `autovideosink` desenha na janela. Avisos de "buffering/queue" no terminal são cosméticos.

!!! success "✅ Teste de aceite"
    Janela com o vídeo ao vivo da câmera frontal (1280×720).

## 4. Câmera — caminho B (validado): VideoClient do SDK, request/response

O caminho que **funcionou** — e ainda entregou resolução maior: **1920×1080**, contra os 720p do stream. É o mesmo padrão request/response da [Etapa 5](05-sdk2.md), agora com carga binária, batendo no `rt/api/videohub/request` (o `/api/videohub/request` da [lista do Jetson](01-rede-go2.md)):

1. `client.Init()` cria o par publisher/subscriber em `rt/api/videohub/request` e `rt/api/videohub/response`;
2. `GetImageSample()` monta uma `unitree_api/Request`: **id único da requisição** + **api_id** da função (descubra com `grep -rn "API_ID" /root/rima_ws/unitree_sdk2_python/unitree_sdk2py/go2/video/`) + `parameter` em JSON (vazio aqui);
3. O serviço `videohub` no robô captura um frame na resolução nativa, comprime em **JPEG** e devolve numa `Response` cuja **carga binária são os bytes crus do JPEG**;
4. O cliente casa a resposta pelo id e devolve `(code, data)` — `code 0` = sucesso.

**Por que isso dribla a aresta do pub/sub:** no pub/sub, o assinante precisa do *schema rico* de cada mensagem — qualquer divergência de contrato quebra a desserialização. No envelope request/response, o contrato é simples e estável, e o frame viaja como **blob opaco** que só o `cv2.imdecode` interpreta. *Schema acoplado vs. envelope + payload opaco* — decisão clássica de projeto de sistemas distribuídos.

```python
import sys, time
import numpy as np, cv2
from unitree_sdk2py.core.channel import ChannelFactoryInitialize
from unitree_sdk2py.go2.video.video_client import VideoClient

if __name__ == "__main__":
    ChannelFactoryInitialize(0, sys.argv[1])
    client = VideoClient()
    client.SetTimeout(3.0)     # quanto esperar cada resposta
    client.Init()
    print("Pedindo frames via VideoClient (q na janela sai)...")
    first = True
    while True:
        code, data = client.GetImageSample()   # 1 round-trip = 1 frame
        if code != 0:
            print("GetImageSample code:", code); time.sleep(0.5); continue
        img = cv2.imdecode(np.frombuffer(bytes(data), np.uint8), cv2.IMREAD_COLOR)
        if img is None: continue
        if first: print("primeiro frame:", img.shape); first = False
        cv2.imshow("Go2 - VideoClient (q sai)", img)
        if cv2.waitKey(1) & 0xFF == ord('q'): break
    cv2.destroyAllWindows()
```

!!! success "✅ Teste de aceite — validado"
    `primeiro frame: (1080, 1920, 3)` + janela com o vídeo. Avisos `QFontDatabase: Cannot find font directory` são cosméticos (o Qt embutido no OpenCV do pip não traz fontes).

Características do caminho: taxa menor que o stream (um pedido-e-resposta por frame) e engasgos ocasionais em loop contínuo são comportamento relatado no ecossistema — em troca, código trivial e a imagem em resolução cheia.

## 4b. Estudo de caso — a aresta do `/frontvideostream` via rclpy

Registro honesto do que aconteceu antes do caminho B funcionar, porque o diagnóstico vale mais que o atalho:

**Sintoma:** o assinante rclpy nunca recebia callback, com o terminal inundado de `serdata.cpp: invalid data size` / `unable initialize generic sequence`.

**A cadeia de evidências:** (a) os erros vinham **em rajada contínua** → cada erro era uma mensagem *chegando* e sendo rejeitada — o videohub publicava o tempo todo; (b) `/sportmodestate`, do **mesmo pacote** `unitree_go`, desserializa perfeitamente → pilha e build corretos, divergência específica do `Go2FrontVideoData`; (c) no Jetson, `ros2 interface show` deu `Unknown package 'unitree_go'` — o Foxy embarcado é baunilha; os serviços Unitree carregam os tipos **embutidos nos binários**, e o `topic list -t` mostra o nome do tipo porque ele viaja na **metadata de descoberta** do DDS (listar não exige o molde; decodificar exige); (d) há relato público idêntico (issue [#102 do unitree_sdk2_python](https://github.com/unitreerobotics/unitree_sdk2_python/issues/102)): mesma definição, tópico a ~33 Hz, assinante Python sem receber nada.

**Decisão de engenharia:** timebox vencido → não lixar a aresta; usar os caminhos documentados (VideoClient e GStreamer). Detalhes operacionais: [troubleshooting](troubleshooting.md#frontvideostream-mudo).

## 5. Os três transportes, lado a lado

| | GStreamer (A) | VideoClient (B — validado) | rclpy em `/frontvideostream` |
|---|---|---|---|
| Transporte | Multicast UDP, fluxo H.264 | Request/response DDS, JPEG por frame | Pub/sub DDS, JPEG por frame |
| Resolução | 1280×720 | **1920×1080** | 720p/360p/180p |
| Taxa | Alta e contínua | Menor (1 round-trip/frame) | ~33 Hz (quando funciona) |
| Consumo no código | Difícil (pipeline de mídia) | Trivial (`imdecode`) | Trivial — **mas bloqueado pela aresta de desserialização** |
| Uso recomendado | Assistir / gravar | Visão computacional no seu código | Evitar até a aresta ser resolvida upstream |
