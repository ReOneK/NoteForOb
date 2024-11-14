---
tags:
  - linux
  - pagecache
  - 内存
---

```
cat /proc/meminfo
```
![[pagecache-meminfo.png]]

```
Buffers + Cached + SwapCached = Active(file) + Inactive(file) + Shmem + SwapCached
```
	两边都是page cache的组成，左边的Buffers更偏向于内核，右边为更具体的

### `Buffers`与`Cached`的区别

- **Buffers**：主要用于块设备的缓存。`Buffers` 缓存的是用于原始块访问的数据，而不是文件系统本身的数据。它代表的是==文件系统元数据==（例如：超级块、目录项等）和==直接块设备操作数据。==
    
- **Cached**：主要用于文件系统的缓存。`Cached` 缓存的是文件系统中的文件数据和目录内容。它缓存了文件的内容，从而加速文件的读取和写入操作。

### Active(file)+Inactive(file)

- **Active(file)**: 活跃文件页缓存，表示最近被访问过的文件页缓存。这些缓存页是“热的”，意味着它们频繁被使用，因此内存管理系统更倾向于把这些数据保存在内存中，以便快速访问。
    
- **Inactive(file)**: 不活跃文件页缓存，表示较长时间未被访问的文件页缓存。这些缓存页是“冷的”，意味着它们使用频率较低。当系统内存压力较大时，这些缓存页更有可能被释放或换出到交换空间。
  
- Active(file) + Inactive(file)代表所有用于缓存文件内容的内存页面总和，不包含共享内存shmem以及Buffers（缓存块设备的io）
  