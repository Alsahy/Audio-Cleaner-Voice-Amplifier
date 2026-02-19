# Phase 1: Technical Exploration & Feasibility

**Objective:** To investigate the "black box" technical challenges identified in Phase 0 and determine the specific libraries, mathematical approaches, and metrics required to build the solution.

---

## 1. The "Unknowns" & Research Questions

### A. Universal Input & Tooling
**The Problem:** The tool must accept any file format that produces sound (MP3, WAV, AAC, MP4, etc.), but I don't know if these files store sound in fundamentally different ways.
* **Question 1:** Is treating an MP3 file different from treating a WAV, AAC, or MP4 file? Do they require different code to "open" and read the sound?
* **Question 2:** If they are different, is there a single tool or "wrapper" that handles all these differences automatically, allowing my code to treat every input as the same raw sound data?
* **Findings:**
    * **Direct Answer:** Yes, they store data differently (MP3/AAC are compressed and lossy; WAV is uncompressed and lossless; MP4/MKV are multi-track containers). However, the *core separation algorithm* will not require different code if we use a universal decoder upfront.
    * **The Wrapper:** `librosa` is a general Python library that handles all formats safely by relying on FFmpeg under the hood. It strips audio from video containers and converts everything into a standardized Float32 NumPy array.
    * **The Lossy Challenge:** Because MP3/AAC compression permanently deletes high-frequency data, the separation process can leave noticeable "holes" in the voice. This requires Bandwidth Extension (BWE) regeneration later in the pipeline.

### B. Separation Logic (The Core)
**The Problem:** We need to distinguish "Voice" from "Music," but I don't know how a computer perceives that difference.
* **Question 3:** What are the distinguishing characteristics that make a "voice" different from "music" or "noise" to a computer?
* **Question 4:** What are the standard approaches or logical methods used in software to separate these two distinct sounds?
* **Findings:**
    * **Direct Answer:** A computer distinguishes them by analyzing the structural patterns in the **frequency domain** (spectrograms). Human voices create distinct wavy horizontal harmonic lines, while percussion creates rigid vertical broadband lines.
    
    * **Approaches Evaluated:**
        * **DSP (Digital Signal Processing):** Mathematically subtracts the sides from the center (since vocals are usually panned dead-center in stereo). *Failure Cases:* Fails on Mono sounds, centered drums, or heavy vocal reverb where Left ≠ Right.
        * **NMF (Non-negative Matrix Factorization):** An unsupervised learning approach that assumes a song is made of repeating sound dictionaries. It attempts to factor the magnitude spectrogram matrix $V$ into two smaller matrices: $W$ (frequency patterns) and $H$ (temporal activations), such that $V \approx W \times H$. *Drawback:* Slow and struggles with complex, constantly changing vocal pitches.
        * **Traditional Bandpass Filtering:** Applying a 300Hz–3kHz filter fails to isolate vocals; it creates a muddy mixture of voice and instruments while causing a muffled "telephone effect."
        * **Deep Learning (Supervised Learning):** AI models learn the visual *texture* of sources on a spectrogram. 
            * *Primary Architectures:* CNNs/U-Nets (Frequency Domain), RNNs/LSTMs (Sequence Modeling), and Hybrid/Time-Domain models (State-of-the-Art).
            

### C. Quality Preservation
**The Problem:** I suspect that removing part of the sound might damage the part I want to keep.
* **Question 5:** Does the process of stripping away background noise inevitably damage or "cut" parts of the voice?
* **Question 6:** If damage occurs, what does it sound like, and are there known ways to minimize it?
* **Findings:**
    * **Direct Answer:** Yes, aggressive masking inevitably causes collateral damage to the vocal track.
    * **Types of Damage:**
        * *Phase Damage (Metallic Effect):* Discarding phase data causes misaligned waveforms during reconstruction.
        * *Frequency Overlap Deletion (Muffling):* Masking deletes vocal consonants that share exact frequencies with broadband instruments (like snares).
        * *Musical Noise (Chirping/Swirling):* Imperfect masks leave random isolated data points, sounding like watery, synthetic chirps.
        * *Bleed/Leakage:* Background instruments slip into the vocal track.
    * **Mitigation Strategies:**
        * **Ensembling:** Averaging outputs from multiple distinct architectures (e.g., combining a Time-Domain model with a Frequency-Domain U-Net) to smooth discrepancies.
        * **Cascaded Restoration:** Passing the stem through specialized blocks: Speech Denoising (scrubs bleed), Bandwidth Extension (hallucinates missing high-harmonics up to 24kHz), and Neural Vocoding (fixes severe phase damage).

