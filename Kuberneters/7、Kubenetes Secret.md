# Kubenetes Secret.

## Secret

作用：加密数据放到etcd中，Pod通过Volume数据卷挂载进行访问



## 创建Secret

yaml文件

![image-20201212211804015](assets/image-20201212211804015.png)

username & password都为加密后的数据

![image-20201212211748946](assets/image-20201212211748946.png)



## 变量形式挂载

![image-20201212212052444](assets/image-20201212212052444.png)

进入容器查看挂载的变量名

```shell
kubectl exec -it mypod bash
```



![image-20201212212234562](assets/image-20201212212234562.png)

## Volume数据卷挂载



![image-20201212212500459](assets/image-20201212212500459.png)

使用该yaml启动pod

```
kubectl apply -f secret-vol.yaml
```



查看pods

```
kubectl get pods
```

进入pod

```
kubectl exec -it mypod bash
```

查看挂载的数据卷

![image-20201212212834615](assets/image-20201212212834615.png)