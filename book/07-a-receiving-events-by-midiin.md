A. RECEIVING EVENTS BY MIDIIN
=============================

Csound provides a variety of opcodes, such as
[cpsmidi](http://www.csounds.com/manual/html/cpsmidi.html "cpsmidi"),
[ampmidi](http://www.csounds.com/manual/html/ampmidi.html "ampmidi") and
[ctrl7](http://www.csounds.com/manual/html/ctrl7.html "ctrl7"), which
facilitate the reading of incoming midi data into Csound with minimal
fuss. These opcodes allow us to read in midi information without us
having to worry about parsing status bytes and so on. Occasionally
though when more complex midi interaction is required, it might be
advantageous for us to scan all raw midi information that is coming into
Csound. The
[midiin](file:///C:/Program%20Files/Csound/doc/manual/midiin.html "midiin")
opcode allows us to do this.

In the next example a simple midi monitor is constructed. Incoming midi
events are printed to the terminal with some formatting to make them
readable. We can disable Csound\'s default instrument triggering
mechanism (which in this example we don\'t want to use) by writing the
line:

    massign 0,0 

just after the header statement (sometimes referred to as instrument 0).

For this example to work you will need to ensure that you have activated
live midi input within Csound, either by using the [-M
flag](http://www.csounds.com/manual/html/CommandFlagsCategory.html#FlagsCatMinusUpperM)
or from within the QuteCsound configuration menu. You will also need to
make sure that you have a midi keyboard or controller connected. You may
also want to include the [-m0
flag](http://www.csounds.com/manual/html/CommandFlags.html#FlagsMinusLowerM)
which will disable some of Csound\'s additional messaging output and
therefore allow our midi printout to be presented more clearly.

The status byte tells us what sort of midi information has been
received. For example, a value of 144 tells us that a midi note event
has been received, a value of 176 tells us that a midi controller event
has been received, a value of 224 tells us that pitch bend has been
received and so on.

The meaning of the two data bytes depends on what sort of status byte
has been received. For example if a midi note event has been received
then data byte 1 gives us the note velocity and data byte 2 gives us the
note number. If a midi controller event has been received then data byte
1 gives us the controller number and data byte 2 gives us the controller
value. 

**   *EXAMPLE 07A01\_midiin\_print.csd***

    <CsoundSynthesizer>
    <CsOptions>
    -Ma -m0
    ; activates all midi devices, suppress note printings
    </CsOptions>

    <CsInstruments>
    ; Example by Iain McCurdy

    ; no audio so 'sr' or 'nchnls' aren't relevant
    ksmps = 32

    ; using massign with these arguments disables default instrument triggering
    massign 0,0

      instr 1
    kstatus, kchan, kdata1, kdata2  midiin            ;read in midi
    ktrigger  changed  kstatus, kchan, kdata1, kdata2 ;trigger if midi data changes
     if ktrigger=1 && kstatus!=0 then          ;if status byte is non-zero...
    ; -- print midi data to the terminal with formatting --
     printks "status:%d%tchannel:%d%tdata1:%d%tdata2:%d%n"\
                                        ,0,kstatus,kchan,kdata1,kdata2
     endif
      endin

    </CsInstruments>
    <CsScore>
    i 1 0 3600 ; instr 1 plays for 1 hour
    </CsScore>
    </CsoundSynthesizer>

The principle advantage of using the *midiin* opcode is that, unlike
opcodes such as *cpsmidi*, *ampmidi* and *ctrl7* which only receive
specific midi data types on a specific channel, *midiin* \'listens\' to
all incoming data including system exclusive messages. In situations
where elaborate Csound instrument triggering mappings that are beyond
the capabilities of the default triggering mechanism are required, then
the use of *midiin* might be beneficial.
