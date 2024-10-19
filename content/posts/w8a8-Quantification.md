---
title: "记录W8A8量化上传流程"
date: 2024-10-17T21:55:31-02:30
draft: false
tags:
  - 量化
  - 高级教程
license: "GNU General Public License v3.0"
author: "某喜欢魔物娘的大佬"
---



>  本文章经作者同意，转载自自由AI阵线社区，版权所有！

首先在autodl注册账号、选择租用/创建实例，进入如下界面：

![image1](/W8A8/image1.png)

选择按量计费，在地区中多做切换，不同地区的可选GPU型号不同。这里我选择四卡4090D，卡的数目可以在租用后随意调整至1~4卡之间，对于22B模型的量化已经足够，因此不必在意这里的选取。

数据盘扩容选择100G就足够了，因为下载的原始22B模型约44G，量化后的W8A8模型22G，共66G。

![image2](/W8A8/image2.png)

最后在镜像处选择基础镜像，我的选择如下图所示：

![image3](/W8A8/image3.png)

完成以上这些，选择立即创建，设备就租到了。设备租到以后不要直接点开机按钮，而是选择“更多”中的“无卡模式开机”，费用很低，所以可以耐心折腾：

![image4](/W8A8/image4.png)

![image5](/W8A8/image5.png)

无卡模式开机后，看到显示了ssh登录口令和快捷工具列表，点击JupyterLab：

![image6](/W8A8/image6.png)

JupyterLab中，点击“终端”按钮，做W8A8量化前的准备工作：

![image7](/W8A8/image7.png)

![image8](/W8A8/image8.png)

打开终端，可以看到它提示了数据盘/root/autodl-tmp，也就是说掏钱扩容100G的空间和初始赠送的50G空间都位于这个叫做autodl-tmp的文件夹里，往这个文件夹里放东西，这拢共150G的数据盘就会越用越少。

接下来安装量化所需的python工具，在终端中输入：

```bash
pip install llmcompressor -i https://mirrors.aliyun.com/pypi/simple/
```

就可以自动安装llmcompressor的最新版本了，截至发帖，它的最新版本是0.2.0：

![image9](/W8A8/image9.png)

![image10](/W8A8/image10.png)

这样就是安装成功了，接下来进入数据盘下载模型和量化所需的语料。先看看当前的位置，在终端中输入ls这两个字母回车：

![image11](/W8A8/image11.png)

可以看见目前终端中的目录层级可以看到和左侧文件视图里一一对应的项目：

![image12](/W8A8/image12.png)

实际上，当在JupyterLab的启动页中创建终端时，新打开的终端总是会与左侧文件视图具有一致的层级关系，所以有时如果懒得敲命令切换目录，可以在左边通过鼠标点击切换到目的目录后在右边创建新的终端来直接得到一个切换好目录的终端：

![image13](/W8A8/image13.png)

![image14](/W8A8/image14.png)

![image15](/W8A8/image15.png)

接下来下载量化所需的校准语料。在位于autodl-tmp目录下的终端中输入：

```bash
export HF_ENDPOINT="https://hf-mirror.com"
```

![image16](/W8A8/image16.png)

看似没有动静，实则已经设置好了huggingface的镜像站以加速下载，否则在中国境内是很难连接huggingface的。

在位于autodl-tmp目录下的终端中输入：

```bash
huggingface-cli download --repo-type dataset --resume-download HuggingFaceH4/ultrachat_200k --local-dir ultrachat_200k
```

![image17](/W8A8/image17.png)

这样就开始下载了，有时下载会变得越来越慢，等不及了可以在终端中按Ctrl和c的组合键强制终端中的下载进程关闭，多按几次：

![image18](/W8A8/image18.png)

然后按方向键的上键切换到下载指令，回车，再一次下载。下载进度不会丢失：

![image19](/W8A8/image19.png)

![image20](/W8A8/image20.png)

看到这个Fetching XX files: 100%，就大概知道是下载好了，不确定也没关系，再多执行几遍下载指令确认也行：

![image21](/W8A8/image21.png)

安装完量化所需的python工具、下载好量化所需的校准语料。最后，下载需要量化的模型。

我选择的模型是Mistral-Small-Instruct-2409-abliterated这个模型，在它的huggingface页面点击复制模型名称按钮，会复制到“byroneverson/Mistral-Small-Instruct-2409-abliterated”，

