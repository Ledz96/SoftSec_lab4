**Why did you need to change is_png_chunk_valid?**

The function is_png_chunk_valid checks for integrity of png chunks by calculating a checksum; since the fuzzer only "randomly" tweaks some regions of the input file (not in a completely random way, but often based on "genetics algorithms", the integrity check would fail, stopping the flow of the program and making the whole process of looking for bugs ultimately way less useful than it would be otherwise.

** Why did we give you exactly TWO seeds for fuzzing? **

A different seed is given for each different input image, palette.png and rgba.png. The fuzzer will then use the two different input seeds and tweak them to find bugs.

** Why did you have to use afl-gcc to compile the source (and not e.g. ordinary gcc)? **

As the official documentation states:
"(afl-gcc) is a helper application which serves as a drop-in replacement for gcc, used to recompile third-party code with the required runtime instrumentation for afl-fuzz."

In short, afl-gcc (or clang) is needed in order to add through compilation runtime instruments for fuzzing: this is necessary as afl is a greybox fuzzer, that needs some lightweight program analysis in order to run.

** How many crashes in total did AFL produce? How many unique crashes? **

In the first instance of running AFL, 244k crashes were produced, of which only 37 unique. This is to be expected, as the fuzzer only slightly tweaks the input based on previous successful crashes, leading to many crashes caused by the same issue.

** Why are hangs counted as bugs in AFL? Whichtype of attack can they be used for? **

Hangs are counted as bugs as they are often undesired behavior of the program, that may be caused by infinite loops, starvation due to race conditions etc.
As hangs may be used, for example, in DoS attacks, AFL reports them.

** Which interface of libpngparser remains untested by AFL (take a look at pngparser.h)? **

As we can notice by analysing the pngparser.h and the execution of the program, the interface store_png remains untested, as it is uncalled in the execution of the program.

** In lab0x01 many students encountered a crash in load_png when the input file with the specified name didn’t exist. Can AFL find that bug in this setup?**

As AFL only edits the content of the input file, this is not a kind of bug that can be found through the use of a fuzzer with the specified settings. 
Despite that, I identified the bug, and detailed it in the third report.

** How long did you run AFL for? If you run it for twice as long, do you expect to find twice as many bugs? Why? **

On average, I run AFL for one minute or so each time. Running it for longer would, in fact, not be much beneficial, as the coverage wall has been hit.
As the fuzzer works by finding inputs that crash the program and using them to adjust its next inputs, it is common for it to get stuck on the same few unique crashes for a long time.
Since only unique crashes are useful to find bugs, we do not expect to find twice as many bugs by running the fuzzer for twice as long.
