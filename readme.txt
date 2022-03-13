update - 3/11/2022
-now compatible with 0cc-Famitracker and Dn-Famitracker
-fixed a bug in text2vol5 (undefined behavior) 
 that might have caused incorrect output
-in famitone asm, made variables contiguous 
 

famitone 5 (unofficial)
text2vol5, replaces text2data from famitone2
nsf2data5, replaces nsf2data
doug fraker, nesdoug.com


additions so far:
	-volume column support
	-full note range
	-1xx,2xx,3xx,4xx,Qxx,Rxx effects
	-duty cycle envelopes
	-sound fx > 256 bytes
	
version5.0:
	-added command line switch -allin to prevent removing unused instruments
	-added command line switch -Wno to ignore warnings about unsupported effects
	-fixed bug, multiple D00 effects (different channels) incorrect pattern length
	-fixed bug, Bxx below D00 effect (different channels) incorrect pattern length
	-fixed bug, Bxx loop back causing wrong instrument inserted at loop point
example of new switches:
text2vol4 filename.txt -ca65 -allin -Wno	

usage notes:
	-use nsf2data5 (included) for sound effects.
	-must use with famitone5 asm files
	-should use with famitracker 0.4.6 (or later) text export
	-use volume column for Square 1, Square 2, and Noise Channels
	-at the start of a song, the volume column is assumed to be 'f'
	-if you loop backwards, use a volume column entry for the first note at the 
	 loop point, or else it will carry over whatever volume is still loaded
	Example - at the start of the loop, you fail to put a volume column entry, 
	 but at the end of the loop, you fade out to zero. When it loops backwards, 
	the volume will still be set to zero.
	-use any note for Square 1 + 2, Triangle, or Noise Channel
	-restrict DPCM to C-1 to D-6
	-pal support fixed.


added features:
	-use the command line -allin to ensure all instruments are output, even if 
	 no song uses it. this may come in handy if you have too many songs and need 
	 to split the famitracker file into 2 parts.
	-the song titles will appear in comments in the asm data. This may come in 
	 handy if you need to cut or move 1 song for space reasons.

Support for 1xx,2xx,3xx,4xx,Qxx,Rxx effects. They apply to the Square and Triangle 
channels. Use 100,200,or 300 to disable the pitch bend. Use 400 to disable to vibrato. 
The vibrato speed (4xy where x is speed) is permanently set to 6. I recommend that you 
write all the vibrato effects like 461 462 463, etc. "a" (like 46a) is the maximum depth.

	Don't mix 1xx,2xx,3xx,Qxx,Rxx effects with arpeggio or pitch envelopes.

You can put a 1xx(2xx) and 4xx effect at the same time on a note, but not on the same 
line...only 1 fx column is allowed(per channel), so 1xx(2xx) on the first line and 4xx 
on the line below that.

	There is no range check for Qxx or Rxx effects. The code assumes that you, the 
musician, would not tell it to bend up to a note that doesn't exist.


	
	
	




Troubleshooting

My song is silent? Possible reasons...

-you are calling a song # too high. The first song is 0, not 1
LDA #song_number before calling FamiToneMusicPlay

-you forgot to initialize the music code, make sure you 
send the correct address of the music data to the init code
LDX #<music_data ;low
LDY #>music_data ;high
LDA #1 ;NTSC = 1, PAL = 0
JSR FamiToneInit

-you forgot to call FamiToneUpdate once per frame

-make sure you processed the song with text2vol5, and not
an earlier version. famitone5 is incompatible with any
other version of famitone.



Sound Fx aren't working? Possible reasons...

-you are calling a sfx # too high. The first sfx is 0, not 1

-you are calling a sfx channel with the wrong value
FT_SFX_STREAMS	= 1...4 
LDX #channel, LDA #which_sfx, JSR FamiToneSfxPlay
but X should have a value 0,15,30, or 45

-forgot to turn on sfx... FT_SFX_ENABLE shouldn't be commented out

-forgot to initialize sfx, you need to send a pointer to the sfx data 
LDX #<sfx_data ;low
LDY #>sfx_data ;high
JSR FamiToneSfxInit

-using the old nsf2data (earlier than v1.15) which has a different
header system to the data. You should be using nsf2data5.



famitone comparison (2021)
-------------------
ROM size
famitracker  5743
famitone2    1478-1645
famitone3    1886-2104
famitone4    2381-2603
famitone5    2975-3197
*smaller size if only NTSC or only PAL
 larger if both, because of the note tables
** sizes are about 250 bytes bigger than the previous
 versions because of the volume table added, as some
 musicians requested an exact copy of famitracker.
 It should save some CPU time, since it no longer
 has to multiply volumes on the fly.

Song size
famitracker 4752  131%
famitone2   3624  100%
famitone3   4859  134%
famitone4   4859  134%
famitone5   4866  134%

CPU Time (estimated)
famitracker 3000  167%
famitone2   1800  100%
famitone3   2000  111%
famitone4   2300  128%
famitone5   2500  139%

Features
famitracker = all effects
famitone2 = Bxx,Fxx,D00 no duty envelopes, no volume column, reduced note range
famitone3 + all notes, volume column
famitone4 + 1xx,2xx,4xx
famitone5 + 3xx,Rxx,Qxx, duty envelopes, sfx > 256 bytes

