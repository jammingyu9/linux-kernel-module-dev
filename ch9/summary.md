# Creating a custom slab cache
* Linux Kernel allows us to create our own custom slab caches
  1. Create a custom slab cache of a given size: *kmem_cache_create()*
  2. Issue the *kmem_cache_alloc()* API to allocate a single instance of the custom objects within your slab cache
  3. Use the object
  4. Free it back to the cache with the *kmem_cache_free()* API
  5. Destroy the custom slab cache when done with kmem_cache_destroy()
* usually the allocated mem size is larger than the requested size

# Understanding slab shrinkers
* By registering shrinker interface, when memory pressure is high, the kernel might invoke several slab shrinker callbacks, which are expected to ease the memory pressure by freeing up slab objects
* The API to register a shrinker function with the kernel is the *register_shrinker()* API

# The slab allocator summary
## pros
* fast (pre-cached memory objects)
* A physically contiguous memory chunk is guaranteed
* Hardware (CPU) cacheline-aligned memory is guaranteed when the SLAB_HWCACHE_ALIGN flag is used when creating the cache
* You can create your own custom slab cache for particular objects
## cons
* A limited amount of memory can be allocated at a time
* Using the APIs incorrectly can lead to internal fragmentation (wastage). It's designed to only really optimize for the common case - for allocations of a size less than one page

# Debugging at the slab layer
* SLUB (the unqueued allocator) implementation of the slab layer is the default on most Linux installations
* If CONFIG_SLUB_DEBUG flag is on, kernel injects special values to memory when (when init, free or at the end of address)
* Kernel may trigger a report when those values are changed before allocation or after free (prevent UAF)

# Understanding and using the kernel vmalloc() API
* vmalloc region is another completely virtual address space within the kernel's address space from where virtual pages can be allocated at will
* Of course, ultimately, once a virtual page is actually used - it's physical page frame that it's mapped to is really allocated via the page allocator
* You can allocate virtual memory from the kernel's vmalloc region using the *vmalloc()* API
  * The vmalloc() API allocates contiguous virtual memory to the caller (no guarantee that it will be physically contiguous)
  * The vmalloc() APIs must only ever be invoked from a process context
  * The actual allocated memory might well be larger than what's requested
* **when you require a larger virtually contiguous buffer of a size greater than the slab APIs can provide, then you should use vmalloc!**
* analogous *vzalloc(), vfree()* APIs provided

# A brief note on memory allocations and demand paging
* when using vmalloc, it only causes virtual memory pages to be allocated
* vmalloc-ed virtual memory has to, at some point, become physical memory
* Physical memory is allocated via the one and only way that it can be in the kernel - via the page allocator
* The actual physical page frames corresponding to the virtual ones only get allocated when virtual pages are touched in any manner, such as for reads, writes, or executions (**demand paging!**)
* vmalloc() and friends, and indeed, for the user space glibc malloc() family of routines, work by demand paging
* When vmalloc/malloc() returns success, all that has really happened so far is that a virtual memory region has been reserved; no physical memory has actually been allocated yet!
* **The actual allocation of a physical page frame only happens on a per-page basis as and when the virtual page is accessed (read/write/exec)

## MMU, TLB and Page fault handler
* The virtual address is interpreted by the Memory Management Unit(MMU), which is a part of the silicon on the CPU core
* The MMU's Translation Lookaside Buffer (TLB) will now be check for a hit
* If so, the memory translation is already available; if not, we have a TLB-miss. If so, the MMU will now walk the paging tables of the process, effectively translating the virtual address and thus obtaining the physical address. It puts this on the address bus, and the CPU goes on its merry way
* **If the MMU cannot find a matching physical address, because like we don't yet have a physical page frame, only a virtual page then it *invokes the OS's page fault handler code*. This page fault handler actually resolves the situation; in our case, with vmalloc(), it requests the page allocator for a single physical page frame (at order 0) and maps it to the virtual page**
* **This is not the case for kernel memory allocations carried out via the buddy system and the slab allocator, because physical page frames are allocated immediately**

# Friends of vmalloc()
* A pattern of usage that emerged in a lot of in-kernel code paths went something like the following pseudocode:
```
kptr = kmalloc(n);
if (!kptr) {
    kptr = vmalloc(n);
    if (unlikely(!kptr))
        <... failed, cleanup ...>
}
<ok, continue with kptr>
```
* The cleaner alternative to this kind of code is the kvmalloc() API
* Further, you can use the kvmalloc_array() API to allocate virtual contiguous memory for an array of items. Its implementation is shown as follows:
```
static inline void *kvmalloc_array(size_t n, size_t size, gfp_t flags)
{
    size_t bytes;
    if (unlikely(check_mul_overflow(n, size, &bytes))) # check this line for IoF!
        return NULL;
    return kvmalloc(bytes, flags);
}
```
* For NUMA awareness, check *kvmalloc_node()*
