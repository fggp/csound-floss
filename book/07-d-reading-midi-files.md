D. READING MIDI FILES
=====================

Instead of using either the standard Csound score or live midi events as
input for a orchestra Csound can read a midi file and use the data
contained within it as if it were a live midi input.

The command line flag to instigate reading from a midi file is
\'[-F](http://www.csounds.com/manual/html/CommandFlags.html#FlagsMinusUpperF)\'
followed by the name of the file or the complete path to the file if it
is not in the same directory as the .csd file. Midi channels will be
mapped to instrument according to the rules and options discussed in
[Triggering Instrument
Instances](http://en.flossmanuals.net/bin/view/Csound/Triggering%20Instrument%20Instances)
and all controllers can be interpretted as desired using the techniques
discussed in [Working with
Controllers](http://en.flossmanuals.net/bin/view/Csound/WORKINGWITHCONTROLLERS).
One thing we need to be concerned with is that without any events in our
standard Csound score our performance will terminate immedately. To
circumvent this problem we need some sort of dummy event in our score to
fool Csound into keeping going until our midi file has completed.
Something like the following, placed in the score, is often used.

    f 0 3600

This dummy \'f\' event will force Csound to wait for 3600 second (1
hour) before terminating performance. It doesn\'t really matter what
number of seconds we put in here, as long as it is more than the number
of seconds duration of the midi file. Alternatively a conventional \'i\'
score event can also keep performance going; sometimes we will have, for
example, a reverb effect running throughout the performance which can
also prevent Csound from terminating. Performance can be interrupted at
any time by typing ctrl+c in the terminal window. 

The following example plays back a midi file using Csound\'s
\'fluidsynth\' family of opcodes to facilitate playing soundfonts
(sample libraries). For more information on these opcodes please consult
the [Csound Reference
Manual](http://www.csounds.com/manual/html/index.html). In order to run
the example you will need to download a midi file and two (ideally
contrasting) soundfonts. Adjust the references to these files in the
example accordingly. Free midi files and soundfonts are readily
available on the internet. I am suggesting that you use contrasting
soundfonts, such as a marimba and a trumpet, so that you can easily hear
the parsing of midi channels in the midi file to different Csound
instruments. In the example channels 1,3,5,7,9,11,13 and 15 play back
using soundfont 1 and channels 2,4,6,8,10,12,14 and 16 play back using
soundfont 2. When using fluidsynth in Csound we normally use an \'always
on\' instrument to gather all the audio from the various soundfonts (in
this example instrument 99) which also conveniently keeps performance
going while our midi file plays back.

**  *EXAMPLE 07D01\_ReadMidiFile.csd* **

    <CsoundSynthesizer>

    <CsOptions>
    ;'-F' flag reads in a midi file
    -F AnyMIDIfile.mid
    </CsOptions>

    <CsInstruments>
    ;Example by Iain McCurdy

    sr = 44100
    ksmps = 32
    nchnls = 1
    0dbfs = 1

    sr = 44100
    ksmps = 32
    nchnls = 2

    giEngine     fluidEngine; start fluidsynth engine
    ; load a soundfont
    iSfNum1      fluidLoad          "ASoundfont.sf2", giEngine, 1
    ; load a different soundfont
    iSfNum2      fluidLoad          "ADifferentSoundfont.sf2", giEngine, 1
    ; direct each midi channels to a particular soundfonts
                 fluidProgramSelect giEngine, 1, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 3, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 5, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 7, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 9, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 11, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 13, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 15, iSfNum1, 0, 0
                 fluidProgramSelect giEngine, 2, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 4, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 6, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 8, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 10, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 12, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 14, iSfNum2, 0, 0
                 fluidProgramSelect giEngine, 16, iSfNum2, 0, 0

      instr 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16 ; fluid synths for channels 1-16
    iKey         notnum                 ; read in midi note number
    iVel         ampmidi            127 ; read in key velocity
    ; create a note played by the soundfont for this instrument
                 fluidNote          giEngine, p1, iKey, iVel
      endin

      instr 99 ; gathering of fluidsynth audio and audio output
    aSigL, aSigR fluidOut           giEngine      ; read all audio from soundfont
                 outs               aSigL, aSigR  ; send audio to outputs
      endin
    </CsInstruments>

    <CsScore>
    i 99 0 3600 ; audio output instrument also keeps performance going
    e
    </CsScore>

    <CsoundSynthesizer>

Midi file input can be combined with other Csound inputs from the score
or from live midi and also bear in mind that a midi file doesn\'t need
to contain midi note events, it could instead contain, for example, a
sequence of controller data used to automate parameters of effects
during a live performance.

Rather than to directly play back a midi file using Csound instruments
it might be useful to import midi note events as a standard Csound
score. This way events could be edited within the Csound editor or
several scores could be combined. The following example takes a midi
file as input and outputs standard Csound .sco files of the events
contained therein. For convenience each midi channel is output to a
separate .sco file, therefore up to 16 .sco files will be created.
Multiple .sco files can be later recombined by using
[\#include](http://www.csounds.com/manual/html/include.html)\...
statements or simply by using copy and paste.

The only tricky aspect of this example is that note-ons followed by
note-offs need to be sensed and calculated as p3 duration values. This
is implemented by sensing the note-off by using the
[release](http://www.csounds.com/manual/html/release.html) opcode and at
that moment triggering a note in another instrument with the required
score data. It is this second instrument that is responsible for writing
this data to a score file. Midi channels are rendered as p1 values, midi
note numbers as p4 and velocity values as p5.

***  EXAMPLE 07D02\_MidiToScore.csd***

    <CsoundSynthesizer>

    <CsOptions>
    ; enter name of input midi file
    -F InputMidiFile.mid
    </CsOptions>

    <CsInstruments>
    ; Example by Iain McCurdy

    ;ksmps needs to be 10 to ensure accurate rendering of timings
    ksmps = 10

    massign 0,1

      instr 1
    iChan       midichn
    iCps        cpsmidi            ; read pitch in frequency from midi notes
    iVel        veloc       0, 127 ; read in velocity from midi notes
    kDur        timeinsts          ; running total of duration of this note
    kRelease    release            ; sense when note is ending
     if kRelease=1 then            ; if note is about to end
    ;           p1  p2  p3    p4     p5    p6
    event "i",  2,  0, kDur, iChan, iCps, iVel ; send full note data to instr 2
     endif
      endin

      instr 2
    iDur        =        p3
    iChan       =        p4
    iCps        =        p5
    iVel        =        p6
    iStartTime  times        ; read current time since the start of performance
    ; form file name for this channel (1-16) as a string variable
    SFileName   sprintf  "Channel%d.sco",iChan
    ; write a line to the score for this channel's .sco file
                fprints  SFileName, "i%d\\t%f\\t%f\\t%f\\t%d\\n",\
                                     iChan,iStartTime-iDur,iDur,iCps,iVel
      endin

    </CsInstruments>

    <CsScore>
    f 0 480 ; ensure this duration is as long or longer that duration of midi file
    e
    </CsScore>

    </CsoundSynthesizer>

The example above ignores continuous controller data, pitch bend and
aftertouch. The second example on the page in the [Csound
Manual](http://www.csounds.com/manual/html/index.html) for the opcode
[fprintks](http://www.csounds.com/manual/html/fprintks.html) renders all
midi data to a score file.
