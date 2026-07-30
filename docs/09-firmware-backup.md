# Etapa 9 — Firmware OTA + energia + backup 🔜

!!! warning "Planejada — será preenchida quando validarmos ao vivo"

**Plano:** app **Unitree Go** → OTA do robô e do controle (anotar versões antes/depois); carregar as 2 baterias + controle + notebook; backup do Jetson a partir do notebook:

```bash
rsync -avz unitree@192.168.123.18:/home/unitree/ ~/backup_go2_jetson/
```

Lembrete: atualização de sistema no Jetson **só** via OTA oficial — nunca `apt upgrade` ([por quê](01-rede-go2.md#3-reconhecimento-do-jetson-o-que-existe-la-dentro)).
