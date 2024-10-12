+++
title = '从零开始的赛博老婆！2 - 基础Python环境搭建+接入大语言模型'
date = 2024-10-11T23:15:06-02:30
draft = false

+++

![](/images/104755143_p0_master1200.jpg)
Pixiv ID: 104755143

本文章开始我们逐步进行实战，本次为搭建python基本环境，然后接入大语言模型，以Open AI Chat GPT为例。

# Python环境与依赖

当你想运行你的赛博伙伴，你的代码是需要依赖库运行的，本文章使用了Python基础库和openai等库。请注意，当你的电脑里的python环境里没有正确安装相应的依赖库，代码无法正常运行。

***注：以下命令可能不适用于你的系统，请根据对应系统查询conda命令***

### 安装python环境或者安装anaconda环境管理器

请去[python官网](https://www.python.org/downloads/)或者[anaconda官网](https://www.anaconda.com/download)里安装软件，二者选其一即可。本文推荐使用anaconda，避免出现好多个环境互相冲突的情况。

如果你使用ARM架构的系统,可以尝试使用MiniForge,这个项目对ARM Python环境有一些优化,还是不错的。

***请注意，安装任意软件的时候请务必将其添加到系统变量或者系统路径中（大佬除外），否则无法使用CMD正常运行脚本，具体安装和排错等问题恕不在本教程范围内***

一般选择默认安装即可，安装好anaconda之后，，用打开你的CMD命令行窗口，运行以下命令检查conda是否安装成功：

```
conda -V 
# 或者
conda --version
```

如果出现“没有此命令”则安装不正常，请自行排错。

```
# 更新conda
conda update conda
# 更新所有包
conda update --all
```

### 创建一个python环境

***注：本教程建议使用python3.11及以上版本，之前的版本没有经过测试，请尽量保证最新版本。***

我们给我们的赛博伙伴项目创建一个虚拟环境，以后我们就使用这个环境运行和赛博伙伴相关的代码，这样后续你统计依赖列表打包Docker镜像的时候很方便。

使用此命令创建一个新的环境：

```
conda create -n env_name
```

例如：

```
conda create -n ai
```

### 激活环境并安装依赖

#### 激活环境

使用以下命令激活特定的环境：

```
conda activate env_name
```

例如我们刚刚创建了一个名为`ai`的环境，现在我们要激活它：

```
conda activate ai
```

激活之后，我们所有的操作使用的就是这个环境里已经安装的包和依赖了，你会发现命令行的开头会有你当前的环境名称。每次你打开命令行，都检查一下是不是你所需要的环境，当然你也可以设置你的默认环境。

你可以使用这个命令：

```
conda env list 
# 或
conda info --envs 
# 查看存在哪些虚拟环境
```

你可能要用到这个命令关闭这个环境（但是先不要关闭，我们还要继续使用这个环境）：

```
conda deactivate env_name
# 例如关闭ai这个环境，关闭之后就是默认环境了
conda deactivate ai
```

#### 安装相关依赖

现在激活我们刚刚创建的环境，然后仍然是在CMD里***依次***输入此命令安装相关依赖：

```
# Openai官方的python库
pip install openai
```

安装完成后我们准备进入下一步。

相关的官方文档比较推荐，官方的文档是准确且值得反复推敲的。

[openai官方文档](https://platform.openai.com/docs/introduction)

# 创建一个python脚本

### 准备你的Open AI API

准备好你的能用的API token并妥善保存

### 开始写代码

有人喜欢用脚本，有人喜欢用notebook交互式python，因人而异，喜欢啥用啥。我个人喜欢用脚本，notebook的用法请自行学习。

建议安装并使用Visual Studio Code这个软件，大佬随意。不建议使用某自主研发会员制CEC IDE（因为穷）。使用VSC创建一个`cyberai.py`文件，保存在你能找得到的地方，然后我们开始输入代码：

首先导入之前安装好的库中我们需要的部分,这个是Openai官方的python SDK库,可以用来调用兼容的API。

```python
from openai import OpenAI
```

然后我们开始基础设置,这个就相当于连接到运行LLM的服务器一样：

1. `base_url`是你的接入点，如果你使用的是非官方的第三方中转API，或者你使用了反向代理，那么你可能会用到这个，在双引号中间填入你的接入点链接，例如`https://api.openai.com/v1/chat/completions`。如果你可以正常连接到官方的接入点，这一行就可以删去不用。
2. `api_key`是你的API Token，在双引号里填写你的api。

```python
chat_model= OpenAI(
	# 你需要把这个替换成你的后端的API地址
	base_url="https://api.openai.com/v1/",
	# 这是用于身份验证的 API Key
	api_key = "sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"
	)
```

好的，我们已经完成一半了，我们将简单定义一个函数并在这个函数里调用我们刚刚设置好的模型：

这个函数接受一个问题 `question`(字符串)作为参数。

消息列表`message`是定义一条将要发送给LLM后端的消息体，其中包含系统消息（设定角色为猫娘并要求输出中文）和用户消息（用户输入的问题）。一般的格式如下,当然不同的LLM支持的消息体格式可能略有不同,例如写本文时,Claude的API就必须User开头,具体请查询对应API Doc。

我们调用 `chat_model.chat.completions.create`方法生成对话响应，指定模型为 `gpt-4o-mini`，并设置温度参数为 0.7。

最后我们从响应中提取生成的文本内容并返回。

```python
def get_response_from_llm(question):
    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
        {"role": "user", "content": question},
    ]
    response = chat_model.chat.completions.create(
                model='gpt-4o-mini',
                messages=message,
                temperature=0.7,
            )
    response_str = response.choices[0].message.content
    return response_str
```

这个函数收到问题的输入就会传给chat_model进行响应，并把AI生成的回复返回。

我们已经接近成功了，我们添加一个简单的命令行交互：

1. 检查是否作为主程序运行。
2. 进入一个无限循环，提示用户输入问题或输入 'exit' 退出。
3. 如果用户输入 'exit'，打印 "再见" 并退出循环。
4. 否则，调用 `get_response_from_llm`函数获取响应并打印。

```python
if __name__ == "__main__":
    while True:
        user_input = input("\n输入问题或者请输入'exit'退出：")
        if user_input.lower() == 'exit':
            print("再见")
            break 
        response = get_response_from_llm(user_input)
        print(response)
```

最后检查一下你的完整代码可能看起来是这样子：

```python
from openai import OpenAI

chat_model= OpenAI(
	# 你需要把这个替换成你的后端的API地址
	base_url="https://api.openai.com/v1/",
	# 这是用于身份验证的 API Key
	api_key = "sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"
	)

def get_response_from_llm(question):

    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
        {"role": "user", "content": question},
    ]

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
        response = get_response_from_llm(user_input)
        print(response)


```

很好，保存之后，我们关闭VSC并打开命令行窗口，我们将要运行这个脚本然后尝试与AI进行单次对话：

在你的命令行窗口里输入以下命令但先不要按下回车：

```
# 注意空格！！！！
python3+空格
```

然后在资源管理器里尝试把你刚刚写好保存的`cyberai.py`文件拖拽进你的命令行窗口，这一步操作主要是把这个文件的路径+文件名+扩展名一起告诉命令行好让它运行这个脚本，最后完整的命令应该是类似于这样子的：

```
python3 /Volumes/path/path/cyberai.py
```

好的我们按下回车，如果一切顺利的话，应该会出现一个提示：
（本脚本作者测试了能够正常运行，如果有任何问题请自行排错请谅解）

```
输入问题或者请输入'exit'退出：
```

那么恭喜你，你成功了，尝试输入问题然后等待AI回复，例如：

```python
输入问题或者请输入'exit'退出：你可以做我的赛博伙伴吗？
很抱歉，作为一个AI语言模型，我没有实体和情感，无法成为你的赛博伙伴。我只能为你提供语言上的交流和帮助。请尊重他人，不要发表不恰当的言论。
```

（这个AI中暑了，我们得赶快抢救一下……）

到现在为止，你应该已经可以正常地运行这个简单的脚本，并与某个大模型进行简单的单次对话了。但是我们可以发现，这AI没有记忆啊，说了这一句忘了上一句，这不行啊。所以接下来我们就要实现LLM的上下文记忆。

所谓上下文记忆，实际上实现的方法有很多，例如使用列表存储对话信息或者使用数据库进行永久存储等都可以。我们暂时使用最简单的列表来存储上下文消息，同时复现Langchain~~（屎山）~~曾经使用的三种上下文记忆*（这一部分后续的记忆章节会专门介绍）*。

我们在刚刚的代码里添加一些东西来实现短期记忆：

添加一个名字叫做`chat_history`的列表

```python
chat_model= OpenAI(
	# 你需要把这个替换成你的后端的API地址
	base_url="https://api.openai.com/v1/",
	# 这是用于身份验证的 API Key
	api_key = "sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"
	)
# 添加列表
chat_history = []
```

然后我们在`get_response_from_llm`函数里面添加历史记录的载入部分：

```python
def get_response_from_llm(question):

  	# 添加输出便于直观看到列表里面是什么样子
    print(chat_history)

    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*4:-1]])
    chat_history_prompt = f"Here is the chat history:\n {chat_history_window}"    

    message = [
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},
      	# 在这里增加了一条消息用于展示上下文记忆
        {"role": "assistant", "content": chat_history_prompt},
        {"role": "user", "content": question},
    ]
    
		# 添加输出便于直观看到传递给LLM的消息是什么样子
    print(message)

    response = chat_model.chat.completions.create(
                model='gpt-4o-mini',
                messages=message,
                temperature=0.7,
            )
    
    response_str = response.choices[0].message.content

    return response_str
```

我们来了解一下这段代码：在这段代码中，`chat_history_window` 和 `chat_history_prompt` 是两个关键变量，它们用于构建聊天历史记录的字符串表示。

首先，`chat_history_window` 通过列表推导式和字符串的 `join` 方法创建。列表推导式 

```python
[f"{role}: {content}" for role, content in chat_history[-2*4:-1]]
```

会遍历 `chat_history` 列表的一个切片。这个切片 `chat_history[-2*4:-1]` 选择了 `chat_history` 列表的最后七个元素（从倒数第八个到倒数第二个）。对于每个元素（它是一个包含角色和内容的元组），列表推导式生成一个格式化的字符串 `"{role}: {content}"`。然后，`"\n".join(...)` 将这些字符串用换行符 `\n` 连接起来，形成一个多行字符串 `chat_history_window`。

这样的话，我们就可以通过改变 `-2*4` 里面的这个数字 `4`，改变成任意数字，就可以控制上下文记忆的轮次了。比如你想让 AI 的记忆有 3 轮，那你就可以改成 `[-7:-1]`，因为一轮对话包含用户输入和机器人的回复条消息嘛，所以是 6 条消息。至于为什么要去掉倒数第一个消息呢，因为后续我们在主函数中，用户的当前输入（AI 还没来得及相应）的时候，就已经被记录到聊天历史了，这会和后面的 `{"role": "user", "content": question}` 重复了，没必要。

接下来，`chat_history_prompt` 是一个包含聊天历史记录的完整字符串。它通过格式化字符串 

```python
f"Here is the chat history:\n {chat_history_window}"
```

创建，其中 `chat_history_window` 被插入到字符串中。这段代码的目的是将最近的聊天历史记录格式化为一个字符串，以便在后续的聊天请求中使用。这样可以为聊天模型提供上下文，使其能够生成更相关和连贯的响应。你可以把它理解为一种模板 + 变量的形式，达到类似于动态提示词的效果，也就是说，`{chat_history_window}` 里面的是变量，会一直变，而之前的内容是固定字符串，便于 LLM 理解这一部分是上下文，避免产生混淆。
多说一句，这里提到的***格式化字符串***在后续的**提示词工程**和**动态提示词**等相关应用中很有用，它让我们的Agent可以根据不同的条件得到最精准的信息，减少Token使用，提高成功率。

最后，我们在主函数里增加上写文记录的操作，大概是这样：

```python
if __name__ == "__main__":
    while True:
        user_input = input("\n输入问题或者请输入'exit'退出：")
        if user_input.lower() == 'exit':
            print("再见")
            break 
        # 添加用户消息到消息列表
        chat_history.append(('human', user_input))
        response = get_response_from_llm(user_input)
        # 添加AI回复到消息列表
        chat_history.append(('ai', response))
        print(response)
```

完整的代码看起来是这样：

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

    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*4:-1]])
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

希望你们可以成功运行代码！
