---
title: java案例：网络编程之实现文件上传
date: 2020-05-14 22:25:51
tags: java
---

# 文件上传分析图解

1. 【客户端】输入流，从硬盘读取文件数据到程序中。
2. 【客户端】输出流，写出文件数据到服务端。
3. 【服务端】输入流，读取文件数据到服务端程序。
4. 【服务端】输出流，写出文件数据到服务器硬盘中。


![image.png](https://halo-1257208482.image.myqcloud.com/image_1589465952019.png!webp)

# 基本实现
**服务端实现**
```java
public class FileUploadServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8888);
        File file = new File(".\\upload");
        if (!file.exists()) {
            file.mkdir();
        }
        System.out.println("服务已启动。。");
        while (true) {
            Socket socket = serverSocket.accept();
            new Thread(){
                @Override
                public void run() {
                    super.run();
                    InputStream is = null;
                    FileOutputStream fos = null;
                    try {
                        is = socket.getInputStream();
                        String fileName = System.currentTimeMillis() + ".jpg";
                        fos = new FileOutputStream(file + File.separator + fileName);
                        int len=0;
                        byte[] bytes = new byte[1024];

                        while ((len = is.read(bytes)) != -1) {
                            fos.write(bytes, 0, len);
                            if (len < 1024) {
                                break;
                            }
                        }
                        socket.getOutputStream().write("上传成功！".getBytes());
                        fos.close();
                        socket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }.start();
        }
//        serverSocket.close();
    }
}
```
**客户端**
```java
public class FileUploadClient {
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream(".\\1.jpg");
        Socket socket = new Socket("localhost", 8888);
        OutputStream os = socket.getOutputStream();
        int len=0;
        byte[] bytes = new byte[1024];
        while ((len=fis.read(bytes))!=-1){
            os.write(bytes, 0, len);
        }
//        socket.shutdownOutput();
        InputStream is = socket.getInputStream();
        while ((len=is.read(bytes))!=-1){
            System.out.println(new String(bytes, 0, len));
        }

        fis.close();
        socket.close();
    }
}
```