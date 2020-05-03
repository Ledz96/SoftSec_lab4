**Name**

Out of bounds access

**Description**

In the function 'convert_color_palette_to_image', two parameters height and width, of the type uint32_t, are used to determine height and width of the image. 
The struct image, however, uses 16 bit unsigned integers for the fields size_x and size_y. This leads to possible mistakes in the casting operation, with discrepancies between the values of img->size_x and width (as well as img->size_y and height). 
This is particularly problematic in the loop that iterates through the image, as the upper bounds are not img->size_x and img->size_y, which represent the actual height and width of the allocated pointer, but rather the variables 'width' and 'height', which are represented in 4 bytes.
As a consequence, in case a 32 bit value higher than the highest representable in 16 bits is provided, an out of bounds access will happen, since the upper limit of the for loop will be higher than the actual upper limit allocated.

**Affected Lines**

pngparser.c: 426, 433 (wrong checks)
pngparser.c: 428-438 (possible out of bounds accesses)

**Expected vs Observed**

We expect the program to correctly allocate the image, and access its elements to color it. Obviously, we expect no out of bounds access. No clear specification is given in case of large images; different solutions will be provided in the final session (suggested fix description).
In practice, the program simply tries a cast to 16 bit integers, and then uses 32 bit large upper bounds: in case of large inputs, this leads to out of bound accesses, that can cause memory corruption or/and segmentation faults.

**Steps to Reproduce**

Possible input (works with any non-valid png input)
./size ../reports/03/Problem_4/id\:000000\,sig\:11\,src\:000000\,op\:flip1\,pos\:16

**Suggested Fix Description**

Multiple solutions might be adopted to fix this bug. I will now illustrate two of them:
1) A simple but effective solution would be to allow the cast to 16 bits integer, and then use the img->size_x and img->size_y fields as upper bounds for the for loop. While this 'corrupts' the input image, the solution avoids out of bound accesses.
2) This solution, that I deemed as better and chose as a consequence, consists in replacing size_x and size_y with 32 bits values, thus making the cast safe. In addition to this, I added a much necessary check on the allocation of the img->px pointer, as large images may not fit. In absence of said check, attempts to dereference the pointer would be made, leading to segmentation faults.
