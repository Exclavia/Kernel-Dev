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

### 7.2.2. Deallocation
Deallocation (freeing) is a little more tricky. As mentioned earlier, this is where the efficiency of a memory-management algorithm is really tested. The problem is effective reclaimation of memory. The naive solution would be to change the given block to a hole and enter it back into the hole index. However, if I do this:
```c
int a = kmalloc(8); // Allocate 8 bytes: returns 0xC0080000 for sake of argument
int b = kmalloc(8); // Allocate another 8 bytes: returns 0xC0080008.
kfree(a);           // Release a
kfree(b);           // Release b
int c = kmalloc(16);// What will this allocation return?
```
Note that in this example the space required for headers and footers have been purposely omitted for readability

Here we have allocated space for 8 bytes, twice. We then release both of those allocations. With the naive release algorithm we would then end up with two 8-byte sized holes in the index. When the next allocation (for 16 bytes) comes along, neither of those holes can fit it, so the kmalloc() call will return 0xC0080010. This is suboptimal. There are 16 bytes of space free at 0xC0080000, so we should be reallocating that!

The solution to this problem in most cases is a varation on a simple algorithm that I call unification - That is, converting two adjacent holes into one. (Please note that this coining of a term is not from a sense of self-importance, merely from the absence of a standardised name).

It works thus: When free()ing a block, look at what is immediately to the left (assuming 0-4GB left-to-right) of the header. If that is a footer, which can be discovered from the value of the magic number, then follow the pointer to it's header and query whether it is a hole or a block. If it is a hole, we can modify it's header's size attribute to take into account both it's size and ours, then point our footer to it's header. We have thus amalgamated both holes into one (and in this case there is no need to do an expensive insert operation on the index).

That is what I call unifying left. There is also unifying right, which should be performed on free() as well. Here we look at what is directly after the footer. If we find a header there, again identified by it's magic number, we check if it is a hole. We can then use it's size attribute to find it's footer. We rewrite the footer's pointer to point to our header. Then, all that needs to be done is to remove it's old entry from the hole index, and add our own.

Note also that in the name of reclaiming space, if we are free()ing the last block in the heap (there are no holes or blocks after us), then we can contract the size of the heap. To avoid this happening constantly, in my implementation I have defined a minimum heap size, below which it will not contract.

#### 7.2.2.1. Pseudocode

1. Find the header by taking the given pointer and subtracting the sizeof(header_t).
2. Sanity checks. Assert that the header and footer's magic numbers remain in tact.
3. Set the is_hole flag in our header to 1.
4. If the thing immediately to our left is a footer:
   - Unify left. In this case, at the end of the algorithm we shouldn't add our header to the hole index (the header we are unifying with is already there!) so set a flag which the algorithm checks later.
5. If the thing immediately to our right is a header:
   - Unify right.
6. If the footer is the last in the heap ( footer_location+sizeof(footer_t) == end_address ):
   - Contract.
7. Insert the header into the hole array unless the flag described in Unify left is set.

## 7.3. Implementing an ordered list

So now we come to the implementation. As usual I'm going to try and explain the utility datatypes and functions first, and finish up with the allocation/free functions themselves.

The first datatype we need it an implementation of an ordered list. This concept will be used multiple times in your kernel (it is a common requirement) so it is probably a good idea to abstract it, so it can be used again.

