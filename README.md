#mol2文件分子名称的批量修改

#1. 目的及思路

&#8194;&#8194;&#8195;一般软件在读取 \*.mol2 文件时，依据文件内部的标注确定分子名称，而非文件名。\*.mol2 文件结构（仅展示包含分子名称的前几行）如下图所示，`#`为首的行是Sybyl软件生成的注释，可以看到，分子名称在该文件的第9行（不一定），即`@<TRIPOS>MOLECULE`的下一行（一定），该分子名为IAA。

```
#	Name:			IAA
#	Creating user name:	XiyangLee
#	Creation time:		Wed Jun 19 15:25:24 2019

#	Modifying user name:	XiyangLee
#	Modification time:	Wed Jun 19 15:27:21 2019

@<TRIPOS>MOLECULE
IAA
   22    23     1     1     0
SMALL
GAST_HUCK


@<TRIPOS>ATOM
      1 C1         -0.7976   -6.3164   -0.6615 C.2       1 BENZYL      0.0193 
      2 N2          0.2258   -5.7519   -0.0713 N.pl3     1 BENZYL     -0.2884 
      3 C2          1.3929   -6.1123   -0.6404 C.ar      1 BENZYL      0.0606 
      4 C3          1.1440   -6.9808   -1.6852 C.ar      1 BENZYL     -0.0193 
...
```

&#8195;&#8195;为了使软件正确读取我们所希望的分子名称，我们在修改分子名称时需要将 \*.mol2 文件内的名称一并修改。本文所介绍方法的思路为：1. 利用cmd批量修改 \*.mol2 文件名；2. 利用Python读取文件名并替换文件内的旧分子名。

#2. 操作方法

##2.1 批量修改 \*.mol2 文件名

&#8195;&#8195;考虑到批处理对象数目一般较为庞大，我们首先从获取文件名称开始。

&#8195;&#8195;在\*.mol2文件所在目录新建一个txt文件，输入`dir /a-d /b *.mol2* >>文件名.txt`，保存后修改该文件后缀为bat，双击执行该bat文件，即可获得该目录下所有\*.mol2文件的名称；

&#8195;&#8195;新建一个Excel文件，在第二列粘贴需要修改的文件名，第一列的单元格填充`ren`，第三列输入修改后的文件名称，与第二列一一对应；随后将该文件另存为csv文件；

&#8195;&#8195;使用Notepad++（或其他文本编辑器）打开保存的csv文件，将所有的`,`替换为` `（空格），这样每一行都符合cmd修改文件名的语法格式：`ren oldname.xxx newname.xxx`，保存后将文件名后缀csv改为bat，双击执行即可批量修改该目录下的 \*.mol2 文件名。

##2.2 批量修改 \*.mol2 文件内的分子名称

&#8195;&#8195;新建Python文档，输入并执行以下代码即可。

```
import os
import re

path = "C:/Users/Admin/Jupyter/MolecularRename/All" # \*.mol2 文件所在文件夹目录，注意删除非mol2文件
files = os.listdir(path) #得到文件夹下的所有文件名称
s = []
for file in files: #遍历文件夹
    if not os.path.isdir(file): #判断是否是文件夹，不是文件夹才打开
        f = open(path+"/"+file,"r+"); #打开文件
        iter_f = iter(f); #创建迭代器
        new=[]
        for line in iter_f: #遍历文件，一行行遍历，读取文本
            new.append(line)
        newname = "".join(re.findall(r'(.+?)\.mol2',file)) #将文件名作为分子名称；"".join()：将list转为str；r'(.+?)\.mol2'：去除文件名后缀；
        new[0]= '#	Name:			'+ newname +'\n'
        new[8]= newname + '\n'
        f.seek(0)
        for n in new:
            f.write(n)
        f.close()
print('Work done.')
```