# Custom Whisper ASR backend with WhisperX and pyannote support
FROM onerahmet/openai-whisper-asr-webservice:v1.5.0-gpu

# Install whisperx and pyannote for speaker diarization
RUN cd /app && \
    .venv/bin/pip install --no-cache-dir whisperx pyannote.audio && \
    .venv/bin/pip install --no-cache-dir torch torchaudio --index-url https://download.pytorch.org/whl/cu118

# Verify installations
RUN cd /app && \
    .venv/bin/python -c "import whisperx; print('✓ whisperx installed:', whisperx.__version__)" && \
    .venv/bin/python -c "import pyannote.audio; print('✓ pyannote.audio installed:', pyannote.audio.__version__)"

# Keep the original entrypoint and command from base image
