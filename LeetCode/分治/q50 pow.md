# pow 求给定数字的多少次方

```
/**
 * 实现 pow(x, n) ，即计算 x 的 n 次幂函数。
 *
 * 示例 1:
 *
 * 输入: 2.00000, 10
 * 输出: 1024.00000
 * 示例 2:
 *
 * 输入: 2.10000, 3
 * 输出: 9.26100
 * 示例 3:
 *
 * 输入: 2.00000, -2
 * 输出: 0.25000
 * 解释: 2-2 = 1/22 = 1/4 = 0.25
 *
 * 来源：力扣（LeetCode）
 * 链接：https://leetcode-cn.com/problems/powx-n
 * 著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
 */
```



求x 的 n次方

可以分为求 x的n/2次方 * x的n/2次方 

```java
public class Solution {

    public static void main(String[] args) {
        int x =2;
        int n = 3;
        System.out.println(myPow(x,n));
    }
    // 分治思想
    public static double quickMul(double x, long N) {
        if (N == 0) {
            return 1.0;
        }
        // 除2 一直得到 0
        double y = quickMul(x, N / 2);
        return N % 2 == 0 ? y * y : y * y * x;
    }

    public static double myPow(double x, int n) {
        long N = n;
        return N >= 0 ? quickMul(x, N) : 1.0 / quickMul(x, -N);
    }
}
```





位运算法

```java
public static double myPow2(double x,int n){
        if(x == 0.0f) return 0.0d;
        long b = n;
        double res = 1.0;
        if(b < 0) {
            x = 1 / x;
            b = -b;
        }
        while(b > 0) {
          	// b & 1 判断是否奇数
            if((b & 1) == 1) res *= x;
            x *= x;
          	// 右移一位，相当于 除2
            b >>= 1;
        }
        return res;
    }
```

