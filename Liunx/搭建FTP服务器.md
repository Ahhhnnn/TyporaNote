# Liunx搭建FTP服务器





# Windows搭建FTP服务器

## 启动或关闭Windows功能

![image-20200425143220765](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200425143220765.png)

## 启用FTP服务器功能

![image-20200425143319746](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200425143319746.png)

## 选中IIS服务——》网站——》添加FTP站点

回到我的桌面，右键我的计算机，选择管理

![image-20200425143525724](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200425143525724.png)

右键网站，选择添加FTP站点

![image-20200425143630835](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200425143630835.png)

完成后即可访问

在浏览器中输入 ftp:ip 即可

![image-20200425145604683](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200425145604683.png)

## Java操作FTP服务器

### 引入依赖

```xml
<!--引入FTP 依赖-->
        <dependency>
            <groupId>commons-net</groupId>
            <artifactId>commons-net</artifactId>
            <version>3.6</version>
        </dependency>
```

### 初始化FTP客户端

```java
/**
     * @Description 初始化FTP客户端
     * @Author heningm
     * @Date 12:34 2020/2/5
     * @Param []
     * @return boolean
     **/
    public static boolean initFtpClient(){
        ftp = new FTPClient();
        int reply;
        try {
            // 连接FTP服务器
            ftp.connect(HOST, PORT);
            //登录, 如果采用默认端口，可以使用ftp.connect(host)的方式直接连接FTP服务器
            ftp.login(USERNAME, PASSWORD);
            ftp.setBufferSize(10240);
            //设置传输超时时间为60秒
            ftp.setDataTimeout(600000);
            //连接超时为60秒
            ftp.setConnectTimeout(600000);
            //FTP以二进制形式传输
            ftp.setFileType(FTP.BINARY_FILE_TYPE);
            //返回ftp服务器上一次返回的响应码
            reply = ftp.getReplyCode();
            //如果响应码不在（reply >= 200 && reply < 300）之间，则说明错误，关闭连接
            if (!FTPReply.isPositiveCompletion(reply)) {
                ftp.disconnect();
                return false;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return true;
    }
```

### 上传文件

```java
 /**
     * Description: 向FTP服务器上传文件
     * @param filePath FTP服务器文件存放路径。例如分日期存放：/2015/01/01。文件的路径为basePath+filePath
     * @param filename 上传到FTP服务器上的文件名
     * @param input 本地要上传的文件的 输入流
     * @return 成功返回true，否则返回false
     */
    public static boolean uploadFile(String filePath, String filename, InputStream input) {
        boolean result = false;
        try {
            filePath = new String(filePath.getBytes("GBK"),"iso-8859-1");
            filename = new String(filename.getBytes("GBK"),"iso-8859-1");
             if (!initFtpClient()){
                 return result;
             };
            //切换到上传目录
            ftp.enterLocalPassiveMode();
            //切换到指定路径，如果失败，说明目录不存在
            if (!ftp.changeWorkingDirectory(BASEPATH+filePath)) {
                //如果目录不存在创建目录
                String[] dirs = filePath.split("/");
                String tempPath = BASEPATH;
                for (String dir : dirs) {
                    if (null == dir || "".equals(dir)){
                        continue;
                    }
                    tempPath += "/" + dir;
                    if (!ftp.changeWorkingDirectory(tempPath)) {
                        if (!ftp.makeDirectory(tempPath)) {
                            return result;
                        } else {
                            ftp.changeWorkingDirectory(tempPath);
                        }
                    }
                }
            }
            //设置上传文件的类型为二进制类型
            ftp.setFileType(FTP.BINARY_FILE_TYPE);
            //上传文件
            ftp.enterLocalPassiveMode();
            if (!ftp.storeFile(filename, input)) {
                return result;
            }
            input.close();
            ftp.logout();
            result = true;
        }
        catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (ftp.isConnected()) {
                try {
                    ftp.disconnect();
                } catch (IOException ioe) {
                }
            }
        }
        return result;
    }
```

### 下载文件

```java
/**
     * @Description 从ftp服务器下载文件到指定输出流
     * @Author heningm
     * @Date 22:30 2020/3/5
     * @Param [remotePath, fileName, outputStream]
     * @return boolean
     **/
    public static boolean downloadFile(String remotePath, String fileName, OutputStream outputStream) {
        boolean result = false;
        try {
            remotePath = new String(remotePath.getBytes("GBK"),"iso-8859-1");
            fileName = new String(fileName.getBytes("GBK"),"iso-8859-1");
            if (!initFtpClient()){
                return result;
            };
            // 转移到FTP服务器目录
            ftp.changeWorkingDirectory(remotePath);
            ftp.enterLocalPassiveMode();
            //检索所有的文件
            FTPFile[] fs = ftp.listFiles();
            for (FTPFile ff : fs) {
                if (ff.getName().equals(fileName)) {
                    ftp.enterLocalPassiveMode();
                    //将指定目录的指定文件写入到指定的输出流
                    ftp.retrieveFile(remotePath+"/"+fileName,outputStream);
                    result = true;
                }
            }
            ftp.logout();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (ftp.isConnected()) {
                try {
                    ftp.disconnect();
                } catch (IOException ioe) {
                }
            }
        }
        return result;
    }
```

### 删除文件

```java
/**
 * @Description 删除文件
 * @Author heningm
 * @Date 11:38 2020/2/6
 * @Param [remotePath, fileName]
 * @return void
 **/
public static boolean deleteFile( String remotePath,String fileName){
    boolean flag = false;
    try {
        remotePath = new String(remotePath.getBytes("GBK"),"iso-8859-1");
        fileName = new String(fileName.getBytes("GBK"),"iso-8859-1");
        if (!initFtpClient()){
            return flag;
        };
        // 转移到FTP服务器目录
        ftp.changeWorkingDirectory(remotePath);
        ftp.enterLocalPassiveMode();
        FTPFile[] fs = ftp.listFiles();
        for (FTPFile ff : fs) {
            if ("".equals(fileName)){
                return flag;
            }
            if (ff.getName().equals(fileName)){
                String filePath = remotePath + "/" +fileName;
                ftp.deleteFile(filePath);
                flag = true;
            }
        }
        ftp.logout();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (ftp.isConnected()) {
            try {
                ftp.disconnect();
            } catch (IOException ioe) {
            }
        }
    }
    return flag;
}
```

### 修改文件

```java
/**
 * @Description 修改文件名称或者文件夹名
 * @Author heningm
 * @Date 21:18 2020/2/11
 * @Param [oldAllName, newAllName] 这里传入了文件的全路径，（存储文件时存储了路径）
 * @return boolean
 **/
public static boolean reNameFile( String oldAllName,String newAllName){
    boolean flag = false;
    try {
        oldAllName = new String(oldAllName.getBytes("GBK"),"iso-8859-1");
        newAllName = new String(newAllName.getBytes("GBK"),"iso-8859-1");
        if (!initFtpClient()){
            return flag;
        };
        ftp.enterLocalPassiveMode();
        //参数中包含了文件路径，此处不需要切换路径
        ftp.rename(oldAllName,newAllName);
        flag = true;
        ftp.logout();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (ftp.isConnected()) {
            try {
                ftp.disconnect();
            } catch (IOException ioe) {
            }
        }
    }
    return flag;
}
```