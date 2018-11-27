# 人工智能写词机

### 项目简介：

以《全宋词》为数据，应用TensorFlow框架和LSTM模型，训练出可以针对给定词牌写出完整的一首词的人工智能写词机。

- 代码地址：https://github.com/koala0qoo/quiz-w11
- 运行地址：https://www.tinymind.com/executions/ajyrcf84

### 文件内容：

- embedding文件夹
    - word2vec_basic_ch.py 执行embedding操作的脚本
    - word2vec_basic_ch.ipynb 对代码的分析
    - tsne.png 对embedding结果的可视化
- QuanSongCi.txt 《全宋词》文本
- flags.py 命令行参数处理
- utils.py 读取与生成训练数据
- model.py 模型定义
- train.py 训练脚本
- sample.py 用最近的checkpoint，对三个词牌进行生成操作
- train_eval.py 执行训练与验证（词牌生成）过程

### 第一部分：Word Embedding

#### 理解 Word Embedding

Word embedding（词嵌入）的基本含义，是把高维空间中的稀疏表达嵌入到一个低维空间，形成在新空间上的稠密表达。从本质上来说，word embedding 的过程就是找到词语从高维空间到低维空间的一个线性映射。同时，映射后的新表达还要尽量保留原来的有用信息。

从神经网络的角度看，embedding 可以看作输入为 one hot、神经元个数为词向量维数的全连接层。而这个全连接层的参数（也即权重矩阵），定义了上述线性变换的映射规则。

这一映射的作用可以看作是进行了一次抽提特征，即从输入的词语中抽提出给定维数的特征。embedding 后生成的词向量，相当于用共同的特征进行了量化表达，因此可以在一定程度上体现出词语之间的关联，而词向量间的距离也可以体现出词语之间的相似性。

至于网络最终会训练出什么样的映射规则取决于学习任务。比如在 Skip-gram 模型中，学习任务为预测给定词的上下文，因此 embedding 层会倾向于提取出更有利于这一任务的特征。

代码解释见 word2vec_basic_ch.ipynb 文件。

#### 结果分析：

可以看到，某些意义相近的词在图中呈现出一定程度的聚集，但是由于语料库较小，效果并不是很好。图中仅举例标示出较明显的几处：

- 除一以外的数字，常作为数量词出现（一比较特殊，会有更多的含义和用法）
- 金、银、玉、瑶等表示玉石宝物的名词
- 雨、雪、霜、露等表示自然现象的名词
- 楼、台、亭、阁等表示建筑的名词
![输入图片说明](https://github.com/koala0qoo/Lyrics_writing_machine/blob/master/embedding/tsne.png?raw=true)
由于本次训练采用的是Skip-gram模型，该模型的学习任务是预测给定词的上下文，因此embedding后具有相似表达的词，更可能具有相同的上下文，因而也更可能具有相似的语法功能或含义。


### 第二部分：RNN 训练写词机

#### 理解 RNN

RNN：

传统DNN中，输入的每一条数据间是没有关联的，即只考虑当前一个状态。而RNN的特点是考虑一个连续的状态序列，每一条数据的输出不再独立，而是依赖于与他相邻的一条或多条数据的输入或输出，相当于赋予网络一定的记忆功能。

这种记忆功能是通过每个状态的隐藏层实现的。每一条输入信息都会转化为“隐藏状态”并在状态序列的隐藏层中向后传递。也就是说，每个时刻的输入由上一时刻的隐藏状态和本时刻的输入信息共同决定。理论上，每一时刻的隐藏状态可以一直向后传递，相当于每个时刻都可以捕获到前面所有时刻的信息。

这种负责传递记忆的隐藏层可以有多个，本项目中设置了3个。

LSTM：

上文提到，理论上隐藏状态可以一直向后传播。但实际上，如果依赖较长，在进行反向传播时容易发生梯度消失，导致权值无法更新。或者说，这样的网络是无法记住较长时间以前的信息的。而 LSTM（Long Short Term Memory）作为对传统RNN的改进，其主要目的是让网络可以拥有“较长的短期记忆”（不是记的时间长，而是记的内容多）。

LSTM 在传统 RNN 的隐藏状态的基础上增加了一个“细胞状态”，用于记忆的储存和传递。RNN 中用于传递记忆的是隐藏状态 h，而 h 每次向下一个时刻传递时都会乘以同一个矩阵 W（也是产生梯度消失或爆炸的原因）；LSTM 中的细胞状态 c 传递给下一个时刻的信息是上一时刻的信息和本时刻新输入信息的加和，因此不会受到矩阵连乘的影响。

但是，由于并不是所有的信息都是有意义的，LSTM 通过一个“遗忘门”控制要遗忘哪些之前的信息，通过“输入门”控制要记住哪些当前输入的信息，而最后的输出则通过“输出门”筛选后得到。这些门都以 sigmoid 函数的形式起作用。不同于权重矩阵，sigmoid 产生的是一个遗忘的概率，在 0-1 之间，接近 0 倾向于遗忘，接近 1 则倾向于记住。这样就保证了有用信息可以长时间传递。

模型实现过程见 model.py 文件。

#### 结果分析：

输出结果

![输入图片说明](https://github.com/koala0qoo/img/blob/master/log.png?raw=true)


