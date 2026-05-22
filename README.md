# MPV per Windows e Linux +Shaders (Vulkan, CUDA, DX3D11)

Guida pratica per installare e configurare **MPV** con profili modulari per Windows e Linux, GPU AMD/NVIDIA/Intel, shader CAS/FSR/NIS, debanding e troubleshooting.

L'obiettivo è avere una configurazione semplice da mantenere, abbastanza leggera da non trasformare ogni film in un benchmark termico, ma abbastanza curata da migliorare nitidezza, scaling e fluidità senza costringerti a contemplare pixel sfocati come un monaco medievale del bitrate.

---

## Indice

- [Obiettivo](#obiettivo)
- [Prerequisiti](#prerequisiti)
- [Installazione su Windows](#installazione-su-windows)
- [Installazione su Linux](#installazione-su-linux)
- [Struttura della configurazione](#struttura-della-configurazione)
- [Shader consigliati](#shader-consigliati)
- [Configurazione Windows](#configurazione-windows)
- [Configurazione Linux](#configurazione-linux)
- [Profili disponibili](#profili-disponibili)
- [Esempi di utilizzo](#esempi-di-utilizzo)
- [input.conf opzionale](#inputconf-opzionale)
- [Verifica shader in esecuzione](#verifica-shader-in-esecuzione)
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
- Git, utile per scaricare e aggiornare questo progetto
- una GPU con driver funzionanti
- un terminale: PowerShell su Windows, shell Bash/Zsh/Fish su Linux
- `unzip` su Linux, se vuoi estrarre gli shader pack `.zip` da terminale

---

## Installazione su Windows

Questa sezione installa MPV, crea le cartelle corrette e copia configurazione + shader dal progetto GitHub.

### 1. Installare Git e MPV

Apri **PowerShell**.

```powershell
winget --version
```

Se `winget` risponde, installa Git e MPV:

```powershell
winget install Git.Git
winget install --id shinchiro.mpv
```

Verifica:

```powershell
git --version
mpv --version
```

Se `mpv` non viene trovato nel `PATH`, chiudi e riapri PowerShell. Sì, Windows ogni tanto vuole essere riavviato anche solo per ricordarsi dove ha messo le chiavi.

### 2. Creare la cartella configurazione

```powershell
mkdir "$env:APPDATA/mpv" -Force
mkdir "$env:APPDATA/mpv/shaders" -Force
```

Percorso finale:

```text
%APPDATA%/mpv
```

Di solito corrisponde a:

```text
C:/Users/<utente>/AppData/Roaming/mpv
```

### 3. Clonare questo progetto

Repository ufficiale di questa guida:

```text
https://github.com/Damocle77/MPV-Player.git
```

Clona il repository:

```powershell
cd "$env:USERPROFILE/Downloads"
git clone https://github.com/Damocle77/MPV-Player.git
cd ./MPV-Player
```

### 4. Copiare configurazione e shader

```powershell
Copy-Item ./mpv.conf "$env:APPDATA/mpv/mpv.conf" -Force

if (Test-Path ./input.conf) {
    Copy-Item ./input.conf "$env:APPDATA/mpv/input.conf" -Force
}

Copy-Item ./shaders/* "$env:APPDATA/mpv/shaders" -Recurse -Force
```

### 5. Estrarre gli shader pack `.zip`

Nel progetto gli shader pack pesanti possono essere tenuti come archivi `.zip`, così GitHub non mostra mille file come se stessi pubblicando l'elenco telefonico della GPU.

```powershell
$shaderDir = "$env:APPDATA/mpv/shaders"

"FSRCNNX", "GLSL_High-end", "GLS_Low-end", "Nnedi3-RAVU" | ForEach-Object {
    $zip = Join-Path $shaderDir "$_.zip"
    $dst = Join-Path $shaderDir $_

    if (Test-Path $zip) {
        Expand-Archive $zip $dst -Force
    }
}
```

Verifica:

```powershell
dir "$env:APPDATA/mpv"
dir "$env:APPDATA/mpv/shaders"
```

Occhio solo a non ottenere cartelle doppie tipo:

```text
FSRCNNX/FSRCNNX/FSRCNNX_x2_16-0-4-1.glsl
```

Se succede, sposta i file al livello corretto. Non è colpa tua: gli ZIP sono piccoli contenitori di caos con compressione opzionale.

### 6. Test rapido

```powershell
mpv --profile=cas-scaled "D:/Video/film.mkv"
```

Test shader extra:

```powershell
mpv --profile=fsrcnnx-x2 "D:/Video/film.mkv"
mpv --profile=ravu-r3 "D:/Video/film.mkv"
```

Log verboso, quando MPV fa il misterioso:

```powershell
mpv --profile=cas-scaled --msg-level=all=v "D:/Video/film.mkv"
```

---

## Installazione su Linux

Questa sezione installa MPV, crea la configurazione in `~/.config/mpv` e copia configurazione + shader dal progetto GitHub.

### 1. Installare MPV, Git e unzip

Fedora / Nobara:

```bash
sudo dnf install mpv git unzip
```

Ubuntu / Debian:

```bash
sudo apt update
sudo apt install mpv git unzip
```

Arch / EndeavourOS / Manjaro:

```bash
sudo pacman -S mpv git unzip
```

openSUSE:

```bash
sudo zypper install mpv git unzip
```

Verifica:

```bash
git --version
mpv --version
```

> Nota Flatpak: se installi MPV da Flathub, la configurazione non sta in `~/.config/mpv`, ma in `~/.var/app/io.mpv.Mpv/config/mpv`. Comodo come un telecomando con tre sportelli batteria, ma almeno funziona.

### 2. Creare la cartella configurazione

```bash
mkdir -p ~/.config/mpv/shaders
```

### 3. Clonare questo progetto

Repository ufficiale di questa guida:

```text
https://github.com/Damocle77/MPV-Player.git
```

Clona il repository:

```bash
cd ~/Downloads
git clone https://github.com/Damocle77/MPV-Player.git
cd MPV-Player
```

### 4. Copiare configurazione e shader

```bash
cp mpv.conf ~/.config/mpv/mpv.conf

if [ -f input.conf ]; then
    cp input.conf ~/.config/mpv/input.conf
fi

cp -r shaders/* ~/.config/mpv/shaders/
```

### 5. Estrarre gli shader pack `.zip`

```bash
cd ~/.config/mpv/shaders

for pack in FSRCNNX GLSL_High-end GLS_Low-end Nnedi3-RAVU; do
    if [ -f "$pack.zip" ]; then
        mkdir -p "$pack"
        unzip -o "$pack.zip" -d "$pack"
    fi
done
```

Verifica:

```bash
find ~/.config/mpv -maxdepth 3 -type f | sort | head -80
```

Se vedi cartelle doppie tipo:

```text
~/.config/mpv/shaders/Nnedi3-RAVU/Nnedi3-RAVU/Vulkan/...
```

sposta il contenuto al livello giusto. Gli archivi ZIP, quando possono complicare una cosa semplice, lo fanno con dedizione quasi artistica.

### 6. Test rapido

```bash
mpv --profile=linux-amd,cas-scaled ~/Video/film.mkv
```

Per Intel:

```bash
mpv --profile=linux-intel,cas-scaled ~/Video/film.mkv
```

Per NVIDIA:

```bash
mpv --profile=linux-nvidia,cas-scaled ~/Video/film.mkv
```

Test shader extra:

```bash
mpv --profile=fsrcnnx-x2 ~/Video/film.mkv
mpv --profile=ravu-r3 ~/Video/film.mkv
```

Log verboso:

```bash
mpv --profile=cas-scaled --msg-level=all=v ~/Video/film.mkv
```

---

## Struttura della configurazione

### Windows

<sub>

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
    ├── FSRCNNX.zip          # archivio; estrarre in FSRCNNX/ per usare i preset
    ├── GLSL_High-end.zip    # Anime4K HQ preset, estrarre in GLSL_High-end/
    ├── GLS_Low-end.zip      # Anime4K light preset, estrarre in GLS_Low-end/
    ├── Nnedi3-RAVU.zip      # contiene cartelle OpenGL/ Vulkan e .hook
    └── (una volta estratti, le relative cartelle FSRCNNX/, GLSL_High-end/, GLSL_Low-end/, Nnedi3-RAVU/ appaiono qui)

        ├── OpenGL\       # shader gather / win_bgfx
        ├── Vulkan\       # shader compute / gpu-next
        ├── nnedi3-*.hook   # fallback universali
        └── ravu-*.hook
```

</sub>

### Linux

<sub>

```text
~/.config/mpv/
├── mpv.conf
├── input.conf
└── shaders/    # stessa struttura di Windows
```

</sub>

> **Suggerimento**: mantieni le cartelle `Nnedi3-RAVU/OpenGL` e `Nnedi3-RAVU/Vulkan` separate. MPV carica il file che indichi, non cerca da solo "la versione giusta". In `vo=gpu-next + gpu-api=vulkan` usa la variant *Vulkan*. In `vo=gpu` (OpenGL) usa *OpenGL*.

---

### Contenuto della cartella `shaders`

<sub>

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

</sub>

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
| RAVU | `https://github.com/bjin/mpv-prescalers` | upscaling avanzato | usare `Vulkan/` con `gpu-next + vulkan`, oppure `OpenGL/` con backend OpenGL |
| Adaptive Sharpen | `https://gist.github.com/igv/8a77e4eb8276753b54bb94c1c50c317e` | sharpening generale | con moderazione |

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

Più alto non significa automaticamente meglio: significa anche più carico sulla GPU e maggior probabilità di trasformare la ventola in un personaggio secondario del film.

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

Dentro il repository le varianti principali sono state rinominate nella guida in modo più leggibile:

```text
temp-mpv-prescalers/Vulkan
temp-mpv-prescalers/OpenGL
```

- `Vulkan/` = shader compute moderni, consigliati con:

```ini
vo=gpu-next
gpu-api=vulkan
```

- `OpenGL/` = shader gather/fallback, consigliati con:

```ini
vo=gpu
```

oppure backend OpenGL legacy.

La distinzione serve solo a evitare il classico dubbio esistenziale:

> "Vulkan o CUDA?"

che per un nuovo utente suona più come un boss di Elden Ring che come una scelta di backend video.

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

Nota pratica: alcuni `.hook` sono pensati per backend o modalità specifiche. Se uno non parte, prova prima una variante meno specifica o cambia backend grafico. Naturalmente non potevano chiamarli tutti `.glsl`, perché la serenità mentale dell'utente evidentemente non rientrava nei requisiti di progetto.

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

Regola: la guida base resta su `CAS`, `CAS-scaled`, `FSR`. Gli extra vanno documentati, non buttati tutti nel preset principale come coriandoli su un server in produzione.

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
notepad "$env:APPDATA/mpv/input.conf"
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

# ------------------------------------------------------------
# Shader rapidi base
# ------------------------------------------------------------
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

# ------------------------------------------------------------
# Shader extra, abilitali se hai estratto gli ZIP
# ------------------------------------------------------------
# Esempi utili. Tienili commentati finché non ti servono.
# Non tutti gli shader sono felici su ogni backend. Che sorpresa, il mondo è imperfetto.

#CTRL+6 no-osd change-list glsl-shaders set "~~/shaders/FSRCNNX/FSRCNNX_x2_16-0-4-1.glsl" ; show-text "Shader: FSRCNNX x2" 2000
#CTRL+7 no-osd change-list glsl-shaders set "~~/shaders/Nnedi3-RAVU/Vulkan/ravu-r3-yuv.hook" ; show-text "Shader: RAVU r3 Vulkan/gpu-next" 2000
#CTRL+8 no-osd change-list glsl-shaders set "~~/shaders/Nnedi3-RAVU/OpenGL/ravu-r3-yuv.hook" ; show-text "Shader: RAVU r3 OpenGL legacy" 2000
#CTRL+9 no-osd change-list glsl-shaders set "~~/shaders/adaptive-sharpen.glsl" ; show-text "Shader: Adaptive Sharpen" 2000

# ------------------------------------------------------------
# Stats / overlay diagnostici
# ------------------------------------------------------------
# i o TAB mostrano l'overlay statistiche.
# CTRL+s mostra direttamente la property glsl-shaders.

i script-binding stats/display-stats-toggle
TAB script-binding stats/display-stats-toggle
? script-binding stats/display-page-5

# ------------------------------------------------------------
# Screenshot
# ------------------------------------------------------------
# Utile per confronto A/B: stesso frame, shader diversi.

s screenshot
S screenshot video

# ------------------------------------------------------------
# Volume
# ------------------------------------------------------------

UP add volume 5
DOWN add volume -5

# ------------------------------------------------------------
# Audio/sub delay
# ------------------------------------------------------------

CTRL+LEFT add audio-delay -0.050
CTRL+RIGHT add audio-delay 0.050
ALT+LEFT add sub-delay -0.050
ALT+RIGHT add sub-delay 0.050
```

Nota pratica: `change-list glsl-shaders set` sostituisce lo shader attivo. È più sicuro di `append` o `toggle`, perché evita di impilare tre upscaler uno sopra l'altro come se stessi costruendo una lasagna di artefatti.

---

## Verifica shader in esecuzione

Questa sezione serve a capire se MPV sta davvero caricando gli shader oppure se stai solo guardando lo stesso video con più speranza, attività nobile ma poco misurabile.

### Metodo 1: OSD durante la riproduzione

Con l'`input.conf` consigliato:

```text
CTRL+s
```

mostra a schermo la property:

```text
glsl-shaders
```

Se tutto funziona, vedrai qualcosa tipo:

```text
Shader attivi: ~~ / shaders / CAS-scaled.glsl
```

oppure:

```text
Shader attivi: ~~ / shaders / FSRCNNX / FSRCNNX_x2_16-0-4-1.glsl
```

Gli spazi qui sopra sono solo per leggibilità: nel player comparirà il path reale. Se la riga è vuota, MPV non ha shader GLSL attivi.

### Metodo 2: overlay statistiche

Durante il video premi:

```text
i
```

oppure:

```text
TAB
```

L'overlay statistiche mostra renderer, decoder, frame drop, timing e informazioni video. Non sempre mostra la lista shader in modo esplicito su tutte le build, quindi usa `CTRL+s` per la verifica diretta degli shader.

### Metodo 3: log da terminale

Windows PowerShell:

```powershell
mpv --profile=cas-scaled --msg-level=all=v "D:/Video/film.mkv" 2>&1 | Tee-Object "$env:TEMP/mpv-shader.log"
Select-String -Path "$env:TEMP/mpv-shader.log" -Pattern "glsl|shader|CAS|FSR|RAVU|FSRCNNX|NIS"
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

Poi premi `s` nello stesso punto del video. Se stai confrontando frame diversi, stai facendo benchmarking creativo, non analisi.

### Metodo 5: verifica path shader

Windows:

```powershell
dir "$env:APPDATA/mpv/shaders"
dir "$env:APPDATA/mpv/shaders/FSRCNNX"
dir "$env:APPDATA/mpv/shaders/Nnedi3-RAVU"
```

Linux:

```bash
ls -lh ~/.config/mpv/shaders
ls -lh ~/.config/mpv/shaders/FSRCNNX
ls -lh ~/.config/mpv/shaders/Nnedi3-RAVU
```

---

## Cosa fanno le opzioni principali

| Opzione | Dove | Significato semplice | Quando cambiarla |
|---|---|---|---|
| `vo=gpu-next` | Windows/Linux | renderer moderno di MPV/libplacebo | lascialo così, salvo bug specifici |
| `gpu-api=d3d11` | Windows | backend grafico stabile su Windows | se usi Vulkan su Windows e sai perché, allora sai anche dove mettere le mani |
| `gpu-api=vulkan` | Linux | backend consigliato per `gpu-next` e shader compute | se dà problemi, prova OpenGL |
| `hwdec=d3d11va` | Windows | decoding hardware via D3D11 | default sano per AMD/NVIDIA/Intel |
| `hwdec=vaapi` | Linux AMD/Intel | decoding hardware Linux | se VAAPI rompe, usa `linux-safe` |
| `hwdec=nvdec` | NVIDIA | decoding hardware NVIDIA | opzionale; se instabile torna ad `auto-safe` |
| `scale=ewa_lanczossharp` | tutti | scaler nitido ma non folle | se troppo tagliente, prova scaler più morbidi |
| `dscale=mitchell` | tutti | downscale controllato | ok per uso generale |
| `deband=yes` | tutti | riduce banding leggero | se impasta, abbassa o disattiva |
| `video-sync=display-resample` | tutti | sincronizza video al refresh monitor | se noti scatti strani, prova `video-sync=audio` |
| `interpolation=yes` | tutti | aiuta fluidità su mismatch fps/refresh | se crea effetto strano, metti `no` |
| `glsl-shaders-clr` | profili | pulisce shader già caricati | fondamentale quando cambi shader, evita il circo multi-upscaler |

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

Perché nascondere le estensioni dei file continua a essere considerata una brillante idea UX da qualche parte nel multiverso.

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

Più shader non significa più qualità. Spesso significa soltanto prendere una sorgente già maltrattata e incidere i suoi difetti nel marmo con zelo quasi religioso.

---

> *Nota pratica:* se inizi a concatenare cinque shader neurali, due sharpen e un downscaler “per vedere cosa succede”, quello che succede è che MPV diventa un benchmark ambulante e la GPU inizia a rivalutare le sue scelte di vita.

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

