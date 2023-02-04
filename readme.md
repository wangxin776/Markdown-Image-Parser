## 背景

本项目主要用于解析markdown文件中的图片链接，并将图片下载到本地。方便进行图床迁移

原作者的博客：[如何导出简书中的全部文章（包括图片）？](https://zhuanlan.zhihu.com/p/121155268)

## 思路
首先是要解析markdown文档，然后获取到其中的所有图片，再把图片按md文件分好目录保存。

### 解析markdown文档
这里我用了misaka模块，据说是python的markdown解析器里性能最好的，不过这个的文档着实是精简，太少内容了，写得不清不楚的，基本功能看来就是把markdown文档解析为html文档，但是好像没有直接操作markdown元素的方法。

没事，我可以像平时写爬虫那样解析html呀，不就曲线救国拿到图片了吗~
这里就用BeautifulSoup啦

### 下载图片
很简单，就是requests，没啥好说的。

## 实现
### 遍历文件
首先要遍历文件夹里面的所有md文档：
```python
def get_files_list(dir):
    """
    获取一个目录下所有文件列表，包括子目录
    :param dir:
    :return:
    """
    files_list = []
    for root, dirs, files in os.walk(dir, topdown=False):
        for file in files:
            files_list.append(os.path.join(root, file))

    return files_list
```

### 解析md文档 获取所有图片
先用misaka把markdown转换成html，然后再拿出所有img。
```python
def get_pics_list(md_content):
    """
    获取一个markdown文档里的所有图片链接
    :param md_content:
    :return:
    """
    md_render = misaka.Markdown(misaka.HtmlRenderer())
    html = md_render(md_content)
    soup = BeautifulSoup(html, features='html.parser')
    pics_list = []
    for img in soup.find_all('img'):
        pics_list.append(img.get('src'))

    return pics_list
```

### 下载图片
```python
def download_pics(url, file):
    img_data = requests.get(url).content
    filename = os.path.basename(file)
    dirname = os.path.dirname(file)
    targer_dir = os.path.join(dirname, f'{filename}.assets')
    if not os.path.exists(targer_dir):
        os.mkdir(targer_dir)

    with open(os.path.join(targer_dir, f'{uuid.uuid4().hex}.jpg'), 'w+') as f:
        f.buffer.write(img_data)
```

### 运行

在当前路径创建 `files`文件夹，并在`files`文件夹下创建img目录；将md文件放入`files`文件夹

```
pip install -r requirements.txt
python spider.py
```

## 欢迎与我交流

- [慕雪的寒舍](https://blog.musnow.top)