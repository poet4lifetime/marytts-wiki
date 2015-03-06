# A Possible Way of Pre-Processing .wav and .txt Files

This Wiki describes steps taken during the pre-processing phase of building a voice from a three-channel recording session:

* a channel containing the voice of the speaker,
* a channel containing prompt-separating beeps,
* a channel containing error markup done through a midi keyboard.

### 1. Resampling

Because the original sampling rate of the files from the recording sessions was 44.1 kHz, in order to avoid compatibility during later steps, the .wav files need to be resampled to 16 kHz. One way to do this is using the Sound eXchange (SoX) command line audio-conversion utility.

SoX with .mp3 support can be downloaded directly from the package manager with the command:
```sh
$ sudo apt-get install sox libsox-fmt-mp3
```

In order to do the conversion, just run the following line:
```sh
$ sox <input file> -b 16 <output file> rate 16000 dither -s
```
where ``<input file>`` refers to the original voice sound file, and ``<output file>`` refers to the desired output file to which the resampled sound is to be saved.

### 2. Manual markup
