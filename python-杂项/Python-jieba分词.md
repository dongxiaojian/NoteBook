#### 1.安装jieba分词
	pip install jieba    #有可能会报错，使用清华源没有报错
### 2.切词的方法：jieba.cut() 和 jieba.cut_for_search()
#####　2.1 jieba.cut()
	第一个参数: 需要分词的字符串。
	第二个参数: cut_all 控制切词的模式。
		切词模式：
		   精确模式：试图将句子最精确地切开，适合文本分析；
           全模式：把句子中所有的可以成词的词语都扫描出来, 速度非常快，但是不能解决歧义问题；
	第三个参数：HMM 参数用来控制是否使用 HMM 模型

#####　2.2 jieba.cut_for_search()
		
	搜索引擎模式：在精确模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。

**以上两种方式切词，返回的结果是一个可迭代的generator对象，可以进行遍历或者转换为列表进行处理。 jieba.lcut 以及 jieba.lcut_for_search 直接返回 list**


### 3.添加自定义词典：jieba.load_userdict()
	
	参数词典文件路径的字符串，文件格式为utf-8

	词典的每行格式分为三个部分（之间用空格隔开）：
			第一部分：词语
			第二部分：词频（也可能是权重）
			第三部分：词性 （可忽略不写）



### 4.关键词提取:jieba.analyse.extract_tags()
　4.1关键词提取

	先from jieba import anallyse
	jieba.analyse.extract_tags(sentence, topK = 20, withWeight = False, allowPOS = ())
	
	参数一：sentence，为提取文本
	参数二：topK 返回几个 TF/IDF 权重最大的关键词，默认值为20。
	参数三：withWeight:是否一并返回关键词权重值，默认值为False。
	参数四：allowPOS:仅包括指定词性的词，默认值为空，即不进行筛选。
	参数五：jieba.analyse.TFIDF(idf_path=None) 新建 TFIDF 实例，idf_path 为 IDF 频率文件。
　
 4.2 关键词提取停用词

	关键词提取所使用停用词（Stop Words）文本语料库可以切换成自定义语料库的路径。
	jieba.analyse.set_stop_words(file_name) #file_name为自定义语料库的路径


### 5.调整词典：add_word()、del_word()和suggest_freq()

	  使用 add_word(word, freq=None, tag=None) 和 del_word(word) 可在程序中动态修改词典.

	  使用 suggest_freq(segment, tune=True) 可调节单个词语的词频，使其能（或不能）被分出来。

	  注意：自动计算的词频在使用 HMM 新词发现功能时可能无效。
	

### 6.并行分词（多进程分词）

	 基于python的multipprocessing模块，目前不支持windows。
	 jieba.enable_parallel(4) # 开启并行分词模式，参数为并行进程数。
	 jieba.disable_parallel()   # 关闭并行分词模式。



*注： 本人水平有限， 如有错误欢迎提出指正！如有引用， 请注明出处！！*
