# python实现葫芦侠刷评论脚本
[quote color="success"]众所周知（并没有），我之前提到过我有一个葫芦侠账号，葫芦侠呢也算是陪伴我比较久的一个平台了，从最早期的找破解版游戏，到现在的各种杂项技术分享以及娱乐交友等等，也都能勉强完成。而且混了这么久勉强混到了好多贡献值，对了，还有一个奖状（至于这个我也没想到），多多少少有点感情了。但是转眼一看自己的评论，还不到一万条，多多少少有点作为老用户的羞耻，但是平时我也没啥时间去挨着挨着评论，突然想到之前看到那么多机器人评论，干脆自己用python写一个，开始干活（水文章）[/quote]
开局先秀波图 :@(脸红) 
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041919100874.jpg)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419191001629.jpg)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419191130294.jpg)
首先，要实现脚本自动评论，我们先通过抓包软件了解一个评论的过程用到了哪些链接和数据
这里用黄鸟抓包发现，一个评论总共产生了三条数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419191612132.jpg)
经过判断，中间这条post数据解析到了刚刚提交的评论数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419191739839.jpg)
接着我们来详细分析一下这条数据包，首先我们看到headers信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419191925384.jpg)
简略说一下，从post数据包中的url参数可以很明显看到诸如key以及device_code之类的字眼，可以推断出url参数是由葫芦侠App基于设备码以及cookie生成的，没必要花时间去解，登录状态后直接抓包获取即可，经测试只要不手动logout，第一条产生的key和device_code可持续使用，我们接着看post请求的主体
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419193130518.jpg)
格式化一下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419193155415.jpg)
我们可以看到，一共有六个参数，挨着来分析
第一个参数post_id，经过筛选数据发现是评论贴子的id（划重点，后面的刷评论会用到）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419193515484.jpg)
第二个参数comment_id，推算和测试后确定是评论的楼层，从0开始计数，此参数对数据包构造影响不大，略过
第三个参数text，从post数据主体的截图我们可以看到，源数据是经过URL编码后的评论内容，后面刷评论需要用到
第四个参数patcha，貌似是评论频率过高会要求输入验证码（目前没有遇到过，貌似是因为我没怎么评论？暂且略过）
第五个参数images，这个就不多讲，明显是评论中的图片，刷评论应该不怎么遇得到，暂且略过，有大佬需要的话可以研究研究
第六个参数remindUsers，经过测试确定是评论回复的用户名，刷评论不怎么用得上，如果做自动回复功能可能会用到，暂且保留
主体大致分析完毕，接下来我们开始构造python代码
首先实现基础评论功能，我们需要用到的模块是requests模块，基于python3结构编写，使用python2的同学记得改改
requests模块应该不是自带的，记得先pip安装一下
首先导入requests模块

    import requests

构造headers头部信息，这里是参考，因为葫芦侠经常更新，具体内容根据抓包得到的数据为准

    headers = {
        'Connection': 'close',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length': '106',
        'Host': 'floor.huluxia.com',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/3.8.1'
    }

接下来设置评论链接以及post数据

    comment_url = "http://floor.huluxia.com/comment/create/ANDROID/2.0?platform=2&gkey=000000&app_version=4.1.1.4&versioncode=324&market_id=tool_web&_key=075B357C3926F30BCC22DF6DDF35BF834AEDEC0CEF7DF73857CBC415C5A4B60A3437D077C741C925CB867319372E116A265F603385488BA9&device_code=%5Bd%5Dd113ce76-27bb-469a-a8b5-0d704ea275a7&phone_brand_type=MI"
    post_data = "post_id=47723375&comment_id=0&text=%E6%B5%8B%E8%AF%95%5B%E6%BB%91%E7%A8%BD%5D&patcha=&images=&remindUsers="

