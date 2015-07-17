---
layout:     post
title:      文件状态改变检测(类似git)
category: blog
description: Python脚本，保存文件系统，对比文件历史更改
---
## 文件系统历史数据改变？

　写这个有毛用？git、svn不就已经解决了么。老大刚刚看了一眼，语重心长把我教导了一番：

> 不要总想搞个大新闻
> THINK MORE, DO LESS
> 不要重复造轮子
> XML已死，JSON也快了，msgpack...

### 引子

### 文件树结构

　文件系统一般是用B+树表示，这里为了实现方便，自己建立多叉树。path表示绝对路径，children保存子文件夹，files是字典类型，保存当前路径下文件的md5

``` 
class node:
    """node for file tree"""    
    def __init__(self,path):
        self.path = path
        self.children = []
        self.files = {}
    def addFolder(self,path):
        _child = node(path)
        self.children.append(_child)
    def addFile(self,name):        
        _md5 = GetFileMD5(name)
        self.files[name] = _md5 
```
### 文件遍历
　这个是老大喷的一个点，为什么有os.walk这么好的模块，还要自己写？性能比人家的好？不要重复造轮子，把精力集中在最重要的事情上去。我服了，的确是这样，正如黄欢讲课时举的栗子，思路不集中，到处串台，从python模块一会就刷到知乎上去了...... concentration
　好了，毕竟也是练习，复习一下DFS也可以，看了一下os.walk源码，实现思路也是一样的,上代码：

``` 
def traverse(node,depth=0):
    '''input root node and build multi-tree recursively '''
    path = node.path
    if(isdir(path)):
        for item in listdir(path):
            i = os.path.join(path,item)
            if (isdir(i)):
                node.addFolder(i)
                traverse(node.children[-1],depth+1)
            else:
                node.addFile(i)
```

没什么可说的，倒是发现几个有关处理路径的函数比较有趣，处理路径字符就直接用库吧：

```  
os.sep 根据win/linux （\,/）
os.getcwd()  还可以通过os.path.abspath('.')的方法获取当前路径.
os.listdir(os.getcwd()) 获取当前路径的所有文件
os.path.realpath(__file__)                     ---------->/root/p.py
os.path.split(os.path.realpath(__file__))  --------->('/root', 'p.py')
os.path.join
os.path.exists()
os.path.isfile(path)
os.path.isdir()
os.path.basename(path)-------------->截取最后的名称
all = [d for d in os.listdir('.')] # os.listdir可以列出文件和目录
``` 

### 文件树导出与重建
#### XML(v1)

　DOM? SAX? 还是都不要用了吧，浪费时间，需要自己写处理函数，导出导入都需要使用DFS进行遍历，性能差

其中一些比较有意思的点：
1. 字符串类型的字典怎样转成字典呢？
非常简单`eval(dic_str)`，eval函数的妙用
2. XML的textnode会包含不确定的空白行
最初的实现是将node.path保存在<path>中，file信息保存在<file>的文本区。然后结果却发现一个坑，文本区不属于<path><file>标签......
最初的设计：

```  
<folder>
  <path>name="E:\360Downloads"</path>
  <file>"{'E:\\360Downloads\\1.txt': 'd41d8cd98f00b204e9800998ecf8427e'}"</file>
  <folder>
  	<path>"E:\360Downloads\Software"</path>
  	<file></file>
  </folder>
</folder>
```
　这样就发现有两个问题，导致XML无法成功读取：
1.只能读出前几层的节点，节点属性为ELEMENT_NODE,<path>标签里面的都成了TEXT_NODE
2.TEXT_NODE会出现大量空白行，长度还不确定，导致无法过滤
所以将所有的信息都保存到了标签的属性中，set、get节点属性即可，非常机智。
后来改的版本：

```  
<folder name="E:\360Downloads">
  <file name="{'E:\\360Downloads\\1.txt': 'd41d8cd98f00b204e9800998ecf8427e'}"/>
  <folder name="E:\360Downloads\Software">
  </folder>
</folder>
```
OK，SB的xml格式到此为止，我再也不想用它了。Never and ever again.

#### cPickle(v2)

　直接把node对象dump到文件中，再使用load反序列化回来

```  
def generatePickle(node,picklefile):
    f = open(picklefile,'w')
    pickle.dump(node, f)
    f.close()
```
```  
def readPickleToTree(picklefile):
    f = open(picklefile)
    return pickle.load(f)
```
　小崩溃，XML载入解析节点搞了一天。pickle只用3行代码！！！
　不过保存自定义的对象时还是需要注意，如果其他程序需要调用生成的文件，那么要相应程序中导入类的定义。如果导入失败会导致对象无法还原，如果类在后期版本中改变了，也会还原失败。解决方法是在类中加入 `_getstate_()` 和 `_setstate_()` 来修改类实例的状态。使用方法见：[pickle用法][2]

### 文件比较
　这部分也是主要功能部分，思路也简单，还是DFS。不过具体写出来的时候，保存数据和递归有写坑。这部分慢慢写。用到一些装包拆包、序列生成的方法。
### 部分代码
#### 多叉树XML导出与重建

```  
def generateXML(node, xmlfile):
    '''from traversed node generate XML'''
    doc = Document()
   
    def gen(node,sroot,depth=0):
        if node == None:return
        root = doc.createElement("folder")
        root.attributes['name'] = node.path
        sroot.appendChild(root)
        
        files = doc.createElement("file")
        files.attributes['name'] = str(node.files)
        root.appendChild(files)        
        
        for item in node.children:
            gen(item,root,depth+1)    
    
    gen(node,doc,depth=0) 
    f = open(xmlfile, 'w')
    f.write(doc.toprettyxml(indent="  ",encoding='utf-8'))
    f.close()
``` 
```  
def readXMLToTree(xmlfile,root_path):
    '''from cmp generate tree nodes'''
    def go(DOMTree,root,d=0):
        for child in DOMTree.childNodes:
            if child.nodeType  != DOMTree.TEXT_NODE:   
                if child.tagName == "folder":
                    ret = node(child.attributes['name'].value)
                    root.children.append(ret)
                    go(child,ret,d+1)
                elif child.tagName == "file":
                    root.files.update(eval(child.attributes['name'].value))
    
    res = node(root_path)          
    DOMTree = xml.dom.minidom.parse(xmlfile)
    go(DOMTree,res)
    return res.children[0] ## No idea why one redundant node :(
```



[1]:    {{ page.url}}  ({{ page.title }})
[2]:	http://www.cnblogs.com/cobbliu/archive/2012/09/04/2670178.html 