# MPV Windows-Linux +shader scaricabili (Vulkan, D3D11)

Guida pratica per installare e configurare **MPV** su Windows e Linux con profili modulari per GPU AMD, NVIDIA e Intel, shader CAS/FSR/NIS scaricabili separatamente, debanding e troubleshooting.

L'obiettivo è avere una configurazione semplice da mantenere, leggera quanto basta da non trasformare ogni film in un test termico, ma curata abbastanza da migliorare nitidezza, scaling e fluidità senza venerare pixel sfocati come reliquie medievali del bitrate.

---

## Indice

- [Obiettivo](#obiettivo)
- [Nota importante su shader e repository](#nota-importante-su-shader-e-repository)
- [Prerequisiti](#prerequisiti)
- [Installazione su Windows](#installazione-su-windows)
- [Installazione su Windows con GUI](#installazione-su-windows-con-gui)
- [Installazione su Linux](#installazione-su-linux)
- [Installazione su Linux con GUI](#installazione-su-linux-con-gui)
- [Clonare questo repository](#clonare-questo-repository)
- [Struttura della configurazione](#struttura-della-configurazione)
- [Shader consigliati](#shader-consigliati)
- [Download shader su Windows](#download-shader-su-windows)
- [Download shader su Linux](#download-shader-su-linux)
- [Configurazione Windows](#configurazione-windows)
- [Configurazione Linux](#configurazione-linux)
- [Profili disponibili](#profili-disponibili)
- [Esempi di utilizzo](#esempi-di-utilizzo)
- [input.conf opzionale](#inputconf-opzionale)
- [Verifica shader in esecuzione](#verifica-shader-in-esecuzione)
- [Debug](#debug)
- [Troubleshooting](#troubleshooting)
- [Regole pratiche](#regole-pratiche)
- [Licenza](#licenza)

---

## Obiettivo

Questa guida fornisce una configurazione MPV per:

- Windows con backend `d3d11`
- Linux con backend `vulkan`
- AMD Radeon Polaris / Vega / RDNA
- NVIDIA GTX / RTX
- Intel iGPU / Arc
- shader CAS, CAS-scaled, FSR e NIS scaricati dall'utente
- debanding leggero di default
- profilo debanding più aggressivo opzionale
- profili modulari richiamabili da riga di comando
- uso da terminale oppure tramite frontend grafico

La configurazione è pensata per uso reale: film, anime, serie TV, video compressi, sorgenti 720p/1080p su display più risoluti.

---

## Nota importante su shader e repository

Questo repository **non include shader** e non deve contenere una cartella `shaders` versionata.

Gli shader citati sono liberamente scaricabili dai rispettivi repository, ma restano proprietà dei rispettivi autori e vanno installati localmente dall'utente nella cartella di configurazione di MPV.

In pratica:

```text
Repository GitHub:
├── README.md
├── mpv.conf        # opzionale, se vuoi tenerlo come file pronto
└── input.conf      # opzionale, se vuoi tenerlo come file pronto
```

Non committare:

```text
shaders/
*.glsl
*.hook
*.zip
```

La cartella `shaders` esiste solo sul PC dell'utente, dentro la directory di configurazione di MPV o del frontend grafico.

---

## Prerequisiti

Servono:

- MPV aggiornato
- Git, utile per scaricare e aggiornare questo progetto e gli shader
- una GPU con driver funzionanti
- un terminale: PowerShell su Windows, Bash/Zsh/Fish su Linux
- `unzip` su Linux, se vuoi estrarre pacchetti `.zip` da terminale
- opzionale: un frontend grafico, per esempio `mpv.net`, `Celluloid`, `Haruna` o `SMPlayer`

---

## Installazione su Windows

Questa sezione installa MPV classico, cioè il player pilotabile da terminale.

### 1. Verificare winget

Apri **PowerShell**:

```powershell
winget --version
```

Se `winget` risponde, procedi.

### 2. Installare Git e MPV

```powershell
winget install -e --id Git.Git
winget install -e --id shinchiro.mpv
```

Verifica:

```powershell
git --version
mpv --version
where.exe mpv
```

Se `mpv` non viene trovato nel `PATH`, chiudi e riapri PowerShell. Windows ogni tanto deve fissare il vuoto cosmico prima di ricordarsi il PATH.

### 3. Creare la cartella di configurazione MPV

```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\mpv" | Out-Null
New-Item -ItemType Directory -Force "$env:APPDATA\mpv\shaders" | Out-Null
```

### 4. Copiare la configurazione

Se nel repository sono presenti `mpv.conf` e `input.conf` come file separati:

```powershell
Copy-Item .\mpv.conf "$env:APPDATA\mpv\mpv.conf" -Force
Copy-Item .\input.conf "$env:APPDATA\mpv\input.conf" -Force
```

Se invece stai usando solo questo README, crea i file manualmente:

```powershell
notepad "$env:APPDATA\mpv\mpv.conf"
notepad "$env:APPDATA\mpv\input.conf"
```

Poi incolla i blocchi delle sezioni [Configurazione Windows](#configurazione-windows) e [input.conf opzionale](#inputconf-opzionale).

---

## Installazione su Windows con GUI

MPV nasce come player minimalista/command-line. Se vuoi una GUI vera, la scelta più sensata su Windows è **mpv.net**: è basato su MPV/libmpv, offre interfaccia grafica moderna e mantiene compatibilità con molte opzioni di MPV.

### 1. Installare mpv.net

```powershell
winget install -e --id mpv.net
```

### 2. Aprire la cartella di configurazione di mpv.net

mpv.net usa una cartella diversa da MPV classico:

```text
%APPDATA%\mpv.net
```

Puoi aprirla anche dalla GUI:

```text
Tasto destro nella finestra di mpv.net -> Config -> Open Config Folder
```

Da PowerShell:

```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\mpv.net" | Out-Null
New-Item -ItemType Directory -Force "$env:APPDATA\mpv.net\shaders" | Out-Null
explorer "$env:APPDATA\mpv.net"
```

### 3. Copiare la configurazione per mpv.net

Se nel repository sono presenti `mpv.conf` e `input.conf`:

```powershell
Copy-Item .\mpv.conf "$env:APPDATA\mpv.net\mpv.conf" -Force
Copy-Item .\input.conf "$env:APPDATA\mpv.net\input.conf" -Force
```

Gli shader vanno scaricati nella cartella locale di mpv.net:

```text
%APPDATA%\mpv.net\shaders
```

Nota: `~~/shaders/NOME.glsl` viene risolto rispetto alla cartella config del player in uso. Quindi MPV classico usa `%APPDATA%\mpv\shaders`, mentre mpv.net usa `%APPDATA%\mpv.net\shaders`.

---

## Installazione su Linux

### Fedora / Nobara

```bash
sudo dnf install mpv git unzip
```

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install mpv git unzip
```

### Arch / EndeavourOS / Manjaro

```bash
sudo pacman -S mpv git unzip
```

### Creare la cartella di configurazione

```bash
mkdir -p ~/.config/mpv/shaders
```

### Verifica

```bash
mpv --version
git --version
```

---

## Installazione su Linux con GUI

Su Linux puoi usare MPV puro oppure un frontend grafico basato su MPV/libmpv.

Scelte consigliate:

| GUI | Toolkit | Quando usarla |
|---|---|---|
| `Celluloid` | GTK | scelta semplice e pulita, ottima su GNOME/Cinnamon |
| `Haruna` | Qt/KDE | ottima su KDE Plasma, più ricca di opzioni |
| `SMPlayer` | Qt | frontend storico, multipiattaforma, molto configurabile |

### Celluloid via Flatpak

```bash
flatpak install flathub io.github.celluloid_player.Celluloid
```

Celluloid può usare i file di configurazione di MPV. Se usi la versione Flatpak e qualcosa non viene letto, controlla le preferenze del programma e la sandbox Flatpak: non è magia nera, solo isolamento applicativo con una maschera da burocrazia.

### Haruna via Flatpak

```bash
flatpak install flathub org.kde.haruna
```

Haruna è basato su libmpv e funziona bene come player grafico moderno, specialmente in ambiente KDE.

### SMPlayer

Fedora / Nobara:

```bash
sudo dnf install smplayer
```

Ubuntu / Debian:

```bash
sudo apt update
sudo apt install smplayer
```

Arch / EndeavourOS / Manjaro:

```bash
sudo pacman -S smplayer
```

Dentro SMPlayer seleziona MPV come motore:

```text
Options -> Preferences -> General -> Multimedia engine -> mpv
```

Nota pratica: i frontend grafici non sempre leggono automaticamente la stessa identica cartella config di `mpv` CLI. Se un profilo funziona da terminale ma non dalla GUI, verifica la cartella config del frontend e copia lì `mpv.conf`, `input.conf` e la cartella locale `shaders`.

---

## Clonare questo repository

Repository:

```text
https://github.com/Damocle77/MPV-Player.git
```

Windows PowerShell:

```powershell
cd "$env:USERPROFILE\Downloads"
git clone https://github.com/Damocle77/MPV-Player.git
cd .\MPV-Player
```

Linux:

```bash
cd ~/Downloads
git clone https://github.com/Damocle77/MPV-Player.git
cd ./MPV-Player
```

Il repository contiene guida e configurazioni. Gli shader si scaricano a parte nelle sezioni dedicate.

---

## Struttura della configurazione

### Windows MPV classico

```text
%APPDATA%\mpv\
├── mpv.conf
├── input.conf
└── shaders\              # cartella locale, NON inclusa nel repository
    ├── CAS.glsl
    ├── CAS-scaled.glsl
    ├── FSR.glsl
    ├── NVScaler.glsl      # opzionale NVIDIA
    └── NVSharpen.glsl     # opzionale NVIDIA
```

### Windows mpv.net

```text
%APPDATA%\mpv.net\
├── mpv.conf
├── input.conf
├── mpvnet.conf            # specifico di mpv.net
└── shaders\              # cartella locale, NON inclusa nel repository
    ├── CAS.glsl
    ├── CAS-scaled.glsl
    ├── FSR.glsl
    ├── NVScaler.glsl
    └── NVSharpen.glsl
```

### Linux MPV classico

```text
~/.config/mpv/
├── mpv.conf
├── input.conf
└── shaders/              # cartella locale, NON inclusa nel repository
    ├── CAS.glsl
    ├── CAS-scaled.glsl
    ├── FSR.glsl
    ├── NVScaler.glsl
    └── NVSharpen.glsl
```

### Shader extra opzionali

Puoi aggiungere shader avanzati creando sottocartelle locali:

```text
shaders/
├── FSRCNNX/
├── Anime4K/
└── Nnedi3-RAVU/
    ├── OpenGL/
    └── Vulkan/
```

La distinzione `OpenGL` / `Vulkan` serve soprattutto per shader `.hook` come RAVU/NNEDI3: MPV carica il file che gli indichi, non indovina la variante giusta leggendo i fondi del caffè.

---

## Shader consigliati

### Shader base universali

| Shader | AMD | NVIDIA | Intel | Uso |
|---|---:|---:|---:|---|
| `CAS.glsl` | sì | sì | sì | sharpening leggero |
| `CAS-scaled.glsl` | sì | sì | sì | preset quotidiano equilibrato |
| `FSR.glsl` | sì | sì | sì | upscaling/sharpening più marcato |

Repository consigliato:

```text
https://github.com/agyild/glsl-shaders
```

### Shader NVIDIA opzionali

| Shader | Uso consigliato |
|---|---|
| `NVScaler.glsl` | NVIDIA Image Scaling, scaling + sharpening |
| `NVSharpen.glsl` | sharpening NVIDIA senza scaling |

Repository:

```text
https://github.com/kevinlekiller/NVScaler
```

Nota: essendo shader GLSL possono anche partire su altre GPU, ma per AMD/Intel il percorso sano resta `CAS`, `CAS-scaled` o `FSR`.

### Extra avanzati opzionali

| Shader / progetto | Da dove scaricarlo | Uso | Note |
|---|---|---|---|
| Anime4K | `https://github.com/bloc97/Anime4K/releases` | anime/cartoon | usare i pacchetti release ufficiali |
| FSRCNNX | `https://github.com/igv/FSRCNN-TensorFlow/releases` | super-resolution avanzata | pesante, utile su sorgenti basse |
| RAVU / NNEDI3 | `https://github.com/bjin/mpv-prescalers` | upscaling avanzato | scegliere variante Vulkan/OpenGL corretta |
| Adaptive Sharpen | `https://gist.github.com/igv/8a77e4eb8276753b54bb94c1c50c317e` | sharpening generale | usare con moderazione |

Regola: la guida base resta su `CAS`, `CAS-scaled`, `FSR`. Gli extra vanno documentati, non buttati nel preset principale come coriandoli in sala server.

---

## Download shader su Windows

### MPV classico

```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\mpv\shaders" | Out-Null
cd "$env:APPDATA\mpv\shaders"

git clone https://github.com/agyild/glsl-shaders.git temp-glsl-shaders
Copy-Item .\temp-glsl-shaders\shaders\CAS.glsl . -Force
Copy-Item .\temp-glsl-shaders\shaders\CAS-scaled.glsl . -Force
Copy-Item .\temp-glsl-shaders\shaders\FSR.glsl . -Force
```

Shader NVIDIA opzionali:

```powershell
git clone https://github.com/kevinlekiller/NVScaler.git temp-nvscaler
Copy-Item .\temp-nvscaler\NVScaler.glsl . -Force
Copy-Item .\temp-nvscaler\NVSharpen.glsl . -Force
```

Verifica:

```powershell
Get-ChildItem "$env:APPDATA\mpv\shaders"
```

### mpv.net

```powershell
New-Item -ItemType Directory -Force "$env:APPDATA\mpv.net\shaders" | Out-Null
cd "$env:APPDATA\mpv.net\shaders"

git clone https://github.com/agyild/glsl-shaders.git temp-glsl-shaders
Copy-Item .\temp-glsl-shaders\shaders\CAS.glsl . -Force
Copy-Item .\temp-glsl-shaders\shaders\CAS-scaled.glsl . -Force
Copy-Item .\temp-glsl-shaders\shaders\FSR.glsl . -Force
```

Shader NVIDIA opzionali per mpv.net:

```powershell
git clone https://github.com/kevinlekiller/NVScaler.git temp-nvscaler
Copy-Item .\temp-nvscaler\NVScaler.glsl . -Force
Copy-Item .\temp-nvscaler\NVSharpen.glsl . -Force
```

Verifica:

```powershell
Get-ChildItem "$env:APPDATA\mpv.net\shaders"
```

Struttura minima consigliata:

```text
CAS.glsl
CAS-scaled.glsl
FSR.glsl
```

Struttura completa con NVIDIA NIS:

```text
CAS.glsl
CAS-scaled.glsl
FSR.glsl
NVScaler.glsl
NVSharpen.glsl
```

---

## Download shader su Linux

```bash
mkdir -p ~/.config/mpv/shaders
cd ~/.config/mpv/shaders

git clone https://github.com/agyild/glsl-shaders.git temp-glsl-shaders
cp temp-glsl-shaders/shaders/CAS.glsl .
cp temp-glsl-shaders/shaders/CAS-scaled.glsl .
cp temp-glsl-shaders/shaders/FSR.glsl .
```

Shader NVIDIA opzionali:

```bash
git clone https://github.com/kevinlekiller/NVScaler.git temp-nvscaler
cp temp-nvscaler/NVScaler.glsl .
cp temp-nvscaler/NVSharpen.glsl .
```

Verifica:

```bash
ls -lh ~/.config/mpv/shaders
```

Per frontend Flatpak, verifica la cartella di configurazione effettiva del frontend. Se non legge `~/.config/mpv`, copia `mpv.conf`, `input.conf` e gli shader nella cartella config del frontend.

---

## Configurazione Windows

Crea o modifica:

```powershell
notepad "$env:APPDATA\mpv\mpv.conf"
```

Per mpv.net:

```powershell
notepad "$env:APPDATA\mpv.net\mpv.conf"
```

Contenuto consigliato:

```ini
# ============================================================
# MPV Windows - AMD / NVIDIA / Intel
# Backend D3D11, profili modulari, shader CAS/FSR/NIS
# ============================================================

vo=gpu-next
gpu-api=d3d11
profile=gpu-hq

# Hardware decoding generico stabile su Windows
hwdec=d3d11va
hwdec-codecs=all

# Scaling
scale=ewa_lanczossharp
cscale=ewa_lanczossharp
dscale=mitchell

correct-downscaling=yes
linear-downscaling=yes
sigmoid-upscaling=no

# Deband leggero di default
deband=yes
deband-iterations=1
deband-threshold=24
deband-range=12
deband-grain=2

# Fluidità
video-sync=display-resample
interpolation=yes
tscale=oversample

# Audio Windows
ao=wasapi
audio-channels=auto
audio-spdif=no

# ============================================================
# PROFILI GPU
# ============================================================

[amd-radeon]
profile-desc=AMD Radeon Windows D3D11
hwdec=d3d11va
hwdec-codecs=all

[nvidia]
profile-desc=NVIDIA Windows D3D11
hwdec=d3d11va
hwdec-codecs=all

[nvidia-nvdec]
profile-desc=NVIDIA Windows NVDEC opzionale
hwdec=nvdec
hwdec-codecs=all

[intel]
profile-desc=Intel iGPU/Arc Windows D3D11
hwdec=d3d11va
hwdec-codecs=all

# ============================================================
# PROFILI SHADER BASE
# glsl-shaders-clr evita di accumulare shader come una collezione
# di problemi in produzione.
# ============================================================

[cas]
profile-desc=CAS sharpening leggero
glsl-shaders-clr
glsl-shader="~~/shaders/CAS.glsl"

[cas-scaled]
profile-desc=CAS scaled, buon compromesso per upscaling leggero
glsl-shaders-clr
glsl-shader="~~/shaders/CAS-scaled.glsl"

[fsr]
profile-desc=FSR upscaling/sharpening
glsl-shaders-clr
glsl-shader="~~/shaders/FSR.glsl"

[nis]
profile-desc=NVIDIA Image Scaling, consigliato solo su NVIDIA
glsl-shaders-clr
glsl-shader="~~/shaders/NVScaler.glsl"

[nvsharpen]
profile-desc=NVIDIA sharpen leggero, consigliato solo su NVIDIA
glsl-shaders-clr
glsl-shader="~~/shaders/NVSharpen.glsl"

[deband-heavy]
profile-desc=Deband aggressivo per banding evidente
deband=yes
deband-iterations=2
deband-threshold=32
deband-range=16
deband-grain=3

# ============================================================
# PROFILI SHADER EXTRA - opzionali, richiedono download manuale
# ============================================================

[adaptive-sharpen]
profile-desc=Sharpening adattivo universale
glsl-shaders-clr
glsl-shader="~~/shaders/adaptive-sharpen.glsl"

[krigbilateral]
profile-desc=Krig Bilateral chroma scaler
glsl-shaders-clr
glsl-shader="~~/shaders/KrigBilateral.glsl"

[ssim-down]
profile-desc=SSimDownscaler, downscale qualità
glsl-shaders-clr
glsl-shader="~~/shaders/SSimDownscaler.glsl"

[ssim-sr]
profile-desc=SSimSuperRes, sharpen/super-res
glsl-shaders-clr
glsl-shader="~~/shaders/SSimSuperRes.glsl"

[fsrcnnx-x2]
profile-desc=FSRCNNX x2 neural SR
glsl-shaders-clr
glsl-shader="~~/shaders/FSRCNNX/FSRCNNX_x2_16-0-4-1.glsl"

[nnedi3]
profile-desc=NNEDI3 nns32 OpenGL gather
glsl-shaders-clr
glsl-shader="~~/shaders/Nnedi3-RAVU/OpenGL/nnedi3-nns32-win8x4.hook"

[ravu-r3]
profile-desc=RAVU r3 YUV OpenGL gather
glsl-shaders-clr
glsl-shader="~~/shaders/Nnedi3-RAVU/OpenGL/ravu-r3-yuv.hook"
```

---

## Configurazione Linux

Crea o modifica:

```bash
nano ~/.config/mpv/mpv.conf
```

Contenuto consigliato:

```ini
# ============================================================
# MPV Linux - AMD / NVIDIA / Intel
# Backend Vulkan, profili modulari, shader CAS/FSR/NIS
# ============================================================

vo=gpu-next
gpu-api=vulkan
profile=gpu-hq

# Hardware decoding Linux
# auto-safe è la scelta più prudente e portabile.
hwdec=auto-safe
hwdec-codecs=all

# Scaling
scale=ewa_lanczossharp
cscale=ewa_lanczossharp
dscale=mitchell

correct-downscaling=yes
linear-downscaling=yes
sigmoid-upscaling=no

# Deband leggero di default
deband=yes
deband-iterations=1
deband-threshold=24
deband-range=12
deband-grain=2

# Fluidità
video-sync=display-resample
interpolation=yes
tscale=oversample

# Audio Linux
ao=pulse,pipewire,alsa
audio-channels=auto
audio-spdif=no

# ============================================================
# PROFILI GPU
# ============================================================

[linux-amd]
profile-desc=Linux AMD Vulkan + VAAPI
hwdec=vaapi
hwdec-codecs=all

[linux-intel]
profile-desc=Linux Intel Vulkan + VAAPI
hwdec=vaapi
hwdec-codecs=all

[linux-nvidia]
profile-desc=Linux NVIDIA Vulkan, decoding automatico sicuro
hwdec=auto-safe
hwdec-codecs=all

[linux-nvidia-nvdec]
profile-desc=Linux NVIDIA NVDEC opzionale
hwdec=nvdec
hwdec-codecs=all

[linux-safe]
profile-desc=Profilo compatibile senza decoding hardware forzato
hwdec=no

# ============================================================
# PROFILI SHADER BASE
# ============================================================

[cas]
profile-desc=CAS sharpening leggero
glsl-shaders-clr
glsl-shader="~~/shaders/CAS.glsl"

[cas-scaled]
profile-desc=CAS scaled, buon compromesso per upscaling leggero
glsl-shaders-clr
glsl-shader="~~/shaders/CAS-scaled.glsl"

[fsr]
profile-desc=FSR upscaling/sharpening
glsl-shaders-clr
glsl-shader="~~/shaders/FSR.glsl"

[nis]
profile-desc=NVIDIA Image Scaling, consigliato solo su NVIDIA
glsl-shaders-clr
glsl-shader="~~/shaders/NVScaler.glsl"

[nvsharpen]
profile-desc=NVIDIA sharpen leggero, consigliato solo su NVIDIA
glsl-shaders-clr
glsl-shader="~~/shaders/NVSharpen.glsl"

[deband-heavy]
profile-desc=Deband aggressivo per banding evidente
deband=yes
deband-iterations=2
deband-threshold=32
deband-range=16
deband-grain=3

# ============================================================
# PROFILI SHADER EXTRA - opzionali, richiedono download manuale
# ============================================================

[adaptive-sharpen]
profile-desc=Sharpening adattivo universale
glsl-shaders-clr
glsl-shader="~~/shaders/adaptive-sharpen.glsl"

[krigbilateral]
profile-desc=Krig Bilateral chroma scaler
glsl-shaders-clr
glsl-shader="~~/shaders/KrigBilateral.glsl"

[ssim-down]
profile-desc=SSimDownscaler, downscale qualità
glsl-shaders-clr
glsl-shader="~~/shaders/SSimDownscaler.glsl"

[ssim-sr]
profile-desc=SSimSuperRes, sharpen/super-res
glsl-shaders-clr
glsl-shader="~~/shaders/SSimSuperRes.glsl"

[fsrcnnx-x2]
profile-desc=FSRCNNX x2 neural SR
glsl-shaders-clr
glsl-shader="~~/shaders/FSRCNNX/FSRCNNX_x2_16-0-4-1.glsl"

[ravu-r3-vulkan]
profile-desc=RAVU r3 YUV Vulkan
glsl-shaders-clr
glsl-shader="~~/shaders/Nnedi3-RAVU/Vulkan/ravu-r3-yuv.hook"
```

---

## Profili disponibili

| Profilo | Sistema | Uso |
|---|---:|---|
| `amd-radeon` | Windows | AMD con D3D11VA |
| `nvidia` | Windows | NVIDIA con D3D11VA |
| `nvidia-nvdec` | Windows | NVIDIA con NVDEC opzionale |
| `intel` | Windows | Intel iGPU / Arc |
| `linux-amd` | Linux | AMD con VAAPI |
| `linux-intel` | Linux | Intel con VAAPI |
| `linux-nvidia` | Linux | NVIDIA con decoding automatico sicuro |
| `linux-nvidia-nvdec` | Linux | NVIDIA con NVDEC |
| `linux-safe` | Linux | niente decoding hardware forzato |
| `cas` | tutti | sharpening leggero |
| `cas-scaled` | tutti | scaling/sharpening equilibrato |
| `fsr` | tutti | upscaling più marcato |
| `nis` | NVIDIA | NVIDIA Image Scaling |
| `nvsharpen` | NVIDIA | NVIDIA sharpening senza scaling |
| `deband-heavy` | tutti | banding evidente |

---

## Esempi di utilizzo

### Windows AMD

```powershell
mpv --profile=amd-radeon,cas-scaled "D:\Video\film.mkv"
mpv --profile=amd-radeon,fsr "D:\Video\film.mkv"
```

### Windows NVIDIA

```powershell
mpv --profile=nvidia,cas-scaled "D:\Video\film.mkv"
mpv --profile=nvidia,nis "D:\Video\film.mkv"
mpv --profile=nvidia,nvsharpen "D:\Video\film.mkv"
```

### Windows mpv.net

Se hai aggiunto mpv.net al PATH:

```powershell
mpvnet --profile=nvidia,cas-scaled "D:\Video\film.mkv"
```

Oppure apri il file dalla GUI. I profili caricati da `mpv.conf` restano disponibili.

### Linux AMD

```bash
mpv --profile=linux-amd,cas-scaled ~/Video/film.mkv
mpv --profile=linux-amd,fsr ~/Video/film.mkv
```

### Linux NVIDIA

```bash
mpv --profile=linux-nvidia,cas-scaled ~/Video/film.mkv
mpv --profile=linux-nvidia-nvdec,nis ~/Video/film.mkv
```

### Linux Intel

```bash
mpv --profile=linux-intel,cas-scaled ~/Video/film.mkv
mpv --profile=linux-intel,fsr ~/Video/film.mkv
```

### Video con banding evidente

Linux:

```bash
mpv --profile=cas-scaled,deband-heavy ~/Video/film.mkv
```

Windows:

```powershell
mpv --profile=cas-scaled,deband-heavy "D:\Video\film.mkv"
```

---

## input.conf opzionale

### Windows MPV classico

```powershell
notepad "$env:APPDATA\mpv\input.conf"
```

### Windows mpv.net

```powershell
notepad "$env:APPDATA\mpv.net\input.conf"
```

### Linux

```bash
nano ~/.config/mpv/input.conf
```

Contenuto consigliato:

```conf
# ============================================================
# MPV input.conf - toggle shader, stats, debug rapido
# ============================================================

# Shader rapidi base
# CTRL+1 / CTRL+2 / CTRL+3 cambiano shader al volo.
# CTRL+0 pulisce la lista shader.
# CTRL+S mostra quali shader risultano attivi.

CTRL+1 no-osd change-list glsl-shaders set "~~/shaders/CAS.glsl" ; show-text "Shader: CAS" 2000
CTRL+2 no-osd change-list glsl-shaders set "~~/shaders/CAS-scaled.glsl" ; show-text "Shader: CAS-scaled" 2000
CTRL+3 no-osd change-list glsl-shaders set "~~/shaders/FSR.glsl" ; show-text "Shader: FSR" 2000
CTRL+4 no-osd change-list glsl-shaders set "~~/shaders/NVScaler.glsl" ; show-text "Shader: NVIDIA NIS" 2000
CTRL+5 no-osd change-list glsl-shaders set "~~/shaders/NVSharpen.glsl" ; show-text "Shader: NVIDIA Sharpen" 2000
CTRL+0 no-osd change-list glsl-shaders clr "" ; show-text "Shader GLSL disattivati" 2000
CTRL+s show-text "Shader attivi: ${glsl-shaders}" 5000

# Shader extra: abilitali solo dopo aver scaricato i file.
#CTRL+6 no-osd change-list glsl-shaders set "~~/shaders/FSRCNNX/FSRCNNX_x2_16-0-4-1.glsl" ; show-text "Shader: FSRCNNX x2" 2000
#CTRL+7 no-osd change-list glsl-shaders set "~~/shaders/Nnedi3-RAVU/Vulkan/ravu-r3-yuv.hook" ; show-text "Shader: RAVU r3 Vulkan" 2000
#CTRL+8 no-osd change-list glsl-shaders set "~~/shaders/Nnedi3-RAVU/OpenGL/ravu-r3-yuv.hook" ; show-text "Shader: RAVU r3 OpenGL" 2000
#CTRL+9 no-osd change-list glsl-shaders set "~~/shaders/adaptive-sharpen.glsl" ; show-text "Shader: Adaptive Sharpen" 2000

# Stats / overlay diagnostici
i script-binding stats/display-stats-toggle
TAB script-binding stats/display-stats-toggle
? script-binding stats/display-page-5

# Screenshot per confronto A/B
s screenshot
S screenshot video

# Volume
UP add volume 5
DOWN add volume -5

# Audio/sub delay
CTRL+LEFT add audio-delay -0.050
CTRL+RIGHT add audio-delay 0.050
ALT+LEFT add sub-delay -0.050
ALT+RIGHT add sub-delay 0.050
```

Nota pratica: `change-list glsl-shaders set` sostituisce lo shader attivo. È più sicuro di `append`, perché evita di impilare tre upscaler uno sopra l'altro come una lasagna di artefatti.

---

## Verifica shader in esecuzione

### Metodo 1: OSD durante la riproduzione

Con l'`input.conf` consigliato:

```text
CTRL+s
```

Se tutto funziona vedrai qualcosa tipo:

```text
Shader attivi: ~~ / shaders / CAS-scaled.glsl
```

Gli spazi qui sopra sono solo per leggibilità. Nel player comparirà il path reale. Se la riga è vuota, MPV non ha shader GLSL attivi.

### Metodo 2: overlay statistiche

Durante il video premi:

```text
i
```

oppure:

```text
TAB
```

L'overlay mostra renderer, decoder, frame drop, timing e informazioni video. Non sempre mostra la lista shader in modo esplicito, quindi `CTRL+s` resta la verifica diretta.

### Metodo 3: log da terminale

Windows:

```powershell
mpv --profile=cas-scaled --msg-level=all=v "D:/Video/film.mkv" 2>&1 | Tee-Object "$env:TEMP\mpv-shader.log"
Select-String -Path "$env:TEMP\mpv-shader.log" -Pattern "glsl|shader|CAS|FSR|RAVU|FSRCNNX|NIS"
```

Linux:

```bash
mpv --profile=cas-scaled --msg-level=all=v ~/Video/film.mkv 2>&1 | tee /tmp/mpv-shader.log
grep -Ei 'glsl|shader|CAS|FSR|RAVU|FSRCNNX|NIS' /tmp/mpv-shader.log
```

Se MPV non trova un file shader, nel log lo dice. Non sempre con poesia, ma lo dice.

### Metodo 4: confronto A/B con screenshot

Usa lo stesso frame, scatta uno screenshot senza shader e uno con shader.

Windows:

```powershell
mpv --no-config --pause "D:/Video/film.mkv"
mpv --profile=cas-scaled --pause "D:/Video/film.mkv"
```

Linux:

```bash
mpv --no-config --pause ~/Video/film.mkv
mpv --profile=cas-scaled --pause ~/Video/film.mkv
```

Poi premi `s` nello stesso punto del video. Se confronti frame diversi, stai facendo benchmarking creativo, non analisi.

### Metodo 5: verifica path shader

Windows MPV classico:

```powershell
Get-ChildItem "$env:APPDATA\mpv\shaders"
```

Windows mpv.net:

```powershell
Get-ChildItem "$env:APPDATA\mpv.net\shaders"
```

Linux:

```bash
ls -lh ~/.config/mpv/shaders
```

---

## Debug

### Versione MPV

Windows:

```powershell
mpv --version
```

Linux:

```bash
mpv --version
```

### Log completo

Windows:

```powershell
mpv --msg-level=all=v "D:\Video\film.mkv"
```

Linux:

```bash
mpv --msg-level=all=v ~/Video/film.mkv
```

### Debug video output e decoder

```bash
mpv --msg-level=vd=debug,vo=debug file.mkv
```

### Test shader

```bash
mpv --profile=cas-scaled --msg-level=all=v file.mkv
```

Cerca nel log riferimenti a:

```text
glsl
shader
CAS
FSR
NVScaler
```

---

## Troubleshooting

### MPV non trova lo shader

Controlla che i file siano davvero nella cartella corretta.

Windows MPV classico:

```powershell
Get-ChildItem "$env:APPDATA\mpv\shaders"
```

Windows mpv.net:

```powershell
Get-ChildItem "$env:APPDATA\mpv.net\shaders"
```

Linux:

```bash
ls -lh ~/.config/mpv/shaders
```

Controlla anche che Windows non abbia creato file tipo:

```text
CAS.glsl.txt
FSR.glsl.txt
```

Nascondere le estensioni dei file continua a essere una delle peggiori idee UX mai sopravvissute al contatto con la civiltà.

### Video troppo tagliente

Scala così:

```text
fsr -> cas-scaled -> cas -> nessuno shader
nis -> nvsharpen -> cas -> nessuno shader
```

### Artefatti, rumore o bordi strani

Evita shader aggressivi su sorgenti brutte. Se il file è già compresso male, FSR/NIS possono solo lucidare il disastro.

Prova:

```bash
mpv --profile=cas file.mkv
```

oppure:

```bash
mpv --profile=linux-safe file.mkv
```

### Scatti o frame pacing irregolare

Prova a disattivare interpolazione e sync avanzato:

```ini
video-sync=audio
interpolation=no
```

Oppure lancia temporaneamente:

```bash
mpv --video-sync=audio --interpolation=no file.mkv
```

### Linux: VAAPI non funziona

Verifica driver e supporto:

```bash
vainfo
```

Se `vainfo` manca:

Fedora / Nobara:

```bash
sudo dnf install libva-utils
```

Ubuntu / Debian:

```bash
sudo apt install vainfo
```

Arch:

```bash
sudo pacman -S libva-utils
```

Se VAAPI dà problemi, usa:

```bash
mpv --profile=linux-safe file.mkv
```

### NVIDIA NVDEC instabile

Su Windows torna a:

```powershell
mpv --profile=nvidia,cas-scaled "D:\Video\film.mkv"
```

Su Linux torna a:

```bash
mpv --profile=linux-nvidia,cas-scaled ~/Video/film.mkv
```

### La GUI ignora i profili

Sintomo: da terminale funziona, dalla GUI no.

Cause probabili:

- la GUI usa una cartella config diversa
- stai usando una versione Flatpak con sandbox
- gli shader sono nella cartella di MPV CLI, ma non nella cartella della GUI
- `mpv.conf` è stato copiato nel posto sbagliato

Verifica prima il path config della GUI, poi copia lì:

```text
mpv.conf
input.conf
shaders/
```

La cartella `shaders` resta locale e non va committata nel repository.

---

## Regole pratiche

### AMD

Preset consigliati:

```text
cas-scaled
fsr
```

`cas-scaled` è il default più sensato. `fsr` ha senso quando la sorgente è sotto-risoluta rispetto al monitor.

### NVIDIA

Preset consigliati:

```text
cas-scaled
nis
nvsharpen
```

`cas-scaled` resta il profilo più neutro. `nis` è più aggressivo ed è pensato per scaling. `nvsharpen` è utile quando vuoi solo sharpening senza upscaling.

### Intel

Preset consigliati:

```text
cas
cas-scaled
fsr
```

Su Intel evita di partire da NIS/NVSharpen. Possono anche funzionare in alcune configurazioni, ma non sono il percorso consigliato. Prima CAS, poi CAS-scaled, poi FSR se serve più spinta.

### CAS

Buono per:

- anime
- Blu-ray morbidi
- encode mediamente compressi
- sorgenti già vicine alla risoluzione del monitor

### CAS-scaled

Buono per:

- uso quotidiano
- 720p/1080p morbidi
- upscaling leggero
- compromesso qualità/prestazioni

### FSR

Buono per:

- 720p verso 1080p/1440p/4K
- 1080p verso 1440p/4K
- sorgenti abbastanza pulite

Da evitare su:

- streaming molto compresso
- vecchi encode pieni di rumore
- video già oversharpened

### NIS / NVSharpen

Da trattare come profili NVIDIA.

`NIS` serve per scaling + sharpening.

`NVSharpen` serve quando vuoi sharpening senza scaling.

### Deband-heavy

Usalo solo quando serve davvero:

- cieli a blocchi
- gradienti visibili
- scene scure compresse male
- anime con fondali uniformi

---

## Cosa non fare

Evita combinazioni tipo:

```text
fsr + nis
nis + nvsharpen
cas-scaled + fsr + nis
```

Più shader non significa più qualità. Spesso significa prendere una sorgente già maltrattata e incidere i suoi difetti nel marmo con zelo quasi religioso.

---

## Setup consigliati rapidi

| Scenario | Profilo consigliato |
|---|---|
| Windows AMD | `amd-radeon,cas-scaled` |
| Windows AMD, upscaling marcato | `amd-radeon,fsr` |
| Windows NVIDIA neutro | `nvidia,cas-scaled` |
| Windows NVIDIA, scaling aggressivo | `nvidia,nis` |
| Windows NVIDIA, solo sharpening | `nvidia,nvsharpen` |
| Windows Intel | `intel,cas-scaled` |
| Windows Intel, upscaling marcato | `intel,fsr` |
| Linux AMD | `linux-amd,cas-scaled` |
| Linux Intel | `linux-intel,cas-scaled` |
| Linux NVIDIA neutro | `linux-nvidia,cas-scaled` |
| Linux NVIDIA NVDEC + NIS | `linux-nvidia-nvdec,nis` |
| Video con banding | `cas-scaled,deband-heavy` |
| Problemi strani | `linux-safe` oppure nessuno shader |

---

## Licenza

Questa guida può essere usata, modificata e adattata liberamente.

Gli shader citati non sono inclusi in questo repository e appartengono ai rispettivi repository/autori. Scaricali sempre dalle fonti originali indicate nella guida e verifica le relative licenze prima di redistribuirli.
