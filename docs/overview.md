# Guile on Windows 2025

I'm on a mission to make Guile run like a champ on Windows, despite my lack of love for the OS—honestly, I'd ditch it if I could. But work demands it, and with LilyPond in need, with Lisp Game Jam coming up, and the Guile community short on Windows sufferers, I’m stepping up.

There’s a pile of patch sets that can cobble together a working Guile on Windows, but they’re stuck in limbo. Why? Maintainers don’t know Windows, can’t judge the patches, and the sets lack the polish to prove they’re legit. I’m here to fix that—delivering a slick, complete patch set with all the proof they’ll need.

Think of it like an RPG: the main quest is getting Guile’s core tree Windows-ready. To keep it bite-sized for reviewers, I’m splitting it into side quests and main missions, each one pushing us closer to that final pull request. Game on.

## 1. [COMPLETE] Sidequest: Linux Machine Lockdown  
_Difficulty_: 1/5  

First move: resurrect a Linux rig. I yanked an old laptop from the tech scrapheap, slapped in a fresh battery, and fired up the latest Fedora. Gnome’s default vibe was sluggish, but a quick main menu tweak and some window controls—Maximize and Minimize buttons via GNOME Tweaks—got it humming.  

## 2. [COMPLETE] Sidequest: Emacs, Debbugs, Mail, IRC Mastery  
_Difficulty_: 3/5  

To crack Guile’s patch game, I had to sync with their comms—mailing lists, Debbugs, IRC. It’s a different world from webmail, Zoom, and Teams, and wrestling email into Emacs was a beast. My survival guide’s below.  
* [Mail Setup](002-mail.html)  
* [IRC Setup](002-IRC.html)  
* [Debbugs Setup](002-debbugs.html)  

## 3. [TODO] Sidequest: Unbreak `make distcheck`  
Guile’s distcheck is broken, and I need it solid to build from a clean slate.  
* [IN REVIEW] Bug 76906: Make distcheck broken

## 4. [TODO] Sidequest: Guile in a Linux Docker Cage  
_Difficulty_: 2/5  

This gig’s about proving Guile can build and run outside the cozy confines of `/usr/lib` and `/usr/share`. It’s a warm-up for container talks—cache files, read-only vs read-write dirs—and a sneak peek at Windows’ wild directory quirks. Groundwork’s getting laid.  

## 5. [TODO] Sidequest: FlatPak Guile Wrap  
_Difficulty_: 2/5  

This is Docker’s cousin with a twist—FlatPak’s got its own directory chaos, permissions, and cache tricks. Hardening Guile to handle weird structures and cache/user dir strategies is the portability key. Mission’s on.

## 6. [TODO] Sidequest: Guile Goes Cygwin  
_Difficulty_: 2/5  

Cygwin’s a Windows lifeline—POSIX vibes, bash shell, and a quick recompile for Linux apps with barely a tweak. This mission’s about getting Guile’s extensions to play nice as DLLs in Cygwin’s world. We’ll crack libtool’s DLL versioning and make sure those DLLs land in a `PATH` or `bin` spot where they’ll actually work.  

## 7. [TODO] Sidequest: Guile Meets MSYS2  
_Difficulty_: 1/5  

MSYS2 is Windows’ Linux impersonator—bash, compilers, autotools, the works. It’s Cygwin’s twin for compiling, but it’s less about runtime and more about building. We’re not aiming for full Guile glory here—just rock-solid GNU Make support with Guile extensions. The catch? Sorting out DLL naming clashes between Cygwin and MSYS2.  

## 8. [TODO] Quest: Guile on Linux—32-bit, No Frills

_Difficulty_: 2/5

The 32-bit, no-threads, no-JIT, no-LTO setup is MinGW-w32’s foundation, but it’s gotta work on Linux first. These oddball build options are often busted. Time to fix ‘em.

Since my Linux box is 64-bit, this will be proved out using a 32-bit Debian distro inside a Docker container.
  
* Bug 36340: Squash the no-networking build glitch  
* Bug 76907: Patch the getsockopt buffer overflow  

## 9. [TODO] Major Quest: Guile on MinGW-w32 — Bare Bones  
_Difficulty_: 4/5  

Guile used to hum on 32-bit MinGW-w32 with threads, JIT, and LTO switched off—until versions 3.0.9 and 3.0.10 tanked it. The interpreter choked on 32-bit setups, and bit rot piled on. Reviving it means untangling a mess: DLL quirks, character encoding fights, and Windows-Linux permission clashes. Good news? MSYS2’s bash shell and standard paths give us a leg up for the build.  

## 10. [TODO] Major Quest: Guile on MinGW-w64 UCRT — Full Throttle  
_Difficulty_: 4/5  

The 32-bit MinGW MSVCRT win was just a warm-up. The real prize is MinGW-w64 with the 64-bit UCRT library. This beast needs LilyPond’s Guile patches to fire up threads and JIT, plus a fix for Windows’ still-wonky character encoding—better with UCRT, but not perfect. Threads will run, but with no SIGALRM signal on Windows, the httpd component’s gonna need extra patch love to limp along.  

## 11. [TODO] Sidequest: Guile Solo on Windows—No Bash Crutch  
_Difficulty_: 3/5  

MinGW-w64 with UCRT leaned hard on bash and Linux-y directories, locking Guile in MinGW’s sandbox. To break free and go full Windows—usable and shareable—it’s gotta vibe with standard Windows folder setups. DLLs need to bunk next to the executable, and it’s got to shine in Microsoft Terminal. Plus, the `guild` tool—a bash script—needs a Windows-native makeover to ditch MinGW’s bash dependency.  

## 12. [TODO] Major Boss: Guile as an MSIX Package  
_Difficulty_: 3/5 + $200  

Time to cash in all our chips and wrap Guile in an MSIX bow. We’ll tap the Win32 API to pick smart app-local cache spots, set file and socket permissions, craft XML to map the MSIX guts, and tackle code signing. Scripts and commands will seal the deal to build this containerized package, suitable for the Windows App Store.

I can guarantee that *no one* in the Free Software community will *ever*
build an MSIX package that can be distributed, since that requires
code signing certificate which costs hundreds of dollars.  But I happen
to have one, so let's take advantage of the fact that I've sold my
soul long ago for filthy lucre, so I can document this process.

## 13. [TODO] Final Boss: Merge the Win32 Patches  
_Difficulty_: 5/5  

This epic trek—from side gigs to the big showdown—delivers a battle-hardened set of Guile patches for Windows. We’ve crushed it: containerization, Cygwin tweaks, standalone MSIX glory. Every test, every fix proves these patches aren’t just working—they’re bulletproof and documented to death. They’ve earned their stripes for the main Guile Savannah tree. With this, Guile’s Windows game levels up, and the community wins big on portability, but perhaps has also taken a step to the dark side. 

The last hurdle? Not tech—it’s people. Have I sold it hard enough to get the merge?

## Conclusion

Guile has limped along in Windows for years but never
really truly worked, and as of 3.0.10, it is mostly broken.
Lilypond needs to run on Windows.  They shouldn't have to carry their
own patchsets.  But Lilypond's solution is specific to their case.
It won't help with the Lisp Game Jam or other places where a Win32 build
could be useful.

To really support Windows is to handle all of this nonsense.

