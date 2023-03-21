---
title: Fixing the qemu serial console terminal size.
tags: ubuntu
---
# Fixing the qemu serial console terminal size.
Qemu can allow you to attach to a serial console of your guest.
This is done with either `-nographic` or `-serial mon:stdio`.
Its great for getting text output, but after you've logged in,
you may find issues editing long command lines or using an editor
such as vi.

To fix this, you simply need to tell linux what the terminal
size you're using is.

First, figure out what your terminal size is.  You have to do this
before connecting to qemu, or some other way such as another tmux
window or terminal.

 * With bash via `$LINES` and `$COLUMNS`:

        $ echo rows $LINES columns $COLUMNS
        rows 75 columns 120

 * With stty, look for 'rows' and 'columns' below:

        $ stty -a
        speed 38400 baud; rows 75; columns 120; line = 0;
        intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>;
        eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R;
        werase = ^W; lnext = ^V; discard = ^O; min = 1; time = 0;
        -parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
        -ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff
        -iuclc -ixany -imaxbel iutf8
        opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
        isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt
        echoctl echoke -flusho -extproc


Then tell linux inside qemu about it via `stty`:

    $ stty rows 75 columns 120

    # now show what it is
    $ stty -a |head -n 1
    speed 115200 baud; rows 75; columns 120; line = 0;

Thats it.
