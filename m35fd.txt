


                                      .!.
                                     !!!!!.
                                  .   '!!!!!.
                                .!!!.   '!!!!!.
                              .!!!!!!!.   '!!!!!.
                            .!!!!!!!!!'   .!!!!!!!.
                            '!!!!!!!'   .!!!!!!!!!'
                              '!!!!!.   '!!!!!!!'
                                '!!!!!.   '!!!'
                                  '!!!!!.   '
                                    '!!!!!
                                      '!'


                          M A C K A P A R    M E D I A






    .---------------------.
----! DCPU-16 INFORMATION !-----------------------------------------------------
    '---------------------'

Name: Mackapar 3.5" Floppy Drive (M35FD)
ID: 0x4fd524c5, version: 0x000b
Manufacturer: 0x1eb37e91 (MACKAPAR)



    .-------------.
----! DESCRIPTION !-------------------------------------------------------------
    '-------------'

The Mackapar 3.5" Floppy Drive is compatible with all standard 3.5" 1440 KB
floppy disks. The floppies need to be formatted in 16 bit mode, for a total of
737,280 words of storage. Data is saved on 80 tracks with 18 sectors per track,
for a total of 1440 sectors containing 512 words each.
The M35FD works is asynchronous, and has a raw read/write speed of 30.7kw/s.
Track seeking time is about 2.4 ms per track.



    .--------------------.
----! INTERRUPT BEHAVIOR !------------------------------------------------------
    '--------------------'

A, B, C, X, Y, Z, I, J below refer to the registers on the DCPU

A: Behavior:

0  Poll device. Sets B to the current state (see below) and C to the last error
   since the last device poll.

1  Set interrupt. Enables interrupts and sets the message to X if X is anything
   other than 0, disables interrupts if X is 0. When interrupts are enabled,
   the M35FD will trigger an interrupt on the DCPU-16 whenever the state or
   error message changes.

2  Read sector. Reads sector X to DCPU ram starting at Y.
   Sets B to 1 if reading is possible and has been started, anything else if it
   fails. Reading is only possible if the state is STATE_READY or
   STATE_READY_WP.
   Protects against partial reads.

3  Write sector. Writes sector X from DCPU ram starting at Y.
   Sets B to 1 if writing is possible and has been started, anything else if it
   fails. Writing is only possible if the state is STATE_READY.
   Protects against partial writes.


    .-------------.
----! STATE CODES !-------------------------------------------------------------
    '-------------'

0x0000 STATE_NO_MEDIA   There's no floppy in the drive.
0x0001 STATE_READY      The drive is ready to accept commands.
0x0002 STATE_READY_WP   Same as ready, except the floppy is write protected.
0x0003 STATE_BUSY       The drive is busy either reading or writing a sector.



    .-------------.
----! ERROR CODES !-------------------------------------------------------------
    '-------------'

0x0000 ERROR_NONE       There's been no error since the last poll.
0x0001 ERROR_BUSY       Drive is busy performing an action
0x0002 ERROR_NO_MEDIA   Attempted to read or write with no floppy inserted.
0x0003 ERROR_PROTECTED  Attempted to write to write protected floppy.
0x0004 ERROR_EJECT      The floppy was removed while reading or writing.
0x0005 ERROR_BAD_SECTOR The requested sector is broken, the data on it is lost.
0xffff ERROR_BROKEN     There's been some major software or hardware problem,
                        try turning off and turning on the device again.



   COPYRIGHT 1987 MACKAPAR MEDIA    ALL RIGHTS RESERVED    DO NOT DISTRIBUTE
