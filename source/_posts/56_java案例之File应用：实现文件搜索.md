---
title: java案例之File应用：实现文件搜索
tags: java
abbrlink: 4e419ad0
date: 2020-05-12 15:40:35
---

> 搜索指定目录中的`.java`文件
# 分析
1. 目录搜索，无法判断多少级目录，所以使用递归，遍历所有目录。
2. 遍历目录时，获取的子文件，通过文件名称，判断是否符合条件。

# 实现
```java
public class Main {
    public static void main(String[] args) {
        File file = new File("E:\\idea");
        getAllFile(file);
    }

    public static void getAllFile(File file) {
        if (file.isDirectory()) {
            // 使用匿名内部类实现
//            File[] files = file.listFiles(new FilenameFilter() {
//                @Override
//                public boolean accept(File dir, String name) {
//                    return new File(dir,name).isDirectory() || name.endsWith(".java");
//                }
//            });
            // 使用Lambda表达式实现
            File[] files = file.listFiles((dir, name) -> new File(dir,name).isDirectory() || name.endsWith(".java"));
            for (File file1 : files) {
                getAllFile(file1);
            }
        } else {
            System.out.println(file);
        }
    }
}
```
