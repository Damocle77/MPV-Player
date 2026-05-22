# Guida MPV per Windows e Linux

Guida pratica per installare e configurare **MPV** con profili modulari per Windows e Linux, GPU AMD/NVIDIA/Intel, shader CAS/FSR/NIS, debanding e troubleshooting.

L'obiettivo è avere una configurazione semplice da mantenere, abbastanza leggera da non trasformare ogni film in un benchmark, ma abbastanza curata da migliorare nitidezza, scaling e fluidità.

---

## Indice

- [Obiettivo](#obiettivo)
- [Prerequisiti](#prerequisiti)
- [Struttura della configurazione](#struttura-della-configurazione)
- [Installazione su Windows](#installazione-su-windows)
- [Installazione su Linux](#installazione-su-linux)
- [Shader consigliati](#shader-consigliati)
- [Configurazione Windows](#configurazione-windows)
- [Configurazione Linux](#configurazione-linux)
- [Profili disponibili](#profili-disponibili)
- [Esempi di utilizzo](#esempi-di-utilizzo)
- [input.conf opzionale](#inputconf-opzionale)
- [Debug](#debug)
- [Troubleshooting](#troubleshooting)
- [Regole pratiche](#regole-pratiche)

---

## Obiettivo

Questa guida fornisce una configurazione MPV per:

- Windows con backend `d3d11`
- Linux con backend `vulkan`
- AMD Radeon Polaris / Vega / RDNA
- NVIDIA GTX / RTX
- Intel iGPU / Arc
- shader CAS, CAS-scaled, FSR, NIS
- debanding leggero di default
- profilo debanding più aggressivo opzionale
- profili modulari richiamabili da riga di comando

La configurazione è pensata per uso reale: film, anime, serie TV, video compressi, sorgenti 720p/1080p su display più risoluti.

---

## Prerequisiti

Servono:

- MPV aggiornato
- Git, utile per scaricare gli shader
- una GPU con driver funzionanti
- un terminale: PowerShell su Windows, shell Bash/Zsh/Fish su Linux

---

## Struttura della configurazione

### Windows

```text
%APPDATA%\mpv\
├── mpv.conf
├── input.conf
└── shaders\
    ├── CAS.glsl
    ├── CAS-scaled.glsl
    ├── FSR.glsl
    ├── NVScaler.glsl        # NVIDIA Image Scaling (opzionale)
    ├── NVSharpen.glsl       # Sharpen NVIDIA (opzionale)
    ├── adaptive-sharpen.glsl
    ├── KrigBilateral.glsl
    ├── SSimDownscaler.glsl
    ├── SSimSuperRes.glsl
    ├── FSRCNNX\          # neural upscaler
    │   ├── FSRCNNX_x2_16-0-4-1.glsl
    │   ├── FSRCNNX_x3_16-0-4-1.glsl
    │   └── FSRCNNX_x4_16-0-4-1.glsl
    ├── GLSL_High-end\     # Anime4K preset HQ
    │   ├── input.conf (preset)
    │   ├── mpv.conf   (preset)
    │   └── shaders\Anime4K_*.glsl
    ├── GLSL_Low-end\      # Anime4K preset leggeri
    │   └── …
    └── Nnedi3-RAVU\
        ├── OpenGL\       # shader gather / win_bgfx
        ├── Vulkan\       # shader compute / gpu-next
        ├── nnedi3-*.hook   # fallback universali
        └── ravu-*.hook
```

### Linux

```text
~/.config/mpv/
├── mpv.conf
├── input.conf
└── shaders/    # stessa struttura di Windows
```

> **Suggerimento**: mantieni le cartelle `Nnedi3-RAVU/OpenGL` e `Nnedi3-RAVU/Vulkan` separate. MPV carica il file che indichi, non cerca da solo "la versione giusta". In `vo=gpu-next + gpu-api=vulkan` usa la variant *Vulkan*. In `vo=gpu` (OpenGL) usa *OpenGL*.

---

### Contenuto della cartella `shaders`

| Voce | Cos'è | Quando ti serve |
|---|---|---|
| `CAS.glsl` / `CAS-scaled.glsl` / `FSR.glsl` | shader core universali | preset di base: sharpening leggero / upscaling leggero / FSR più spinto |
| `NVScaler.glsl` / `NVSharpen.glsl` | NVIDIA Image Scaling & Sharpen | su GPU NVIDIA per scaling+sharpen o solo sharpen |
| `adaptive-sharpen.glsl` | sharpening adattivo | video già nativi ma un po' soft |
| `KrigBilateral.glsl` | chroma scaler bilaterale | dopo un upscaler luma (es. RAVU) per qualità cromatica |
| `SSimDownscaler.glsl` | downscaler SSIM | quando riduci 4K → 1080p e vuoi qualità |
| `SSimSuperRes.glsl` | super‑resolution SSIM | boost di dettaglio leggero senza neural SR pesanti |
| **FSRCNNX/** | sub‑dir con shader neural SR x2/x3/x4 | sorgenti basse + GPU robusta |
| **GLSL_High‑end/** | preset Anime4K HQ (include `mpv.conf`/`input.conf`) | anime/cartoon su PC potente |
| **GLSL_Low‑end/** | preset Anime4K leggeri | anime su laptop/IGPU |
| **Nnedi3-RAVU/** | NNEDI3 & RAVU in varianti `OpenGL/` e `Vulkan/` + fallback root | upscaling avanzato: NNEDI3 per line‑art, RAVU per film/anime moderni |

In pratica: **scarichi, copi qui dentro, poi richiami il profilo** (`--profile=ravu-r3`, `--profile=fsrcnnx-x2`, ecc.). Se un profilo extra non parte, controlla di aver scelto la variante corretta per il backend (OpenGL vs Vulkan) e che il file sia davvero nel path indicato.

---

## Shader consigliati

La guida separa gli shader in tre gruppi:

1. **Core multipiattaforma**, consigliati per AMD, NVIDIA e Intel.
2. **NVIDIA-specifici**, da usare soprattutto su GPU NVIDIA.
3. **Extra avanzati**, utili per utenti che vogliono sperimentare e hanno GPU abbastanza robusta.

### Core consigliati

Questi sono gli shader da installare sempre:

```text
CAS.glsl
CAS-scaled.glsl
FSR.glsl
```

Repository:

```text
https://github.com/agyild/glsl-shaders
```

Uso pratico:

| Shader | AMD | NVIDIA | Intel | Uso consigliato |
|---|---:|---:|---:|---|
| `CAS.glsl` | sì | sì | sì | sharpening leggero |
| `CAS-scaled.glsl` | sì | sì | sì | preset quotidiano equilibrato |
| `FSR.glsl` | sì | sì | sì | upscaling più marcato |

### Shader NVIDIA-specifici

Questi shader implementano NVIDIA Image Scaling / Sharpening e nella guida vengono trattati come profili NVIDIA:

```text
NVScaler.glsl
NVSharpen.glsl
```

Repository:

```text
https://github.com/kevinlekiller/NVScaler
```

Uso pratico:

| Shader | Uso consigliato |
|---|---|
| `NVScaler.glsl` | upscaling NVIDIA Image Scaling |
| `NVSharpen.glsl` | sharpening NVIDIA quando non serve scalare |

Nota: essendo shader GLSL possono anche partire su altre GPU, ma non li consideriamo preset consigliati per AMD/Intel. Per AMD e Intel la strada sana è `CAS`, `CAS-scaled` o `FSR`.

### Extra avanzati opzionali

Questi non fanno parte del setup base, ma possono essere citati nella guida come estensioni.

| Shader / progetto | Da dove scaricarlo | Uso | Note |
|---|---|---|---|
| Anime4K | `https://github.com/bloc97/Anime4K/releases` | anime/cartoon | usare i pacchetti release ufficiali |
| FSRCNNX | `https://github.com/igv/FSRCNN-TensorFlow/releases` | super-resolution avanzata | scaricare il file `.glsl` desiderato dalla release |
| KrigBilateral | `https://gist.github.com/igv` oppure mirror community | chroma scaling | utile in catene shader avanzate |
| SSimDownscaler | `https://gist.github.com/igv` oppure mirror community | downscaling di qualità | utile quando riduci sorgenti più grandi |
| SSimSuperRes | `https://gist.github.com/igv` oppure mirror community | sharpening/super-res avanzato | più da profilo enthusiast |
| RAVU | `https://github.com/bjin/mpv-prescalers` | upscaling avanzato | usare la cartella `compute` o `gather` |
| Adaptive Sharpen | `https://gist.github.com/igv/8a77e4eb8276753b54bb94c1c50c317e` | sharpening generale | da usare con moderazione |

#### Download Anime4K

Anime4K è meglio scaricarlo dalle release ufficiali, perché contiene molti shader e preset già raggruppati.

Windows/Linux:

```text
https://github.com/bloc97/Anime4K/releases
```

Scarica lo ZIP della release, poi copia i file `.glsl` desiderati nella cartella `shaders` di MPV.

#### Download FSRCNNX

Scarica i file `.glsl` dalle release:

```text
https://github.com/igv/FSRCNN-TensorFlow/releases
```

Esempi comuni:

```text
FSRCNNX_x2_8-0-4-1.glsl
FSRCNNX_x2_16-0-4-1.glsl
```

Più alto non significa automaticamente meglio: significa anche più carico sulla GPU.

#### Download RAVU

RAVU sta nel repository `mpv-prescalers`:

Windows PowerShell:

```powershell
cd "$env:APPDATA\mpv\shaders"
git clone https://github.com/bjin/mpv-prescalers.git temp-mpv-prescalers
```

Linux:

```bash
cd ~/.config/mpv/shaders
git clone https://github.com/bjin/mpv-prescalers.git temp-mpv-prescalers
```

Dentro il repository guarda soprattutto:

```text
temp-mpv-prescalers/compute
temp-mpv-prescalers/gather
```

Molti shader RAVU hanno estensione `.hook`, per esempio:

```text
ravu-r4-yuv.hook
ravu-r4-yuv-opengl.hook
ravu-3x-r4-yuv.hook
```

In MPV l'estensione `.hook` indica semplicemente uno shader scritto nel formato user-shader/hook di MPV. Non è uno script esterno e non va messo nella cartella `scripts`: va caricato come normale shader con `glsl-shader` o `glsl-shaders`.

Esempio:

```ini
[linux-ravu]
glsl-shaders-clr
glsl-shader="~~/shaders/ravu-r4-yuv.hook"
```

Su Windows:

```ini
[windows-ravu]
glsl-shaders-clr
glsl-shader="~~/shaders/ravu-r4-yuv-opengl.hook"
```

Nota pratica: alcuni `.hook` sono pensati per backend o modalità specifiche. Se uno non parte, prova prima una variante meno specifica o cambia backend grafico. Naturalmente non potevano chiamarli tutti `.glsl`, sarebbe stato troppo misericordioso.

#### Download shader IGV singoli

(...)

---

### Quando usare gli shader extra

> Descrizioni lampo, così sai cosa caricare senza aprire un trattato di videocompressione.

| Shader / profilo | Quando usarlo | Nota pratica |
|---|---|---|
| `adaptive-sharpen` | Video già alla risoluzione nativa ma un po' soft; vuoi un tocco di nitidezza in più senza upscaling. | Sostituto di `cas` quando senti l'immagine molliccia. |
| `krigbilateral` | Dopo un upscaler luma (RAVU/FSRCNNX) per scalare il chroma con qualità. | Va **dopo** l'upscaling luma, non prima. |
| `ssim-down` (SSimDownscaler) | Stai riducendo 4K → 1080p e non vuoi perdere dettaglio. | Da usare **solo** se fai downscale. |
| `ssim-sr` (SSimSuperRes) | Vuoi un boost di dettaglio leggero su 1080p→1440p / 1440p→4K senza neural SR pesanti. | Poco più leggero di FSRCNNX. |
| `fsrcnnx-x2` | Sorgenti bassissima risoluzione (480p/720p) e GPU robusta. | Molto pesante; non combinarlo con RAVU. |
| `nnedi3` | Anime/line‑art nostalgico, ti serve nitidezza massima e accetti carico GPU elevato. | Variante `nns32` è compromesso; `nns128+` ≈ incendio. |
| `ravu-r3` / `ravu-r4` | Upscaling generale moderno. `r3` bilanciato, `r4` qualità massima. | Preferito su film live action; scegli cartella `Vulkan` o `OpenGL` in base al backend. |

Regola: la guida base resta su `CAS`, `CAS-scaled`, `FSR`. Gli extra vanno documentati, non buttati tutti nel preset principale come coriandoli su un server in produzione. `CAS`, `CAS-scaled`, `FSR`. Gli extra vanno documentati, non buttati tutti nel preset principale come coriandoli su un server in produzione.

---

## Download shader su Windows

### Shader core AMD/NVIDIA/Intel

```powershell
cd "$env:APPDATA\mpv\shaders"

git clone https://github.com/agyild/glsl-shaders.git temp-glsl-shaders
copy .\temp-glsl-shaders\shaders\CAS.glsl .
copy .\temp-glsl-shaders\shaders\CAS-scaled.glsl .
copy .\temp-glsl-shaders\shaders\FSR.glsl .
```

### Shader NVIDIA opzionali

```powershell
git clone https://github.com/kevinlekiller/NVScaler.git temp-nvscaler
copy .\temp-nvscaler\NVScaler.glsl .
copy .\temp-nvscaler\NVSharpen.glsl .
```

Verifica:

```powershell
dir "$env:APPDATA\mpv\shaders"
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

### Shader core AMD/NVIDIA/Intel

```bash
cd ~/.config/mpv/shaders

git clone https://github.com/agyild/glsl-shaders.git temp-glsl-shaders
cp temp-glsl-shaders/shaders/CAS.glsl .
cp temp-glsl-shaders/shaders/CAS-scaled.glsl .
cp temp-glsl-shaders/shaders/FSR.glsl .
```

### Shader NVIDIA opzionali

```bash
git clone https://github.com/kevinlekiller/NVScaler.git temp-nvscaler
cp temp-nvscaler/NVScaler.glsl .
cp temp-nvscaler/NVSharpen.glsl .
```

Verifica:

```bash
ls -lh ~/.config/mpv/shaders
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

## Configurazione Windows

Crea o modifica:

```powershell
notepad "$env:APPDATA\mpv\mpv.conf"
```

Incolla:

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
# PROFILI SHADER
# glsl-shaders-clr evita di accumulare shader come se fosse
# una collezione di problemi in produzione.
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
# PROFILI SHADER EXTRA (opzionali / sperimentali)
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
profile-desc=SSimDownscaler (downscale qualità)
glsl-shaders-clr
glsl-shader="~~/shaders/SSimDownscaler.glsl"

[ssim-sr]
profile-desc=SSimSuperRes (sharpen/super‑res)
glsl-shaders-clr
glsl-shader="~~/shaders/SSimSuperRes.glsl"

[fsrcnnx-x2]
profile-desc=FSRCNNX x2 neural SR
glsl-shaders-clr
glsl-shader="~~/shaders/FSRCNNX/FSRCNNX_x2_16-0-4-1.glsl"

[nnedi3]
profile-desc=NNEDI3 nns32 (OpenGL gather)
glsl-shaders-clr
glsl-shader="~~/shaders/Nnedi3-RAVU/OpenGL/nnedi3-nns32-win8x4.hook"

[ravu-r3]
profile-desc=RAVU r3 YUV (OpenGL gather)
glsl-shaders-clr
glsl-shader="~~/shaders/Nnedi3-RAVU/OpenGL/ravu-r3-yuv.hook"

```

---

## Configurazione Linux

Crea o modifica:

```bash
nano ~/.config/mpv/mpv.conf
```

Incolla:

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
# PROFILI SHADER (base)
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
```

---

## Profili disponibili

| Profilo | Sistema | Uso |
|---|---:|---|
| `amd-radeon` | Windows | AMD con D3D11VA |
| `nvidia` | Windows | NVIDIA con D3D11VA |
| `nvidia-nvdec` | Windows | NVIDIA con NVDEC, opzionale |
| `intel` | Windows | Intel iGPU / Arc |
| `linux-amd` | Linux | AMD con VAAPI |
| `linux-intel` | Linux | Intel con VAAPI |
| `linux-nvidia` | Linux | NVIDIA con decoding automatico sicuro |
| `linux-nvidia-nvdec` | Linux | NVIDIA con NVDEC |
| `linux-safe` | Linux | niente decoding hardware forzato |
| `cas` | Tutti | sharpening leggero, sicuro su AMD/NVIDIA/Intel |
| `cas-scaled` | Tutti | scaling/sharpening equilibrato, preset quotidiano |
| `fsr` | Tutti | upscaling più marcato, utile su sorgenti sotto-risolute |
| `nis` | NVIDIA | NVIDIA Image Scaling |
| `nvsharpen` | NVIDIA | NVIDIA sharpening senza scaling |
| `deband-heavy` | Tutti | banding evidente |

---

## Esempi di utilizzo

### Windows AMD

```powershell
mpv --profile=amd-radeon,cas-scaled "D:\Video\film.mkv"
```

```powershell
mpv --profile=amd-radeon,fsr "D:\Video\film.mkv"
```

### Windows NVIDIA

```powershell
mpv --profile=nvidia,cas-scaled "D:\Video\film.mkv"
```

```powershell
mpv --profile=nvidia,nis "D:\Video\film.mkv"
```

```powershell
mpv --profile=nvidia,nvsharpen "D:\Video\film.mkv"
```

### Linux AMD

```bash
mpv --profile=linux-amd,cas-scaled ~/Video/film.mkv
```

```bash
mpv --profile=linux-amd,fsr ~/Video/film.mkv
```

### Linux NVIDIA

```bash
mpv --profile=linux-nvidia,cas-scaled ~/Video/film.mkv
```

```bash
mpv --profile=linux-nvidia-nvdec,nis ~/Video/film.mkv
```

### Linux Intel

```bash
mpv --profile=linux-intel,cas-scaled ~/Video/film.mkv
```

```bash
mpv --profile=linux-intel,fsr ~/Video/film.mkv
```

### Video con banding evidente

```bash
mpv --profile=cas-scaled,deband-heavy ~/Video/film.mkv
```

Su Windows:

```powershell
mpv --profile=cas-scaled,deband-heavy "D:\Video\film.mkv"
```

---

## input.conf opzionale

### Windows

```powershell
notepad "$env:APPDATA\mpv\input.conf"
```

### Linux

```bash
nano ~/.config/mpv/input.conf
```

Contenuto consigliato:

```conf
# Toggle shader rapidi
CTRL+1 change-list glsl-shaders set "~~/shaders/CAS.glsl"
CTRL+2 change-list glsl-shaders set "~~/shaders/CAS-scaled.glsl"
CTRL+3 change-list glsl-shaders set "~~/shaders/FSR.glsl"
CTRL+4 change-list glsl-shaders set "~~/shaders/NVScaler.glsl"
CTRL+0 change-list glsl-shaders clr ""

# Screenshot
s screenshot
S screenshot video

# Stats MPV
TAB script-binding stats/display-stats-toggle
i script-binding stats/display-stats-toggle

# Volume
UP add volume 5
DOWN add volume -5

# Audio delay
CTRL+LEFT add audio-delay -0.050
CTRL+RIGHT add audio-delay 0.050

# Subtitle delay
ALT+LEFT add sub-delay -0.050
ALT+RIGHT add sub-delay 0.050
```

Nota: per verificare shader, renderer e decoding, apri le statistiche durante la riproduzione con `i` o `TAB`, a seconda della build e della configurazione.

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

Windows:

```powershell
dir "$env:APPDATA\mpv\shaders"
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

Perché nascondere le estensioni dei file è apparentemente considerato design, non sabotaggio.

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

Fedora/Nobara:

```bash
sudo dnf install libva-utils
```

Ubuntu/Debian:

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

Più shader non significa più qualità. Spesso significa soltanto prendere una sorgente già maltrattata e inciderne i difetti sulla pietra con zelo notarile.

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

Questa guida può essere usata, modificata e adattata liberamente. Gli shader citati appartengono ai rispettivi repository/autori.

