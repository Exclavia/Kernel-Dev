# 7. The Heap
In order to be responsive to situations that you didn't envisage at the design stage, and to cut down the size of your kernel, you will need some kind of dynamic memory allocation. The current memory allocation system (allocation by placement address) is absolutely fine, and is in fact optimal for both time and space for allocations. The problem occurs when you try to free some memory, and want to reclaim it (this must happen eventually, otherwise you will run out!). The placement mechanism has absolutely no way to do this, and is thus not viable for the majority of kernel allocations.

As a sidepoint of general terminology, any data structure that provides both allocation and deallocation of contiguous memory can be referred to as a heap (or a pool). There is, as such, no standard 'heap algorithm' - Different algorithms are used depending on time/space/efficiency requirements. Our requirements are:

(Relatively) simple to implement.Able to check consistency - debugging memory overwrites in a kernel is about ten times more difficult than in normal apps!

The algorithm and data structures presented here are ones which I developed myself. They are so simple however, that I am sure others will have used it first. It is similar to (though more simple than) [Doug Lea's malloc](https://gee.cs.oswego.edu/dl/html/malloc.html) which is used in the GNU C library.

## 7.1. Data structure description
The algorithm uses two concepts: blocks and holes. Blocks are contiguous areas of memory containing user data currently in use (i.e. malloc()d but not free()d). Holes are blocks but their contents are not in use. So initially by this concept the entire area of heap space is one large hole.
<img src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/heap_format.png" >
For every hole there is a corresponding descriptor in an index table. The index table is always ordered ascending by the size of the hole pointed to.

Blocks and holes each contain descriptive data - a header and a footer. The header contains the most information about the block - the footer merely contains a pointer to the header (the reason for the footer will become apparent soon). Pseudocode:
```c
typedef struct
{
  u32int magic;  // Magic number, used for error checking and identification.
  u8int is_hole; // 1 if this is a hole, 0 if this is a block.
  u32int size;   // Size of the block, including this and the footer.
} header_t;

typedef struct
{
  u32int magic;     // Magic number, same as in header_t.
  header_t *header; // Pointer to the block header.
} footer_t;
```
Notice that each also has a 'magic number' field. This is for error checking, and later will play a part in our 'free' algorithm. This is just a sentinel number - an unusual number that will stand out from others - much like 0xdeadbaba that we used in chapter 2. In the sample code I've gone for 0x123890AB arbitrarily.

Note also that within this tutorial I will refer to the size of a block being the number of bytes from the start of the header to the end of the footer - so within a block of size x, there will be x - sizeof(header_t) - sizeof(footer_t) user-useable bytes.

## 7.2. Algorithm description
### 7.2.1. Allocation
Allocation is straightforward, if a little long-winded. Most of the steps are error-checking and creating new holes to minimise memory leaks.

1. Search the index table to find the smallest hole that will fit the requested size. As the table is ordered, this just entails iterating through until we find a hole which will fit.
      If we didn't find a hole large enough, then:
      1. Expand the heap.
      2. If the index table is empty (no holes have been recorded) then add a new entry to it.
      3. Else, adjust the last header's size member and rewrite the footer.
      4. To ease the number of control-flow statements, we can just recurse and call the allocation function again, trusting that this time there will be a hole large enough.
2. Decide if the hole should be split into two parts. This will normally be the case - we usually will want much less space than is available in the hole. The only time this will not happen is if there is less free space after allocating the block than the header/footer takes up. In this case we can just increase the block size and reclaim it all afterwards.
3. If the block should be page-aligned, we must alter the block starting address so that it is and create a new hole in the new unused area.
   If it is not, we can just delete the hole from the index.
4. Write the new block's header and footer.
5. If the hole was to be split into two parts, do it now and write a new hole into the index.
6. Return the address of the block + sizeof(header_t) to the user.
