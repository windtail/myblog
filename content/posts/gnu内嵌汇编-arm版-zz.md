---
title: "GNU内嵌汇编 ARM版 （ZZ）"
date: 2011-04-13T21:35:00+08:00
draft: false
---

本文转载自 <http://wenku.baidu.com/view/b7bb0e116c175f0e7cd13733.html>
                     

  

一、格式  

    asm volatile (“asm code”：output：input：changed);    //必须以‘;’结尾，不管有多长对C都只是一条语句  

      

    asm                 内嵌汇编关键字  

      

    volatile            告诉编译器不要优化内嵌汇编，如果想优化可以不加  

      

    ANSI C规范的关键字：    （ANSI C把asm用于其它用途，不能用于内嵌汇编语句，GCC可以）  

        \_\_asm\_\_   

        \_\_volatile\_\_        //前面和后面都有两个下划线，它们之间没有空格  

          

    如果后面部分没有内容，‘：’可以省略，前面或中间的不能省略‘：’  

    没有asm code也不可以省略‘“”’，没有changed必须省略‘：’  

      

二、asm code  

      

    asm code必须放在一个字符串内，但是字符串中间是不能直接按回车键换行的  

    可以写成多个字符串，只要字符串之间不加任何符号编译完后就会变成一个字符串  

           

         "mov r0,r0/n/t"        //指令之间必须要换行，/t可以不加，只是为了在汇编文件中的指令格式对齐  

        "mov r1,r1/n/t"  

        "mov r2,r2"  

      

    字符串内不是只能放指令，可以放一些标签、变量、循环、宏等等  

      

    还可以把内嵌汇编放在C函数外面，用内嵌汇编定义函数、变量、段等汇编有的东东，总之就跟直接在写汇编文件一样  

      

    在C函数外面定义内嵌汇编时不能加volatile：output：input：changed  

      

    注意：编译器不检查asm code的内容是否合法，直接交给汇编器  

      

三、output（ASM --> C）和input（C --> ASM）  

    1、    指定输出值  

            \_\_asm\_\_ \_\_volatile\_\_ (  

                    "asm code"  

                    :“constraint”（variable）  

            );  

      

            constraint定义variable的存放位置：  

                                r            使用任何可用的通用寄存器  

                                m            使用变量的内存地址  

  

            output修饰符：  

                                +            可读可写  

                                =            只写  

                                &            该输出操作数不能使用输入部分使用过的寄存器，只能 +& 或 =& 方式使用  

                                  

    2、    指定输入值  

            \_\_asm\_\_ \_\_volatile\_\_ (  

                    "asm code"  

                    :  

                    :“constraint”（variable / immediate）  

            );  

      

            constraint定义variable / immediate的存放位置：  

                                r            使用任何可用的通用寄存器（变量和立即数都可以）  

                                m            使用变量的内存地址（不能用立即数）  

                                i             使用立即数（不能用变量）  

                      

    3、    使用占位符  

            int a = 100,b = 200;  

            int result;  

            \_\_asm\_\_ \_\_volatile\_\_ (  

                    “mov    %0,%3/n/t”        //mov     r3,#123                %0代表result，%3代表123（编译器会自动加 # 号）  

                    “ldr    r0,%1/n/t”        //ldr     r0,[fp, #-12]        %1代表 a 的地址  

                    “ldr    r1,%2/n/t”        //ldr     r1,[fp, #-16]        %2代表 b 的地址  

                    “str    r0,%2/n/t”        //str     r0,[fp, #-16]        因为%1和%2是地址所以只能用ldr或str指令  

                    “str    r1,%1/n/t”        //str     r1,[fp, #-12]        如果用错指令编译时不会报错，要到汇编时才会  

                    ：“=r”(result),“+m”(a),“+m”(b)                            out1是%0，out2是%1，...，outN是%N-1  

                    ：“i”(123)                                                    in1是%N，in2是%N+1，...  

            );  

      

    4、引用占位符  

            int num = 100;  

          \_\_asm\_\_ \_\_volatile\_\_ (  

                "add    %0,%1,#100/n/t"  

                : "=r"(a)  

                : "0"(a)                        //"0"是零，即%0，引用时不可以加 %，只能input引用output,  

                );                                            //引用是为了更能分清输出输入部分  

      

    5、    & 修饰符  

            int num;  

          \_\_asm\_\_ \_\_volatile\_\_ (                //mov     r3, #123            //编译器自动加的指令  

                "mov    %0,%1/n/t"            //mov     r3,r3                //输入和输出使用相同的寄存器  

                : "=r"(num)  

                : "r"(123)  

             );  

               

             int num;  

          \_\_asm\_\_ \_\_volatile\_\_ (                //mov     r3, #123  

                "mov    %0,%1/n/t"            //mov     r2,r3                //加了&后输入和输出的寄存器不一样了  

                : "=&r"(num)                    //mov     r3, r2                //编译器自动加的指令  

                : "r"(123)  

             );  

      

四、changed  

      

    告诉编译器你修改过的寄存器，编译器会自动把保存这些寄存器值的指令加在内嵌汇编之前，再把恢复寄存器值的指令加在内嵌汇编之后  

      

        1    void test()                                                        test:  

        2    {                                                                            str     fp, [sp, #-4]!  

        3        \_\_asm\_\_ \_\_volatile\_\_ (                                        add     fp, sp, #0  

       4             "mov    r4,#123"                                        mov     r4,#123  

        5          );                                                                    add     sp, fp, #0  

        6    }                                                                            ldmfd   sp!, {fp}  

        7                                                                                bx      lr  

        8                                                                                  

        9    void test()                                                        test:  

        10    {                                                                            stmfd   sp!, {r4, fp}  

        11        \_\_asm\_\_ \_\_volatile\_\_ (                                        add     fp, sp, #0  

       12             "mov    r4,#123"                                        mov     r4,#123  

        13                :                                                            add     sp, fp, #0  

        14                  :                                                            ldmfd   sp!, {r4, fp}  

        15                  :"r4"                                                        bx            lr  

        16            );  

        17    }  

                              

    汇编的第 2 行与第 6 行没有保存和恢复 R4（R4是通用寄存器变量必须保护，见APCS），第 10 行与第 14 行有保存和恢复 R4  

      

    如果修改了没有在输入或输出中定义的任何内存位置，必须在changed列表里加上“memory”  



