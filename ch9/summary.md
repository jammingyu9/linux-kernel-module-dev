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
