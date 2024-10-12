+++
title = '从零开始的赛博老婆！4 - 短期记忆与记忆体系'
date = 2024-10-11T23:23:47-02:30
draft = false

+++

![](/images/94008036_p0_master1200.jpg)

PIXIV: 94008036  @苏翼丶

# Agent记忆体系浅谈

（我写完这篇文章再重新回看，记忆这两个词太多了，我发现我不认识“记忆”这两个字了，大家点点赞吧写教程真的消耗阳寿……(｡ ́︿ ̀｡)

从GPT3破圈那一刻起，大家对LLM的记忆体系就众说纷纭。从技术上来讲，一方面Transformer架构衍生出的一些技术促进了记忆体系的发展（如 Embedding/Vector Database），另一方面有需求催生了另一些技术（如 Graph Database）。但就从2023年发展到现在，Agent的记忆体系进展比较缓慢。

我在这些技术和概念的帮助下，设计了一套针对于赛博老婆的记忆体系。它的特点就是自动存取，同时可以保证分级的持久化记忆。

## 记忆概念分类

我想把赛博老婆的记忆体系分成三类：短期记忆，中期记忆，长期记忆。我们来分别解释这几个概念：

### 短期记忆

短期记忆有多短期呢？我认为短期记忆是包含最丰富的，最原始的信息的。就像人的记忆一样，人类每天都会忘掉很多事情的很多细节，但是你可能会记得刚刚发生的事情的并能够说出细节。我认为**在LLM还没有开始*总结或简化*那一部分最原始的记忆之前的内容**，都算作短期记忆。或者通俗一点讲，对于赛博老婆，**每一个会话的上下文**，或者**每一天当天（昨天）的记忆**，都算作短期记忆。因为这一部分记忆，包含了你和赛博老婆交互的原始记录。但是在这里，我想稍微延伸一点：我把当天的总结记忆和每一次会话的记忆也算作短期记忆之内。之所以如此规定，是因为在实际的记忆载入中，这类的近期记忆往往频率更高，同时需要相对详细的细节，他们往往是一起无条件载入的。也就是说，上下文记忆，默认是必要的，而上几次的对话，或者昨天聊了什么，也会有更大概率被提起，他们的载入频率也差不多，都很高。

> **会话：**你和赛博老婆互动，聊了几句，然后你就去做别的事情了，比如说吃了个饭。当你吃完饭，可能已经过了会话超时时间，而你又不想继续之前的话题，而是聊点别的，于是赛博老婆就会开一个新的会话。这在后续的记忆结构化中有利于**实体提取**与**记忆网**的生成。

### 中期记忆

中期记忆有别于短期记忆，中期记忆是经过简化和总结的，也就是丢弃掉了某些细节而得到的记忆。但我们为什么称之为中期记忆呢？这仍然是属于对人脑记忆的模仿，人脑会暂存一部分正在进行的事情，并简化其中的内容，等到这个事件结束了，有了结果了，就会大幅度遗忘，可能最后只会保留一个印象，类似于：我曾经做过什么事情，当时的结果是……而对曾经事情的过程和细节，完全遗忘。而这类中期记忆也不会对永久记忆网产生较大的影响。

我们在这里举例说明：你在某个网购官方平台购买了一个全新显卡RTX 9090 Ti Super Plus Max Ultra 1024TB，你付款商家发货寄出快递。由于你一直心心念念这款超强的显卡，你一直记着有这么个快递在路上，于是没事儿就点开查询快递到哪里了。**这就是属于中期记忆的*“事件中”***。过了几天，你的快递终于到了，你非常开心，迫不及待地拆了包装，安装上去，这时你可能已经忘记自己看了多少次快递跟踪了，也忘了它是几天内到达的，因为这个时候你很想把它安装到电脑上看看效果。这就是中期记忆的**遗忘特性**，**阶段性目标达成**可能会遗忘掉很多细节，但是事件存在周期又**远超**“短期记忆”。你安装到了电脑，但是你发现用这个显卡跑LLM慢的要死，然后你惊讶的发现，由于某黄的精准刀法，RTX 9090 Ti Super Plus Max Ultra 1024TB虽然拥有超大的显存，但是其显存位宽却只有可怜的*128bit*——连下一代的10070 Ti Super Plus Max Ultra Gaming的升级版本的*150bit*都没有！在你不得不感叹某黄精准刀法的同时不由得骂：“Fxxk you Nvxdxa !!” 你很难过，在某海鲜市场折价出手，期间还没事就看一看某鱼，生怕买家到手刀或者无理由退款等逆天操作……终于，你是幸运的，你成功地卖出了显卡，这段风波画上句号。等到几年后你再想起这件事情，可能只会记得那印象深刻的阉割和那句“Fxxk you Nvxdxa !!”

这就是中期记忆的特点了，不是所有记忆都会变成中期记忆的，只是某些特定事件，周期不短不长的才有资格被称为中期记忆。另外，上述案例是虚构的，绝对不是真实事件改编！！！

### 长期记忆

好，终于聊到这里了。我们要开始加速了。