最后使用requests模块的post方法提交数据并获取返回值
最终得到的代码如下

    import requests
    #设置头部信息
    headers = {
        'Connection': 'close',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length': '106',
        'Host': 'floor.huluxia.com',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/3.8.1'
    }
    
    #设置回复url及post数据
    post_data = "post_id=47723375&comment_id=0&text=%E6%B5%8B%E8%AF%95%5B%E6%BB%91%E7%A8%BD%5D&patcha=&images=&remindUsers="
    comment_url = "http://floor.huluxia.com/comment/create/ANDROID/2.0?platform=2&gkey=000000&app_version=4.1.1.4&versioncode=324&market_id=tool_web&_key=075B357C3926F30BCC22DF6DDF35BF834AEDEC0CEF7DF73857CBC415C5A4B60A3437D077C741C925CB867319372E116A265F603385488BA9&device_code=%5Bd%5Dd113ce76-27bb-469a-a8b5-0d704ea275a7&phone_brand_type=MI"
    
    #post提交评论数据
    response = requests.post(url=comment_url, data=post_data, headers=headers)
    print(response.text)

运行一下，成功返回
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419200749833.png)
评论成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210419200904112.jpg)
基础功能写好了，大致也就差不多了，刚刚是实现了发一条评论
要想达到刷评论的效果，需要解决以下问题：持续评论，评论频率过快，无法连续评论相同内容等等
这里给的思路是轮询或者循环语句，适当调整评论间隔，随机字符或调用机器人api
所以我们期待一下加亿点细节，加上细节后的代码是这样的

    import requests
    import random
    import urllib.parse
    import json
    import time
    
    # 设置头部信息
    headers = {
        'Connection': 'close',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length': '106',
        'Host': 'floor.huluxia.com',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/3.8.1'
    }
    
    
    # 生成随机评论
    def random_comment():
        post_id = random.randint(40000000, 50000000)
        comment_id = random.randint(1, 10)
        comment_api = "http://api.btstu.cn/yan/api.php?charset=utf-8"
        comment = urllib.parse.quote("%s" % comment_id + "[睡觉][茶杯]")
        # comment_info = requests.get(comment_api, verify=False)
        # comment = urllib.parse.quote("每日毒鸡汤时间：" + comment_info.text + "[滑稽]")
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    def main():
        # 设置回复url及post
        post_data = random_comment()
        comment_url = "http://floor.huluxia.com/comment/create/ANDROID/2.0?platform=2&gkey=000000&app_version=4.1.1.4&versioncode=324&market_id=tool_web&_key=6862A43A7126C3C5D7AB0BFE61BAE16FA5852495DFCFB4DCD5D1D34F3B9678F18C11DF30A4EAC58ED693FD5A1E4E7DCA522FA2837AB0F714&device_code=%5Bd%5Dd113ce76-27bb-469a-a8b5-0d704ea275a7&phone_brand_type=MI"
        post_url = "http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=%d" % post_data[1]
    
        # 获取帖子信息(1)
        post_response = requests.get(post_url)
        post_json_data = json.loads(post_response.text)
    
        # post提交评论数据
        print(comment_url)
        print(post_data[0])
        print(headers)
        response = requests.post(url=comment_url, data=post_data[0], headers=headers)
        json_data = json.loads(response.text)
        if json_data['code'] == 104:
            print("----------------------------")
            print("话题不存在，跳过此ID")
        else:
            # 获取帖子信息(2)
            post_info = post_json_data['post']
            post_category = post_info['category']
            # 返回状态信息
            print("----------------------------")
            print("评论状态：" + json_data['msg'])
            print("评论内容：" + post_data[2])
            print("帖子名称：" + post_info['title'])
            print("所属版块：" + post_category['title'])
    
    
    for i in range(1, 99999):
        main()
        time.sleep(5)

可以算是这个脚本的1.0版本吧
不过，经过前几天的脚本测试，我发现有不少id的帖子存在话题被删除或者话题不存在等等状况，在执行代码的过程中是很影响效率的
所以需要有一个脚本专门用来收集有效帖子id


----------


首先我们还是通过抓包来获取帖子的主体信息，最终我们得到这样一个url

    http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=1

这个url返回的是一串json结构的数据，我们格式化一下方便看清结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420205547585.png)

我们再来看一个被删除的帖子链接

    http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=112333

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420205756579.png)

最后来看一个不存在的帖子链接

    http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=112333231221321

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420205937760.png)

观察一下他们的特征，被删除和不存在的帖子返回的特征更容易被爬虫捕捉，我们就以这两种情况作为判定条件
开始构造python代码，首先导入我们要用到的模块并定义好url

    import requests
    import json
    
    url = "http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=%s"

