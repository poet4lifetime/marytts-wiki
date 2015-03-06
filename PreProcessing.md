# A Possible Way of Pre-Processing .wav and .txt Files

This Wiki describes steps taken during the pre-processing phase of building a voice from a three-channel recording session:

* a channel containing the voice of the speaker,
* a channel containing prompt-separating beeps,
* a channel containing error markup done through a midi keyboard.

However, most the steps described below are also applicable, or offer an alternative solution for working with a single-channel recording.

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

An easy way for extracting sound files containing individual prompts from a single sound file is automatizing the process by using a script written in e.g. Praat. In order to be able to do this, it is necessary to have a set of boundaries determining the beginning and endpoint of the interval containing each individual prompt.

In case there is a channel containing the recording of prompt-separating beeps, this could be converted to a .TextGrid file with
//Find a way to highlight this command inline// Praat's Annotate > To TextGrid(silences)... option that can be accessed from the object window.
// Add explanation of how to label silences and how to label rest (i do not remember options by hand).

// Picture of accessing this option in Praat?

// Describe how to open TextGrid and Sound files together in Praat (suggest opening the sound file as long sound).

In case the recording was a single-channel one, the boundaries could be set by hand. 
// Explain how to set boundaries in Praat.

// Explain how error markup with midi keyboard helps here.

### 3. Praat scripts

Sound extraction script:
```
snd = Open long sound file... <voice .wav file>
tg = Read from file... <.TextGrid file with 'sound' labels>

tier = 2
for interval to 900
  select tg
  label$ = Get label of interval... tier interval
    if label$ == "sound"
    label$ > utt_'interval'.wav
    start = Get start point... tier interval
    end = Get end point... tier interval
    select snd
    Extract part... start end yes
  endif
endfor
```

Sound saving and renaming script:
```
ns = numberOfSelected()
for s from 1 to ns
  snd_'s' = selected(s)
endfor

for s to ns
  select snd_'s'
  Save as WAV file... sound_'s'.wav
endfor
```