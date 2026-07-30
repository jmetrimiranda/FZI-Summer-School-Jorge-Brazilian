# Etapa 8 — Internet no robô (NAT pelo notebook) ✔

*Validada em 30/07 — o Jetson do Go2 navegando na internet através do Wi-Fi do notebook.*

## O problema que esta etapa resolve

O robô só tem o cabo até você. Para rodar `git clone`, `pip` ou baixar qualquer coisa **dentro** dele, alguém precisa emprestar internet. A solução: o notebook vira um **"roteador de bolso"** — recebe internet pelo Wi-Fi e reparte pelo cabo:

```
internet ──Wi-Fi──► NOTEBOOK ──cabo──► JETSON (robô)
                  (NAT: traduz e repassa)
```

## Passo a passo, com o porquê de cada comando

**No host** (Wi-Fi do notebook conectado numa rede COM internet):

```bash
nmcli con mod go2 ipv4.method shared ipv4.addresses 192.168.123.99/24
nmcli con up go2
```

- `ipv4.method shared` muda o perfil `go2` do modo "só IP fixo" para o modo **compartilhar**: o NetworkManager passa a fazer NAT (repassar a internet do Wi-Fi por essa interface) e liga um DHCP embutido.
- Mantivemos o `.99` no mesmo comando — por isso **nada do que já funciona quebra**: tópicos, SDK e RViz continuam normais durante o NAT.

**No Jetson** (`ssh unitree@192.168.123.18`):

```bash
sudo ip route replace default via 192.168.123.99
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

- A **rota padrão** ensina ao robô o portão de saída: *"tudo que não for da rede local, entrega para o `.99`"* (o notebook). Sem ela, o Jetson nem tenta sair.
- O **DNS** (8.8.8.8) permite traduzir nomes: sem ele, o robô alcança IPs mas `google.com` falha.

!!! success "✅ Teste de aceite"
    De **dentro do Jetson**: `ping -c3 8.8.8.8` respondendo (a rota funciona) **e** `ping -c3 google.com` respondendo (o DNS funciona).

??? failure "Deu errado?"
    [Checklist do NAT](troubleshooting.md#nat-jetson-sem-internet)

## Voltar ao modo direto (sem NAT)

```bash
nmcli con mod go2 ipv4.method manual
nmcli con up go2
```

No Jetson, a rota adicionada **some sozinha no próximo reboot do robô**; o DNS gravado é inofensivo de manter.

## Onde isso se encaixa — e onde não

- **No evento:** os robôs são da FZI, já pendurados na infraestrutura deles — **você não fará isso lá**.
- **Serve para:** atualizar/instalar coisas no seu Go2 em casa, e como habilidade genérica de bancada — "dar internet a um dispositivo via notebook" salva o dia em laboratórios e demos.
- **Projeto pós-viagem:** a "ilha sem fio" (robô + notebook no roteador de viagem com LAN `192.168.123.0/24`, sem cabo no notebook) — esboço registrado, sem pressa.
