**Name**

Dereference of null pointer

**Description**

When the png format is not recognized, the 'load_png' function returns a non-zero value. 
In the file 'size.c', however, the result returned by 'load_png' is checked in the wrong way, as if a non-zero value was returned in case of success. 
As a consequence, two undesired results are generated: 
* in case the file has been loaded correctly, the program returns 1 immediately, without freeing the allocated pointers or printing the image size.
* in case the file has not been loaded correctly, the program tries to access the image's dimensions. However, the image pointer is still null, leading to a segmentation fault. In addition, even removing this line, an attempt to free a non-allocated pointer would be made: this is undefined behavior, and leads to a segmentation fault in most cases.

**Affected Lines**

size.c: 16 (wrong check)
size.c: 20 (dereference of null pointer)
size.c: 21,22 (free on non allocated pointer + dereference in line 21)

**Expected vs Observed**

We expect the program to check whether or not the png has been loaded correctly: in case it has been, then the program should print its dimensions and free the memory; in case it has not, it should just return.
In practice, the program only returns if a png has been loaded correctly; in case it has not, it tries to access a field of a null pointer to struct, leading to a segmentation fault.

**Steps to Reproduce**

Possible input (works with any non-valid png input)
./size ../reports/00/id:000000,sig:11,src:000000,op:flip1,pos:0

**Proof of concept**

(attached: id:000000,sig:11,src:000000,op:flip1,pos:0)

**Suggested Fix Description**

In order to fix the bug, it's enough to delete the '!' in line 16 of the code. This way, the program will correctly check whether or not a png has been correctly loaded (i.e.: a non 0 result is returned in case the loading is not successful), and only dereference the pointer in case the image has been loaded correctly.