人脑的长期记忆是一个非常复杂的系统。我们搞赛博老婆只能尽量**平衡**“精简”和“功能”。然而，赛博老婆可不像人，当你真的需要她回忆很重要的细节的时候，可不能像人一样，卖个萌然后糊弄过去。赛博老婆的长期记忆需要能够**回溯**到最原始的对话记忆或相关文档。同时，大多数情况，仅仅是一些**关键信息**的提示。这就对索引和查询系统提出了很高的要求，长期记忆系统设计的关键，就是如何***高效而精确***地自动存取，尤其是***取***。

我把长期记忆的查询和索引分成两种模式，“模糊检索”和“关系检索”，对应RAG和Graph Query。这只是两种不同的检索方法，实际情况大多是混合的，也就是并行的。为什么要这么做呢？我们来看实际例子。

模糊检索：你需要让赛博老婆回忆一下你曾经都在哪几天**消费*超过*500**元。这时，“超过500元”是一个语意，他是和价值/开支相关的，这种情况记忆执行的是语意数量的范围检索，是相对模糊的。（因为记忆库是通用的，我们没有一个专门的账本数据库，不然的话直接表达式>500就可以解决）此时，语意模糊检索占大头，而关系检索占小头，关系检索仅作为实体或后续进一步细节查询的补充。

关系检索：你让你的赛博老婆帮你分析你自己现在的**人际网**并找出你人际网中其他人的**潜在联系**。此时，就会用到关系检索。关系检索依赖于图数据库，它会把记忆转化为实体并建立联系，当我们执行实体关系连锁检索时，Query的三元组会激活好几个初始节点，代表初始的查询实体。然后通过检索与这些实体相连接的其他实体的信息，返回结果。这时，与Query在时空上，或语意上或其他任何形式的关联，都会返回。后续可以进行多次的知识网跳跃或者进行网图归类重排，或者定位到某个范围很小的节点群，执行模糊检索等。

在上述案例中，我们不难发现，两大类检索方式经常时**混合**使用的，这也是目前的趋势。实际上，短中长期记忆在实际应用里也是混合使用的，只是可能每一次调用时**侧重点**或者**强度**不同。

## 记忆系统的相关技术

### 非关系型数据库（NoSQL）

例如MongoDB，作为一种非关系型数据库，在存储文本音频和图片等多媒体数据的时候很有用。因为我们所有的原始数据都存在这里，上级数据库溯源到这里，需要原始数据相对方便一点，后续更改添加条目，备份维护都比较简单。MongoDB的python SDK还是很不错的，同时Mongo DB部署的平台支持也不错。

### 嵌入（Embedding）

一种数据量化技术，与图数据库和向量数据库配合，对多媒体数据进行多角度量化（语意，声学特征，图像特征等），便于数据库和LLM进行处理。

### 向量数据库（Vector Database）

存储向量化的嵌入数据和对应的经过总结的二级数据，执行语意搜索，RAG，混合重排等。向量数据库很善于相似搜索，包括图片文本音频等。

### 图数据库（Graph Database）

图数据库善于处理实体关系和网络。通过图数据库，LLM可以高效地存储和处理实体之间的关系网络，这对于需要理解和导航复杂关联的应用场景非常有用。另一个例子是知识图谱的构建，Agent可以利用图数据库存储知识点及其相互关联的信息，并结合互联网搜索帮助你发现新的潜在内容，甚至可以进行时空事件预测。

### 其他相关技术

还有一些本项目的细节可能会用到，比如聚类，回归，Q学习，决策树，降维映射等等等等。由于本篇还算入门，这些内容等后续开对应进阶教程再说。（挖了无数深坑（;￣O￣））

# 短期记忆实战

说了这么多，我们今天先来从相对简单的短期记忆开始实践。

## 三种上下文记忆

这三个记忆类型实际上是继承了曾经Langchain的方式，但其实我们了解了方法之后，可以排列组合，根据不同的场景选择。

### 上下文记忆

我们在第二篇文章中就已经引入了**上下文记忆**这个概念，在这里我们系统性地说一下。第一种上下文记忆是没有限制的，当你的程序在运行，只要后台不关，这个记忆列表就一直存在内存（RAM）里面，如果程序关闭，机器断电，上下文记忆就没了，所以不是**持久记忆**。而且第一种上下文是**无限长**的，聊的多，上下文就无限多，没有卸载机制，所以有很多隐患。这里仅放出代码：

其实就是改了这个：

```python
# 把这个
chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*4:-1]])
# 变成了这个
chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history])
# 找不同
```

完整代码在这里：

```python
from openai import OpenAI

chat_model= OpenAI(
	# 你需要把这个替换成你的后端的API地址
	base_url="https://api.openai.com/v1/",
	# 这是用于身份验证的 API Key
	api_key = "sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"
	)

chat_history = []

def get_response_from_llm(question):

    print(chat_history)

    # 没有了窗口，就是无限长的上下文，但是超出了LLM上下文最大限制会报错
    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history])
    chat_history_prompt = f"Here is the chat history:\n {chat_history_window}"    


    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
        {"role": "assistant", "content": chat_history_prompt},
        {"role": "user", "content": question},
    ]

    print(message)

    response = chat_model.chat.completions.create(
                model='gpt-4o-mini',
                messages=message,
                temperature=0.7,
            )
    
    response_str = response.choices[0].message.content

    return response_str

if __name__ == "__main__":
    while True:
        user_input = input("\n输入问题或者请输入'exit'退出：")
        if user_input.lower() == 'exit':
            print("再见")
            break 
        chat_history.append(('human', user_input))
        response = get_response_from_llm(user_input)
        chat_history.append(('ai', response))
        print(response)

```

