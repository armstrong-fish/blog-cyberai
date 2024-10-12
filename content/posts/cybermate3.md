+++
title = '从零开始的赛博老婆！3 - TTS浅谈与实践'
date = 2024-10-11T23:22:21-02:30
draft = false

+++

![Image Description](/images/71888962_p0_master1200.jpg)

PIXIV: 71888962 [@Yuuri](https://www.pixiv.net/users/1822272)

## TTS浅谈

TTS（**T**ext **T**o **S**peech）是语音合成（**Speech synthesis**）的一个分支，**文本转语音**（**TTS**）系统将正常语言文本转换为语音。TTS系统对于提高交互性有帮助，最近技术发展迅猛，TTS项目百花齐放。

从RVC的实时变声和动态语调优化，再到效果优秀的VITS，生成对抗网络（GAN）的加持使其声音比以前自然了很多。随着SO-VITS强大的声调控制能力的实现，优秀数据集加持下的Diff-SVC，虚拟歌姬的声音媲美真人，但也随之带来了一系列问题。在我浅显的认知中，我对语音合成还停留在初音未来那个时代，但是时代变化如此之快。目前TTS大有互相融合的趋势，例如[fish-diffusion](https://github.com/fishaudio/fish-diffusion)或者Fishaudio旗下的其他项目等，都有不小的进步。

开源项目如此繁盛，商用发展当然也不落后。看向商业阵营，各大公司都有其绝活。比如**Acapela Group**专门搞已故名人的TTS，这非常有特色。还有众多公司专注于高质量或情感丰富的TTS、定制声音等。而在商用TTS行业中，Azure TTS可以说是龙头，我们本篇文章也会使用微软的免费EdgeTTS服务作为示例。

这里有一些相关的项目链接：

- [Fish Audio](https://github.com/fishaudio)
- [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS)
- [Retrieval-based-Voice-Conversion-WebUI](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI)

这里还有两个哔哩哔哩up主，专注于音乐翻唱相关的，也推荐参考：

- [领航员未鸟](https://space.bilibili.com/2403955/dynamic)

东洋雪莲也比较有实力，但似乎不公开分享很多技术细节，故仅作介绍。

注意，现在互联网上有非常多的TTS引擎，本文章仅使用较为稳定的Edge-TTS，初衷是用于实际应用。如果你需要完全本地使用TTS或者为了好玩个性化训练声音和特殊模型，本教程不适用，请移步到其他个性化TTS炼丹教程，比如GPT-SO-VITS等等。

## Edge TTS

TTS接受文字输入，然后输出音频。对于Edge TTS的Python版本，它实际上是通过网络请求工作的。因此，它可以很好地在边缘计算平台上运行，因为音频合成是在云端计算并返回给你的。我们使用的是[Edge TTS Python](https://pypi.org/project/edge-tts/)库，这里还有GitHub的项目网址：[edge-tts](https://github.com/rany2/edge-tts)。

### 安装Edge TTS

首先安装：

```bash
pip install edge-tts
```

Edge TTS有两种工作方式，一种是使用命令行交互的方式，如果你只想使用命令模式，你可以使用pipx安装它（来自官网的教程）：

```bash
pipx install edge-tts
```

命令行模式我们不用，这里一笔带过。

### Edge TTS使用

首先创建一个`Edgetts.py`文件，然后导入所需要的模块：

```python
# 导入需要的库
import asyncio
import edge_tts
import os
```

然后初始化TTS引擎。为了获取语言和声音的可选项，可以在命令行窗口输入命令来获取支持的列表：

```bash
edge-tts --list-voices
```

你会发现输出了一大坨列表，这里就不放出来了。选择需要的`NAME`然后继续写初始化代码：

```python
# 设置要转化为语音的文本
TEXT = "你好啊，这里是lico！欢迎来到lico的元宇宙！"  
# 设置语音的语言和声音，注意这个名字是大小写敏感的
VOICE = "zh-CN-XiaoyiNeural" 
# 设置输出文件的路径，文件将保存在脚本所在的文件夹中
OUTPUT_FILE = os.path.join(os.path.dirname(os.path.realpath(__file__)), "test.mp3")
```

使用edgetts的函数进行语音生成。注意，异步运行：

```python
# 定义主函数
async def _main() -> None:  
    # 创建一个Communicate对象，用于将文本转化为语音
    communicate = edge_tts.Communicate(TEXT, VOICE)  
    # 将语音保存到文件中
    await communicate.save(OUTPUT_FILE)  
```

最后加上主函数，整体的代码如下：

```python
# 导入需要的库
import asyncio  
import edge_tts  
import os

# 设置要转化为语音的文本
TEXT = "你好啊，这里是lico！欢迎来到lico的元宇宙！"  
# 设置语音的语言和声音
VOICE = "zh-CN-XiaoyiNeural" 
# 设置输出文件的路径，文件将保存在脚本所在的文件夹中
OUTPUT_FILE = os.path.join(os.path.dirname(os.path.realpath(__file__)), "test.mp3")

# 定义主函数
async def _main() -> None:  
    # 创建一个Communicate对象，用于将文本转化为语音
    communicate = edge_tts.Communicate(TEXT, VOICE)  
    # 将语音保存到文件中
    await communicate.save(OUTPUT_FILE)  

# 如果这个脚本是直接运行的，而不是被导入的，那么就运行主函数
if __name__ == "__main__":  
    asyncio.run(_main())
```

在命令行运行之后，你就会发现脚本旁边生成了一个`test.mp3`，这就是生成的语音，你可以尝试在**不社会性死亡**的情况下播放或者修改成更社死的文本。

Edge TTS还提供了流式生成的选项，这里不过多叙述，因为流式仅指输出，输入是没办法流式的，因为需要对整个文本进行音调和音素的规划，不过我们还是有机会解决这个问题，后续有机会再写吧。

```python
# 官方的流式生成的示例
async def amain() -> None:
    """Main function"""
    communicate = edge_tts.Communicate(TEXT, VOICE)
    with open(OUTPUT_FILE, "wb") as file:
        async for chunk in communicate.stream():
            if chunk["type"] == "audio":
                file.write(chunk["data"])
            elif chunk["type"] == "WordBoundary":
                print(f"WordBoundary: {chunk}")
```

### LLM与TTS的结合

我们接下来尝试给我们上次的代码加上TTS语音输出，不过在此之前，我们需要先把TTS脚本改一下，让它变成一个函数，这样我们需要的时候调用它就好了。

我们想要使用命令行交互，并且能够播放声音。考虑到多平台的兼容性，我们安装一个库`pygame`：

```bash
pip install pygame
```

然后我们修改之前的脚本，试试能不能播放：

```python
# 导入需要的库  
import asyncio  
import edge_tts  
import os  
import pygame  

# 设置要转化为语音的文本  
TEXT = "你好啊，这里是lico！欢迎来到lico的元宇宙！"  
# 设置语音的语言和声音  
VOICE = "zh-CN-XiaoyiNeural"   
# 设置输出文件的路径，文件将保存在脚本所在的文件夹中  
OUTPUT_FILE = os.path.join(os.path.dirname(os.path.realpath(__file__)), "test.mp3")  

# 定义主函数  
async def _main() -> None:  
    # 创建一个Communicate对象，用于将文本转化为语音  
    communicate = edge_tts.Communicate(TEXT, VOICE)  
    # 将语音保存到文件中  
    await communicate.save(OUTPUT_FILE)  

    print(f"文件已保存到: {OUTPUT_FILE}")

    # 检查文件是否存在  
    if not os.path.exists(OUTPUT_FILE):  
        print("错误: 文件未生成")  
        return  

    # 检查文件大小  
    if os.path.getsize(OUTPUT_FILE) == 0:  
        print("错误: 文件大小为0")  
        return  

    # 初始化pygame混音器  
    pygame.mixer.init()  
    # 加载音频文件  
    pygame.mixer.music.load(OUTPUT_FILE)  
    # 播放音频文件  
    pygame.mixer.music.play()  

    # 等待音频播放结束  
    while pygame.mixer.music.get_busy():  
        await asyncio.sleep(1)  

# 如果这个脚本是直接运行的，而不是被导入的，那么就运行主函数  
if __name__ == "__main__":  
    asyncio.run(_main())
```

如果你成功地运行了代码，TTS的声音会通过默认的扬声器播放出来。由于第一次使用Pygame会有一定的载入时间，因此运行脚本后可能要等几秒才会播放出声音。

现在我们有了音频播放函数了，我们可以把这个函数和我们之前的函数合在一起，达到TTS和文字同时交互的效果，整体的代码看起来是这样：

```python
import asyncio  # 导入异步IO模块  
import edge_tts  # 导入edge_tts模块，用于文本转语音  
import os  # 导入os模块，用于文件路径操作  
import pygame  # 导入pygame模块，用于音频播放  
from openai import OpenAI  # 导入OpenAI模块，用于与OpenAI的API交互  

# 初始化OpenAI的聊天模型  
chat_model = OpenAI(
    # 你需要把这个替换成你的后端的API地址  
    base_url="https://api.openai.com/v1/",  
    # 这是用于身份验证的 API Key  
    api_key="sk-SbmHyhKJHt3378h9dn1145141919810D1Fbcd12d"  
)

# 设置语音的语言和声音  
VOICE = "zh-CN-XiaoyiNeural"  
# 设置输出文件的路径，文件将保存在脚本所在的文件夹中，每一次生成后会覆盖之前的，仅用来测试  
OUTPUT_FILE = os.path.join(os.path.dirname(os.path.realpath(__file__)), "test.mp3")  
# 设置聊天记录列表  
chat_history = []  
# 在脚本开头就初始化pygame混音器，避免每次调用都初始化浪费时间  
pygame.mixer.init()

# 定义主函数，异步执行  
async def _main(text: str = '测试') -> None:  
    # 创建一个Communicate对象，用于将文本转化为语音  
    communicate = edge_tts.Communicate(text, VOICE)  
    # 将语音保存到文件中  
    await communicate.save(OUTPUT_FILE)  

    print(f"文件已保存到: {OUTPUT_FILE}")

    # 检查文件是否存在  
    if not os.path.exists(OUTPUT_FILE):  
        print("错误: 文件未生成")  
        return  

    # 检查文件大小  
    if os.path.getsize(OUTPUT_FILE) == 0:  
        print("错误: 文件大小为0")  
        return  

    # 加载音频文件  
    pygame.mixer.music.load(OUTPUT_FILE)  
    # 播放音频文件  
    pygame.mixer.music.play()  

    # 等待音频播放结束  
    while pygame.mixer.music.get_busy():  
        await asyncio.sleep(0.5)  

# 定义一个函数，从语言模型获取响应  
def get_response_from_llm(question):  
    # 打印当前的聊天记录  
    print(f'Here is the history list: {chat_history}')  
    # 获取最近的聊天记录窗口  
    chat_history_window = "\n".join([f"{role}: {content}" for role, content in chat_history[-2*4:-1]])  
    # 生成聊天记录提示  
    chat_history_prompt = f"Here is the chat history:\n {chat_history_window}"  
    # 构建消息列表  
    message = [  
        {"role": "system", "content": "You are a catgirl! Output in Chinese."},  
        {"role": "assistant", "content": chat_history_prompt},  
        {"role": "user", "content": question},  
    ]  
    # 打印发送到后端的消息  
    print(f'Message sent to backend: {message}')  
    # 调用OpenAI的API获取响应  
    response = chat_model.chat.completions.create(
        model='gpt-4o-mini',  
        messages=message,  
        temperature=0.7,  
    )  
    # 获取响应内容  
    response_str = response.choices[0].message.content  
    return response_str  

# 主程序入口  
if __name__ == "__main__":  
    while True:  
        # 获取用户输入  
        user_input = input("\n输入问题或者请输入'exit'退出：")  
        if user_input.lower() == 'exit':  
            print("再见")  
            break  
        # 将用户输入添加到聊天记录  
        chat_history.append(('human', user_input))  
        # 获取语言模型的响应  
        response = get_response_from_llm(user_input)  
        # 打印响应  
        print(response)  
        # 将响应添加到聊天记录  
        chat_history.append(('ai', response))  
        # 异步运行主函数，将响应转化为语音并播放  
        asyncio.run(_main(response))  
```

这次功能合并有几个优化：

1. 把pygame初始化移到函数外部，脚本首次执行的时候初始化，避免了每次都在函数内初始化，提高速度。

2. 函数增加了一个参数`text`，它的类型为`str`（字符串），默认的内容为“测试”，避免由于后端出错没有文字可说的情况导致的报错。

3. 几个`print`增加了格式化字符串输出，提高命令行交互的可读性。

这个脚本有几个潜在问题（由于是教程就无所谓修复了，但还是拿出来说一下）：

1. 生成的文件没有重新命名导致每次音频文件都会覆盖，你可以加一个重命名的操作，这样所有生成的语句都可以回溯。

2. 语音生成函数里缺乏阻断机制，也就是说如果你短时间内问了AI两次，上一句话没说完下一句就也会同时开始播放，就像两个人同时在说话，会出现声音重叠的问题。（由于异步的问题）此问题也好解决，开一个缓存区就可以，但这不是入门教程该有的内容，后续进阶可能会聊。

### 拓展：全流式TTS（FSTTS）与语音合成标记语言（SSML）

全流式TTS是一个概念，它应该叫“Full-Stream Text to Speech”，简单地说，TTS可以**同时**流式地接受文字的输入和输出，并且达到相对自然而流畅的声音。这个技术的难点在于流式输入，在自然语言里，完整的句子对于语气和语调影响很大，如何在句子还没完整输入的时候就直接能够定下这句话的语气呢？我个人认为这需要LLM和TTS引擎的双方配合，一是LLM能根据训练的参数和内置的情感模型预先输出一个情感标记，用于给接下来要输出音频的句子定下情感基调，另一方面还需要TTS引擎能够接受情感基调并流式生成语音。这需要两个LLM紧密配合且都具有相关功能。**另一个实现方法**是采用多模态输出。多模态很好地解决了两个引擎的协调问题，但问题就在于，这个配合的过程是不可控的，是黑箱。相同的一句话在不同的场合可能有千百种语气（参考“卧槽”的好几种用法），如果把这个控制过程完全交给多模态模型，最终效果很大程度上取决于这个模型的能力，这就造成上限不高，后期可拓展性欠佳的缺点。考虑到作为赛博老婆，语音识别（ASR），声纹识别 (Voiceprint Recognize) 还有离线唤醒（Offline Wake up）等技术都要与LLM进行互动，我们很难把这么多模块合并成一个超大的多模态模型，同时还有好效果，所以我们暂时使用模块化的结构，不同的功能模块各司其职。

我简单画了一个流程图，大概是这样的系统逻辑： ![](images/Audio-interaction-system.png)

语音合成标记语言是微软Azure Speech里的一个概念，它是一种基于XML的标记语言，可用于微调文本转语音输出属性，例如音调、发音、语速、音量等。与纯文本输入相比，它可以提供更多的控制权和灵活性。相比较简单的情感基调定义，SSML可以更精细地控制语音的细节，当然对于SSML的生成也有更大的挑战，需要能力较强的LLM完成这样复杂的任务，也需要并发和异步保证交互的流畅性。（希望开源的赶快有类似的，这个太贵了）SSML的开发需要大量优质标记的数据集作为支撑，同时实际生产时为了提高速度，可以使用数据库匹配的方式，预先从已有的类似场景中直接加载类似的配置，降本增效。

