# 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。



```java
package com.he.堆栈and队列.包含min函数的栈;

import java.util.Stack;

public class MinStack {
    Stack<Integer> A,B;
    /** initialize your data structure here. */
    public MinStack() {
        A = new Stack<>();
        B = new Stack<>();
    }

    public void push(int x) {
        A.add(x);
        if(B.isEmpty() || B.peek() >= x){
            B.add(x);
        }
    }

    public void pop() {
        if( A.pop().equals(B.peek())){
            B.pop();
        }
    }

    public int top() {
        return A.peek();
    }

    public int min() {
        return B.peek();
    }

    public static void main(String[] args) {
        MinStack minStack = new MinStack();
        minStack.push(-3);
        minStack.push(2);
        minStack.push(0);
        minStack.push(-4);
        System.out.println(minStack.min());
        System.out.println(minStack.top());
        minStack.pop();
        System.out.println(minStack.min());
        System.out.println(minStack.top());
    }
}

```

