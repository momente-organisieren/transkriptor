# 🎙️ Transkriptor – Lokale Spracherkennung mit Sprecher-Diarisierung

Eine vollständig lokal gehostete Web-Anwendung zur automatischen Transkription von Audio- und Videodateien mit Sprecherzuweisung.

## ✨ Features

- **🎯 Lokale Verarbeitung** – Deine Daten verlassen nie deinen Server
- **👥 Sprecher-Diarisierung** – Automatische Erkennung verschiedener Sprecher
- **🌍 Multi-Sprache** – Unterstützt 20+ Sprachen (Deutsch, Englisch, etc.)
- **⏱️ Zeitstempel** – Wortgenaue oder Segment-Zeitstempel
- **✏️ Editor** – Transkripte direkt im Browser bearbeiten
- **📁 Export** – TXT, SRT, VTT, JSON, Word-Format

## 🖥️ Systemanforderungen

- **Docker** mit Docker Compose
- **NVIDIA GPU** mit mindestens 8 GB VRAM (für große Modelle)
- **NVIDIA Container Toolkit** installiert
- **Hugging Face Account** (kostenlos)

### GPU Setup (falls noch nicht vorhanden)

```bash
# NVIDIA Container Toolkit installieren (Ubuntu/Debian)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update && sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## 🚀 Installation

### 1. Repository klonen/Dateien kopieren

```bash
mkdir whisper-transcriber && cd whisper-transcriber
# Alle Dateien in dieses Verzeichnis kopieren
```

### 2. Hugging Face Token einrichten

1. Erstelle einen kostenlosen Account auf [huggingface.co](https://huggingface.co)
2. Generiere einen **READ** Token unter: https://huggingface.co/settings/tokens
3. Akzeptiere die Nutzungsbedingungen für diese Modelle:
   - https://huggingface.co/pyannote/segmentation-3.0
   - https://huggingface.co/pyannote/speaker-diarization-3.1

### 3. Konfiguration erstellen

```bash
# .env Datei aus Vorlage erstellen
cp .env.example .env

# Token eintragen
nano .env
# Ersetze "hf_DEIN_TOKEN_HIER" mit deinem echten Token
```

### 4. Starten

```bash
docker compose up -d
```

⏳ **Erster Start dauert länger** – Die Modelle werden heruntergeladen (~10 GB).

### 5. Öffnen

Öffne im Browser: **http://localhost:3000**

## 📋 Nutzung

1. **Datei hochladen** – Audio (MP3, WAV, M4A...) oder Video (MP4, MKV...)
2. **Optionen wählen**:
   - Sprache (automatisch oder manuell)
   - Max. Anzahl Sprecher
   - Sprecher-Erkennung ein/aus
   - Wort-Zeitstempel ein/aus
3. **Warten** – Je nach Dateigröße einige Sekunden bis Minuten
4. **Bearbeiten** – Sprecher umbenennen, Text korrigieren
5. **Exportieren** – Format wählen und herunterladen

## 🎛️ Modelle & Performance

| Modell | VRAM | Qualität | Geschwindigkeit |
|--------|------|----------|-----------------|
| `tiny` | ~1 GB | ⭐⭐ | ⚡⚡⚡⚡⚡ |
| `base` | ~1 GB | ⭐⭐⭐ | ⚡⚡⚡⚡ |
| `small` | ~2 GB | ⭐⭐⭐⭐ | ⚡⚡⚡ |
| `medium` | ~5 GB | ⭐⭐⭐⭐ | ⚡⚡ |
| `large-v3` | ~10 GB | ⭐⭐⭐⭐⭐ | ⚡ |

Modell ändern in `docker-compose.yml`:
```yaml
environment:
  - ASR_MODEL=medium  # oder: tiny, base, small, large-v3
```

## 🔧 Konfiguration

### docker-compose.yml Optionen

```yaml
environment:
  # ASR Engine (whisperx für Diarization)
  - ASR_ENGINE=whisperx
  
  # Modellgröße
  - ASR_MODEL=large-v3
  
  # Hugging Face Token (für Sprecher-Erkennung)
  - HF_TOKEN=${HF_TOKEN}
```

### Ports ändern

Frontend: Port `3000` → z.B. `8080`:
```yaml
frontend:
  ports:
    - "8080:80"
```

API direkt: Port `9000`

## 🐛 Troubleshooting

### "API offline"

```bash
# Logs prüfen
docker compose logs whisper-api

# Container neustarten
docker compose restart whisper-api
```

### GPU nicht erkannt

```bash
# NVIDIA Docker testen
docker run --rm --gpus all nvidia/cuda:12.0-base nvidia-smi

# Falls Fehler: nvidia-container-toolkit neu konfigurieren
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### Diarization funktioniert nicht

1. HF_TOKEN in `.env` korrekt eingetragen?
2. Nutzungsbedingungen der pyannote-Modelle akzeptiert?
3. Container neu starten nach Token-Änderung

### Out of Memory (GPU)

Kleineres Modell verwenden:
```yaml
- ASR_MODEL=small  # statt large-v3
```

## 📁 Verzeichnisstruktur

```
whisper-transcriber/
├── docker-compose.yml    # Container-Konfiguration
├── .env                  # Secrets (HF_TOKEN)
├── .env.example          # Vorlage
└── frontend/
    ├── Dockerfile
    ├── nginx.conf
    ├── index.html
    ├── styles.css
    └── app.js
```

## 🔒 Sicherheit

- Alle Daten werden **lokal verarbeitet**
- Keine Daten werden an externe Server gesendet
- Hugging Face Token wird nur für Model-Downloads verwendet

## 📄 Lizenz

MIT License – Frei verwendbar für private und kommerzielle Zwecke.

## 🙏 Credits

- [WhisperX](https://github.com/m-bain/whisperX) – ASR mit Alignment und Diarization
- [whisper-asr-webservice](https://github.com/ahmetoner/whisper-asr-webservice) – API Backend
- [pyannote-audio](https://github.com/pyannote/pyannote-audio) – Speaker Diarization
