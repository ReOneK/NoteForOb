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
	两边都是page cache的组成，