### 上下文窗口

```python
# 变成这个就是窗口，也就是仅仅从列表中抽取最近的一部分上下文给LLM，数量固定，由最后面的表达式[-2*n:-1]决定
chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*4:-1]])
```

把n改变成任意数字，就可以控制上下文记忆的轮次了。比如你想让 AI 的记忆有 3 轮，那你就可以改成 `[-7:-1]`，因为一轮对话包含用户输入和机器人的回复条消息嘛，所以是 6 条消息。

### 上下文摘要总结

简单来说，就是计数达到一定的轮次后，抽出来最近的那几条，然后对这个进行总结，减少token使用或者为了中期和长期记忆准备。LLM接受几个总结，并根据经过总结后的上下文继续对话。这种方法会舍弃一部分信息，特定情况使用。
我们首先按照之前的套路新加一个函数用于给聊天记录总结：

```python
def summarize_chat_history(chat_history_window):
    """
    对最近的聊天记录进行总结
    :param chat_history_window: 最近的聊天记录
    :return: 总结后的字符串
    """
    # 创建总结提示
    summary_prompt = f"请总结以下对话内容：\n{chat_history_window}"

    print(f"正在对以下内容生成总结: {summary_prompt}")
    # 调用 LLM 生成总结
    summary_response = chat_model.chat.completions.create(
        model='gpt-4o-mini',
        messages=[{"role": "user", "content": summary_prompt}],
        temperature=0.7,
    )
    # 获取总结内容
    summary_str = summary_response.choices[0].message.content
    return summary_str
```

然后我们需要在之前的LLM响应函数里**添加**一些**计数和判断**，这样好把总结后的内容添加到prompt message里面：

```python
    # 如果聊天记录达到总结阈值
    if len(chat_history) >= SUMMARY_THRESHOLD:
        # 获取最近的聊天记录窗口
        chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-SUMMARY_THRESHOLD*2-1:-1]])
        # 生成总结
        summary = summarize_chat_history(chat_history_window)
        # 将总结添加到总结记录中
        summary_history.append(summary)
        # 保留最后一条聊天记录
        chat_history = chat_history[-1:]
```

这样的话，LLM响应函数会变成这样：

```python
def get_response_from_llm(question):
    """
    获取 LLM 的响应
    :param question: 用户的问题
    :return: LLM 的响应
    """
    global chat_history, summary_history

    # 如果聊天记录达到总结阈值
    if len(chat_history) >= SUMMARY_THRESHOLD:
        # 获取最近的聊天记录窗口
        chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-SUMMARY_THRESHOLD*2-1:-1]])
        # 生成总结
        summary = summarize_chat_history(chat_history_window)
        # 将总结添加到总结记录中
        summary_history.append(summary)
        # 保留最后一条聊天记录
        chat_history = chat_history[-1:]

    # 获取最近的聊天记录窗口
    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*SUMMARY_THRESHOLD:-1]])
    chat_history_prompt = f"Here is the chat history:\n {chat_history_window}"    

    # 创建消息列表
    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
        {"role": "assistant", "content": chat_history_prompt},
        {"role": "user", "content": question},
    ]

    # 如果有总结记录，将其添加到消息列表中
    if summary_history:
        summary_prompt = "\n".join(summary_history)
        message.insert(1, {"role": "assistant", "content": f"Summary of previous conversations:\n{summary_prompt}"})

    # 调用 LLM 获取响应
    response = chat_model.chat.completions.create(
        model='gpt-4o',
        messages=message,
        temperature=0.7,
    )
    
    print(f"message: {message}")
    # 获取响应内容
    response_str = response.choices[0].message.content

    return response_str
```

最后是完整的代码：

