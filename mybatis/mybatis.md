## 

![image-20200401202755930](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401202755930.png)



## Mybatis 执行流程

![image-20200401203729097](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401203729097.png)



![image-20200401213505593](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401213505593.png)

## 自定义Mybatis插件

![image-20200401210650540](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401210650540.png)



在intercep方法中需要实现三个核心业务

![image-20200401210905009](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401210905009.png)

在plugin方法中，将自己写的插件加入mybatis

![image-20200401210926056](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401210926056.png)

在自定义插件上 使用@Intercepts注解，告诉mybatis在执行到StatementHandler时就执行我们自己的插件（相当于在哪个位置进行拦截）

![image-20200401211207928](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401211207928.png)

拿到原始sql

![image-20200401211536455](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401211536455.png)



增加分页参数

![image-20200401211742833](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401211742833.png)

对需要分页的方法对象拦截，并操作

![image-20200401212119151](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401212119151.png)

执行查询总数的方法

![image-20200401212135623](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401212135623.png)



根据原始sql，生成一个带limit的sql，进行分页

![image-20200401212621828](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401212621828.png)

![image-20200401212725288](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401212725288.png)

将替换后sql放回mybatis 上下文

![image-20200401212837206](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200401212837206.png)

