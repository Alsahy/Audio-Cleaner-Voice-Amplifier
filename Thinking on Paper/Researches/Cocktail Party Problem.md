### The Difference in Acoustic Modeling

Separating a voice from a guitar is relatively straightforward for AI because the two sources have vastly different harmonic structures and textures. Separating a voice from another voice is exponentially harder because the frequency ranges, harmonic structures, and textures are nearly identical.

To achieve multi-speaker separation, the engine must utilize specialized models (such as SepFormer, Conv-TasNet, or DPRNN) that employ different strategies:

### 1. Speaker Embeddings (Voice Biometrics)

Instead of just looking for "human frequencies," these models require a brief sample of the target speaker's voice. The AI generates a mathematical "voiceprint" (an embedding) and actively scans the overlapping audio, masking out any speech that does not match the target speaker's specific vocal tract characteristics.

### 2. Spatial Filtering (Beamforming)

If the audio was recorded with multiple microphones (stereo or spatial arrays), algorithms can use the microscopic time delays between the left and right channels to calculate where a sound is coming from. The system can mathematically "mute" voices coming from the sides and only preserve the voice coming from the direct center.
