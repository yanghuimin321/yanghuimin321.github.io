---
title: Python爬虫实战：协程+线程爬取在线m3u8视频
date: 2024-08-27 02:59:20
tags:
---

根据学习的[爬虫视频](https://www.bilibili.com/video/BV1uN4y1W7Du?p=68\&vd_source=95f31f606414d946786c34f2feac0a92)实战项目，进行的扩展实战

**扩展实现功能**：

1.  根据片子id、播放源、下载集数，爬取在线播放网站的m3u8视频，并转化为mp4文件；
2.  下载m3u8对应ts文件采用协程操作；
3.  对于不同的集数采用线程操作

# 1. 思路分析

打开随便一部片子详情页，如下图，发现路径由 `域名+/tv/+片子id组成`

![1724748888705.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/9bb196f544494168b92a739bbfc17d7d~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=R4x3SLGeVlPyoS%2BIi0YhDEXospg%3D)

`f12`查看请求信息发现它返回一个`html文件`，其中引入了一个`js文件`

![1724750202552.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/79889de92d5043d282868f0dd9676f2e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=QDGcpl%2FFQwLgTkjQkDlsvZyzTfo%3D)

继续查看请求的`js文件`内容，发现是一个`存放各种播放源、集数对应的m3u8链接列表文件`

![1724751240212.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/09f2e5d833e849ee9f9df179aca8e761~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=ewJpxq4CyQ%2BrHaStWhmN9R7AaPA%3D)

**那么现在，我们就知道通过片子id、播放源、集数获取对应的m3u8文件的逻辑了**：先通过 `域名+/tv/+片子id` 获取详情页的`html文件`，然后找到引入的`js文件路径`，请求js文件，通过查找播放源、集数，就能匹配到对应的`m3u8文件请求路径`啦

这样就完啦？以网站中`良子线`为例，我们请求看看上面对应的m3u8文件路径，如图，发现根本没有想象中的那些ts文件路径，这是因为真正的m3u8文件路径，其实是包含在这个文件中，即图片中圈住的内容

![1724751538215.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/0c68075f57b24b2abe49eb8847237bbb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=%2ByQ5A6e%2FIJYGYxIi5RQA0hrdcIE%3D)

![1724752069360.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/30d985e696b14fc4896bc828fe7ce043~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=pRHAROiGqx%2FlfNMOd%2BsCxZAFb4o%3D)

所以对于有些播放源，我们得进行**二次处理获取真正的m3u8文件源**

获取到真正的m3u8文件后，我们要做的就是`拼接ts请求路径`，发起请求把ts文件都请求下来即可；但是下载下来的ts文件都能直接播放？不不不，有些播放源，偷偷给你做了点小处理

![1724752878255.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/8a827e00785e433dbfad4837fc103deb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=iXyFMtFCel2JdZEWBT2MWLALnxo%3D)

如上，有些播放源对ts文件是做了**加密**的，想要获得能直接播放的ts文件，还得先进行解密操作；一般都是`AES`加密，获取到解密需要的`key`和`iv`值即可，其中`key`值，是需要根据**URI拼接请求路径**获取到真正的`key`值的，如图

![1724760670767.jpg](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/471041aaad6944b18fb2e291b71e008b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=F1El3YB7ALLNKPV7u8Ux3qZrgtM%3D)

这下，我们终于能获得一堆能直接播放的ts文件啦，但是实际上都是很短的片段，想要看完整内容，还得把ts文件都合并成一个完整的，可以使用`FFmpeg`来进行合并操作，甚至是后续将`ts文件`转为`mp4格式文件`

> **FFmpeg** 是一个开源的多媒体框架，能够记录、转换和流式传输音频和视频。它支持几乎所有的音频和视频格式，可以用于各种多媒体处理任务，如格式转换、视频编辑、音频提取、压缩等。

# 2. 编码环节

## 2.1 获取m3u8文件链接列表

拥有`片子id`我们可以拼接出详情页请求路径，获取html文件，再通过`正则匹配`得到存有m3u8链接列表的js文件，最后再根据`播放源`，匹配出所有的m3u8请求链接

```python
# tv_id：片子id  
# play_type：播放源类型
# 返回：id对应播放源的所有集组成的m3u8请求链接列表
def get_data(tv_id, play_type):
    # 页面url
    url = f'http://feijisu36.com/tv/{tv_id}/'
    response = requests.get(url)
    # 获取页面中js文件请求url，其中保存的播放源和集数对应的m3u8请求地址
    url = re.findall(r'<script type="text/javascript" src="(?P<url>.*?)"></script>', response.text)[1]
    # 拼接成有效的url地址
    if not url.startswith('http'):
        url = 'http:' + url
    # 请求获取js文件内容
    resp = requests.get(url)
    temp = re.finditer('playarr_' + play_type + r'[.*?]="(?P<url>.*?)";', resp.text)
    play_list = []
    for it in temp:
        play_list.append(it.group('url').split(',')[0])
    return play_list
```

## 2.2 获取m3u8文件内容

**get\_m3u8**：获取`m3u8文件`内容，并保存，`协程`调用后续获取`ts文件`的操作\
**deal\_m3u8\_content**：就是用来处理上面分析的需要二次请求文件才能得到真正的m3u8文件的问题
**deal\_ad**：对于特定的播放源，m3u8文件中有一些`广告的ts内容`，在这里进行一下处理

```python
def deal_m3u8_content(res, m3u8_url):
    # 良子,酷播，飞飞, 酷U 播放源需要根据第一次获取的m3u8文件，再次获取到第二个真正的m3u8文件
    if video_play_type in ['lz', 'kb', 'ff', 'uk', 'wj', 'fs']:
        uri = ''
        for line in res.text.splitlines():
            if not line.startswith('#'):
                uri = line.strip()
        if video_play_type in ['uk', 'fs']:
            m3u8_url = '/'.join(m3u8_url.split('/')[:3]) + uri
        else:
            m3u8_url = m3u8_url.replace(m3u8_url.split('/')[-1], uri)
        response = requests.get(m3u8_url)
        return {
            'content': response.content,
            'url': m3u8_url
        }
    else:
        return {
            'content': res.content,
            'url': m3u8_url
        }


# 处理一下澳门新葡京:)
def deal_ad(file_name):
    if video_play_type == 'uk':
        lines = open(file_name, 'r').readlines()
        new_line = []
        flag = False
        for line in lines:
            if(line.startswith('#EXT-X-KEY')):
                flag = not flag
            if flag:
                new_line.append(line)
        with open(file_name, 'w') as f:
            f.writelines(new_line)
            

def get_m3u8(v_id, m3u8_url):
    file_name = f'temp/{video_id}_{video_play_type}_{v_id}.m3u8'
    # 获取m3u8文件
    result = deal_m3u8_content(requests.get(m3u8_url), m3u8_url)
    content = result['content']
    with open(file_name, 'wb') as f:
        f.write(content)
    print(f'获取{file_name}的m3u8文件完毕')
    deal_ad(file_name)
    # 下载ts文件
    if video_play_type in ['lz', 'ff', 'kb']:
        asyncio.run(get_ts(file_name, result['url'], v_id))
    else:
        asyncio.run(get_ts(file_name, m3u8_url,v_id))
    # 获取完数据删除本地m3u8文件
    os.remove(file_name)
```

## 2.3 获取ts文件

**get\_ts**：在这里处理m3u8文件内容（如果有加密，还要处理获得解密对象），拼接获取文件中所有ts文件请求路径，并各自创建异步任务调用`download_ts`方法来下载ts文件\
**download\_ts**：ts文件下载。如果ts文件被加密，还要调用解密对象处理ts文件\
**deal\_ts\_url**：根据不同的播放源，用不同的方式拼接ts请求路径

```python
async def download_ts(ts_url, file_name, session, des_info, ts_info):
    try:
        async with session.get(ts_url, timeout=60) as response:
            # 判断一下请求是否正常
            if not response.status == 200:
                for it in ts_info:
                    if it['name'] == file_name:
                        it['code'] = response.status
                        return  # 请求状态码不对，直接结束
            content = await response.content.read()
            # 判断是否加密，如果加密了对内容进行解密
            if des_info['is_aes'] == 1:
                content = des_info['cipher'].decrypt(pad(content, AES.block_size))
            async with aiofiles.open(file_name, 'wb') as f:
                await f.write(content)
        print(f'{file_name}文件获取完毕')
    except TimeoutError:
        return await download_ts(ts_url, file_name, session, des_info, ts_info)
    except ServerDisconnectedError:
        print('ServerDisconnectedError')
        

def deal_ts_url(line, m_url):
    ts_url = line.strip()
    # 良子,酷播，飞飞，百度等 播放源需要拼接url
    if video_play_type in ['lz', 'kb', 'ff', 'bd']:
        ts_url = m_url.replace(m_url.split('/')[-1], ts_url)
    elif video_play_type in ['uk', 'fs']:
        ts_url = '/'.join(m_url.split('/')[:3]) + ts_url
    return ts_url


async def get_ts(m3u8_name, m_url, v_id):
    tasks = []
    ts_info = []
    async with aiohttp.ClientSession() as session:
        async with aiofiles.open(m3u8_name, mode='r') as f:
            des_info = {"is_aes": 0}
            content = await f.read()
            # 判断是否加密
            if '#EXT-X-KEY' in content:
                des_info['is_aes'] = 1
                des_info['cipher'] = await get_cipher(m_url, session, content)
            for line in content.splitlines():
                if not line.startswith('#'):
                    file_name = f'temp/{video_id}_{video_play_type}_{v_id}_{line.strip().split("/")[-1]}'
                    # 需要判断播放源，不同播放源请求url组成不一样
                    ts_url = deal_ts_url(line, m_url)
                    tasks.append(asyncio.create_task(download_ts(ts_url, file_name, session, des_info, ts_info)))
                    ts_info.append({
                        'name': file_name,
                        'code': 200
                    })
            print(f'共{len(ts_info)}条ts文件')
            await asyncio.wait(tasks)
            deal_ts_files(ts_info, v_id)
```

## 2.4 设置ts解密对象

**get\_cipher**就是在处理m3u8文件中内容时，会调用的一个`设置解密对象`的方法；如果是对ts文件有加密的播放源，在这个方法中会获取`key`和`iv`值生成一个解密对象用于后续操作

```python
# 处理信息，并返回解密对象，判断播放源
async def get_cipher(url, session, content):
    # 获取加密URI和IV
    uri = re.search(r'URI="(?P<uri>.*?)"', content).group('uri')
    iv = b'0000000000000000'
    if video_play_type in ['sn', 'hn']:
        iv = re.search(r'IV=(?P<iv>.*)', content).group('iv')
        iv = bytes.fromhex(iv[len(iv) % 16:])
    res_url = ''
    if video_play_type in ['uk', 'fs']:
        res_url = '/'.join(url.split('/')[:3]) + uri
    else:
        res_url = url.replace(url.split('/')[-1], uri)
    async with session.get(res_url) as response:
        key = await response.content.read()
        cipher = AES.new(key, AES.MODE_CBC, iv)
        return cipher
```

## 2.5 合并ts文件，转成mp4格式

**deal\_ts\_files**：在ts片段文件都获取完之后调用的`合并`、`转格式`的方法（**处理完成后会将原先的ts片段文件等都删除**）

```python
# 下载ts文件完成后合并转化为MP4格式
# ts_files: ts文件列表，v_id:集数
def deal_ts_files(ts_files, v_id):
    temp_file = f'temp/{video_id}_{video_play_type}_{v_id}_temp.txt'
    output_ts = f"temp/{video_id}_{video_play_type}_{v_id}_merged.ts"  # 合并后的 .ts 文件
    output_mp4 = f"temp/{video_id}_{video_play_type}_{v_id}_正片.mp4"  # 最终的 .mp4 文件
    # 遍历一下video_dict，如果有video_id对应的片名，则输出的mp4文件用片名替代
    for key, value in video_dict.items():
        if value == video_id:
            output_mp4 = f"temp/{key}_{video_play_type}_{v_id}_正片.mp4"
    with open(temp_file, "w") as f:
        for ts_file in ts_files:
            if ts_file['code'] == 200:
                name = ts_file['name'].split('/')[-1]
                f.write(f"file '{name}'\n")
    # 使用 FFmpeg 合并文件
    subprocess.run(["ffmpeg", "-f", "concat", "-safe", "0", "-i", temp_file, "-c", "copy", output_ts])
    # 将合并后的 .ts 文件转换为 .mp4 格式
    subprocess.run(["ffmpeg", "-i", output_ts, "-c", "copy", output_mp4])
    # 删除中间文件
    os.remove(output_ts)
    os.remove(temp_file)
    for ts_file in ts_files:
        if ts_file['code'] == 200:
            os.remove(ts_file['name'])
    print(f'{output_mp4}合并完成')
```

## 2.6 主函数

通过在代码中自行设定`片子id`，`播放源`，`集数范围`，可以下载不同的视频；片子的每一集，都是以`线程`的方式来进行加载处理

```python
video_id = '3736'  # 片子id
video_play_type = 'sn'  # 片子播放源
video_range = [1, 5]   # 集数范围
# play_dict = {
#     'hn': '牛牛',  
#     'sn': '新朗',  
#     'lz': '良子',  
#     'fs': 'F速',  
#     'ff': '飞飞',  
#     'bd': '百度',  
#     'uk': '酷U',   
#     'wj': '无天', 
#     'kb': '酷播', 
# }
video_dict = {
    '海贼王': '3736',
}

if __name__ == '__main__':
    # 根据片子id和播放源获取对应的一系列m3u8的url列表
    url_list = get_data(video_id, video_play_type)
    # 获取需要的集数对应的m3u8下载地址列表
    m3u8_url_list = url_list[video_range[0]-1:video_range[1]]
    # 多线程获取m3u8数据进行后续下载操作
    with ThreadPoolExecutor(3) as executor:
        for index in range(len(m3u8_url_list)):
            executor.submit(get_m3u8,  video_range[0]+index, m3u8_url_list[index])
```

# 3. 运行效果

程序运行效果：

![1724759070690.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/4436cafe4d51402c96d1bd35bf9ccf84~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=cuw%2FEhrD8iddybmnOMwXPjEb%2BZs%3D)

运行时下载的ts片段文件如图：

![1724759274992.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/aa71f8f361a2480ea08fe7763260435a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=tNa65OgLRb8Wy0IBAnC2D6fAJzc%3D)

下载完成执行完合并和转格式操作后：

![1724759806562.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/02708c256423477cac9073f7ec9bfc26~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=D%2Fnr73%2BafFMLTMWiX4jtkUoNWFs%3D)

正常播放：

![1724759879209.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/7c6e0dbd6ea5404e81b5e69e08c1a7fc~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgeWht:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiOTY0MTI3NTY2MjIwMzAifQ%3D%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727809185&x-orig-sign=Iey9zclET3Sm0oSvV5YL6bRlZfE%3D)

***

如果文章中有哪里没有讲明白，或者讲解有误的地方，欢迎在评论区批评指正。\
感谢阅读 ( ´▽｀)