### 7.3.1. ordered_array.h
```c
// ordered_array.h -- Interface for creating, inserting and deleting
// from ordered arrays.
// Written for JamesM's kernel development tutorials.

#ifndef ORDERED_ARRAY_H
#define ORDERED_ARRAY_H

#include "common.h"

/**
  This array is insertion sorted - it always remains in a sorted state (between calls).
  It can store anything that can be cast to a void* -- so a u32int, or any pointer.
**/
typedef void* type_t;
/**
  A predicate should return nonzero if the first argument is less than the second. Else
  it should return zero.
**/
typedef s8int (*lessthan_predicate_t)(type_t,type_t);
typedef struct
{
   type_t *array;
   u32int size;
   u32int max_size;
   lessthan_predicate_t less_than;
} ordered_array_t;

/**
  A standard less than predicate.
**/
s8int standard_lessthan_predicate(type_t a, type_t b);

/**
  Create an ordered array.
**/
ordered_array_t create_ordered_array(u32int max_size, lessthan_predicate_t less_than);
ordered_array_t place_ordered_array(void *addr, u32int max_size, lessthan_predicate_t less_than);

/**
  Destroy an ordered array.
**/
void destroy_ordered_array(ordered_array_t *array);

/**
  Add an item into the array.
**/
void insert_ordered_array(type_t item, ordered_array_t *array);

/**
  Lookup the item at index i.
**/
type_t lookup_ordered_array(u32int i, ordered_array_t *array);

/**
  Deletes the item at location i from the array.
**/
void remove_ordered_array(u32int i, ordered_array_t *array);

#endif // ORDERED_ARRAY_H
```
Notice that in the name of abstraction we have made the 'less than' function user-defineable. We will use this in the heap implementation to order by size and not pointer address. Note also we have two methods of defining an ordered_array. create_ordered_array will use kmalloc() to get some space. place_ordered_array will use the given start location. As we want to put our heap in a specific place (and because kmalloc isn't yet working!) we use place_ordered_array in our heap code.

### 7.3.2. ordered_map.c
```c
// ordered_array.c -- Implementation for creating, inserting and deleting
// from ordered arrays.
// Written for JamesM's kernel development tutorials.

#include "ordered_array.h"

s8int standard_lessthan_predicate(type_t a, type_t b)
{
   return (a<b)?1:0;
}

ordered_array_t create_ordered_array(u32int max_size, lessthan_predicate_t less_than)
{
   ordered_array_t to_ret;
   to_ret.array = (void*)kmalloc(max_size*sizeof(type_t));
   memset(to_ret.array, 0, max_size*sizeof(type_t));
   to_ret.size = 0;
   to_ret.max_size = max_size;
   to_ret.less_than = less_than;
   return to_ret;
}

ordered_array_t place_ordered_array(void *addr, u32int max_size, lessthan_predicate_t less_than)
{
   ordered_array_t to_ret;
   to_ret.array = (type_t*)addr;
   memset(to_ret.array, 0, max_size*sizeof(type_t));
   to_ret.size = 0;
   to_ret.max_size = max_size;
   to_ret.less_than = less_than;
   return to_ret;
}

void destroy_ordered_array(ordered_array_t *array)
{
// kfree(array->array);
}

void insert_ordered_array(type_t item, ordered_array_t *array)
{
   ASSERT(array->less_than);
   u32int iterator = 0;
   while (iterator < array->size && array->less_than(array->array[iterator], item))
       iterator++;
   if (iterator == array->size) // just add at the end of the array.
       array->array[array->size++] = item;
   else
   {
       type_t tmp = array->array[iterator];
       array->array[iterator] = item;
       while (iterator < array->size)
       {
           iterator++;
           type_t tmp2 = array->array[iterator];
           array->array[iterator] = tmp;
           tmp = tmp2;
       }
       array->size++;
   }
}

type_t lookup_ordered_array(u32int i, ordered_array_t *array)
{
   ASSERT(i < array->size);
   return array->array[i];
}

void remove_ordered_array(u32int i, ordered_array_t *array)
{
   while (i < array->size)
   {
       array->array[i] = array->array[i+1];
       i++;
   }
   array->size--;
}
```
Hopefully nothing there should surprise you. On insert the item is placed at the correct position and all larger other items shifted up one position. As always with these satellite datatypes, any implementation will work. There are better implementations of ordered arrays than this (c.f. heap-ordering, binary search trees), but I decided to go with a simple one for teaching purposes.

## 7.4. The heap itself
### 7.4.1. kheap.h
Some #defines and function prototypes are useful:
```c
#define KHEAP_START         0xC0000000
#define KHEAP_INITIAL_SIZE  0x100000
#define HEAP_INDEX_SIZE   0x20000
#define HEAP_MAGIC        0x123890AB
#define HEAP_MIN_SIZE     0x70000

/**
  Size information for a hole/block
**/
typedef struct
{
   u32int magic;   // Magic number, used for error checking and identification.
   u8int is_hole;   // 1 if this is a hole. 0 if this is a block.
   u32int size;    // size of the block, including the end footer.
} header_t;

typedef struct
{
   u32int magic;     // Magic number, same as in header_t.
   header_t *header; // Pointer to the block header.
} footer_t;

typedef struct
{
   ordered_array_t index;
   u32int start_address; // The start of our allocated space.
   u32int end_address;   // The end of our allocated space. May be expanded up to max_address.
   u32int max_address;   // The maximum address the heap can be expanded to.
   u8int supervisor;     // Should extra pages requested by us be mapped as supervisor-only?
   u8int readonly;       // Should extra pages requested by us be mapped as read-only?
} heap_t;

/**
  Create a new heap.
**/
heap_t *create_heap(u32int start, u32int end, u32int max, u8int supervisor, u8int readonly);
/**
  Allocates a contiguous region of memory 'size' in size. If page_align==1, it creates that block starting
  on a page boundary.
**/
void *alloc(u32int size, u8int page_align, heap_t *heap);
/**
  Releases a block allocated with 'alloc'.
**/
void free(void *p, heap_t *heap);
```
I have decided, arbitrarily, to put the kernel heap at 0xC0000000, give it's index a size of 0x20000 bytes, and give it a minimum size of 0x70000 bytes. The header and footer structures are the same as those given at the top of the chapter. We can actually have more than one heap (in my own kernel I have a user-mode heap as well), so for ease of portability I have decided to implement the heap as a datatype itself. heap_t keeps track of the heap's index, start/end/max addresses and the modifiers to give alloc_page when requesting more memory.

### 7.4.2. kheap.c
Finding the smallest hole that will fit a certain number of bytes is a common task that gets called on every allocation. It would therefore be nice to wrap this up in a function:
```c
static s32int find_smallest_hole(u32int size, u8int page_align, heap_t *heap)
{
   // Find the smallest hole that will fit.
   u32int iterator = 0;
   while (iterator < heap->index.size)
   {
       header_t *header = (header_t *)lookup_ordered_array(iterator, &heap->index);
       // If the user has requested the memory be page-aligned
       if (page_align > 0)
       {
           // Page-align the starting point of this header.
           u32int location = (u32int)header;
           s32int offset = 0;
           if ((location+sizeof(header_t)) & 0xFFFFF000 != 0)
               offset = 0x1000 /* page size */  - (location+sizeof(header_t))%0x1000;
           s32int hole_size = (s32int)header->size - offset;
           // Can we fit now?
           if (hole_size >= (s32int)size)
               break;
       }
       else if (header->size >= size)
           break;
       iterator++;
   }
   // Why did the loop exit?
   if (iterator == heap->index.size)
       return -1; // We got to the end and didn't find anything.
   else
       return iterator;
}
```
I feel I should explain two lines:
```c
if ((location+sizeof(header_t)) & 0xFFFFF000 != 0)
  offset = 0x1000 /* page size */  - (location+sizeof(header_t))%0x1000;
```
It's important to note that when the user requests memory to be page-aligned, he is requesting the memory that he has access to to be page-aligned. That means that the header address will actually not be page-aligned. The address that we want to fall on a boundary is location + sizeof(header_t).

Creating a heap is a simple procedure. The only part worthy of note is that we set aside the first HEAP_INDEX_SIZE*sizeof(type_t) bytes as the index. The index is put there using place_ordered_array, and the effective start address is shifted forwards. That is why, when testing your kernel, you will see allocations starting at 0xC0080000 instead of the more obvious 0xC0000000. Also note that we create a custom less_than function for the index array. This is because with the standard less_than function the array would be sorted by pointer address, instead of by size.