![image21](/W8A8/image22.png)



然后最终的下载指令就是：

```bash
huggingface-cli download --resume-download byroneverson/Mistral-Small-Instruct-2409-abliterated --local-dir Mistral-Small-Instruct-2409-abliterated
```

这里的“--local-dir Mistral-Small-Instruct-2409-abliterated”是说下载下来的文件夹叫做Mistral-Small-Instruct-2409-abliterated。

![image21](/W8A8/image23.png)

![image22](/W8A8/image24.png)

好，经过多次Ctrl+c中断并重新下载，终于下载好了。最后还要下载另外两个东西，lm_eval和vllm，这是两个用于在量化结束后评估模型的工具，简单来说就是用来跑分的，简称benchmark，其实还需要一套叫做GSM8K的数学题集的，但GSM8K可以到了要跑分时再下载，这里就只安装这两个工具：

```bash
pip install lm_eval -i https://mirrors.aliyun.com/pypi/simple/
pip install vllm -i https://mirrors.aliyun.com/pypi/simple/
```

![image23](/W8A8/image25.png)

![image23](/W8A8/image26.png)

接下来关机，结束无卡模式。

![image24](/W8A8/image27.png)

至此，所有准备工作都已完成，接下来点击开机按钮，开始烧钱。

![image25](/W8A8/image28.png)

![image23](/W8A8/image29.png)

老规矩，左边切到autodl-tmp目录，右边开个新终端直接切到autodl-tmp目录里：

![image26](/W8A8/image30.png)

![image23](/W8A8/image31.png)

然后把我准备的脚本文件quant.py从你的桌面或者是别的什么地方拖拽到JupyterLab左侧的文件管理视图的autodl-tmp目录里：

![image27](/W8A8/image32.png)

![image23](/W8A8/image33.png)

好，最后需要编辑quant.py里的参数，双击JupyterLab左侧文件管理视图autodl-tmp目录里的quant.py文件，进入quant.py编辑界面，有两个地方要改：

![image28](/W8A8/image34.png)



上面的“Qwen2.5-32B-Instruct”改成下载的原始模型的文件夹名称，在左侧的文件夹上右键重命名然后复制一下文件名就好了。

![image23](/W8A8/image35.png)

下面的这个0.9是计算出来的，计算方式是用1-（6/模型层数），例如我下载的这个模型有56层，那么6/56是0.1071428571428571，而1-0.1071428571428571约等于0.9，那么就是0.9，最后改完的quant.py文件如下图，注意文件名旁边的按钮是个黑色小圆点，说明编辑器里的修改还没有保存到文件里，切记按ctrl+s或是点击黑色小圆点来保存文件：

![image29](/W8A8/image36.png)

![image29](/W8A8/image37.png)

![image29](/W8A8/image38.png)

![image29](/W8A8/image39.png)

最后，在终端中输入python3 quant.py敲下回车，开始模型量化吧：

![image30](/W8A8/image40.png)

![image30](/W8A8/image41.png)

![image30](/W8A8/image42.png)

从12点29到14点48，一转眼两个小时过去了，估计还要半个多小时才能量化完，这里再说说量化耗时的事情，量化耗时主要和选取的校准语料的数目有关，也就是“NUM_CALIBRATION_SAMPLES”的值，这个值在脚本里写的是2048，是一个相对来说比较大的值，也是量化耗时的主要原因。基本上NUM_CALIBRATION_SAMPLES翻一倍就会导致量化耗时增加一倍。2048已经可以确保量化充分的进行了。

![image31](/W8A8/image43.png)

接近完成，内存占用也来到了75GB，顺带一提，这个监控内存占用的界面是点击“AutoPanel”打开的：

![image32](/W8A8/image44.png)

![image32](/W8A8/image45.png)

![image32](/W8A8/image46.png)

到这里为止，模型量化就算是完成了，但量化得好不好还要另说，此时就轮到评估工具出场来跑benchmark了，原始模型跑一次，量化模型跑一次。两次结果相差越小，说明量化效果越好。在控制台输入：

```bash
export HF_ENDPOINT="https://hf-mirror.com"
```

来开启huggingface镜像站加速，这是为了下载gsm8k试题集。

然后在控制台输入以下指令开始测试量化后的w8a8模型：

