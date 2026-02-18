---

# What are different formats of audio?

Media formats that handle sound generally fall into two categories: purely audio formats and video formats (which act as containers for both visual and audio data).

Here is a breakdown of the most common standard formats for both.

### **1. Audio Formats**

Audio files are typically categorized by whether they are compressed and how that compression happens.

**Lossy Formats (Smaller file size, some quality loss)**

These are best for streaming, portable devices, and general listening.

- **MP3 (MPEG-1 Audio Layer III):** The most universal audio format. It offers a good balance of sound quality and file size, though it is technically outdated compared to newer standards.
- **AAC (Advanced Audio Coding):** The standard for YouTube, Apple devices, and gaming consoles. It generally achieves better sound quality than MP3 at the same bit rate.
- **OGG (Ogg Vorbis):** An open-source, patent-free alternative to MP3/AAC, commonly used by Spotify and in gaming development.
- **WMA (Windows Media Audio):** Developed by Microsoft. It is less common now but still found in older Windows environments.

**Lossless Formats (Original quality, large file size)**

These preserve the exact audio data from the original source, making them ideal for archiving and professional editing.

- **WAV (Waveform Audio File Format):** The standard for uncompressed audio on Windows/PCs. It is widely used in professional recording and editing because it requires no processing to decode.
- **FLAC (Free Lossless Audio Codec):** Compresses audio to about 60% of its original size without losing a single bit of data. It is the gold standard for high-fidelity (Hi-Fi) listening.
- **ALAC (Apple Lossless Audio Codec):** Apple’s alternative to FLAC, compatible with iOS and macOS devices.

---

### **2. Video Formats (Containers)**

Video files are actually "containers" that hold a video stream, an audio stream, and metadata (like subtitles) in a single package.

- **MP4 (MPEG-4 Part 14):** The most compatible and popular format today. It works on virtually every player, phone, and browser. It usually uses H.264 or H.265 video compression and AAC audio.
- **MKV (Matroska Video):** An open-standard format famous for its ability to hold an unlimited number of video, audio, picture, or subtitle tracks in one file. It is preferred for high-quality movie rips but isn't supported by all default media players.
- **WebM:** Developed by Google specifically for the web. It is optimized for HTML5 video and offers high quality with small file sizes, used heavily by YouTube.
- **MOV (QuickTime Movie):** Developed by Apple. It is high quality and commonly used in video editing workflows, though file sizes can be large.
- **AVI (Audio Video Interleave):** An older format introduced by Microsoft. It is less efficient than MP4 or MKV but is still used for legacy compatibility.

### **3. A Note on Codecs vs. Containers**

It is helpful to distinguish between the **Format (Container)** and the **Codec**.

- **The Container** (e.g., `.mp4`, `.mkv`) is the box that holds the data.
- **The Codec** (e.g., H.264, AAC, MP3) is the software used to compress and decompress the actual media inside that box.

For example, an `.mp4` file (container) might contain video compressed with `H.264` (codec) and audio compressed with `AAC` (codec).

Would you like to know which format is best suited for a specific project, such as archiving or streaming?

<aside>
💡

We are going (initially) to focus only on most used audio and video format:

- Audio: (mp3, wav, aac).
- Video: (mp4, mkv).
</aside>

# What are differences between (mp3, acc, wav)?

- **The Problem:** If your input file is an MP3 or AAC (lossy or compressed), the compression algorithm has already thrown away "less audible" data to save space. It essentially "smears" the sound frequencies together.
- **The Consequence:** To a human ear, it sounds fine. To a separation algorithm (which looks for mathematical patterns), these compression artifacts look like noise or "ghost" data.
- **Converting back to WAV:** You are taking that small, like blurry JPEG and blowing it up to a huge poster size. **It is now a "large" file, but it is still blurry.**
    - Solution for this problem is to generate (predict or guess) the missing audio using a GAN like voice mixer after separating music from audio not before the operation.
- **Bandwidth Extensions (BWE):** are used to fix the missing audio
    - **Slow / Heavy (Avoid for real-time):** *AudioSR*, *NuWave*. (Great quality, but takes forever).
    - **Fast / Efficient (Good for Production):** *VoiceFixer* (uses a ResUNet), *HiFi-GAN* (very fast inference).
- **Treating Video Sounds:** (mp4, mkv) are containers that contains audio and video, we just extract the sounds from that containers and treat it normally.

# Is There Library or Any Tool Can handle Those Differences?

## **Librosa** (or sometimes `pydub`).

These libraries act as a "funnel." You throw **any** file at them (MP3, MKV, FLAC, OGG, MP4), and they return the exact same thing every time: **A simple array of numbers (Raw PCM),** it doesn’t affect the quality at all.

- Librosa secretly calls FFmpeg to decode the compressed data (MP3/AAC) into Raw PCM (WAV data) in your RAM.
    - FFmpeg is "Universal Wrapper" that the entire audio industry relies on to solve this problem.
- Automatically converts the audio from "Integer" (16-bit, values like 32,767) to "Floating Point" (32-bit, values between -1.0 and +1.0).

<aside>
💡

**Findings:**

- Pipeline is like following:
    
    Input (any type of audio file) → Universal Decoder (Librosa/FFmpeg) → Separating Algorithm (??) → Fixing Holes (BWE) → Cleaning Voice (??) → Output (Final data again).
    
</aside>
