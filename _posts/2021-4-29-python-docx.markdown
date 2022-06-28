---
layout: post
title:  "python-docx"
date:   2021-4-29 20:35:35 +0800
categories: command
---

#### doc2docx

```
import os
import win32com.client

def conv_doc2docx(filename):
    """

    :param filename: doc文档的文件名
    :return: None
    """
    # 用os.path.splitext把文件名和扩展名拆开来，分别存为filename_base和filename_ext
    filename_base,filename_ext = os.path.splitext(filename)
    # 文件名加上.docx扩展名就是转换以后的文件名了。当然实际的转换在后面完成
    filename_conv = filename_base+'.docx'
    # 下面两行是检验前面提到的两个问题
    assert '~$' not in filename_base, f"~$ in filename:{filename}"
    assert filename_ext=='.doc', f'{filename} should be .doc file'

    # 下面一行用basedir目录和文件名形成word文档的相对链接
    path = os.path.join(basedir,filename)
    # 下面一行用basedir目录和文件名形成转换后word文档（docx）的相对链接
    path_conv = os.path.join(basedir,filename_conv)
    # 下面两行是把相对链接转换成绝对链接
    ori_path_abs = os.path.abspath(path)
    conv_path_abs = os.path.abspath(path_conv)
    word = win32com.client.Dispatch('Word.application')
    try:
        doc = word.Documents.Open(ori_path_abs)
        doc.SaveAs2(conv_path_abs, FileFormat=16)
        doc.Close()
    except Exception as e:
        print(f'Fail to convert:{filename}')
        print(e)
```

#### 字体对应
```
四号 == 14pt
小四 == 12pt
```


#### 读取docx文件

```
import docx

class DocProc:
    def __init__(self, filename):
        self.doc = docx.Document(filename)
    
    def get_text(self):
        for para in self.doc.paragraphs:
            print(para.text)
```


#### 常用方法

Word文档的主要结构单位是『段落』（Paragraph）。标题也好，目录也好，正文也好，都是段落。通过赋予段落不同的样式，形成不同功能的文档结构。这种做法一直可以追溯到Word 6.0时代，猜测是当时为了节省紧俏的存储资源而设定的。  

比段落更小的单位是『游程』（Run）。比方说，你想在一个段落里加重某几个字，那么加重的字就形成一个游程。一个段落至少包含一个游程。  

```
from  docx import  Document
from  docx.shared import  Pt
from  docx.oxml.ns import  qn
from docx.shared import Inches

#打开文档
document = Document()

#加入不同等级的标题
document.add_heading('Document Title',0)
document.add_heading(u'二级标题',1)
document.add_heading(u'二级标题',2)

#添加文本
paragraph = document.add_paragraph(u'添加了文本')
#设置字号
run = paragraph.add_run(u'设置字号')
run.font.size=Pt(24)



#设置字体
run = paragraph.add_run('Set Font,')
run.font.name='Consolas'

#设置中文字体
run = paragraph.add_run(u'设置中文字体，')
run.font.name=u'宋体'
r = run._element
r.rPr.rFonts.set(qn('w:eastAsia'), u'宋体')

#设置斜体
run = paragraph.add_run(u'斜体、')
run.italic = True

#设置粗体
run = paragraph.add_run(u'粗体').bold = True

#增加引用
document.add_paragraph('Intense quote', style='Intense Quote')

#增加有序列表
document.add_paragraph(
    u'有序列表元素1',style='List Number'
)
document.add_paragraph(
    u'有序列别元素2',style='List Number'
)

#增加无序列表
document.add_paragraph(
    u'无序列表元素1',style='List Bullet'
)
document.add_paragraph(
    u'无序列表元素2',style='List Bullet'
)

#增加图片（此处使用相对位置）
document.add_picture('jdb.jpg',width=Inches(1.25))

#增加分页
document.add_page_break()

#保存文件
document.save('demo.docx')
```

#### 表格

```
#增加表格
table = document.add_table(rows=3,cols=3)
hdr_cells=table.rows[0].cells
hdr_cells[0].text="第一列"
hdr_cells[1].text="第二列"
hdr_cells[2].text="第三列"

hdr_cells = table.rows[1].cells
hdr_cells[0].text = '2'
hdr_cells[1].text = 'aerszvfdgx'
hdr_cells[2].text = 'abdzfgxfdf'

hdr_cells = table.rows[2].cells
hdr_cells[0].text = '3'
hdr_cells[1].text = 'cafdwvaef'
hdr_cells[2].text = 'aabs zfgf'

# 建立表格
from docx import Document

document = Document()
table = document.add_table(rows=9,cols=10,style = 'Table Grid')
cell_1 = table.cell(1,2)
cell_2 = table.cell(4,6)
# 合并表格
cell_1.merge(cell_2)
document.save('table-1.docx')

document = Document('table-1.docx')
table = document.tables[0]
for row,obj_row in enumerate(table.rows):
    for col,cell in enumerate(obj_row.cells):
        cell.text = cell.text + "%d,%d " % (row,col)

document.save('table-2.docx')
```

#### 段落
![images](/source/python_docx1.png)

1. ParagraphFormat.alignment （选择一个WD_PARAGRAPH_ALIGNMENT）
2. ParagraphFormat.left_indent（长度）
3. ParagraphFormat.right_indent（长度）
4. ParagraphFormat.first_line_indent（长度）
5. ParagraphFormat.space_before（长度）
6. ParagraphFormat.space_after（长度）
7. ParagraphFormat.line_spacing_rule（选择一个WD_LINE_SPACING）
8. ParagraphFormat.line_spacing（长度）
9. ParagraphFormat.widow_control（True：设置，None：继承Style设置）
10. ParagraphFormat.keep_with_next（True：设置，None：继承Style设置）
11. ParagraphFormat.keep_together（True：设置，None：继承Style设置）
12. ParagraphFormat.page_break_before（True：设置，None：继承Style设置）

# 设置段落
```
from docx.enum.text import WD_ALIGN_PARAGRAPH

p = document.add_paragraph(u'字体在右边', line_style)
p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
```


#### 查看样式
```
styles = document.styles
print("\n".join([s.name for s in styles if s.type == WD_STYLE_TYPE.PARAGRAPH]))
```

