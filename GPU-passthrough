# 🎮 Proxmox 9.2 – NVIDIA GPU Passthrough to LXC (Jellyfin)

## 🧭 Overview
This guide documents a **working, production-grade setup** for passing an **NVIDIA RTX GPU** from a **Proxmox 9.2.x host** into a **privileged LXC container**, and then exposing it to **Docker / Jellyfin** using the NVIDIA container runtime.

Validated with:
- Proxmox VE 9.2.x
- NVIDIA RTX 4060
- Driver 580.x
- Privileged LXC
- Docker + Jellyfin (linuxserver.io)

Result: `nvidia-smi` works on host, inside LXC, and inside the Jellyfin Docker container.

---

## 📦 Architecture

[ Proxmox Host ]
  └─ NVIDIA Driver (580.x)
  └─ /dev/nvidia* devices
       ↓ bind mount
[ Privileged LXC ]
  └─ nvidia-utils + Docker
  └─ nvidia-container-runtime
       ↓ runtime=nvidia
[ Jellyfin Container ]
  └─ NVDEC / NVENC hardware transcoding

---

## 1️⃣ Host checks (Proxmox)

```bash
nvidia-smi
ls -l /dev/nvidia*
```

You must see `/dev/nvidia0`, `/dev/nvidiactl`, `/dev/nvidia-modeset`,
`/dev/nvidia-uvm`, and `/dev/nvidia-caps`.

---

## 2️⃣ LXC configuration (REQUIRED)

The container **must be privileged**.

Edit:
```bash
nano /etc/pve/lxc/<CTID>.conf
```

```ini
# NVIDIA GPU passthrough
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file

lxc.cgroup2.devices.allow: c 504:* rwm
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file

lxc.cgroup2.devices.allow: c 507:* rwm
lxc.mount.entry: /dev/nvidia-caps dev/nvidia-caps none bind,optional,create=dir
```

Restart container:
```bash
pct stop <CTID>
pct start <CTID>
```

---

## 3️⃣ Verify inside LXC

```bash
ls -l /dev/nvidia*
ls -l /dev/nvidia-caps
nvidia-smi
```

If this works, GPU passthrough is done.

---

## 4️⃣ Install NVIDIA userspace (LXC)

```bash
apt update
apt install -y nvidia-utils-580
```

Do NOT install kernel drivers.

---

## 5️⃣ Install NVIDIA container runtime (LXC)

```bash
apt install -y nvidia-container-toolkit
nvidia-ctk runtime configure --runtime=docker
systemctl restart docker
```

Verify:
```bash
docker info | grep -i runtime
```

Must show: `runc nvidia`

---

## 6️⃣ Jellyfin docker-compose.yml

```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    runtime: nvidia
    environment:
      - TZ=Europe/Paris
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
    volumes:
      - /docker-configs/docker/jellyfin/jellyfin-cfg:/config
      - /media:/media
    ports:
      - 8096:8096
      - 8920:8920
      - 7359:7359/udp
      - 1900:1900/udp
    restart: unless-stopped
```

Start:
```bash
docker compose up -d
```

---

## 7️⃣ Final validation

```bash
docker exec -it jellyfin nvidia-smi
watch -n 1 nvidia-smi
```

During playback you should see `ffmpeg` using the GPU.

---

## 🎬 Jellyfin settings

Dashboard → Playback → Transcoding:
- Enable NVIDIA NVDEC
- Enable NVIDIA NVENC
- Enable HEVC / H.264 / AV1

Restart Jellyfin.

---

## ✅ Result

✔ GPU passthrough working  
✔ Docker NVIDIA runtime configured  
✔ Jellyfin hardware transcoding active  

Happy transcoding 🚀

