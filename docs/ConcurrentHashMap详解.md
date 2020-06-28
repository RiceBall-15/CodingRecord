java7中的ConcurrnetHashMap 的初始化逻辑

- 必要的参数校验
- 校验并发级别 concurrencyLevel 大小，如果大于最大值，重置为最大值。无参构造默认值是16；
- 寻找并发级别 concurrencyLevel  之上最近的 2的幂次方值，作为初始化容量的大小，默认是16；
- 记录 segmentShift 偏移量，这个值为【容量 = 2的N次方】中的N，在后面Put时计算位置时会用到。默认是32 - sshift = 28；
- 记录segmentMask，默认是ssize-1 = 16 - 1 = 15；
- 初始化 segments[0]，默认大小为 2，负载因子 0.75，扩容阀值是 2*0.75=1.5，插入第二个值时才会进行扩容。