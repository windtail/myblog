---
title: "gcc 的宏替换 __stringify"
date: 2011-04-13T21:58:00+08:00
draft: false
---

 


  每次看到类似如下的代码，总是不解，为什么要分两级定义呢，分明可以直接将\_\_stringify定义为 #x嘛。但大家都这么做，理论上是不会有问题的啊。今天忍不住搜了下，看来还是我没往深入地想啊，唉，鄙视自己！


 


 #define \_\_stringify\_1(x...) #x   

 #define \_\_stringify(x...) \_\_stringify\_1(x) 




 


 


 假设：直接 

#define \_\_stringify(x) #x 



 


  且 #define AA bb



  那么 \_\_stringify(AA) 展开为 "AA"



 


  而像最上面的定义方法，\_\_stringify(AA) 将展开为 "bb"



 


 


  不过，最后我还想问一句，gcc支持嵌套多少层宏定义，无限？


 


 


 


  参考：<http://hi.baidu.com/%B1%B1%B7%E7%B1%B1%B5%C4%D6%ED/blog/item/a2d8e031942aa20591ef39ae.html>
  