```python
from openai import OpenAI

# 初始化 OpenAI 模型
chat_model= OpenAI(
	# 你需要把这个替换成你的后端的API地址
	base_url="https://api.openai.com/v1/",
	# 这是用于身份验证的 API Key
	api_key = "sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"
	)

# 聊天记录和总结记录的列表
chat_history = []
summary_history = []
SUMMARY_THRESHOLD = 4  # 定义达到多少轮次后进行总结

def summarize_chat_history(chat_history_window):
    """
    对最近的聊天记录进行总结
    :param chat_history_window: 最近的聊天记录
    :return: 总结后的字符串
    """
    # 创建总结提示
    summary_prompt = f"请总结以下对话内容：\n{chat_history_window}"

    print(f"正在对以下内容生成总结: {summary_prompt}")
    # 调用 LLM 生成总结
    summary_response = chat_model.chat.completions.create(
        model='gpt-4o-mini',
        messages=[{"role": "user", "content": summary_prompt}],
        temperature=0.7,
    )
    # 获取总结内容
    summary_str = summary_response.choices[0].message.content
    return summary_str

def get_response_from_llm(question):
    """
    获取 LLM 的响应
    :param question: 用户的问题
    :return: LLM 的响应
    """
    global chat_history, summary_history

    # 如果聊天记录达到总结阈值
    if len(chat_history) >= SUMMARY_THRESHOLD:
        # 获取最近的聊天记录窗口
        chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-SUMMARY_THRESHOLD*2-1:-1]])
        # 生成总结
        summary = summarize_chat_history(chat_history_window)
        # 将总结添加到总结记录中
        summary_history.append(summary)
        # 保留最后一条聊天记录
        chat_history = chat_history[-1:]

    # 获取最近的聊天记录窗口
    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*SUMMARY_THRESHOLD:-1]])
    chat_history_prompt = f"Here is the chat history:\n {chat_history_window}"    

    # 创建消息列表
    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
        {"role": "assistant", "content": chat_history_prompt},
        {"role": "user", "content": question},
    ]

    # 如果有总结记录，将其添加到消息列表中
    if summary_history:
        summary_prompt = "\n".join(summary_history)
        message.insert(1, {"role": "assistant", "content": f"Summary of previous conversations:\n{summary_prompt}"})

    # 调用 LLM 获取响应
    response = chat_model.chat.completions.create(
        model='gpt-4o',
        messages=message,
        temperature=0.7,
    )
    
    print(f"message: {message}")
    # 获取响应内容
    response_str = response.choices[0].message.content

    return response_str

if __name__ == "__main__":
    while True:
        user_input = input("\n输入问题或者请输入'exit'退出：")
        if user_input.lower() == 'exit':
            print("再见")
            break 
        # 将用户输入添加到聊天记录中
        chat_history.append(('human', user_input))
        # 获取 LLM 的响应
        response = get_response_from_llm(user_input)
        # 将 LLM 的响应添加到聊天记录中
        chat_history.append(('ai', response))
        # 打印 LLM 的响应
        print(response)
```

## 数据库与记忆持久化

### Mongo数据库部署

我们用Docker部署Mongo DB，大佬不限部署方式。我们暂时安装Docker Engine，你如果喜欢用桌面版的Desktop Docker，请你自行配置。这里大多使用Docker Engine命令行操作和Docker Compose。

