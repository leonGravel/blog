---
title: Base64转CommonsMultipartFile 
categories: ["java"]
tags: ["java","Base64"]
date: 2018-08-27 23:01:41 
author: gravel
---

今天在项目中遇到一个问题，需要把Base64的字符串转为CommonsMultipartFile。
首先需要对Base64的字符串进行解码

<!--more-->

```
 public static byte[] base64ToData(String base64) {
        return Base64.decodeBase64(base64.substring("data:image/png;base64,".length()));
    }
```
这里需要注意的是，解码之前需要将data的格式说明截去。

然后将Byte数组转为InputStream：
```
InputStream in = new ByteArrayInputStream(is);
```
然后将InputStream的数据，复制到temp文件。

```
File temp = new File(.....);
FileUtils.copyInputStreamToFile(in, temp);
```
生成FileItem：```FileItem fileitem = createFileItem(file.getName());```

```
 private FileItem createFileItem(String filePath)  
    {  
        FileItemFactory factory = new DiskFileItemFactory(16, null);  
        FileItem item = factory.createItem("file", "image/png", true, filePath);  
        File newfile = new File(filePath);  
        int bytesRead = 0;  
        byte[] buffer = new byte[8192];  
        try  
        (  
            FileInputStream fis = new FileInputStream(newfile);  
            OutputStream os = item.getOutputStream();){
            while ((bytesRead = fis.read(buffer, 0, 8192))  
                != -1)  
            {  
                os.write(buffer, 0, bytesRead);  
            }  
        }  
        catch (IOException e)  
        {  
            e.printStackTrace();  
        }  
        return item;  
    }
```
CommonsMultipartFile的构造函数可以直接通过FileItem生成CommonsMultipartFile：


```
 CommonsMultipartFile multipartFile = new CommonsMultipartFile(fileitem);
```

这样，就可以了。