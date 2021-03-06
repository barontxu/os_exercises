# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
```
  + 采分点：说明64bit CPU架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了64bit CPU支持的物理内存大小限制（1分）
  - 正确描述了64bit CPU下的多级页表的级数和多级页表的结构或反置页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表或反置页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- [x]  

>  64位CPU支持2^64bit，即16EB，但目前用不到这么大，实际上的物理内存也没用这么大。比如64位的Linux支持46位4TB的物理地址空间。64位CPU体系结构下的页表一般是3级或4级，由于二者的原理相近，在这里只解释三级页表的结构。1级页表PTE大小为32位，虚拟地址结构为页号+页内地址；二级页表，虚拟地址组成为目录地址+页表地址+页内偏移；对32位系统而言加入物理地址扩展PAE后二级页表无法满足要求，故引入三级页表，增加PMD中间目录一级。
多级页表：cr3寄存器->PGD的首地址->页全局目录->页上级目录->页中间目录->页表->页->物理地址
在利用反置页表进行地址变换时，是用进程标志符和页号去检索反置页表；若检索完整个页表都未找到与之匹配的页表项，表明此页此时尚未调入内存，对于具有请求调页功能的存储器系统应产生请求调页中断，若无此功能则表示地址出错；如果检索到与之匹配的表项，则该表项的序号i便是该页所在的物理块号，将该块号与页内地址一起构成物理地址。

## 小组思考题
---

（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。 

- [x]  

>设x 为不存在页面访问时间，那么有效访问时间应该为 0.9 \* 150 + 0.1 \* x ,又总时间为500ns，所以有下式  
500=0.9\*150+0.1\*x
解得x为3650us

（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
Virtual Address 6c74
Virtual Address 6b22
Virtual Address 03df
Virtual Address 69dc
Virtual Address 317a
Virtual Address 4546
Virtual Address 2c03
Virtual Address 7fd7
Virtual Address 390e
Virtual Address 748b
```

