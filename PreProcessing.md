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

An easy way of extracting sound files containing individual prompts from a single sound file is automatizing the process by using a script written in e.g. Praat. In order to be able to do this, it is necessary to have a set of boundaries determining the beginning and endpoint of the interval containing each individual prompt.

In case there is a channel containing the recording of prompt-separating beeps, this could be converted to a .TextGrid file with Praat's `Annotate > To TextGrid(silences)...` option that can be accessed from the object window. Each beep should be labeled as a silence, and the area between them (esentially the areas containing the prompts) should be labelled as sound.

To open `TextGrid` and `Sound files` together in Praat:
* open the files individually (`.wav` using `Open > Open long sound file` and `TextGrid` using `Open > Read from file`)
* select the opened files and `View & Edit`
* proceed to boundry editting

In case the recording was a single-channel one, the boundaries could be set by hand.
To set a boundary in Praat (mark the beginning/end of an utterance), click on the appropriate place of the waveform. Once you click on a circle that appeared on a tier, a boundary is created. After a second boundary is set, the transcription/marking of the utterance can be added. Boundaries can be adjusted by clicking on them and pulling the blue bar. To delete a boundary, highlight it and press `Alt+Backspace`.

Having a third channel for error markup proves to be beneficial: any imperfections of an utterance can be marked with the help of a midi-keyboard during the prompt recording. At the stage of further data processing, annotated midi markings merged with the sound file help to spot the parts with the boundaries that need attention, which speeds up the clean-up of the sound file significantly.

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
In order for the second script to run, the files to be saved need to be selected in Praat's object window.

### 4. Python script

This Python script can be used to extract prompts from a text file containing a list of prompts with labels.

```
promptList = []

# reading in individual prompts
with open("<original prompt file>") as promptFile:
    for line in promptFile:
        line = line.strip('\n')
        line = line.strip()
        # stripping the label, number, and quotation marks from the prompt
        line = line[11:-2]
        promptList.append(line)

# saving individual prompts as files
i = 1
for element in promptList:
	promptTranscriptFile = open(("sound_" + str(i) + ".txt"), 'a')
	promptTranscriptFile.write(element)
	promptTranscriptFile.close()
	i += 1
```

### 5. Final steps

The previous sections described the creation of sets of .wav and .txt files necessary as inputs for MaryTTS voicebuilding. Once they are ready, they should be copied to the voicebuilding folder into their respective folders. Note that the names of .wav and .txt files have to match in order for the files to be matched properly.
