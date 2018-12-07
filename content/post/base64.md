---
title: Base64转CommonsMultipartFile 
categories: ["java"]
tags: ["java","Base64"]
date: 2018-08-27 23:01:41 
author: gravel
---

在做一个上传组件的时候，需要把前端传过来的 `Base64` 的字符串转为 `CommonsMultipartFile`，然后解析保存。

<!--more-->

这里我直接使用的 `apache` 的 `Base64` 类进行转码：

```
 public static byte[] base64ToData(String base64) {
        return Base64.decodeBase64(base64.substring("data:image/png;base64,".length()));
    }
```
这里需要注意的是，解码之前需要将 `data` 的格式说明截去。

然后将 `Byte` 数组转为 `InputStream`：
```
InputStream in = new ByteArrayInputStream(is);
```
然后将 `InputStream` 的数据，复制到临时的 `temp` 文件。

```
File temp = new File(.....);
FileUtils.copyInputStreamToFile(in, temp);
```
生成 `FileItem` ：```FileItem fileitem = createFileItem(file.getName());```

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
`CommonsMultipartFile` 的构造函数可以直接通过 `FileItem` 生成 `CommonsMultipartFile`：


```
 CommonsMultipartFile multipartFile = new CommonsMultipartFile(fileitem);
```

这样，就可以了。