接着我们写一个for循环并从1开始依次赋值，用以遍历帖子id，并使用json.loads解析返回的json数据

    for post_id in range(1, 50000000):
        post_data = requests.get(url % post_id)
        post_json_data = json.loads(post_data.text)

从刚刚的几张图片可以很明显看到特征：
帖子被删除的状态，返回的title是**/* 话题已删除 */**
帖子不存在的状态，返回的json中有个键值对是**"code":104**
除去这两种状态，剩下的即为正常
由此可以写出一个if-elif循环来匹配字符串

    if '"code":104' in post_data.text:
        print("帖子ID：%d" % post_id + " 状态：话题不存在")
    elif "/* 话题已删除 */" in post_data.text:
        print("帖子ID：%d" % post_id + " 状态：话题已删除")
    else:
        print("帖子ID：%d" % post_id + " 状态：此话题正常")

满足“此话题正常”条件时，我们write功能将有效id写入文本文件
顺便另一个文本文件随着循环的id持续更新，方便下次回溯爬取进度
完整的代码如下：

    import requests
    import json
    
    url = "http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=%s"
    
    for post_id in range(29042, 50000000):
        post_data = requests.get(url % post_id)
        post_json_data = json.loads(post_data.text)
        # print(post_json_data)
        if '"code":104' in post_data.text:
            print("帖子ID：%d" % post_id + " 状态：话题不存在")
        elif "/* 话题已删除 */" in post_data.text:
            print("帖子ID：%d" % post_id + " 状态：话题已删除")
        else:
            print("帖子ID：%d" % post_id + " 状态：此话题正常")
            valid_id = open("valid_id.txt", "a")
            valid_id.write(str(post_id))
            valid_id.write("\n")
            valid_id.close()
            end_id = open("end_id.txt", "w")
            end_id.write(str(post_id))
            end_id.close()

