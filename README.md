# Hardware specifications for 0x10c

May 2014

This repository serves as a historical archive containing specifications for the fictional hardware of the game 0x10c. The game was to be a multiplayer sandbox game set in space, with a fully programmable CPU controlling a ship. The game was [cancelled in 2013][1] to much dismay of fans. A number of fan projects appeared aiming at continuing development, but they also appear to be abandoned.

There are a large number of fan works on GitHub, mainly implementations of the DCPU-16 hardware or code to run on it. GitHub still has a list of [DCPU-16 ASM trending repositories][2]. These usually included links to the official specifications which were either hosted on Pastebin or 0x10c.com. The later has been been offline since February 2014 (weirdly the domain was renewed for another year in April 2014), so this is my attempt to archive them for future reference.

## Official Specifications

* [DCPU-16](dcpu16.txt) - 16bit CPU.
* [LEM1802](lem1802.txt) - 128x96 pixel color display.
* [SPED-3](sped3.txt) - 3D vector display.
* [M35FD](m35fd.txt) - Floppy disk drive.
* [SPC2000](spc2000.txt) - Deep sleep chamber. This is part of 0x10c's backstory (the number of years to sleep was entered in the wrong byte order).
* [Clock](clock.txt) - Basic real time clock.
* [Keyboard](keyboard.txt) - Keyboard interface.

## Community Adopted Specifications

The floppy drive specification was released approximately six months after the release of the initial specifications. Before this time a fan work specification for the [HMD2043 floppy disk drive](https://gist.github.com/DanielKeep/2495578) was released. A number of emulators continue to use this rather than the official M35FD.

Although the specification of the LEM1802 says it must be initialised before use, most emulators start with the device already initialised with video memory mapped to 0x8000.

Although there is a specification for a keyboard device, Notch's alpha releases of 0x10c which included a functional DCPU-16 system did not follow this. Instead the keyboard is interfaced through a 16 letter ring buffer mapped at 0x9000 (non configurable). An address is 0 before a key is pressed, and should be written as 0 again after being read so it can be checked later. Most emulators follow this functionality.

The official specifications don't go too in depth into the format of the assembly language, and there has been no official assembler released. The code in the specification is similar to NASM and community assemblers are also based on this. Most support labels, and some add macros. The language has come to be known as DASM (DCPU-16 Assembly) and typically has the extension `dasm` or `dasm16`. Object code has the extension `bin`, `dcpu` or `dcpu16`.

## Copyright

The copyright notices in the specifications are assumed to be fictional due to the copyright dates being dated when the game was set (1980s), not when the fictional works were written (2012/2013). Even so, these fictional works are still assumed to be under the copyright of Mojang. They are being reproduced here to serve as a non-commerical archive which is permitted under most copyright laws around the world (including the US where GitHub is based, and the UK where I am based).

The implementation of these specifications in your own commerical work is permitted (it is just a specification and is not patented), however you probably should not reproduce these specification documents as is. Care should be taken with hardware and manufacturer names, as they may also be under the copyright of Mojang.

[Notch has said on Reddit][3] that Mojang didn't want these files to be distributed originally, as they didn't want the hardware to become fragmented before the game was finished. He has also said that implementing the hardware in your own games is allowed.

[1]: http://www.rockpapershotgun.com/2013/08/19/0x10c-cancelled-for-good-but-fans-plan-to-do-it-anyway/
[2]: https://github.com/trending?l=dcpu-16-asm
[3]: http://www.reddit.com/r/dcpu16/comments/1zykmx/hey_guys_what_sort_of_copyright_is_the_dcpu16/cfy7igf
