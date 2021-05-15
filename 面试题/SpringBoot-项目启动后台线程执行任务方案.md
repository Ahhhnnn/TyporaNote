## SpringBoot项目，启动项目后后台启动线程执行任务

场景：项目启动后需要一个后台线程去执行某个任务，可以是轮训的定时任务， 可以是一个监听器等等。

实现方式：

使用Spring的控制反转，将Bean放入IOC容器，在类的无参构造器中调用方法执行任务。

```java
package com.baidu.he.config;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * @author hening
 */
@Component
public class EventSubscriber implements DisposableBean, Runnable {

    // 定时任务线程池
    private ScheduledExecutorService executorService;

    public EventSubscriber(){
        doInsert();
    }
    @Override
    public void run() {
        SimpleDateFormat smf = new SimpleDateFormat("HH:mm:ss");
        try {
            System.out.println("模拟执行任务,开始时间："+ smf.format(new Date()));
            Thread.sleep(2000);
            System.out.println("任务结束,结束时间："+ smf.format(new Date()));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void doInsert() {
        executorService = Executors.newSingleThreadScheduledExecutor();
        SimpleDateFormat smf = new SimpleDateFormat("HH:mm:ss");
        System.out.println("开启线程池："+ smf.format(new Date()));
        executorService.scheduleAtFixedRate(this, 1000, 3000, TimeUnit.MILLISECONDS);
    }

    @Override
    public void destroy() throws Exception {

    }
}
```