运行脚本后会将有效id输出到文件valid_id.txt
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021042114063934.png)
格式是这样子
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421140812368.png)
[quote color="primary"]那么接下来，我们就可以利用这个脚本生成的文件来自动获取有效id了，效率提升了不少
经过完善后添加了以下功能：
通过读取文件内容自动获取评论，可随时更改评论配置文件，添加评论语句
自动识别帖子所属版块并选择不同的评论配置文件（笨办法，重复写，代码有点冗杂，因为太菜了，希望有大佬帮我优化优化，嘤嘤嘤）[/quote]
完整代码如下：

    import requests
    import random
    import urllib.parse
    import json
    import time
    import linecache
    
    # 设置账号key链接
    comment_url = "http://floor.huluxia.com/comment/create/ANDROID/2.0?platform=2&gkey=000000&app_version=4.1.1.4&versioncode=324&market_id=tool_web&_key=6862A43A7126C3C5D7AB0BFE61BAE16FA5852495DFCFB4DCD5D1D34F3B9678F18C11DF30A4EAC58ED693FD5A1E4E7DCA522FA2837AB0F714&device_code=%5Bd%5Dd113ce76-27bb-469a-a8b5-0d704ea275a7&phone_brand_type=MI"
    
    # 设置头部信息
    headers = {
        'Connection': 'close',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length': '106',
        'Host': 'floor.huluxia.com',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/3.8.1'
    }
    
    
    # 获取文本行数，用以随机评论
    def get_lines(filept):  # filept传递文件路径
        f = open(file=filept, encoding='utf-8')  # 根据文件内容调整encoding编码
        n = len(f.readlines())
        return n
    
    
    # 获取帖子ID
    def get_post_id():
        lines = get_lines(filept="valid_id.txt")  # 配合post_scan.py收集到的有效帖子id进行读取
        id_2 = random.randint(1, lines)  # 随机获取行数
        post_id = linecache.getline(filename="valid_id.txt", lineno=id_2)  # 每行一个帖子id
        return int(post_id)
    
    
    # 获取帖子信息
    def get_post_info(post_id):
        post_url = "http://floor.huluxia.com/post/detail/ANDROID/4.1?post_id=%d" % post_id
        post_response = requests.get(post_url)
        post_json_data = json.loads(post_response.text)
        return post_json_data
    
    
    # 判断帖子所属版块
    def category_decide(post_info):
        post = post_info['post']
        category = post['category']
        return category['title']
    
    
    # 实用软件类评论生成，以”感谢分享，拿走了“为主要意图的评论为主
    def comment_syrj(post_id):
        post_id = post_id
        lines = get_lines(filept="comment/syrj.txt")  # 获取回复配置文件行数
        flag = random.randint(1, lines)  # 配合下一语句，随机从回复配置文件中抽取行数
        comment = linecache.getline(filename="comment/syrj.txt", lineno=flag)
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="  # 评论post数据主体
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    # 新番源类评论生成，以”挺好看，感谢分享“为主要意图的评论为主
    def comment_xfy(post_id):
        post_id = post_id
        lines = get_lines(filept="comment/xfy.txt")  # 获取回复配置文件行数
        flag = random.randint(1, lines)  # 配合下一语句，随机从回复配置文件中抽取行数
        comment = linecache.getline(filename="comment/xfy.txt", lineno=flag)
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="  # 评论post数据主体
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    # 泳池类评论生成，以较水的评论为主
    def comment_yc(post_id):
        post_id = post_id
        lines = get_lines(filept="comment/yc.txt")  # 获取回复配置文件行数
        flag = random.randint(1, lines)  # 配合下一语句，随机从回复配置文件中抽取行数
        comment = linecache.getline(filename="comment/yc.txt", lineno=flag)
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="  # 评论post数据主体
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    # 许愿类评论生成，以“感谢分享，等大佬分享”为主要意图的评论为主
    def comment_xy(post_id):
        post_id = post_id
        lines = get_lines(filept="comment/xy.txt")  # 获取回复配置文件行数
        flag = random.randint(1, lines)  # 配合下一语句，随机从回复配置文件中抽取行数
        comment = linecache.getline(filename="comment/xy.txt", lineno=flag)
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="  # 评论post数据主体
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    # 美腿类评论生成，以夸奖称赞的评论为主
    def comment_mt(post_id):
        post_id = post_id
        lines = get_lines(filept="comment/mt.txt")  # 获取回复配置文件行数
        flag = random.randint(1, lines)  # 配合下一语句，随机从回复配置文件中抽取行数
        comment = linecache.getline(filename="comment/mt.txt", lineno=flag)
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="  # 评论post数据主体
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    # 游戏类评论生成，以“是否玩过以及立场”为主要意图的评论为主
    def comment_yx(post_id):
        post_id = post_id
        lines = get_lines(filept="comment/yx.txt")  # 获取回复配置文件行数
        flag = random.randint(1, lines)  # 配合下一语句，随机从回复配置文件中抽取行数
        comment = linecache.getline(filename="comment/yx.txt", lineno=flag)
        comment_data = "post_id=%d" % post_id + "&comment_id=0&text=%s" % comment + "&patcha=&images=&remindUsers="  # 评论post数据主体
        return comment_data, post_id, urllib.parse.unquote(comment)
    
    
    # 主函数，用以发送评论数据包主体
    def main():
        post_id = get_post_id()
        post_info = get_post_info(post_id)
        # print(str(post_info))
        # print(type(post_info))
        if "'code': 104" in str(post_info):  # 判断帖子是否有效
            print("----------------------------")
            print("话题不存在，跳过此ID")
        elif "/* 话题已删除 */" in str(post_info):
            print("----------------------------")
            print("话题已删除，跳过此ID")
        else:  # 不满足上面两种特殊条件时说明帖子内容正常，进行下一步构造数据包
    
            category = category_decide(post_info)  # 获取帖子所属版块，用以选择不同版块的回复配置文件
            post_data = post_info['post']  # 获取帖子信息，用于后续打印返回状态
    
            if category == "实用软件" or category == "手机美化" or category == "玩机教程" or category == "技术分享" or category == "原创技术" or category == "福利活动" or category == "三楼学院" or category == "头像签名":  # 以实用软件版块为主的类似版块
                comment_data = comment_syrj(post_id)  # 评论数据
                response = requests.post(url=comment_url, data=comment_data[0].encode('utf-8'),
                                         headers=headers)  # 接收评论返回数据包
                response_json = json.loads(response.text)  # 评论返回数据包转换为json/dict格式，用于后续读取键值
                # 返回评论状态信息
                print("----------------------------")
                print("评论状态：" + response_json['msg'])
                print("评论内容：" + comment_data[2])
                print("帖子名称：" + post_data['title'])
                print("所属版块：" + category)
            elif category == "新番源" or category == "三两影":
                comment_data = comment_xfy(post_id)  # 评论数据
                response = requests.post(url=comment_url, data=comment_data[0].encode('utf-8'),
                                         headers=headers)  # 接收评论返回数据包
                response_json = json.loads(response.text)  # 评论返回数据包转换为json/dict格式，用于后续读取键值
                # 返回评论状态信息
                print("----------------------------")
                print("评论状态：" + response_json['msg'])
                print("评论内容：" + comment_data[2])
                print("帖子名称：" + post_data['title'])
                print("所属版块：" + category)
            elif category == "泳池" or category == "娱乐天地" or category == "恶搞" or category == "模型玩具" or category == "玩机广场":
                comment_data = comment_yc(post_id)  # 评论数据
                response = requests.post(url=comment_url, data=comment_data[0].encode('utf-8'),
                                         headers=headers)  # 接收评论返回数据包
                response_json = json.loads(response.text)  # 评论返回数据包转换为json/dict格式，用于后续读取键值
                # 返回评论状态信息
                print("----------------------------")
                print("评论状态：" + response_json['msg'])
                print("评论内容：" + comment_data[2])
                print("帖子名称：" + post_data['title'])
                print("所属版块：" + category)
            elif category == "许愿" or category == "制图工坊":
                comment_data = comment_xy(post_id)  # 评论数据
                response = requests.post(url=comment_url, data=comment_data[0].encode('utf-8'),
                                         headers=headers)  # 接收评论返回数据包
                response_json = json.loads(response.text)  # 评论返回数据包转换为json/dict格式，用于后续读取键值
                # 返回评论状态信息
                print("----------------------------")
                print("评论状态：" + response_json['msg'])
                print("评论内容：" + comment_data[2])
                print("帖子名称：" + post_data['title'])
                print("所属版块：" + category)
            elif category == "美腿" or category == "次元阁" or category == "自拍":
                comment_data = comment_mt(post_id)  # 评论数据
                response = requests.post(url=comment_url, data=comment_data[0].encode('utf-8'),
                                         headers=headers)  # 接收评论返回数据包
                response_json = json.loads(response.text)  # 评论返回数据包转换为json/dict格式，用于后续读取键值
                # 返回评论状态信息
                print("----------------------------")
                print("评论状态：" + response_json['msg'])
                print("评论内容：" + comment_data[2])
                print("帖子名称：" + post_data['title'])
                print("所属版块：" + category)
            elif category == "游戏" or category == "Steam" or category == "LOL手游" or category == "王者荣耀" or category == "DNF手游" or category == "和平精英" or category == "使命召唤手游" or category == "穿越火线" or category == "原神" or category == "跑跑卡丁车" or category == "QQ飞车" or category == "球球大作战" or category == "我的世界" or category == "英雄联盟" or category == "地下城与勇士" or category == "明日之后" or category == "天天酷跑" or category == "新游推荐":
                comment_data = comment_yx(post_id)  # 评论数据
                response = requests.post(url=comment_url, data=comment_data[0].encode('utf-8'),
                                         headers=headers)  # 接收评论返回数据包
                response_json = json.loads(response.text)  # 评论返回数据包转换为json/dict格式，用于后续读取键值
                # 返回评论状态信息
                print("----------------------------")
                print("评论状态：" + response_json['msg'])
                print("评论内容：" + comment_data[2])
                print("帖子名称：" + post_data['title'])
                print("所属版块：" + category)
            else:
                print("----------------------------")
                print("无预先定义版块：" + category + "，跳过")
    
    
    for i in range(1000):
        main()
        time.sleep(6)

具体的功能描述都在注释里面，我就不多做解释了
评论配置文件以comment参数读取的文件名为主，放在同目录下comment目录内，配置文件需要utf-8编码储存，否则脚本可能会报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421142121506.png)

所有的代码我都会放在github里，后续可能会有更新吧，想关注后续更新的大佬们可以star一手（不要脸的求波follow）
github项目链接：
<a href="https://github.com/Y5neKO/huluxia_auto_comment" target="_blank">https://github.com/Y5neKO/huluxia_auto_comment</a>
