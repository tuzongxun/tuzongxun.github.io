---
title: 使用python和java一键替换word文件内容
date: 2020-11-30 15:43:58
categories: python #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 问题整理
---
## 需求

一键替换word文档内容，是个比较常见的需求，office和wps也都有全部替换功能。

但是这个功能只能针对当前打开的文档，如果有多个文档，就需要一个一个的来。同时，如果要替换的内容也是多个，同样需要一个一个的来。

在文件少的情况下可能也没什么，但是如果文件有几十个，那么可能就很需要耐心了，因此一键替换所有符合要求的文档中内容的需求就有了。

<!--more-->


## 技术选型

### poi

java中操作word，比较熟知或者说比较官方的api是poi，对于word普通的操作其实也可以了。

但是对于文档内容替换，实际使用时却发现有很多的坑，并不能让人满意。

首先，poi中处理docx文件使用的是`XWPFDocument`，出来doc文件使用的是`HWPFDocument`，实际使用发现在替换doc文件时，文档中自动生成的标题序号无法读取，只能读到级别，而不能读到具体的数值。

因此，从某种程度而言，这就破坏了原本的文档结构，所以也就不是那么靠谱。

同时，不知何种原因，不论是docx还是doc，总会有一些内容就是替换不掉，这就更加不靠谱了。

基于两种原因，在文档内容替换的技术选型上，便不得不放弃poi，而相关的替换代码也就没有贴出来的必要。

### spire.doc

这个技术其实是最近了解到的，相对poi其实简单很多，但是这个并不是开源的，免费版功能有限，并且经过他处理的文件，都会在文档第一页增加一段类似版权说明之类的内容。

很显然，这种直接加文档无关内容的做法，也是当前需求不允许的。

同时，和poi一样，也总会有些内容怎么都替换不掉。

于是乎，只能另求他解。

### python-docx

解决业务问题是不应该局限于技术本身的，在找了众多java资料，结果验证都有这样那样的问题之后，也了解到了python也可以处理这种word文档内容替换的业务，依赖的模块是python-docx。

经过验证，python确实能够完全替换掉相应文档中的所有内容了。但是，新的问题是，这个python-docx只能处理docx的文档，而不能处理doc文档。

看网上资料，处理doc文档还需要依赖`win32com`，但是可能是网络问题，想了很多办法都下载不了这个依赖后，只好换了另一种思路，即把doc文件先转成docx文件，然后再用python-docx一键替换。

## 可用方案

经过一番折腾，确定了一个可用方案，经过验证也确实是可用的，一定程度上的确减轻了这种文档替换的工作量。

只不过操作还是略显得复杂，并不能算是最终方案，仅供参考。

方案结果是：

1. 使用`spire.doc`把某个目录下的doc文档一键转换成docx文档；
2. 使用`python`把docx文档中`spire.doc`额外增加的内容以及其他需要替换的内容一键替换。

### doc转换为docx的代码如下

```
public static void main(String [] args){
		File file = new File(args[0]);
		if (file.isDirectory()) {
			System.out.println(file.getName() + " is a directory");
			// 如果是一个目录，则列出目录中的子目录或者文件
			File[] files = file.listFiles();
			for (int i = 0; i < files.length; i++) {
				File file1 = files[i];
				String fileName = file1.getName();
				if (file1.isDirectory()) {
					System.out.println(fileName + " is a directory");
				}
				else {
					System.out.println(fileName + " is a file");
					String fileExtension = fileName.substring(fileName.lastIndexOf("."), fileName.length());
					if (".doc".equals(fileExtension)) {
						Document document = new Document(file1.getAbsolutePath());
						document.saveToFile(args[1]+"\\"+fileName+"x", FileFormat.Docx);
						document.close();
					}
				}
			}
		}
	}
```

以上代码的maven依赖如下：

```
<dependency>
	<groupId>e-iceblue</groupId>
	<artifactId>spire.doc</artifactId>
	<version>3.11.0</version>
</dependency>
```

### python-docx一键替换文档内容

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import os
import docx
import sys

#需替换文档目录
path = u'E:\\test'
#自动创建
tlog =  path + u'\替换文档列表.txt'
err_log = path + u'\替换出错列表.txt'

if sys.getdefaultencoding() != 'utf-8':
    reload(sys)
    sys.setdefaultencoding('utf-8')
#两个日志
def log(text):
    with open( err_log,"a+" ) as f:
        f.write(text)
        f.write('\n')
def log2(text):
    with open( tlog,"a+" ) as f:
        f.write(text)
        f.write('\n')

#替换内容（文档名称，旧的内容，新的内容）
def info_update(doc,old_info,new_info):
    #替换文档中所有文字内容
    for para in doc.paragraphs:
        for run in para.runs:
            run.text = run.text.replace(old_info,new_info)
    #替换文档中表格中的内容
    for table in doc.tables:
        for row in table.rows:
            for cell in row.cells:
                cell.text = cell.text.replace(old_info,new_info)

def pageHeader_update(doc,old_info,new_info):
    for se in doc.sections:
        for pa in se.header.paragraphs:
            pa.text = pa.text.replace(old_info,new_info)
        for ta in se.header.tables:
            for row in ta.rows:
                for ce in row.cells:
                    ce.text = ce.text.replace(old_info,new_info)
                
def thr(old_info,new_info):
    #遍历目录中的docx文档
    print('开始替换')
    for parent, dirnames, filenames in os.walk(path):
        for fn in filenames:
            filedir = os.path.join(parent, fn)
            if fn.endswith('.docx'):
                try:
                    #定义文档路径
                    doc = docx.Document(filedir)
                    #调用函数修改文档内容
                    info_update(doc,old_info,new_info)
                    pageHeader_update(doc,old_info,new_info)
                    #保存文档
                    doc.save(filedir)
                    #写入修改日志
                    log2(filedir + ' 修改完成')
                    print(filedir + ' 修改完成')
                except Exception as e:
                    #写入修改失败日志
                    log(filedir)
                    
if __name__ == '__main__':
    thr('Evaluation Warning: The document was created with Spire.Doc for JAVA.','')
    thr('数据','tuzongxun')
    print('----全部替换完成----')
```



## python-docx安装（win10-python39）

之所以一块额外拿出来说，是因为各种网络问题会导致很多简单的操作莫名其妙的失败，单纯的`pip install python-docx`并不能达到想要的效果。

经过最终验证成功后的总结，整体步骤如下：

1、pip list查看setuptools是否存在，如果存在，安装wheel：

```
pip install wheel
```

如果不存在，先安装setuptools，似乎一般都会有：



2、安装好wheel之后安装lxml，这个文件在线一直安装失败，最终离线安装，先下载文件lxml-4.6.2-cp39-cp39-win_amd64.whl（注意下载对应的版本，不一定要这个），下载地址https://pypi.org/project/lxml/#files
文件下载之后在提供的文件所在目录执行如下命令（直接这里执行pip的前提是python和pip的环境变量配好了）：

```
pip install lxml-4.6.2-cp39-cp39-win_amd64.whl
```



3、然后安装python-docx：

```
pip install python-docx
```

如果在线安装不了，尝试离线安装（这个安装包找不到下载地址的，若有需要可联系我，也可以在csdn下载https://download.csdn.net/download/weixin_45174975/12152734），在下载的安装包所在目录执行：

```
pip install python-docx-0.8.10.tar.gz
```

都安装完之后即可执行python脚本。