---
title: Python爬虫实战：电影天堂关键词搜索获取片源信息及下载种子
typora-root-url: ./Python爬虫实战：电影天堂关键词搜索获取片源信息及下载种子
date: 2024-08-27 21:10:59
tags:
    - python
    - 爬虫
categories: 
    - 实战
description: 根据设置的搜索关键字，爬取电影天堂对应片源信息及下载地址的数据，生成csv文件
---

根据学习的[爬虫视频](https://www.bilibili.com/video/BV1uN4y1W7Du?p=25\&vd_source=95f31f606414d946786c34f2feac0a92)实战项目，进行的扩展实战

**原实战实现功能**：爬取主页电影排行榜片源信息及下载种子

**扩展实现功能**：破解反爬策略，根据搜索关键字，获取查找出来的片源信息列表以及下载种子

# 1. 思路分析

## 1.1 数据获取

首先获取搜索的请求链接，查看请求参数

![图片](./pic0.png)

多进行几次请求，发现除了`keyboard`参数，其他的参数每次都是固定的，可知`keyboard`即为我们输入的搜索关键字转码而来，这里使用的是`gb2312`编码的方式

![图片](./pic1.png)

搜索请求返回一个`html文件`数据，我们采用`BeautifulSoup`来进行数据提取，获取`片源信息`以及`详情页的链接`

![图片](./pic2.png)

点击进入详情页，发现详情页的页面地址就等于：`域名+上面的链接`

![图片](./pic3.png)

我们再对每个片源的详情页进行访问，用BeautifulSoup提取出对应的下载种子即可

## 1.2 反爬处理

如果直接用`requests`库直接对网页进行请求，每次返回都是

![图片](./pic4.png)

这是因为网站有做反爬，具体的解决方法可以参考[破解反爬虫策略 /\_guard/auto.js（一） 原理](https://blog.csdn.net/cjc000/article/details/140476371)

> **requests** 是一个非常流行且强大的 Python 库，用于发送 HTTP 请求。

请求里面的关键参数主要是headers中的user-agent和cookies中的`guardok`

> 当请求返回为`<script src="/_guard/auto.js"></script>`时，响应数据中会返回一个`guard`的cookie，通过`auto.js文件`对`guard值`的加密处理，会返回一个`guardret值`的cookie，带着它再一次进行请求，响应数据中就会返回`guardok值`啦
>
> 拿到`guardok`就可以正常进行网络请求啦，所以主要难点就是对`auto.js文件`中加密方法的处理

# 2. 编码环节

## 2.1 反爬处理

主要是对auto.js文件进行反混淆处理，提取出`guardret值`生成的关键代码，具体操作参考[破解反爬虫策略 /\_guard/auto.js（一） 原理](https://blog.csdn.net/cjc000/article/details/140476371)，反混淆处理后的关键代码如下：

```javascript
function x(input, _0x56ee7b) {
    var  _0x453109 = {
        'kkhgO': function(_0xa38b79, _0x106f39) {
            return _0xa38b79 + _0x106f39;
        },
        'ENirv': function(_0x2f779a, _0x416b4b) {
            return _0x2f779a !== _0x416b4b;
        },
        'OuMHF': 'SpoxN',
        'ocuSY': function(_0x44bb0a, _0x227faa) {
            return _0x44bb0a ^ _0x227faa;
        }
    };
    let output = '';
    var _0x56ee7b = _0x453109["kkhgO"](_0x56ee7b, 'PTNo2n3Ev5');
    for (let _0x4e905d = 0x0; _0x4e905d < input['length']; _0x4e905d++) {
        if (_0x453109['ENirv'](_0x453109["OuMHF"], "SpoxN"))
            _0x98e486 = 0x2;
        else {
            const charCode = _0x453109["ocuSY"](input.charCodeAt(_0x4e905d), _0x56ee7b.charCodeAt(_0x4e905d % _0x56ee7b['length']));
            output += String["fromCharCode"](charCode);
        }
    }
    return output;
}

function b(input) {
    var  _0x2056fd = {
        'HKVGJ': function(callee, _0x55533f) {
            return callee(_0x55533f);
        }
    };
    return _0x2056fd["HKVGJ"](btoa, input);
}


function get_guardret(guard) {
    var CHccew = {
        'AZRsQ': '2|5|6|4|1|0|7|3',
        'DkXRu': function(_0x24bd17, _0x319a20) {
            return _0x24bd17 - _0x319a20;
        },
        'DwVzJ': function(_0x1d9e50, _0x282088) {
            return _0x1d9e50 * _0x282088;
        },
        'VhuiU': function(_0x4676ea, _0x492298) {
            return _0x4676ea === _0x492298;
        },
        'UqSnX': "undefined",
        'zECQl': "guardret="
    }
        , _0x4a12e7 = CHccew['AZRsQ']['split']('|')
        , _0x143f9d = 0x0;
    var _0x1db96b = guard.substr(0x0, 0x8);
    var _0x6339b6 = parseInt(guard['substr'](0xc));
    CHccew['VhuiU']('object', CHccew['UqSnX']) && (_0x6339b6 = 0x2);
    var _0x56549e = CHccew['DkXRu'](CHccew['DwVzJ'](_0x6339b6, 0x2) + 0x12, 0x2);
    var encrypted = x(_0x56549e.toString(), _0x1db96b);
    var guard_encrypted = encrypted.toString();
    return b(guard_encrypted)
}
```

拥有一个获取`guardret值`的方法，剩下就是按照上面`guardok`获取逻辑来处理cookie啦

```python
def get_cookie():
    # 1. 请求主页面
    response = requests.get(domain, cookies=cookies, headers=headers)
    # 2. 设置cookie，第一次请求获取guard，处理得到guardret
    for cookie in response.cookies:
        cookies[cookie.name] = cookie.value
    with open('guard处理.js', 'r', encoding='utf-8') as f:
        js_code = f.read()
    # 编译JavaScript代码
    ctx = execjs.compile(js_code)
    # 调用函数
    cookies['guardret'] = ctx.call('get_guardret', cookies['guard'])
    # 3. 二次请求获取到guardok，放入cookie，才能正常请求界面
    response = requests.get(domain, cookies=cookies, headers=headers)
    for cookie in response.cookies:
        cookies[cookie.name] = cookie.value
```

> **ExecJS** 是一个 Python 库，它允许你直接在 Python 中执行 JavaScript 代码

## 2.2 请求参数处理

对于查询的关键字，请求前需要对内容进行`gb2312`转码处理，具体如下：

```python
def get_data(text):
    str = 'classid=0&show=title%2Csmalltext&tempid=1'
    # 将字符串编码为 GB2312
    encoded_text01 = text.encode('gb2312')
    encoded_text02 = '立即搜索'.encode('gb2312')
    # 将编码后的字节数据转换为 URL 编码格式
    keyboard = urllib.parse.quote(encoded_text01)
    submit = urllib.parse.quote(encoded_text02)
    return str + f'&keyboard={keyboard}&Submit={submit}'
```

> **urllib.parse** 是 Python 标准库中的一个模块，专门用于处理 URL 的解析、合成、编码和解码操作。

## 2.3 查询页面数据提取

通过`BeautifulSoup`和`正则`对页面数据进行分析，获取`片名`、`日期`、`详情页路径`

```python
def deal_query_html(response, queryStr):
    result = []
    max_len = 0
    soup = BeautifulSoup(response.text, 'lxml')
    ul_div = soup.find('div', class_='co_content8')
    if ul_div is None:
        print('没有搜索结果')
    else:
        ul = ul_div.find('ul')
        li_list = ul.find_all('table')
        print(f'搜索成功啦~ 共{len(li_list)}条结果')
        for li in li_list:
            data = {}
            data['full_title'] = li.find('a').get('title')
            data['href'] = domain + li.find('a').get('href')[1:]
            data['title'] = re.search(r'《(?P<title>.*?)》', data['full_title']).group('title')
            data['time'] = re.search(r'日期：(?P<time>.*?)\n',li.find('font').text).group('time').strip()
            # 获取对应的下载种子源
            length = get_source(data)
            result.append(data)
            # 更新一下下载地址最多有几个
            if length > max_len:
                max_len = length
        # 数据存入文件
        save_file_csv(result, max_len, queryStr)
        print(f'数据获取完成，存入文件：temp/{queryStr}_下载种子.csv')
```

> **BeautifulSoup** 是一个用于解析 HTML 和 XML 文件的 Python 库，通常用于网页抓取（Web Scraping）\
> **re** 是 Python 的正则表达式模块，用于在字符串中执行模式匹配操作。

## 2.4 详情页面种子数据提取

`deal_query_html`函数处理了页面查询数据，获取了对应的详情页的路径，通过`get_source`函数获取详情页的数据，得到每个片源的种子数据

```python
# 获取对应数据的种子链接
def get_source(info):
    resp = requests.get(info["href"], cookies=cookies, headers=headers)
    resp.encoding = 'gb2312'
    # print(resp.text)
    obj3 = re.compile(r'<td style="WORD-WRAP: break-word" bgcolor="#fdfddf">.*?<a href="(?P<download>.*?)">.*?</td>', re.S)
    download_list = obj3.finditer(resp.text)
    # 关闭response
    resp.close()
    length = 0
    for dl in download_list:
        length = length + 1
        info[f'download{length}'] = re.sub(r'\s+', '', dl.group("download"))
    print(f'{info["title"]}种子链接获取完成, 共{length}个')
    # 返回有几个下载地址
    return length
```

## 2.5 爬取数据写入文件

获取了数据，当然要进行保存操作啦，这里用`csv`文件对爬取的数据进行保存

```python
# 写入csv文件
def save_file_csv(data_list, max_len, query_str):
    # 设置csv文件表头
    head = ["完整片名", "网页路径","片名", "日期"]
    for i in range(max_len):
        head.append(f'下载地址{i+1}')
    # 打开文件，准备写入
    with open(f'{query_str}_下载种子.csv', 'w', newline='', encoding='utf-8') as csvfile:
        # 创建一个csv.writer对象
        writer = csv.writer(csvfile)
        # 写入标题行
        writer.writerow(head)
        # 遍历数据
        for row in data_list:
            writer.writerow(row.values())
        # 关闭
        csvfile.close()
```

> **csv** 是 Python 标准库中的一个模块，用于处理 CSV（逗号分隔值）文件。它提供了方便的工具来读取和写入 CSV 格式的数据，非常适合处理结构化数据。

## 2.6 主函数

```python
domain = 'https://www.dytt89.com/'
cookies = {}
headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36'}


if __name__ == '__main__':
    # 1. 搜索的关键字， 拼接查询参数
    queryStr = '神偷奶爸'
    data = get_data(queryStr)
    # 2. 处理获取请求需要的cookie
    get_cookie()
    headers['content-type'] = 'application/x-www-form-urlencoded'
    print(f'开始搜索：{queryStr}')
    # 3. 发起搜索界面请求
    response = requests.post(domain + 'e/search/index.php', cookies=cookies, headers=headers, data=data)
    response.encoding = 'gb2312'
    # 4. 处理返回的搜索结果界面信息
    deal_query_html(response, queryStr)
```

# 3. 运行效果

启动程序，爬虫顺利运行，下面是运行效果：

![图片](./pic5.png)

爬取数据也顺利保存进本地csv文件

![图片](./pic6.png)

***

如果文章中有哪里没有讲明白，或者讲解有误的地方，欢迎在评论区批评指正。\
感谢阅读 ( ´▽｀)
