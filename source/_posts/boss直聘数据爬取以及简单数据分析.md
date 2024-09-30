---
title: boss直聘数据爬取以及简单数据分析
typora-root-url: ./boss直聘数据爬取以及简单数据分析
date: 2024-08-28 03:01:32
tags:
    - python
    - 爬虫
    - 数据分析
categories: 
    - 实战
description: 爬取boss网页数据，并对数据进行存储，使用tableau进行简单数据分析
---

最近面临找工作，想看看现在深圳前端招聘环境怎么样，想着写个爬虫获取一下boss上面的前端岗位招聘信息，然后做一个简单的数据分析

**结果爬取数据的时候发现有3个主要问题：**

1.  网站搜索最多只能查询10页的数据，超过10页就查找不了
2.  这个网站反反爬好难，本菜鸡能力有限暂时还解决不了这种，而且zp\_stoken的生成验证了好多环境实在也不会弄
3.  请求频繁了还会触发图形验证，而且还据说很容易封ip，处理起来都好麻烦(╥﹏╥)

**但是条条大路通罗马，没有好办法，总还有笨办法可以进行解决的嘛，本菜鸡翻来覆去，勉强想到几个笨办法：**

1.  既然只能查找10页数据，那我就多换几个筛选条件，最后综合综合一下数据嘛，本来我也只是想搜搜深圳的前端岗位，干脆就按区域和工作年限来分别查数据好了，这样总数据量不就上来了嘛
2.  虽然zp\_stoken生成时验证的环境我弄不好，但是我可以直接用rpc远程调用来生成zp\_stoken，直接用浏览器的环境，我不就可以不用解决环境问题啦(◔౪◔)
3.  感觉请求如果没有很快，好像也不那么容易触发图形验证，而且我本身爬取数据量也不大，所以碰到图形验证，就直接手动验证就可以啦，然后通过生成随机数，每次请求间隔一些时间，感觉应该能够避免些被封ip的风险吧

# 1. zp\_stoken生成

请求信息的时候，`cookie`中`zp_stoken`这个参数很重要，它就是反爬的关键参数。如果请求携带的`zp_stoken`不对，返回的数据只能是`您的访问行为异常`，如图

![图片](./pic0.png)

而`zp_stoken`的生成，就是来源于上面请求返回的`zpData`中的`seed`和`ts`

那么接下来，就是找到生成`zp_stoken`调用的函数方法，然后注入`ws服务`，这样之后每次调用`requests`请求的时候，如果`zp_stoken`失效，直接调用方法传入`seed`和`ts`，就能得到新的有效的`zp_stoken`啦\~

直接全局搜索`__zp_stoken__`，如图，可以发现`zp_stoken`生成调用的方法就是圈出的位置

![图片](./pic1.png)

