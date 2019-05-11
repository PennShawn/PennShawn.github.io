---
title: 《Effective Java》Item 9. Perfer try-with-resources to try-finally
categories :
- 技术
tags :
- Java
- Effective Java
---

Always use try-with-resources in preference to try-finally when working with resources that must be closed.

```
try(InputStream in =new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)){

    byte[] buf = new byte[BUFFER_SIZE]
    ...    

}
```




