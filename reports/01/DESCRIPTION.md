**Name**

Double free

**Description**

In the function read_png_chunk, an attempt to allocate a chunk and fill it with data (through an fread) is made. In case this fails, the chunk data is then freed.
This, however, might be a problem, as chunk->chunk_data, if not NULL, is freed a second time in the function load_png. 
This might be a problem in case the allocation was successful, but the fread was not: in this case, read_png_chunk frees the memory a first time, but the pointer value does not become NULL. As a consequence, load_png will attempt at freeing it a second time, leading to undefined behavior.

**Affected Lines**

pngparser.c: 301 (first free, deleted, along with the 'if' in the submitted file)
pngparser.c: 706, 724 (second free)

**Expected vs Observed**

We expect the program to read the png chunk, and correctly terminate in case this could not happen correctly.
In practice, in case the png chunk can not be read correctly, the program attempts a double free, often causing segmentation faults.

**Steps to Reproduce**

./size ../reports/01/id\:000000\,sig\:11\,src\:000000\,op\:flip1\,pos\:8

**Proof-of-Concept Input**
(attached: id:000000,sig:11,src:000000,op:flip1,pos:8) 

**Suggested Fix Description**

In order to fix the bug, it is enough to delete the first free located in the read_png_chunk function. As load_png will free the memory anyway, no memory leak will occur either, and the program will correctly terminate.
