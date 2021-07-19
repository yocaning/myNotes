问题1：如何用rand(7)实现rand(10)
首先明白拒绝采样的方法：使用rand(7)实现rand(5)，只需要在生成6和7时拒绝后重新生成即可实现rand(5)
而实现rand(10)需要扩容，有几种扩容方法：
第一种直接rand(7)*2%10，但这是伪随机本质还是rand(7)
第二种(rand(7)+rand(7))%10，但7有3+4和4+3等多种组合方式，不是等概率的
答案是第三种(rand(7)-1)*7+rand(7)生成[1,49]之间的随机数，然后拒绝40-49，实现1-40的等概率随机均为40分之一，然后求余数加1答案为(((rand(7)-1)*7+rand(7))-1)%10+1
思想来源地址：https://leetcode-cn.com/problems/implement-rand10-using-rand7/solution/xiang-xi-si-lu-ji-you-hua-si-lu-fen-xi-zhu-xing-ji/