```bash
lm_eval --model vllm --model_args pretrained="/root/autodl-tmp/output",add_bos_token=true,tensor_parallel_size=4,max_model_len=2048,gpu_memory_utilization=0.99 --tasks gsm8k --num_fewshot 5 --limit 250 --batch_size 'auto'
```

![image32](/W8A8/image47.png)

![image32](/W8A8/image48.png)

这样就测完了量化后的W8A8模型，可以看到两个评测结果分别是0.856和0.844，这个成绩的高低无所谓，关键是和原始模型足够接近，这样才能称得上是原汁原味。另外可以看到左边多轮好些个文件夹，其中internlm2_5-20b-chat-abliterated是我在量化的时候在另一个控制台标签里下载的，我并不打算只量化一个模型。output当然就是量化后的W8A8格式的模型所在的目录，验证后如果靠谱是要上传huggingface的。root-W8A8-Dynamic-Per-Token则是output文件夹的复制，删掉就行。顺带一提，被删掉的文件只是被藏到回收站里了，仍然在占用硬盘空间，可以在autodl-tmp目录下输入“rm -rf .Trash-0”指令来清空回收站，用“du -h .”指令来查看当前目录下各个文件夹占据磁盘空间的情况。

最后，来测测原始模型的gsm8k成绩：

```bash
lm_eval --model vllm --model_args pretrained="/root/autodl-tmp/Mistral-Small-Instruct-2409-abliterated",add_bos_token=true,tensor_parallel_size=4,max_model_len=2048 --tasks gsm8k --num_fewshot 5 --limit 250 --batch_size 'auto'
```

![image34](/W8A8/image49.png)

![image34](/W8A8/image50.png)

嗯，原始模型的两个成绩分别是0.844和0.836，不错哦，和量化模型非常非常接近，甚至由于0.023左右的误差而略低于量化后的模型，总的来说非常圆满！那么接下来就是上传量化后模型至huggingface了。删除原始模型和root-W8A8-Dynamic-Per-Token，再输入指令清空回收站永绝后患！

![image34](/W8A8/image51.png)

![image34](/W8A8/image52.png)

由于我不打算浪费时间，我会在上传量化后的W8A8模型文件之前插入对internlm2_5-20b-chat-abliterated的量化，就像流水线一样不要停，6/48=0.125，分别填好文件夹名称和0.88：

![image34](/W8A8/image53.png)

![image34](/W8A8/image54.png)

![image34](/W8A8/image55.png)

重命名output文件夹为output1，避免量化下个模型的脚本删掉了上个模型的量化结果，然后启动量化脚本：

![image37](/W8A8/image56.png)

新模型的量化不再多提，现在演示如何上传模型。在圆满完成一个模型的量化后，为了节省钱，会关机并再次以无卡模式开机：

![image38](/W8A8/image57.png)

![image38](/W8A8/image58.png)

![image38](/W8A8/image59.png)

准备生成一个可用于上传模型的huggingface身份token，这方面网上教程很多这里就不多做说明了，只要注意创建身份token的时候不要全默认，全默认创建的身份token是没有上传权限的，不知道该勾哪些权限就全勾上好了。这里我已经有一个全权限勾选的token了，所以直接刷新它来重新生成就行：

![image38](/W8A8/image60.png)

![image38](/W8A8/image61.png)

![image38](/W8A8/image62.png)

然后在终端中开启镜像加速，输入指令登录：

```bash
huggingface-cli login
```

这里粘贴刚刚复制的token然后回车，粘进去看不到变化也无所谓，它就是会隐藏掉的，是正常情况：

![image40](/W8A8/image63.png)

![image40](/W8A8/image64.png)

看到Login successful就是登录成功了，然后输入上传指令：

```bash
huggingface-cli upload Mistral-Small-Instruct-2409-abliterated-W8A8-Dynamic-Per-Token ./output1 .
```

这里Mistral-Small-Instruct-2409-abliterated-W8A8-Dynamic-Per-Token就是网友们看到的你的模型仓库的名称了，我在原始模型后面添加了“-W8A8-Dynamic-Per-Token”的后缀，指令里的“output1”当然就是需要被上传的数据所在的目录了：

![image41](/W8A8/image65.png)

好，等上传完成后就完成了模型Mistral-Small-Instruct-2409-abliterated的全部量化、上传流程，上传慢就ctrl + c中断再次执行相同的上传命令，会断点续传的。

---

这就是提取后的完整内容和相应的图片标记。我已经用 Markdown 格式添加了所有图片引用。