比如答案可以如下表示：
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
      
Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```
Virtual Address 0x6c74
  --> pde index:0x1b pde contents:(valid 1, pfn 0x20)
    --> pte index:0x03 pte contents:(valid 1, pfn 0x61)
      --> Translates to Physical Address 0xc34 --> Value: 0x06
Virtual Address 0x6b22
  --> pde index:0x1a pde contents:(valid 1, pfn 0x52)
    --> pte index:0x19 pte contents:(valid 1, pfn 0x47)
      --> Translates to Physical Address 0x8e2 --> Value: 0x1a
Virtual Address 0x03df
  --> pde index:0x00 pde contents:(valid 1, pfn 0x5a)
    --> pte index:0x1e pte contents:(valid 1, pfn 0x05)
      --> Translates to Physical Address 0x0bf --> Value: 0x0f
Virtual Address 0x69dc
  --> pde index:0x1a pde contents:(valid 1, pfn 0x52)
    --> pte index:0x0e pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
Virtual Address 0x317a
  --> pde index:0x0c pde contents:(valid 1, pfn 0x18)
    --> pte index:0x0b pte contents:(valid 1, pfn 0x35)
      --> Translates to Physical Address 0x6ba --> Value: 0x1e
Virtual Address 0x4546
  --> pde index:0x11 pde contents:(valid 1, pfn 0x21)
    --> pte index:0x0a pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
Virtual Address 0x2c03
  --> pde index:0x0b pde contents:(valid 1, pfn 0x44)
    --> pte index:0x00 pte contents:(valid 1, pfn 0x57)
      --> Translates to Physical Address 0xae3 --> Value: 0x16
Virtual Address 0x7fd7
  --> pde index:0x1f pde contents:(valid 1, pfn 0x12)
    --> pte index:0x1e pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
Virtual Address 0x390e
  --> pde index:0x0e pde contents:(valid 0, pfn 0x7f)
    --> Fault (page directory entry not valid)
Virtual Address 0x748b
  --> pde index:0x1d pde contents:(valid 1, pfn 0x00)
    --> pte index:0x04 pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

选择第一组   Virtual Address 6c74
            Virtual Address 6b22

Virtual Address 6c74:
--> pde index:0x1b pde contents:(valid 1, pfn 0x20)
--> pte index:0x3 pte contents:(valid 1, pfn 0x61)
--> Translates to Physical Address 0xc34 --> Value: 6

Virtual Address 6b22:
--> pde index:0x1a pde contents:(valid 1, pfn 0x52)
--> pte index:0x19 pte contents:(valid 1, pfn 0x47)
--> Translates to Physical Address 0x8e2 --> Value: 1a




（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

table = """7f 7f 7f 7f 7f 7f b2 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
01 19 1d 11 05 1a 1e 15 0f 0b 0a 06 14 0d 06 03 14 09 0f 0a 1b 1b 00 06 0a 1e 18 0a 06 14 19 1c 
ba 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 9f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f a4 7f 7f 7f 7f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 9c 
1e 12 0c 05 0f 1e 17 10 1a 07 0f 1d 11 0e 08 10 1d 00 18 19 1b 16 19 10 11 0d 01 1a 11 06 0f 0f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f bd 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
06 17 04 06 05 0b 01 0b 12 15 1a 02 13 06 0a 11 19 14 1e 00 09 0e 01 01 0b 11 04 02 09 16 11 1e 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0f 1c 07 09 12 11 11 0c 01 12 08 13 0d 08 1e 09 1e 0e 10 08 05 15 0e 12 05 0f 14 17 14 0c 15 12 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
09 08 01 00 04 0c 1c 13 07 15 08 0f 09 1d 0a 18 13 1a 12 1a 0c 15 13 10 11 10 0c 1b 13 11 01 0f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
7f 7f 7f f1 7f 7f 7f b4 7f 7f ca c2 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
da f7 f2 a8 96 c5 9d 94 c8 b9 7f c4 98 e5 7f 7f d3 a1 82 8f a6 fb bf f0 7f 84 d2 a0 88 80 c9 92 
7f 7f dd 7f 7f 7f 7f 7f 7f 7f 7f 95 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
1c 00 04 14 01 0b 04 0e 1a 1c 01 01 1a 01 08 03 02 00 13 17 15 1a 14 0c 13 1e 13 07 01 1c 12 0a 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f ad 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
15 11 1c 05 0c 16 01 03 10 08 03 08 13 10 02 02 1a 13 05 1a 00 0b 1a 16 08 1b 12 1a 1b 0d 14 10 
7f 7f 7f 7f 7f d9 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
15 09 1c 0d 0e 00 00 03 13 14 1d 0d 15 0a 02 0d 15 18 1d 19 11 11 0f 0e 15 11 1a 0d 0e 19 14 10 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f b5 7f 7f fa 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 93 7f 7f 7f 7f 7f 7f 
0d 1b 11 11 0c 06 13 02 18 01 17 0b 15 12 04 10 02 07 0b 08 14 1e 0b 19 06 15 0a 10 02 19 06 0e 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
17 01 15 16 0a 0f 05 16 01 1e 19 13 08 1c 1d 04 18 1a 14 15 0d 1c 1e 0b 18 0d 05 0a 03 1c 1d 03 
07 06 0a 0e 1b 14 13 1a 0b 0d 07 19 1b 1c 12 03 02 18 1b 10 1c 1c 17 0b 0e 0a 0c 0e 00 03 05 11 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f a5 7f 7f 7f 7f 7f 7f 7f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
07 02 11 02 02 13 09 18 0f 1a 04 0f 18 02 1d 05 18 1e 19 09 03 0f 1c 09 1b 04 1c 00 09 1b 18 1c 
7f 7f 7f e1 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f db 7f 7f 7f 
7f 7f 7f 7f 7f 7f 99 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f d5 7f 7f 7f 7f 7f 7f 7f 7f 7f 
11 1c 0e 08 18 19 00 14 02 03 1b 01 07 1e 0d 03 05 07 17 03 1c 0b 1e 1c 09 07 0e 03 14 01 00 1e 
0c 12 1e 15 09 10 0d 0f 0c 05 0c 0c 06 19 0b 04 11 1a 18 13 16 0a 04 07 0a 08 0e 04 04 07 06 18 
1e 07 0f 16 15 0d 18 0f 16 0f 1e 16 02 04 1d 07 00 07 0e 1d 0c 16 19 02 12 1e 11 10 0a 1d 1e 03 
06 1b 07 11 0e 0a 0c 00 05 13 17 06 05 1c 03 18 1b 04 19 04 01 08 0a 12 0d 18 14 03 06 17 03 1d 
7f 7f 7f 7f 7f a7 7f 7f 7f 7f 7f 7f 7f f5 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 97 7f 7f 
0f 12 11 1e 05 13 1e 16 11 0b 0c 1d 1b 14 1b 1c 1a 0b 0e 18 0b 17 0e 0a 0e 03 0a 0c 00 05 08 1a 
7f 7f 7f e8 7f 7f aa 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 8b 7f 7f 7f 7f 7f 7f 7f 7f 7f f9 7f 7f 7f 7f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
1e 14 18 0e 0e 0f 17 16 06 0d 10 11 1c 1a 04 0b 03 00 02 05 17 14 18 08 04 1e 19 1d 00 0c 13 16 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
13 12 0e 00 07 05 12 1a 18 18 19 07 1a 14 02 00 00 17 07 03 13 0a 02 04 10 05 08 16 0e 09 0e 00 
18 1c 09 1b 0a 14 1e 17 01 0d 0b 09 19 0d 15 13 10 01 14 1b 05 13 0f 0a 16 1e 00 1b 0b 19 05 16 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0a 17 0c 0b 11 06 02 07 00 14 0e 13 11 01 19 19 00 0d 18 0c 1a 15 14 1c 18 12 01 0e 15 10 12 0a 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0d 0c 1a 12 16 1b 17 0f 17 13 14 13 0c 13 13 1d 18 1a 17 19 12 0a 08 15 1b 10 04 19 0f 0e 01 0f 
18 1e 1c 14 0a 07 18 1c 1d 05 12 0f 0d 18 1d 16 15 15 14 10 07 18 03 13 0b 11 13 0e 1e 07 00 1d 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
09 07 17 00 05 0d 00 13 12 19 06 08 10 08 12 07 15 18 19 1e 10 0f 1a 17 11 0b 08 03 19 03 17 10 
03 1b 0f 06 12 19 00 00 04 16 04 15 15 10 06 1e 18 10 06 14 0c 11 09 13 01 09 1e 1b 09 1a 09 1d 
0b 14 00 0f 1d 15 0c 15 12 06 06 1b 08 02 19 10 0b 0f 16 05 14 16 19 08 12 07 11 05 18 1a 0a 06 
07 08 10 0e 0c 03 0b 14 10 10 1a 16 15 00 09 15 04 1c 04 1b 06 1a 1a 0a 1b 04 1a 0b 0d 03 12 08 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
7f 7f 7f 7f 7f 7f 7f 7f 7f cb 7f 7f 7f 7f cc 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f a3 
1b 07 0d 15 1c 15 13 0b 0d 13 0b 18 00 17 17 09 00 0a 12 18 1a 06 02 03 1e 14 03 15 1d 12 07 1d 
0e 0f 05 10 0d 1b 16 0e 04 04 1e 12 06 19 06 0e 1b 03 03 01 04 0b 09 08 00 0f 0d 16 09 12 09 17 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
14 1a 00 05 0a 09 19 07 09 0f 1d 09 13 01 0d 1c 06 0b 14 11 11 12 14 0a 0a 0c 10 11 1b 0c 0d 19 
00 07 1b 01 14 0f 1e 1e 03 08 1e 0a 05 1c 13 09 11 0d 0e 11 05 13 1d 12 18 08 04 00 1e 03 0b 14 
7f 7f e7 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f e2 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
0d 15 16 07 0f 0e 06 0c 11 1c 1c 08 14 01 02 0f 1b 04 17 1b 09 15 1a 0b 15 16 12 1a 1b 1d 11 05 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
01 0a 0c 06 1b 0d 14 16 17 16 17 07 1e 04 1c 1a 1a 01 02 19 0e 0b 1e 01 10 0d 03 0c 15 1b 00 10 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
d7 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f ac 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f be e9 b7 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
11 13 02 05 1d 08 02 08 16 08 06 08 0e 16 09 1b 1e 19 02 09 1c 1a 04 0d 0b 17 00 09 17 1b 01 12 
05 1e 1a 03 0a 16 16 1d 0d 19 14 09 12 1b 1a 0f 12 01 07 18 0c 05 11 15 14 0b 0d 0f 18 10 0c 0f 
9b 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
7f 7f 7f 7f 7f 7f 7f dc 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f ec 7f fc 7f 7f 7f 
09 10 08 00 09 02 0f 0f 1a 17 17 1e 08 0b 07 03 0f 0f 04 1c 1e 02 00 01 16 1a 02 00 10 0a 00 00 
13 17 07 01 19 09 19 1d 13 03 1a 13 07 06 0f 03 1c 15 19 0b 1c 04 16 07 00 03 06 17 0b 0e 13 08 
09 01 1c 1e 1e 03 06 13 1e 10 15 14 08 10 09 07 02 08 1e 0d 14 13 1d 0c 09 0a 09 1a 1b 09 0a 10 
00 1d 1c 13 0b 11 1b 0e 18 12 0d 1c 0c 12 01 0e 01 15 00 01 03 04 0f 0b 08 1e 1c 14 18 19 07 19 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0d 19 0d 19 01 16 03 0c 0d 05 0d 1a 01 06 1e 0d 0c 1c 18 05 12 05 18 11 18 02 1c 07 1a 0d 1b 03 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0d 04 00 04 08 0e 00 02 18 1e 0d 0e 12 03 10 14 1d 13 10 0c 1c 10 0e 16 0d 02 12 1a 0b 02 03 1c 
7f 7f 7f 7f 7f 7f 7f c6 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f c7 7f 7f df d1 7f 7f 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f af 7f 7f 7f 7f 7f c0 7f 7f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
08 0e 18 1a 18 14 02 0c 14 09 0f 1c 03 1a 03 0b 1c 06 10 0c 1b 1e 08 0f 1b 10 06 17 0a 0f 00 1e 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0f 1e 07 16 02 05 1c 06 17 12 1a 0b 0a 09 1a 12 1d 1b 04 11 03 01 02 1a 18 19 0a 13 18 0b 11 06 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
07 0f 0f 1c 02 10 11 04 0b 04 1b 0a 02 0e 10 1b 16 06 1c 00 15 01 19 05 18 19 17 03 0c 03 16 1e 
7f 7f 7f b1 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 85 7f 
04 0e 13 04 02 12 07 13 05 1a 09 0d 11 1b 1c 1a 15 02 05 10 1e 16 05 0b 1d 0f 1a 1a 18 13 14 0a 
19 1e 07 06 17 17 0f 0f 0c 1b 18 12 01 1a 0e 05 09 15 00 03 09 1b 17 1e 10 11 11 10 10 19 1d 0c 
07 16 01 0f 11 15 1c 18 11 0f 00 11 11 17 05 12 01 16 19 0d 15 14 09 02 17 0b 05 0d 19 1d 11 1e 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
15 05 0d 04 16 14 07 01 1d 19 11 10 1a 0e 0c 0a 07 00 14 0c 11 01 0b 04 03 08 19 0c 0c 12 07 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0f 0d 14 18 02 00 19 0d 17 00 0d 16 07 1d 1b 00 00 10 1d 0b 06 0d 00 06 0d 0f 07 07 06 0e 08 00 
0f 11 0b 09 0d 10 0e 1a 02 06 1d 12 13 13 07 0c 06 04 1e 17 1b 00 15 09 0f 14 00 0b 11 1b 0f 09 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
7f 7f 7f 7f 7f 7f 7f 7f 7f ff 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 89 7f 7f 7f 7f 
02 01 0b 16 09 04 18 19 1a 09 0d 07 11 0a 0a 18 1d 12 03 14 0d 1e 16 19 15 10 1b 19 09 04 0b 10 
0a 03 04 07 02 10 15 11 15 07 06 11 1b 0d 00 00 09 13 02 06 15 1e 0a 12 10 00 0a 04 07 17 15 01 
09 0a 0a 17 0f 10 04 1c 0a 0a 02 1e 0e 1c 1c 1e 19 1c 1c 18 04 10 11 1e 18 15 17 0b 1d 13 0c 0e 
1d 11 13 16 09 12 0e 0f 0c 10 06 07 06 12 07 18 1c 17 0a 1d 0a 0b 12 14 0c 18 1a 08 06 0c 15 14 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 1b 13 09 0d 17 18 05 13 05 1e 0d 1c 16 12 08 10 04 04 16 0b 17 07 16 16 09 03 0c 0f 03 05 01 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0d 1e 06 03 04 12 03 08 06 01 1d 0a 1d 16 06 1c 00 1e 01 07 11 00 17 02 19 01 10 06 08 0f 0b 0c 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f cf 7f 7f 7f 7f 7f 7f 7f 7f 81 b6 7f 7f 7f 7f 7f 
06 01 15 19 1d 13 0a 19 03 15 02 0c 0f 0b 05 07 19 0e 11 06 16 0a 12 1c 1e 01 18 1a 09 0b 11 1c 
7f 7f f3 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 8d 7f 7f 7f a2 7f 7f 7f 7f 7f 7f 7f 7f e6 7f 7f 7f 
0e 19 05 04 1c 08 01 0f 1e 19 0c 1e 18 1a 14 0c 1c 0e 1c 11 1c 0e 0d 12 09 04 12 1a 08 1a 18 18 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
0f 1a 1e 17 0d 08 03 19 04 11 0e 01 06 19 10 1c 1c 02 15 01 0d 0d 1d 12 05 0f 10 06 0b 1b 1e 18 
08 10 0d 17 1a 07 08 15 0c 04 06 11 12 1d 10 12 04 0c 08 15 08 06 0b 0e 0a 12 05 1b 15 10 01 0a 
7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f bb 7f 7f 7f 7f 7f 7f 7f 7f 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
01 08 13 1d 06 07 1a 0a 0c 0b 01 1d 15 00 03 04 0b 1b 04 07 09 0f 1b 10 1c 10 0d 1b 12 0e 0f 0b 
12 18 1a 06 02 12 0b 16 09 0d 19 02 0c 04 10 16 1e 17 04 0d 10 13 15 1e 1d 06 04 1e 04 1e 03 12 
7f f6 7f 7f 7f 7f 7f ef 7f 7f cd 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 7f 
09 0f 0b 10 00 0d 0d 09 0c 18 15 0f 14 0b 06 00 08 12 1b 19 0f 1e 0e 19 0c 17 1e 09 05 13 10 0b 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
11 03 11 00 1b 0a 0b 11 13 12 0f 13 1a 0d 0f 19 00 04 0a 06 1b 04 03 09 0f 19 1e 1a 12 01 01 13 """

table = [each.split() for each in table.splitlines()]
table = [[eval("0x" + a) for a in each] for each in table]

def PhysicalAddress(virtual_address, memory_block, pdbr):
    print "Virtual Address %x:"%(virtual_address)
    pde_no = virtual_address >> 10;
    pte_no = virtual_address >> 5 & 0x1f;
    offset = virtual_address & 0x1f;
    pde = memory_block[pdbr][pde_no]
    pde_valid = pde >> 7
    pde_address = pde & 0x7f
    print "  --> pde index:0x%x  pde contents:(valid %d, pfn 0x%x)"%(pde_no, pde_valid, pde_address)
    if pde_valid == 0:
        print "    --> Fault (page directory entry not valid)"
        return
    pte = memory_block[pde_address][pte_no]
    pte_valid = pte >> 7
    pte_address = pte & 0x7f
    print "    --> pte index:0x%x  pte contents:(valid %d, pfn 0x%x)"%(pte_no, pte_valid, pte_address)
    if pte_valid == 0:
        print "      --> Fault (page directory entry not valid)"
        return
    
    physicalAddress = offset + pte_address << 5
    content = memory_block[pte_address][offset]
    print "      --> Translates to Physical Address 0x%x --> Value: %x"%(physicalAddress, content)

if __name__ == "__main__":
    PhysicalAddress(0x6b22, table, 0x11)



（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2015/lecture06#head-1f58ea81c046bd27b196ea2c366d0a2063b304ab)
--- 

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。

--- 