接下来就是使用`rpc远程调用`，具体操作方法可以参考[js逆向之rpc远程调用](https://blog.csdn.net/weixin_51111267/article/details/130435642)

![图片](./pic2.png)

![图片](./pic3.png)

浏览器注入`ws服务`如上，然后就是python开启调用ws服务，如下：

```python
import asyncio
import websockets

connected = set()


async def socket_server(websocket):
    connected.add(websocket)
    try:
        async for message in websocket:
            for conn in connected:
                if conn != websocket:
                    await conn.send(message)
    finally:
        connected.remove(websocket)

start_server = websockets.serve(socket_server, "127.0.0.1", 8080)
asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()
```

最后就是通过`flask`部署到本地服务上，供后续请求调用，如下：

```python
import asyncio
import json

import websockets
from flask import Flask,request
app = Flask(__name__)
loop = asyncio.get_event_loop()


async def hello(message):
    # 连接 websocket 并发送消息 获取相应
    async with websockets.connect("ws://127.0.0.1:8080") as websocket:
        await websocket.send(message)
        return await websocket.recv()


def get_token(seed, ts):
    return str(loop.run_until_complete(hello(json.dumps({
        "seed": seed,
        "ts": ts
    }))))


@app.route('/',methods = ['POST', 'GET'])
def hello_world():
    if request.method == 'GET':
        print(request.args)
        seed = request.args.get('seed')
        ts = request.args.get('ts')
        return get_token(seed, ts)


if __name__ == '__main__':
    app.run(host="0.0.0.0",port=80,debug=True)
```

调用效果如下：

![图片](./pic4.png)

# 2. 请求封装和图形验证提示

每一次请求都需要有效的`zp_stoken`，那当然是用函数统一封装一下请求比较好啦，如下：

```python
def request_method(url, data):
    print(f'params: {data}')
    responses = requests.get(url, params=data,cookies=cookies, headers=headers)
    print(f'res: {responses.text}')
    # 如果返回请求内容code为37，说明__zp_stoken__失效，需要重新获取
    if responses.json()['code'] == 37:
        res = requests.get(f'http://127.0.0.1:80', params={
            'seed': responses.json()['zpData']['seed'],
            'ts': responses.json()['zpData']['ts'],
        })
        cookies['__zp_stoken__'] = res.text
        return request_method(url, data)
    elif responses.json()['code'] == 35:
        print('请进行手动图形验证！！！！')
        # 放个音频提醒进行手动验证
        play_mp3()
        return request_method(url, data)
    data = responses.json()['zpData']
    responses.close()
    return data
```

返回`code`为`35`的时候，说明触发`图形验证`了，需要到浏览器里`手动进行验证`，因为爬取数据时间较长，所以我用`pygame`来播放音频来进行提示，免得我注意不到哈哈哈

```python
def play_mp3():
    # 初始化 pygame
    pygame.mixer.init()
    # 加载音频文件
    pygame.mixer.music.load("temp/心电心-王心凌.mp3")
    # 播放音频
    pygame.mixer.music.play(-1)
    while True:
        text = input('是否已经手动验证完毕？')
        if text == 'y':
            pygame.mixer.music.stop()
            print("音频已停止播放")
            # 卸载音频模块
            pygame.mixer.quit()
            # 卸载所有 pygame 模块
            pygame.quit()
            print("pygame 已成功清理和退出")
            break
```

# 3. 请求岗位数据

由于只能请求`10页`数据，我要先获取城市各个`区域列表`和`工作年限的列表`数据，然后再通过循环遍历的方式获取岗位数据，所有获取的数据我会先放入`csv文件`中

```python
csv_head = ['jobName','brandName','cityName','areaDistrict','businessDistrict','salaryDesc','jobExperience','jobDegree','bossName','bossTitle','brandIndustry','brandStageName','brandScaleName','skills','welfareList','securityId','lid','jobLabels','postDescription','address','activeTimeDesc']
headers = {
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36',
}
cookies = {
    '__zp_stoken__': '',
}
joblist_params = {
    'scene': '1',
    'query': '前端',  # 搜索关键字
    'city': '101280600',  # 城市（深圳）
    'experience': '',  # 工作经验
    'multiBusinessDistrict': '',  # 城市区域
    'page': '1',
    'pageSize': '30',
}


if __name__ == '__main__':
    # 获取城市的各个区域
    responses = request_method('https://www.zhipin.com/wapi/zpgeek/businessDistrict.json', {
        'cityCode': joblist_params['city'],
    })
    district_code = []
    for obj in responses['businessDistrict']['subLevelModelList']:
        district_code.append({
            "code": obj['code'],
            "name": obj['name']
        })
    # 获取需要请求的工作经验范围
    experience_code = request_method('https://www.zhipin.com/wapi/zpgeek/search/job/condition.json', {})['experienceList'][3:7]
    with open('temp/bossData.csv', 'w',newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(csv_head)
        # 开始循环请求数据
        for district in district_code:
            for experience in experience_code:
                joblist_params['multiBusinessDistrict'] = district['code']
                joblist_params['experience'] = experience['code']
                joblist_params['page'] = '1'
                print(f"开始请求数据，区域：{joblist_params['multiBusinessDistrict']}；工作经验：{joblist_params['experience']}")
                query_joblist(joblist_params, writer)
                print(f"数据请求完毕，区域：{joblist_params['multiBusinessDistrict']}；工作经验：{joblist_params['experience']}")
```

岗位数据获取方法如下，防止请求过于频繁被封ip啥的，所以我会生成0-5的`随机数`，在每次请求完成后，休眠几秒，这样子虽然会把爬取数据时间拉长，但是起码稳定些也不用怕被封ip啦

```python
# 获取岗位详细描述信息
def get_content(security_id, lid):
    response = request_method('https://www.zhipin.com/wapi/zpgeek/job/card.json', {
        'securityId': security_id,
        'lid': lid,
        'sessionId': '',
    })
    return response['jobCard']


def sleep():
    # 访问过于频繁会出发图形验证，通过随机数等待几秒后再请求
    random_integer = random.randint(0, 5)
    print(f'等待{random_integer}秒')
    time.sleep(random_integer)

# 请求数据
def query_joblist(params, writer):
    # 获取页面数据
    responses = request_method('https://www.zhipin.com/wapi/zpgeek/search/joblist.json', params)
    for i in responses['jobList']:
        obj = re.compile(f"{joblist_params['query']}", re.S)
        # 搜索的数据职位名称不包含搜索关键字的就不进行后续的详情查询和数据写入csv了
        if len(obj.findall(i['jobName'])) == 0:
            continue
        sleep()
        i.update(get_content(i['securityId'], i['lid']))
        data = []
        for key in csv_head:
            data.append(i[key])
        writer.writerow(data)
    # 有下一页数据，则循环请求下去（boss最多请求10页数据）
    if responses['hasMore'] == True:
        params['page'] = int(params['page']) + 1
        sleep()
        query_joblist(params, writer)
```

# 4. 运行效果

个人实际体验爬取数据两千多条，大概花费可能2小时左右，触发图形验证频率，还算能接受，具体的爬取效果如下：

![图片](./pic5.png)

爬取完成生成文件效果：

![图片](./pic6.png)

# 5. 数据分析

前面爬取的数据都是直接存入`csv文件`中，如果对数据进行进一步处理，存入到`数据库`中（可选），就可以用这些数据进行简单的`数据分析`啦，如图是我用爬取的数据做的一些简单的数据分析（不知为啥好大一部分爬取的岗位数据活跃度不高耶，半年前活跃？）

![图片](./pic7.png)

***

以上就是我实现的一个简单爬取boss直聘岗位数据并且进行数据分析的案例

如果文章中有哪里没有讲明白，或者讲解有误的地方，以及如果有更优美的实现方式的话，欢迎在评论区指导。

感谢阅读 ( ´▽｀)
