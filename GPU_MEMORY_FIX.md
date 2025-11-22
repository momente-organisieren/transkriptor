# CUDA Out of Memory - Lösungen

## Problem
```
RuntimeError: CUDA failed with error out of memory
```

Dieser Fehler tritt auf, wenn:
- Das Whisper-Modell zu groß für Ihre GPU ist
- Die Audio-Datei zu lang ist
- Diarization + große Modelle gleichzeitig laufen

## ✅ Angewandte Lösung

**Modell von `large-v3` auf `medium` reduziert**

- `large-v3`: ~10 GB VRAM, beste Qualität
- `medium`: ~5 GB VRAM, **hohe Qualität** (empfohlen für die meisten Fälle)

Die Qualität von `medium` ist für deutsche Sprache bei klarer Audioqualität sehr gut und der Unterschied zu `large-v3` ist minimal.

## Weitere Optimierungen

### 1. Wie viel VRAM hat Ihre GPU?

Prüfen Sie mit:
```bash
nvidia-smi
```

Modell-Empfehlungen:
- **8 GB VRAM oder weniger**: `medium` (5 GB) oder `small` (2 GB)
- **12 GB VRAM**: `medium` oder `large-v3`
- **16+ GB VRAM**: `large-v3` problemlos

### 2. Audio-Datei kürzer machen

Bei sehr langen Dateien (>60 Minuten):
- Teilen Sie die Datei in kleinere Segmente (z.B. 30-Minuten-Chunks)
- Verwenden Sie Tools wie `ffmpeg`:
  ```bash
  ffmpeg -i input.mp3 -ss 00:00:00 -t 00:30:00 -c copy part1.mp3
  ffmpeg -i input.mp3 -ss 00:30:00 -t 00:30:00 -c copy part2.mp3
  ```

### 3. Modell wechseln

Bearbeiten Sie `.env`:

```env
# Für 8 GB VRAM GPUs
ASR_MODEL=medium

# Wenn immer noch Out of Memory
ASR_MODEL=small

# Wenn Sie 12+ GB VRAM haben
ASR_MODEL=large-v3
```

Nach Änderung:
```bash
docker compose down
docker compose up -d
```

### 4. GPU-Cache leeren

Falls das Modell "hängt":
```bash
# Container komplett herunterfahren
docker compose down

# Optional: Volumes löschen (Modelle werden neu heruntergeladen!)
# docker volume rm whisper_cache huggingface_cache

# Neu starten
docker compose up -d
```

### 5. Batch Size reduzieren (Fortgeschritten)

Das whisper-asr-webservice Image unterstützt `WHISPER_BATCH_SIZE`. In `docker-compose.yml` hinzufügen:

```yaml
environment:
  - WHISPER_BATCH_SIZE=8  # Standard: 16, niedriger = weniger VRAM
```

## Qualitätsvergleich

Für **deutsche Sprache mit guter Audioqualität**:

| Modell | VRAM | Qualität | WER* | Empfehlung |
|--------|------|----------|------|------------|
| tiny | ~1 GB | ⭐⭐ | ~12% | Nur für Tests |
| base | ~1 GB | ⭐⭐⭐ | ~8% | Schnelle Vorschau |
| small | ~2 GB | ⭐⭐⭐⭐ | ~5% | Gut für die meisten Fälle |
| **medium** | ~5 GB | ⭐⭐⭐⭐⭐ | ~3.5% | **Beste Balance** ✅ |
| large-v3 | ~10 GB | ⭐⭐⭐⭐⭐ | ~3% | Marginaler Unterschied zu medium |

*WER = Word Error Rate (niedriger = besser)

## Wichtig für Ihre 8-Sprecher-Diskussion

Mit dem `medium` Modell sollten Sie:
- ✅ Genug GPU-Speicher haben
- ✅ Sehr gute Transkriptions-Qualität bekommen
- ✅ Die Speaker Diarization voll nutzen können

Die Speaker-Erkennung hängt vom **pyannote-Modell** ab, nicht von der Whisper-Modellgröße. Das pyannote-Modell läuft separat und ist nicht betroffen.

## Testen Sie jetzt

1. **Container laufen**: `docker compose ps` sollte "healthy" zeigen
2. **Browser neu laden**: http://localhost:3000 (Strg+F5)
3. **Einstellungen**:
   - Min. Sprecher: 6
   - Max. Sprecher: 10
   - Sprache: Deutsch
4. **Audio hochladen** und prüfen

Das `medium` Modell sollte ausreichend sein und keine Out-of-Memory-Fehler mehr verursachen!
