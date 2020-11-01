# reaper-pyramid-reclock
Reaper JSFX to insert SPP messages into a MIDI Realtime stream from Squarp Pyramid

## The problem:

(1) Pyramid does not send SPP - so if you have Reaper listening for Pyramid midi control messages, you can only ever record from the start of the project.

(2) Pyramid and Reaper’s beat clocks slowly drift apart, depending on various things including the temperature of the room you’re in. And because reaper basically ignores Pyramid's timing messages, if you record for more than a couple of bars your beats are going to start falling in wonky places relative to Reaper. This is because what Pyramid says is 120 BPM might be something like 120.04 BPM in Reaper - this is just the way clocks on computers work. That means the beats just will slowly drift out of sync with the REAPER bar lines. Maybe that’s a problem for you, maybe it’s not. For me, it's a problem.

## The solution:

(1) Use a JSFX input plugin to inject desired SPP messages into the MIDI Realtime message stream coming from Pyramid, send the modified stream out to a virtual loopback device and have Reaper listen for sync messages from that loopback device.

(2) Track the time intervals between MIDI Clock messages coming from Pyramid over a long period of time to get a highly accurate BPM that you can set in Reaper to mostly compensate for the difference in clock speeds.

## How (1) works, in detail:

Hold on to your shirts, this is hacky as hell. (Track numbers refer to template project):

* MIDI Realtime messages from Pyramid arrive in Reaper, on Track 1 (“midi ctl”).
* JSFX input FX script on Track 1 listens to these messages and injects SPP messages to jump to a particular bar when START or CONTINUE message is received.
* Output from Track 1 (which consists of the MIDI Realtime messages duplicated from the Pyramid along with additional SPP on start/continue) is sent to virtual midi loopback device (IAC b1)
* MIDI Realtime + SPP messages from IAC b1 arrive in Reaper, and Reaper uses them to sync.

Surprisingly, it works.


# How to use it

## (1) SPP message injector

1. Copy `damian_MIDI_reclock` to (macOS) `~/Library/Application Support/REAPER/Effects/JSFX` (or wherever it should go on Windows).

2. Setup a virtual MIDI loopback port (on the image below I have two port, you only need one). On macOS you can do this with the built-in IAC driver in Audio MIDI Setup. I don’t know how to do this on Windows.

![IAC settings](https://github.com/damian0815/reaper-pyramid-reclock/raw/main/How-to/Audio%20MIDI%20Setup%20-%20IAC%20driver.png)

3. Set Reaper MIDI settings as shown - for the inputs, you want the loopback port (IAC b1 for me) to say “Enabled+Control” and the Pyramid to say just “Enabled”, and for the outputs, you want the loopback port to say “Enabled” (i don’t know if “\[low latency\]” is necessary but it can’t hurt)

![Reaper MIDI settings](https://github.com/damian0815/reaper-pyramid-reclock/raw/main/How-to/REAPER%20-%20MIDI%20settings.png)

4. Set Reaper Sync settings as shown - you want “Enable synchronization to timecode”, “Playback”, “Recording”, and “Start playback on valid timecode when stopped” to all be checked, and “Use input” should be set to the virtual MIDI loopback port you setup in step 2. Set “Freewheel on missing time code” to 1000 and “Synchronize by seeking ahead” to 100 - if you want to mess with these, just beware that if you set them to values that are too low, weird things can happen and you might have to restart Reaper and/or your computer.

![Reaper sync settings](https://github.com/damian0815/reaper-pyramid-reclock/raw/main/How-to/REAPER%20-%20sync%20settings.png)

5. Open project-template.RPP. Press Stop on Pyramid.

6. **THIS IS EXTREMELY IMPORTANT** - Press Pause in Reaper AND THEN press Record in Reaper. There will be no flashing “Waiting for sync” message. Press Play on Pyramid to make Reaper will start recording. **IF YOU ONLY PRESS RECORD IN REAPER WITHOUT PRESSING PAUSE FIRST YOU’LL GET THE FLASHING RED WINDOW BUT RECORDING WILL NOT START WHEN YOU PRESS PLAY ON PYRAMID**. I think I know why this is happening but it would be complicated to explain and I might be wrong.

7. On the Plugin UI, you can set the bar/beat to jump to on play by setting the sliders as shown. Or you can move the playhead to where you want to start recording and click one of the numbered buttons next to "triggers" (playhead position will get rounded to the nearest beat).

![JSFX plugin UI](https://github.com/damian0815/reaper-pyramid-reclock/raw/main/How-to/plugin-ui.png)


## (2) Super-accurate BPM calculater

1. Press Pause in Reaper AND THEN press Record in Reaper. Press Play on Pyramid. Note the percentage counter increasing to 100%. 

2. When the status percentage counter gets to 100% you can trust the reported BPMs. Note the “smoothed smoothed” value and set your Reaper project BPM to *exactly* this value (including all the numbers after the decimal point). Of the three BPMs this one has the least amount of jitter, but it takes longer to respond to changes. I’ve been able to record continously for 15 minutes using this value and have my beats offset by less than a millsecond relative to the Reaper bar lines at the end of that 15 minutes.

3. To reset the counter, change the accuracy (and then change it back). Higher values here are more accurate but the BPM takes longer to calculate.

## Thanks

Hope this helps someone :)

Damian
