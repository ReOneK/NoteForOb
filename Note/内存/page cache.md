---
tags:
  - linux
  - pagecache
  - 内存
---
## 定义
```
cat /proc/meminfo
```
![[pagecache-meminfo.png]]

```
Buffers + Cached + SwapCached = Active(file) + Inactive(file) + Shmem + SwapCached
```
	两边都是page cache的组成，左边的Buffers更偏向于内核，右边为更具体的

### `Buffers`与`Cached`的区别

- **Buffers**：主要用于块设备的缓存。`Buffers` 缓存的是用于原始块访问的数据，而不是文件系统本身的数据。它代表的是文件系统元数据（例如：超级块、目录项等）和直接块设备操作数据。
    
- **Cached**：主要用于文件系统的缓存。`Cached` 缓存的是文件系统中的文件数据和目录内容。它缓存了文件的内容，从而加速文件的读取和写入操作。