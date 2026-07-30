# Etapa 6 — Sensores: LiDAR no RViz + câmera 🔜

!!! warning "Planejada — será preenchida quando validarmos ao vivo"

**Objetivo:** nuvem do LiDAR L1 renderizando no RViz do notebook e vídeo da câmera frontal na tela.

**Plano:** no `casa`, após o `setup.sh` da Etapa 4: descobrir o `frame_id` com `ros2 topic echo /utlidar/cloud --no-arr | grep frame_id`, abrir `rviz2`, Fixed Frame = frame descoberto, Add → By topic → `/utlidar/cloud`. Câmera: pipeline GStreamer oficial da Unitree (`udpsrc address=230.1.1.1 port=1720 multicast-iface=<IF_ROBO> ...`). Alternativa ROS: tópico `/frontvideostream`.