首先这里有官方的安装教程：[Install Docker](https://docs.docker.com/engine/install/)

我们使用命令行CMD，依次输入每条指令：

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

如果不出意外的话，会提示docker安装成功的相关信息。

安装成功后，运行这个命令测试你的docker是否正常安装：

```shell
sudo docker run hello-world
```

如果正常，这个命令会打印输出确认信息并退出。命令行输入这条信息检测是否一起安装了compose，现在的docker都是默认带compose的，如果没有请自行安装。

```shell
sudo docker compose version
```

在任意一个你能找到位置的路径新建一个目录，就命名为cyberai吧，我们将会在这里创建`docker-compose.yaml`文件，让容器管理更方便。

```shell
sudo mkdir cyberai
cd cyberai
```

然后用nano或者vim或者vscode新建一个`docker-compose.yaml`文件，我这里就直接用vim了，你用什么都可以，如果你是小白或者没有安装vim，那还是老老实实用vscode吧，能方便点。

```shell
sudo vim docker-compose.yaml
```

然后复制这一段配置文件，再选中命令行CMD窗口，然后用快捷键粘贴就可以，如果无法粘贴，你可以按下键盘上的“i”键进入insert模式，你会发现命令行窗口左下角会有一个“insert”，然后再粘贴。

```yaml
version: '3.8'  # 指定 Docker Compose 文件的版本

services:
  mongodb:
    image: mongo:latest  # 使用最新版本的 MongoDB 镜像
    volumes:
      - ./data/db:/data/db  # 将当前目录下 ./data/db 挂载到容器内的 /data/db 目录，用于持久化数据库数据
      - ./data/backup:/data/backup  # 将当前目录下 ./data/backup 挂载到容器内的 /data/backup 目录，用于备份数据
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin  # 设置 MongoDB 初始化的 root 用户名,可以自行更改
      - MONGO_INITDB_ROOT_PASSWORD=secret  # 设置 MongoDB 初始化的 root 用户密码,可以自行更改
    networks:
      - internal_network  # 将服务连接到名为 internal_network 的内部网络
    restart: always  # 设置容器在失败后自动重启
    ports:
      - "127.0.0.1:27017:27017"  # 将容器的 27017 端口映射到主机的 127.0.0.1:27017，只在本地暴露端口

networks:
  internal_network:
    driver: bridge  # 定义一个名为 internal_network 的内部网络，使用 bridge 驱动
```

然后按下键盘上的ESC键，退出编辑模式，然后用键盘按下shift+；，就相当于打出一个冒号：，输入wq，然后回车。这一步就是写入保存，退出的意思。总结一下就是，按Esc，然后输入`:wq`，然后按下回车，就会退回到之前的命令行界面。

我们每次启动关闭这个镜像的时候，你要么用这个`docker-compose.yaml`的绝对路径，你要么cd到这个yaml的文件夹内操作。要不然compose找不到yaml文件在哪里。我们在文件夹内，输入命令：

```shell
docker compose up -d
```

你会发现它开始下载镜像并解压，解压完成后会创建容器开始运行。

你可以使用这些命令观察日志或管理容器：

```shell
# 列出所有容器
docker ps -a
# 实时显示容器的日志，ID或者名字从刚刚命令返回的列表里复制
docker logs -f <容器ID或者容器名字>
# 关闭这个compose
docker compose down
```

我们还需要安装对应的python pymongo库：

```shell
pip install pymongo
```

然后我们新建一个py文件，嗯，就叫它`pymongo_test.py`吧：

```python
# pip install pymongo
import pymongo
from pymongo import MongoClient
import os

def test_mongodb_connection():
    # 从环境变量中获取 MongoDB 配置信息
    mongo_host = os.getenv('MONGO_HOST', 'localhost')
    mongo_port = int(os.getenv('MONGO_PORT', 27017))
    mongo_user = os.getenv('MONGO_USER', 'admin')
    mongo_password = os.getenv('MONGO_PASSWORD', 'secret')
    # 创建一个名字叫做 chat_history 的数据库
    mongo_db_name = os.getenv('MONGO_DB_NAME', 'chat_history')

    # 创建 MongoDB 客户端
    client = MongoClient(
        host=mongo_host,
        port=mongo_port,
        username=mongo_user,
        password=mongo_password
    )

    try:
            # 连接到指定的数据库
            db = client[mongo_db_name]
            
            # 插入一个文档到测试集合中
            test_collection = db['daily']
            test_document = {"name": "test", "value": 123}
            test_collection.insert_one(test_document)
            
            # 尝试获取集合列表以验证连接和插入操作
            collections = db.list_collection_names()
            print(f"成功连接到数据库 '{mongo_db_name}'，集合列表：{collections}")
    except Exception as e:
        print(f"连接到 MongoDB 数据库失败：{e}")
    finally:
        # 关闭 MongoDB 客户端
        client.close()

if __name__ == "__main__":
    test_mongodb_connection()
```

我们运行他会得到一堆输出，如果没有报错就证明成功连接到数据库了。

### 短期记忆与原始数据

我们已经准备好了外部数据库和LLM了。接下来我们要准备使用LLM对数据进行处理，然后存入数据库了。一部分存入的数据作为短期记忆，另一部分用于更高阶的数据库进行回溯和提炼用。

我的方案是使用会话管理机制，有会话超时时间，比如我们规定，一次会话就是30分钟，如果超过了这个事件没有任何对话，后端就会把这个会话过期存档。当然这个超时时间我们是可以随意更改的。这样会有好处，那就是会话独立管理便于回溯和后期的检索/整理。

随着功能模块逐渐变多，从现在开始，我们将会使用Python的自定义模块和类（Class）来定义不同的功能模块并把他们分类放在一起。这样便于后续的维护和添加功能/测试。

### 自定义模块与类

我们是*“从零”*教程，所以这里简单介绍一下自定义模块和类，大佬请直接跳过即可。

#### 创建自定义模块

在Python中，自定义模块实际上就是一个包含Python代码的文件。通过创建自定义模块，你可以将相关的功能代码组织在一起，便于重用和维护。

1. **创建一个Python文件**：首先，在你的项目目录下创建一个新的Python文件。文件名就是模块的名称。例如，创建一个名为`mymodule.py`的文件。

2. **编写模块代码**：在该文件中，你可以定义函数、变量和类。例如：

   ```python
   # mymodule.py
   
   def add(a, b):
       return a + b
   
   def subtract(a, b):
       return a - b
   
   class Calculator:
       def __init__(self):
           self.value = 0
   
       def add(self, amount):
           self.value += amount
   
       def subtract(self, amount):
           self.value -= amount
   
       def get_value(self):
           return self.value
   ```

##### 使用模块

要使用自定义模块，你需要在项目中的其他文件中导入它。假设你有一个名为`main.py`的文件想要使用`mymodule`中的功能：

```python
# main.py
# 注意，如果你的自定义模块和主函数脚本文件不在同一个文件夹，你需要提供相对与main.py文件的相对路径，虽然Python编译的时候会自动搜索很多路径，但是为了避免问题，尽量使用准确的相对路径。
import mymodule

# 使用函数
result = mymodule.add(5, 3)
print(f"Addition: {result}")

# 使用类，初始化一个叫calc的对象，名字随意，指向我们的自定义模块
calc = mymodule.Calculator()
# 后续调用就直接用这个名字+我们类里面的函数名称，就好了
calc.add(10)
calc.subtract(3)
print(f"Calculator Value: {calc.get_value()}")
```

#### 创建和使用类

类是Python中实现面向对象编程的基础。使用类，你可以创建对象来封装数据和功能。

##### 定义类

在Python中，定义类使用`class`关键字。下面是一个简单的类定义示例：

```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def bark(self):
        print(f"{self.name} says woof!")

    def get_age(self):
        return self.age
```

##### 使用类

一旦定义了类，你可以创建该类的实例（对象）并使用它们的方法和属性：

```python
# 创建Dog类的实例
my_dog = Dog("Buddy", 3)

# 调用实例方法
my_dog.bark()  # 输出: Buddy says woof （＾_＾）!

# 获取属性
print(f"My dog is {my_dog.get_age()} years old.")
```

#### 组织模块和类

将模块和类结合使用，可以为你的项目提供一个清晰的结构。每个模块可以包含一个或多个相关的类，这样可以使代码更易于管理。例如，你可以为每个主要功能创建一个模块，并在模块内部定义相关的类。接下来，我们就要自定义一个模块用于管理MongoDB数据库操作，我们会在这个模块里定义一个类，并在里面创建几个函数用于写入或者查询数据库。

### 创建MongoDB自定义模块

我们要开始整理我们的项目文件夹了，请大家做好备份，无论你是用Git还是传统的复制副本，请一定要记得备份。

我们在项目文件夹的根目录新建一个文件夹，就叫他`cyberaimodules`吧，我们将会在这个文件夹里创建自定义模块，并在主脚本里面导入它。

我们的文件目录结构看起来像是这样：

```text
your_project/
│
├── main.py
└── cyberaimodules/
    ├── __init__.py
    ├── cyberaimongo.py # 我们将要创建的自定义模块，用于管理数据库操作
    └── module2.py

```

- `your_project/`是你的项目的根目录。
- `main.py`是主程序文件。
- `cyberaimodules/`是我们自定义模块的目录。
- `__init__.py`是一个特殊的文件，用来将`cyberaimodules`目录标识为一个Python包。虽然现在没有这个也能用但还是尽量加上，这个文件是空的，只需要新建这个文件，重命名，然后内容留空就好了。
- `cyberaimongo.py`和`module2.py`是你在`cyberai`目录下创建的Python模块文件。

我们在`cyberaimodules`文件夹下新建一个`cyberaimongo.py`。这个模块用来管理mongo数据库，那它就必须要有三大核心功能：写入，查询和初始化。这里先导入我们需要的模块：

```python
import json
from pymongo import MongoClient
import uuid
from datetime import datetime
```

我们定义模块的初始化：

```python
class MongoManager:
    def __init__(self, host='mongodb', port=27017, username='admin', password='secret', db_name='chat_history'):
        self.client = MongoClient(f'mongodb://{username}:{password}@{host}:{port}/')
        self.db = self.client[db_name]
```

#### `__init__` 的作用

- **自动初始化**：当你创建一个 `MongoManager` 对象时，比如说 `manager = MongoManager()`，Python 会自动调用 `__init__` 函数。这就像是开车时，你坐进车里，发动引擎，一切准备就绪。

- **设置初始状态**：`__init__` 函数会为新创建的对象设置初始状态。它会把你传入的参数（如数据库地址、用户名、密码等）存储起来，以便后续使用。

- **参数默认值**：函数里设置了一些默认值，比如 `host='mongodb'` 和 `port=27017`。这意味着如果你没有特别指定这些参数，系统会使用这些默认值。这就像是你买了一台电视，出厂时已经调好了默认频道和音量，你不调整它也能正常使用。

#### `self` 的作用

- **表示当前对象**：`self` 是一个特殊的变量，它总是指向当前的对象实例。通过 `self`，我们可以在类的方法中访问和修改这个对象自己的属性。

- **属性存储**：当 `__init__` 函数中写 `self.client = ...` 时，我们就是在给这个对象添加一个名为 `client` 的属性，并把 MongoDB 连接信息存储在这里。以后，任何时候你都可以通过这个对象（比如 `manager.client`）访问这个属性。

- **保持一致性**：`self` 确保你在对象的不同方法中都能访问到同样的数据。比如，`self.db` 是在 `__init__` 里创建的，你可以在类的其他方法中随时用 `self.db` 来进行数据库操作。

`__init__` 和 `self` 就像是为一个新的物体（比如一个玩具车）装上了引擎、轮子和方向盘。`__init__` 是组装的过程，把所有零件（参数）放到位，而 `self` 是确保你在任何时候都能找到和使用这些零件。通过这种方式，你创建的每一个 `MongoManager` 对象都是独立的，且已经准备好可以使用。

我目前用的是json存聊天记录，如果你喜欢别的类型随意。

我们创建一个函数用来预先生成将要存在数据库里的json：

```python
def generate_chat_json(self, ai_reply, human_message, session_id, timestamp=None):
    """
    生成一个包含聊天信息的 JSON 字符串。

    参数：
    - ai_reply (str): AI 的回复信息。
    - human_message (str): 人类的消息。
    - session_id (str): 会话的唯一标识符。
    - timestamp (datetime, optional): 消息的时间戳。如果未提供，则使用当前时间。

    返回：
    - str: 包含聊天信息的 JSON 字符串。
    """

    # 如果没有提供时间戳，则使用当前时间
    if timestamp is None:
        timestamp = datetime.now()

    # 将时间戳格式化为字符串，格式为 ISO 8601（不带秒）
    timestamp_str = timestamp.strftime("%Y-%m-%dT%H:%M")

    # 创建一个字典来存储聊天数据
    chat_data = {
        'message_id': str(uuid.uuid4()),  # 生成一个唯一的消息ID
        'ai_reply': ai_reply,             # 存储AI的回复
        'human_message': human_message,   # 存储人类的消息
        'timestamp': timestamp_str,       # 存储格式化后的时间戳
        'session_id': session_id          # 存储会话ID
    }

    # 将字典转化为 JSON 字符串并返回
    return json.dumps(chat_data, ensure_ascii=False, indent=4)
    # ensure_ascii=False 确保非 ASCII 字符（如中文）能够正确显示
    # indent=4 使生成的 JSON 更加易读

```

接下来创建一个函数用于写入单次的对话记录到数据库：

```python
def insert_chat(self, collection_name, ai_reply, human_message, session_id, timestamp=None):
    """
    将聊天记录插入到指定的 MongoDB 集合中。

    参数：
    - collection_name (str): MongoDB 中集合的名称。
    - ai_reply (str): AI 的回复信息。
    - human_message (str): 人类的消息。
    - session_id (str): 会话的唯一标识符。
    - timestamp (datetime, optional): 消息的时间戳。如果未提供，则使用当前时间。

    返回：
    - ObjectId: 插入的文档的唯一 ID。
    """

    # 生成聊天记录的 JSON 数据
    json_data = self.generate_chat_json(self, ai_reply, human_message, session_id, timestamp)

    # 获取指定名称的集合对象
    collection = self.db[collection_name]

    # 确保 json_data 是字典类型，以便插入到 MongoDB
    if isinstance(json_data, str):
        json_data = json.loads(json_data)

    # 插入聊天记录到集合中，并返回插入的文档ID
    return collection.insert_one(json_data).inserted_id

```

我们还要添加一些用于简单查询的函数：

```python
    # 获取某个时间段内的所有数据
    def get_mem_in_time_range(self, collection_name, start_time, end_time):
        """
        在指定的集合中获取某个时间段内的所有数据。

        参数：
        - collection_name (str): 集合的名称。
        - start_time (datetime): 起始时间。
        - end_time (datetime): 结束时间。

        返回：
        - list: 包含查询结果的列表。
        """
        collection = self.db[collection_name]

        # 使用 find 查询符合条件的文档
        cursor = collection.find({
            'timestamp': {
                '$gte': start_time.isoformat(),
                '$lte': end_time.isoformat()
            }
        })

        # 将查询结果转换为列表并返回
        return list(cursor)

    # 根据 ID 获取数据
    def get_data_by_id(self, collection_name, id):
        """
        在指定的集合中根据文档 ID 获取数据。

        参数：
        - collection_name (str): 集合的名称。
        - id (ObjectId): 文档的唯一 ID。

        返回：
        - dict or None: 查询到的文档数据，如果未找到则返回 None。
        """
        collection = self.db[collection_name]

        # 使用 find_one 查询指定 ID 的文档
        data = collection.find_one({'_id': id})

        return data

```

最后整体代码看起来是这样：

```python
import json
import uuid
from datetime import datetime
from pymongo import MongoClient

class MongoManager:
    def __init__(self, host='mongodb', port=27017, username='admin', password='secret', db_name='chat_history'):
        self.client = MongoClient(f'mongodb://{username}:{password}@{host}:{port}/')
        self.db = self.client[db_name]

    def generate_chat_json(self, ai_reply, human_message, session_id, timestamp=None):
        if timestamp is None:
            timestamp = datetime.now()
        timestamp_str = timestamp.strftime("%Y-%m-%dT%H:%M")
        chat_data = {
            'message_id': str(uuid.uuid4()),
            'ai_reply': ai_reply,
            'human_message': human_message,
            'timestamp': timestamp_str,
            'session_id': session_id
        }
        return json.dumps(chat_data, ensure_ascii=False, indent=4)

    def insert_chat(self, collection_name, ai_reply, human_message, session_id, timestamp=None):
        json_data = self.generate_chat_json(self, ai_reply, human_message, session_id, timestamp)
        collection = self.db[collection_name]
        if isinstance(json_data, str):
            json_data = json.loads(json_data)
        return collection.insert_one(json_data).inserted_id

    def get_mem_in_time_range(self, collection_name, start_time, end_time):
        collection = self.db[collection_name]
        cursor = collection.find({
            'timestamp': {
                '$gte': start_time.isoformat(),
                '$lte': end_time.isoformat()
            }
        })
        return list(cursor)

    def get_data_by_id(self, collection_name, id):
        collection = self.db[collection_name]
        data = collection.find_one({'_id': id})
        return data

```

好的，到目前为止，我们自定义模块就暂时告以段落了，下面我们将会在主函数里导入并使用它们。

```python
import os 
import uuid
# 导入之前创建的 cyberaimodules 模块
from cyberaimodules import cyberaimongo
```

这里我们暂时使用的测试用的环境变量，实际生产环境建议保存好敏感数据，避免直接将授权信息放在代码里。

```python
# 创建虚拟的环境变量暂时用来测试
mongo_host = os.getenv('MONGO_HOST', 'localhost')
mongo_port = int(os.getenv('MONGO_PORT', 27017))
mongo_user = os.getenv('MONGO_USER', 'admin')
mongo_password = os.getenv('MONGO_PASSWORD', 'secret')
# 数据库的名字叫做 chat_history
mongo_db_name = os.getenv('MONGO_DB_NAME', 'chat_history')

# 初始化 MongoDB 客户端
mongo_manager = cyberaimongo.MongoManager(
    host=mongo_host,
    port=mongo_port,
    username=mongo_user,
    password=mongo_password,
    db_name=mongo_db_name,
)
```

然后我们先简单测试一下对话过程中单次对话的数据库写入操作，更复杂的会话与总结功能请等待下一篇。

我们在get_response_from_llm这个函数里增加一点内容来测试一下刚刚的模块有没有正常工作（请注意你的Mongo DB对本地端口开放）：

```python
  # 在llm回复后先尝试插入  本次聊天数据
  chat_id = mongo_manager.insert_chat(
        collection_name='daily',
        ai_reply=response_str,
        human_message=question,
    		# 先暂时生成一个会话ID测试，我们后续会根据超时时间进行处理会话
        session_id=str(uuid.uuid4()),
    )

  # 然后再使用返回的唯一chat_id查询这条对话，看看能不能回溯
    query = mongo_manager.get_data_by_id(
        collection_name='daily',
        id=chat_id
    )
	# 如果有，打印出来康康：
    if query:
        print(f"Chat data: {query}")
```

完整的代码看起来是这样：

```python
# main.py
import os
import uuid
from openai import OpenAI

# 导入之前创建的 cyberaimodules 模块
from cyberaimodules import cyberaimongo

mongo_host = os.getenv('MONGO_HOST', 'localhost')
mongo_port = int(os.getenv('MONGO_PORT', 27017))
mongo_user = os.getenv('MONGO_USER', 'admin')
mongo_password = os.getenv('MONGO_PASSWORD', 'secret')
# 数据库的名字叫做 chat_history
mongo_db_name = os.getenv('MONGO_DB_NAME', 'chat_history')

# 初始化 MongoDB 客户端
mongo_manager = cyberaimongo.MongoManager(
    host=mongo_host,
    port=mongo_port,
    username=mongo_user,
    password=mongo_password,
    db_name=mongo_db_name,
)

# 初始化 OpenAI 模型
chat_model= OpenAI(
	# 你需要把这个替换成你的后端的API地址
	base_url="https://api.openai.com/v1/",
	# 这是用于身份验证的 API Key
	api_key = "sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"
	)

# 聊天记录和总结记录的列表
chat_history = []
summary_history = []
SUMMARY_THRESHOLD = 4  # 定义达到多少轮次后进行总结

def summarize_chat_history(chat_history_window):
    """
    对最近的聊天记录进行总结
    :param chat_history_window: 最近的聊天记录
    :return: 总结后的字符串
    """
    # 创建总结提示
    summary_prompt = f"请总结以下对话内容：\n{chat_history_window}"

    print(f"正在对以下内容生成总结: {summary_prompt}")
    # 调用 LLM 生成总结
    summary_response = chat_model.chat.completions.create(
        model='gpt-4o-mini',
        messages=[{"role": "user", "content": summary_prompt}],
        temperature=0.7,
    )
    # 获取总结内容
    summary_str = summary_response.choices[0].message.content
    return summary_str

def get_response_from_llm(question):
    """
    获取 LLM 的响应
    :param question: 用户的问题
    :return: LLM 的响应
    """
    global chat_history, summary_history

    # 如果聊天记录达到总结阈值
    if len(chat_history) >= SUMMARY_THRESHOLD:
        # 获取最近的聊天记录窗口
        chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-SUMMARY_THRESHOLD*2-1:-1]])
        # 生成总结
        summary = summarize_chat_history(chat_history_window)
        # 将总结添加到总结记录中
        summary_history.append(summary)
        # 保留最后一条聊天记录
        chat_history = chat_history[-1:]

    # 获取最近的聊天记录窗口
    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*SUMMARY_THRESHOLD:-1]])
    chat_history_prompt = f"Here is the chat history:\n {chat_history_window}"    

    # 创建消息列表
    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
        {"role": "assistant", "content": chat_history_prompt},
        {"role": "user", "content": question},
    ]

    # 如果有总结记录，将其添加到消息列表中
    if summary_history:
        summary_prompt = "\n".join(summary_history)
        message.insert(1, {"role": "assistant", "content": f"Summary of previous conversations:\n{summary_prompt}"})

    # 调用 LLM 获取响应
    response = chat_model.chat.completions.create(
        model='gpt-4o',
        messages=message,
        temperature=0.7,
    )
    
    print(f"message: {message}")
    # 获取响应内容
    response_str = response.choices[0].message.content

    chat_id = mongo_manager.insert_chat(
        collection_name='daily',
        ai_reply=response_str,
        human_message=question,
        session_id=str(uuid.uuid4()),
    )

    query = mongo_manager.get_data_by_id(
        collection_name='daily',
        id=chat_id
    )

    if query:
        print(f"Chat data: {query}")

    return response_str

if __name__ == "__main__":
    while True:
        user_input = input("\n输入问题或者请输入'exit'退出：")
        if user_input.lower() == 'exit':
            print("再见")
            break 
        # 将用户输入添加到聊天记录中
        chat_history.append(('human', user_input))
        # 获取 LLM 的响应
        response = get_response_from_llm(user_input)
        # 将 LLM 的响应添加到聊天记录中
        chat_history.append(('ai', response))
        # 打印 LLM 的响应
        print(response)
```

到此，我们的记忆部分第一篇就大概结束了。下一篇我们将会在这个基础上，慢慢地把**短期记忆**和**常态记忆**加载聊完。大概就是，每一次LLM聊天的时候，会先记起昨天或者刚刚发生了什么事情，这样的话，我们的赛博老婆，就慢慢地拥有记忆了。祝你在赛博朋克的世界里玩得开心！

