# SQL 编程题

原表:
courseid coursename score
\-------------------------------------
1 [Java ](http://lib.csdn.net/base/java)70
2 oracle 90
3 xml 40
4 jsp 30
5 servlet 80
\-------------------------------------
为了便于阅读, 查询此表后的结果显式如下( 及格分数为60):
courseid coursename score mark
\---------------------------------------------------
1 [Java](http://lib.csdn.net/base/javase) 70 pass
2 oracle 90 pass
3 xml 40 fail
4 jsp 30 fail
5 servlet 80 pass
\---------------------------------------------------
写出此查询语句



select courseid, coursename ,score ,decode（sign(score-60),-1,'fail','pass') as mark from course

**decode可以比较大小**

**sign()函数根据某个值是0、正数还是负数，分别返回0、1、-1**





## 50道sql练习题

https://zhuanlan.zhihu.com/p/38354000

## mysql分组取topN问题

https://mp.weixin.qq.com/s/MuxjlFV0gi1GydOrYfiSeQ



## mysql 行列互换问题

https://mp.weixin.qq.com/s/6Kll4Q6Xp37i2PiLUh4cMA