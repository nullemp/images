# pg_dum dev相关问题

> 1.头文件

![image-20220902143541996](C:\Users\HIGHGO\AppData\Roaming\Typora\typora-user-images\image-20220902143541996.png)

```makefile
#在Makefile中指定头文件的路径
PG_CPPFLAGS = -I$(libpq_srcdir)
SHLIB_LINK_INTERNAL = $(libpq)
```

#include<stdbool.h>

