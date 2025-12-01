# Timestamp Drift Fix

## Problem Description

When transcribing audio with unclear sections (noise, silence, poor quality), you may notice that timestamps start aligned at the beginning but progressively drift out of sync as the audio progresses.

### What Causes Timestamp Drift?

Whisper models can experience timestamp drift due to:

1. **Hallucinations during silence** - The model generates text even when there's no speech
2. **Unclear audio sections** - Poor audio quality causes the model to "guess" incorrectly
3. **Accumulated timing errors** - Small errors compound over longer audio files
4. **Padding and repetition** - The model may repeat previous text when uncertain

## Solutions

### 1. Enable Word-Level Timestamps (Recommended)

Word-level timestamps provide more precise alignment and reduce drift:

**In the UI:**
- ✅ Check "Wort-Zeitstempel aktivieren" before uploading

**Why it helps:** Word-level timestamps use more granular alignment algorithms that are less susceptible to drift.

### 2. WhisperX Default Configuration

Good news! WhisperX (which your application uses) already has better defaults than standard Whisper:

- `condition_on_previous_text=False` (default in WhisperX) - Prevents the model from repeating previous output when it encounters unclear audio
- VAD-based segmentation - Uses Voice Activity Detection to identify actual speech

### 3. Add API Parameters for Better Accuracy

You can pass additional parameters to the `/asr` endpoint to improve timestamp accuracy:

#### Available Parameters

| Parameter | Default | Recommended | Description |
|-----------|---------|-------------|-------------|
| `word_timestamps` | false | **true** | Enable word-level timestamps for better alignment |
| `condition_on_previous_text` | false (WhisperX) | **false** | Prevent hallucination by not conditioning on previous text |
| `suppress_blank` | true | **true** | Suppress blank outputs during silence |
| `vad_filter` | false | **true** (if supported) | Filter out non-speech segments using VAD |

### 4. Pre-process Your Audio

Before transcription, improve audio quality:

- **Normalize audio levels** - Use tools like ffmpeg to normalize volume
- **Remove background noise** - Use noise reduction filters
- **Convert to proper format** - WAV 16kHz mono is optimal

```bash
# Example: Normalize and clean audio with ffmpeg
ffmpeg -i input.mp3 -ar 16000 -ac 1 -af "highpass=f=200, lowpass=f=3000" output.wav
```

### 5. Use Smaller Segments

For very long audio files (>30 minutes), consider:
- Breaking the file into smaller chunks before transcription
- This prevents error accumulation over time

## Implementation for Your Application

### Quick Fix: Enable Word Timestamps

Simply check the "Wort-Zeitstempel aktivieren" checkbox in the UI before uploading. This is the easiest and most effective solution.

### Advanced: Additional Parameters

The WhisperX API supports additional parameters that can be added to your application. Here's what the API already receives:

**Current Parameters (from frontend/app.js:770-785):**
```javascript
params.set('output', 'json');
params.set('language', 'auto');  // or specific language
params.set('diarize', 'true');
params.set('min_speakers', '1');
params.set('max_speakers', '30');
params.set('word_timestamps', 'true');  // ✅ Enable this!
```

**Potential Additional Parameters:**
- `initial_prompt` - Provide context to guide the model
- `temperature` - Lower temperature (e.g., 0.0) for more consistent output
- `compression_ratio_threshold` - Detect hallucinations (default: 2.4)
- `logprob_threshold` - Skip low-confidence segments (default: -1.0)
- `no_speech_threshold` - Threshold for detecting silence (default: 0.6)

## Testing the Fix

After enabling word-level timestamps:

1. **Upload a test file** with known unclear sections
2. **Check alignment** at the beginning, middle, and end
3. **Compare with audio player** - Click segments to verify they play at the correct time
4. **Monitor drift** - If timestamps still drift, the audio quality may be too poor

## When Timestamps Still Drift

If you still experience drift after enabling word timestamps:

1. **Check audio quality** - Very poor audio may still cause drift
2. **Use a larger model** - `medium` or `large-v3` models are more accurate than `small`
3. **Manually adjust** - Use the audio player to identify correct timestamps and manually edit segments
4. **Consider re-recording** - If possible, improve the original recording quality

## References

- [WhisperX GitHub](https://github.com/m-bain/whisperX) - Official WhisperX repository
- [Whisper Hallucination Discussion](https://github.com/openai/whisper/discussions/679) - Community solutions
- [WhisperX Paper](https://arxiv.org/abs/2303.00747) - Technical details on alignment improvements

## Technical Details

### How WhisperX Improves Timestamps

WhisperX enhances standard Whisper with:

1. **Forced alignment** - Uses phoneme recognition to align words with audio
2. **VAD preprocessing** - Detects and removes silence before transcription
3. **Batched inference** - Processes speech segments efficiently
4. **Character-level timestamps** - Even more precise than word-level

### Why condition_on_previous_text=False Helps

Standard Whisper uses previous context to predict the next segment. This can cause:
- **Repetition loops** when audio is unclear
- **Timestamp creep** as the model fills gaps with repeated text
- **Accumulated drift** over long files

Setting this to `False` forces the model to process each segment independently, reducing hallucinations.

## Summary

**Best Practice for Timestamp Accuracy:**

1. ✅ **Enable "Wort-Zeitstempel aktivieren"** in the UI (most important!)
2. ✅ Use a good quality audio file (minimal noise, clear speech)
3. ✅ Use `medium` or `large-v3` model for better accuracy
4. ✅ WhisperX already has `condition_on_previous_text=False` by default
5. ✅ Use the audio player to verify alignment after transcription

**Expected Results:**
- Timestamps should remain aligned throughout the entire audio
- Clicking a segment should jump to the exact moment in the audio
- Exported SRT/VTT files should sync correctly with video players
