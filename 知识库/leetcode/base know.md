
#### 排序
	sort.Ints(nums)

#### 哈希表
1. 可以避免重复遍历
2. key一般是需要比较的值
3. value 一般是索引

#### 链表
1. 哨兵：dummy:=&ListNode{-1,nil}

#### 整数的一些基本用法
- 字符串转整数
	- `int(x - '0')`
- 整除
	- x/num
		-用来判断两个数相加是否超过十，需要进1
- 取余数
	- x%num
- 逆转数字
	- `nums=123
	- `res=0`
	- `res=res*10+nums%10`
	- `nums=nums/10`

#### 滑动窗口
1. 边界判断
	1. 如果使用hashmap，left指针务必小于map里已存的right指针
		`index,ok:=map[s[r]];ok && index>=left`

#### 回文子串
1. 回文串判断
	1. 中心扩展法
	2. func expand(s string,l,r int) int
	3. l,r为中心点
		1. 奇数中心点：expand(s,i,i)