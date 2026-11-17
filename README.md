# ASUS ROG Strix with Intel Processors — CachyOS Audio Fix & ROG Setup

> ![OS](https://img.shields.io/badge/OS-CachyOS-blue?style=for-the-badge&logo=archlinux&logoColor=white)
![Laptop](https://img.shields.io/badge/Laptop-ASUS_ROG_Strix-red?style=for-the-badge&logo=asus&logoColor=white)
![Audio](https://img.shields.io/badge/Audio-ALC294_CS35L41-brightgreen?style=for-the-badge)

🌐 **[English](#-english)** | **[Español](#-español)**

---

## English

Post-installation hardware configuration guide for **ASUS ROG Strix laptops with Intel processors** (tested on **G614JV-AS74**) running **CachyOS (Arch Linux)**. This guide resolves speaker static/crackling caused by DMA stream errors and sets up native ASUS ROG control tools.

### Table of Contents
1. [Fix Audio Static Noise & DMA Errors](#1-fix-audio-static-noise--dma-errors)
2. [Fix Low Volume & Save ALSA Levels](#2-fix-low-volume--save-alsa-levels)
3. [Install `asusctl` & ROG Control Center](#3-install-asusctl--rog-control-center)
4. [Quick Verification](#4-quick-verification)

---

### 1. Fix Audio Static Noise & DMA Errors

#### Issue
When playing audio through the internal speakers, heavy static/radio-like interference occurs. Checking `dmesg` reveals repeated DMA transfer errors from the legacy `snd_hda_intel` driver:
```text
snd_hda_intel 0000:00:1f.3: stream 7 dma error: 0x28
```

#### Solution
Switch from the legacy HDA driver to **Sound Open Firmware (SOF)**:

1. **Install SOF firmware and UCM profiles:**
   ```bash
   sudo pacman -S --needed sof-firmware alsa-ucm-conf alsa-utils
   ```

2. **Force the SOF DSP driver (`dsp_driver=3`):**
   ```bash
   echo "options snd-intel-dspcfg dsp_driver=3" | sudo tee /etc/modprobe.d/asus-audio.conf
   ```

3. **Reboot** your laptop to apply the kernel module configuration.

---

### 2. Fix Low Volume & Save ALSA Levels

#### Issue
After switching to `sof-hda-dsp`, the audio is crystal clear, but maximum speaker volume is too low because ALSA hardware channels default to a capped level.

#### Solution
1. Open the ALSA mixer in your terminal:
   ```bash
   alsamixer
   ```
2. Press **`F6`** and select **`sof-hda-dsp`**.
3. Raise **Speaker** and **Headphone** sliders to `100` (ensure they are unmuted with `00` at the bottom).
4. Press `Esc` to exit and store the mixer state permanently across reboots:
   ```bash
   sudo alsactl store
   ```

---

### 3. Install `asusctl` & ROG Control Center

1. **Import and sign the official ASUS Linux GPG key:**
   ```bash
   sudo pacman-key --recv-keys 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
   sudo pacman-key --finger 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
   sudo pacman-key --lsign-key 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
   ```

2. **Add the `[g14]` repository to `/etc/pacman.conf`:**
   ```bash
   echo -e "\n[g14]\nServer = https://arch.asus-linux.org" | sudo tee -a /etc/pacman.conf
   ```

3. **Install `asusctl` and `rog-control-center`:**
   ```bash
   sudo pacman -Syu asusctl rog-control-center
   ```

4. **Enable the power profiles daemon:**
   ```bash
   sudo systemctl enable --now power-profiles-daemon.service
   ```

---

### 4. Quick Verification

* **Verify Cirrus Logic CS35L41 smart amplifiers loaded factory calibration:**
  ```bash
  sudo dmesg | grep -iE "cs35l41|alc294"
  ```
* **Confirm zero DMA errors during audio playback:**
  ```bash
  sudo dmesg | grep -i "dma error"
  ```

---
---

## Español

Guía de configuración post-instalación para laptops **ASUS ROG Strix con procesadores Intel** (probado en **G614JV-AS74**) en **CachyOS (Arch Linux)**. Resuelve el ruido de estática en los altavoces provocado por errores de transferencia DMA y configura las herramientas nativas de control ASUS ROG.

### Tabla de Contenidos
1. [Solución al ruido de estática y errores DMA](#1-solución-al-ruido-de-estática-y-errores-dma)
2. [Aumentar volumen máximo y guardar niveles en ALSA](#2-aumentar-volumen-máximo-y-guardar-niveles-en-alsa)
3. [Instalación de `asusctl` y ROG Control Center](#3-instalación-de-asusctl-y-rog-control-center)
4. [Comandos de verificación rápida](#4-comandos-de-verificación-rápida)

---

### 1. Solución al ruido de estática y errores DMA

#### Problema
Al reproducir audio en los altavoces internos se escucha estática o interferencia similar a una radio mal sintonizada. Al revisar `dmesg`, el controlador legado `snd_hda_intel` arroja errores continuos:
```text
snd_hda_intel 0000:00:1f.3: stream 7 dma error: 0x28
```

#### Solución
Cambiar el controlador legado por **Sound Open Firmware (SOF)**:

1. **Instalar el firmware de SOF y los perfiles UCM:**
   ```bash
   sudo pacman -S --needed sof-firmware alsa-ucm-conf alsa-utils
   ```

2. **Forzar el uso del controlador SOF (`dsp_driver=3`):**
   ```bash
   echo "options snd-intel-dspcfg dsp_driver=3" | sudo tee /etc/modprobe.d/asus-audio.conf
   ```

3. **Reiniciar el equipo** para aplicar el cambio en el kernel.

---

### 2. Aumentar volumen máximo y guardar niveles en ALSA

#### Problema
Tras cambiar a `sof-hda-dsp`, el sonido es nítido pero el volumen máximo se escucha bajo porque los canales de hardware en ALSA vienen limitados por defecto.

#### Solución
1. Abrir el mezclador en la terminal:
   ```bash
   alsamixer
   ```
2. Presionar **`F6`** y elegir la tarjeta **`sof-hda-dsp`**.
3. Subir las barras de **Speaker** y **Headphone** hasta `100` (asegurándose de que tengan `00` abajo y no `MM`).
4. Salir con `Esc` y guardar la configuración de forma permanente para que no se reinicie al apagar:
   ```bash
   sudo alsactl store
   ```

---

### 3. Instalación de `asusctl` y ROG Control Center

1. **Importar y firmar la llave GPG oficial de ASUS Linux:**
   ```bash
   sudo pacman-key --recv-keys 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
   sudo pacman-key --finger 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
   sudo pacman-key --lsign-key 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
   ```

2. **Añadir el repositorio `[g14]` a `/etc/pacman.conf`:**
   ```bash
   echo -e "\n[g14]\nServer = https://arch.asus-linux.org" | sudo tee -a /etc/pacman.conf
   ```

3. **Instalar `asusctl` y `rog-control-center`:**
   ```bash
   sudo pacman -Syu asusctl rog-control-center
   ```

4. **Habilitar el servicio de perfiles de energía:**
   ```bash
   sudo systemctl enable --now power-profiles-daemon.service
   ```

---

### 4. Comandos de verificación rápida

* **Verificar que los amplificadores Cirrus Logic CS35L41 cargaron su calibración de fábrica:**
  ```bash
  sudo dmesg | grep -iE "cs35l41|alc294"
  ```
* **Confirmar que no hay errores de DMA al reproducir audio:**
  ```bash
  sudo dmesg | grep -i "dma error"
  ```