### D. The "Cocktail Party" Problem (Voice vs. Voice)
**The Problem:** Sometimes the "noise" isn't music, but other people talking.
* **Question 7:** If we can successfully clean a human voice from instruments, does that same approach work for separating a specific human voice from *other* background voices?
* **Findings:**
    * **Direct Answer:** No. Separating voice from guitar relies on vastly different harmonic textures. Separating voice from voice is exponentially harder because the structures are nearly identical.
    * **Required Approaches:**
        * **Speaker Embeddings (Voice Biometrics):** The AI generates a mathematical "voiceprint" from a brief sample of the target speaker and masks out any speech that doesn't match that specific vocal tract (e.g., SepFormer, Conv-TasNet).
        * **Spatial Filtering (Beamforming):** Uses micro-delays between multi-microphone recordings to mathematically mute voices coming from the sides, preserving only the center voice.

### E. Amplification
**The Problem:** I need to make the voice louder, but I don't know how to do it without making it sound bad or boosting the noise I just tried to remove.
* **Question 8:** How do we programmatically amplify a sound file?
* **Question 9:** How do we ensure that increasing the volume doesn't cause distortion or make any remaining background noise overwhelmingly loud?
* **Findings:**
    * **Direct Answer:** Amplification is done via Normalization and Make-up Gain, but it mathematically increases background hiss exactly as much as the target voice.
    * **Safe Amplification Methods:** * *Peak Normalization:* Scales audio so the loudest sample hits a predefined maximum (e.g., -1.0 dB).
        * *LUFS Normalization:* Adjusts volume based on human-perceived loudness targets (e.g., -14 LUFS).
        * *Dynamic Range Compression:* Actively reduces loud volume spikes so the overall track can be safely amplified without digital clipping.
    * **Noise Mitigation (Strict Order of Operations):** The audio must be managed in a highly specific sequence to prevent amplifying the noise floor: Separation $\rightarrow$ Denoise $\rightarrow$ Compression $\rightarrow$ Normalization.

### F. Defining "Clean" (Evaluation Metrics)
**The Problem:** "Clean" is a highly subjective term. We need mathematical standards to validate the engine's output rather than relying solely on human hearing.
* **Question 10:** What standard mathematical metrics exist to objectively measure audio separation quality and naturalness?
* **Findings:**
    * **SI-SDR (Scale-Invariant Signal-to-Distortion Ratio):** The foundational metric for source separation. Measures the power ratio between the intended source and introduced distortion (Scores > 15 dB indicate excellent isolation).
    * **PESQ (Perceptual Evaluation of Speech Quality):** Simulates human ear perception to catch robotic artifacts (Scale -0.5 to 4.5; scores > 3.5 are highly natural).
    * **STOI (Short-Time Objective Intelligibility):** Measures how understandable the spoken words are (Scores > 0.85 indicate high intelligibility).
    * **FAD (Fréchet Audio Distance):** Compares the statistical distribution of the output against pristine studio audio. Essential for evaluating the Bandwidth Extension phase.
    * **MOS (Mean Opinion Score):** Subjective human rating of clarity (1 to 5). AI models like DNSMOS can estimate this programmatically.

---

### Master System Pipeline
Based on the findings across all sections, the synthesized architecture for the engine is:
> `Input (Any Format) → FFmpeg/Librosa Decoder → Primary AI Separation Model → Speech Denoiser (Artifact Removal) → Bandwidth Extension (BWE/Hole Fixing) → Dynamic Range Compressor → LUFS Normalization → Output`

---

## 2. Feasibility Decision (Initial Selection)

*(Decisions will be finalized after the Proof of Concept Phase).*
