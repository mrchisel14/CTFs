# Mov it! (Rev 250pts 54 Solves)

The [binary](./mov.out).

By the name of this binary we get the hint this is probably [movfuscated](https://github.com/xoreaxeaxeax/movfuscator) code, and it was. First I ran file and then strings on the binary. I found this was a 32-bit ELF file and there were way too many strings to look at. Next I tried to run the binary and there was no output at all. So I ran strace and got a bunch of SIGSEGV and SIGILL commands.

```
--- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, si_addr=0} ---
--- SIGILL {si_signo=SIGILL, si_code=ILL_ILLOPN, si_addr=0x804babd} ---
--- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, si_addr=0} ---
--- SIGILL {si_signo=SIGILL, si_code=ILL_ILLOPN, si_addr=0x804babd} ---
.
.
.
```
Nothing else seemed that interesting to me other than the two rt_sigaction calls that were probably hooking the two signals, since normally these signals would cause a binary to crash rather than repeat. I decided to try one more thing to get an idea for what was going on in the binary and that was to run ltrace. I got the following output.

```
sigaction(SIGSEGV, { 0x8048260, <>, 0, 0 }, nil)              = 0
sigaction(SIGILL, { 0x80482e7, <>, 0, 0 }, nil)               = 0
--- SIGSEGV (Segmentation fault) ---
strlen("movmustbeturingcomplete")                             = 23
--- SIGILL (Illegal instruction) ---
--- SIGSEGV (Segmentation fault) ---
strlen("movmustbeturingcomplete")                             = 23
.
.
.
```
So my thought was the flags length was being compared with the length of "movmustbeturingcomplete" which would be 23. I also noticed the strlen method was called 23 times. Next I decided to run the binary in gdb and got:
```
Stopped reason: SIGSEGV
0x0804b2c9 in main ()
```
I was unable to continue the program using the 'c' command. Looking at the disassembly of main confirmed this was a movfuscated binary (there was nothing but mov instructions). I knew the binary used the SIGSEGV and SIGILL instructions several times but I couldn't figure out how to debug with GDB because it would ust stop when ever either of these signals were thrown. After, trying to analyze the binary statically and reading about movfuscated code I realized in order to call library functions movfuscated code uses signals and signal handlers to make those calls, which made the ltrace output make sense. This knowledge made me want to try GDB again and see if I could get past GDB terminating the program after the first SIGSEGV. I realized I needed to pass these signals to the program rather than allow GDB to catch them. So I used the following commands to do that.

```
handle SIGSEGV pass
handle SIGILL pass
```
This tells GDB to pass the signals specified to the program. Then I tried to just hit continue and run through the program. Using peda I saw the flag printing on the stack. 

![FLAG](gdb.gif)

### flag: gigem{Th4t_is_Terr1bl3}
