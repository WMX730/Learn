# AI 术语

## 1、LLM 大语言模型
目前大部分主流LLM基于Transformer架构  。
模型的生成机制就是预测最可能的下一个Token。  
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/23907b07-ebce-44be-8ce1-acbd574a55c9" />

## 2、Prompt 提示词
### 好prompt的四要素 
- **角色设定**：你是谁？
- **任务描述**：做什么？
- **输出约束**：格式和长度
- **上下文信息**：参考资料

## 3、Token 词元（计费单位）
大模型处理文本的最小单位  
需求评审：单次Token消耗 ✖ 日均调用量 ✖ 单价 = 月成本
- **英文**：一个词大于1~1.5个Token
- **中文**：一个汉字大约1.5~2个Token

## 4、RAG (Retrieval-Augmented Generation) 检索增强生成
### 核心流程
文档切段做Embedding存入向量数据库 → 用户问题也转成Embedding → 在向量库中检索语义最相似的文档段 → 把检索到的段落拼进Prompt让模型生成回答

## 5、Fine-tunning 微调
### 什么时候用微调？
- **需要特定的输出风格或语气**
- **需要模型学会你自己定义的分类体系，标准和通用标准不一样**
- **Prompt Engineering已经调到天花板了，加再多示例也没法提升效果**
### 决策路径
Prompt Engineering → RAG → 微调 

## 6、Agent 智能体
Agent能自己拆解复杂任务、规划步骤、调用外部工具执行操作、拿到结果后判断下一步怎么做。
- **核心能力**：规划 → 工具调用 → 观察调整
- **架构**：Agent = LLM + Prompt + Tool + Memory
- <img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/029ebddd-aab2-40e6-85b9-1cfbf61a31c8" />

## 7、Function Calling 函数调用
把工具描述当 API 文档写，名称、功能、参数、返回值、使用场景都要写清楚。
### 定义四件事
- **工具集**：Agent 能用哪些工具，每个工具的功能描述要写到模型能理解的程度。
- **使用条件**：什么情况调什么工具，边界要清晰。
- **优先级**：多个工具都能用时先用哪个。
- **错误处理**：工具调用失败了怎么办，超时了怎么办，返回空结果怎么办。

## 8、Embedding 向量嵌入
应用场景：RAG检索
### 三个关注点
- **Embedding 模型对中文的支持度，有些模型英文效果好中文效果差**
- **向量维度，越高越精确但存储和检索成本也越高**
- **向量数据库选型，不同数据库在检索速度、支持规模、运维成本上差异很大**

## 9、Hallucination 幻觉
主流模型在事实性问答场景下的幻觉率大概在10%-30%之间。
### 防范
- **RAG接地**：让模型基于真实文档回答而不是凭记忆编。
- **输出校验**：模型输出后做二次检查，比如数字和规则是否匹配。
- **信任度设计**：标注信息来源，低置信度转人工，让用户知道哪些是有依据的哪些是不确定的。

## 10、多轮对话与上下文管理
### 上下文保留策略
- **滑动窗口**：只保留最近N轮对话，简单直接但会忘掉早期重要信息。
- **摘要压缩**：让模型把早期对话总结成一段摘要放在前面，省Token但会丢失细节。
- **关键信息提取**：对话中的关键实体和决策单独提取存储，占Token最少但实现复杂。

## 11、Streaming 流式输出
SSE协议：Server-Sent Events。后端和模型之间建立一个持久连接，模型每预测出一个Token就实时推送到前端，不等全部生成完。
### 需求文档
- **TTFT**：(Time to First Token) 首Token延迟，用户发请求到看见第一个字的时间。
- **TPS**：(Tokens Per Second) 每秒输出Token数，决定文字蹦出来的速度。是衡量模型吞吐量和生成速度的核心指标。
- **断流重连**：网络波动时连接断了怎么续上。

## 12、MCP（Model Context Protocol） 模型上下文协议
核心思路：把模型怎么用工具这件事标准化。  
MCP定义了统一的通信格式，模型发出工具调用请求，MCP Server接收并执行，结果按标准格式返回。 
只要工具包装成MCP Server，任何支持MCP的模型和应用都能直接调用，不用单独写对接代码。
### 三个核心角色
- **MCP Host**：大模型所在的应用
- **MCP Client**：协议通信的中间层
- **MCP Server**：具体工具的服务端封装
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/6c504c2f-1184-431b-9483-e3a3ef6f8ea0" />

## 13、Temperature 温度参数
本质上是对模型输出概率分布的缩放系数。模型预测下一个Token时，会算出每个候选词的概率。  
设为0时，模型永远选概率最高的那个词，输出几乎完全一样，非常确定。  
设为1或更高时，低概率的词也有机会被选中，输出变得多样但同时不确定性变大。  
### 为什么设为0不代表不出错？
因为Temperature控制的是概率采样的随机性，不是事实准确性。如果模型对一个错误答案的概率预测本身就是最高的，Temperature设0反而会让它每次都选那个错误答案。Temperature管的是稳不稳定，不管对不对。
#### 实际产品经验
客服场景设0.1-0.3，你需要回答一致稳定；创意写作场景设0.7-0.9，你需要多样性；数据提取和格式化场景设0，你需要结果完全可预测。  
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/cddbbfbc-0e32-4943-860f-c03f3eaf19de" />

## 14、System Prompt 系统提示词
### 区分System Prompt和用户Prompt
System Prompt是对话开始前预设给模型的人设说明书，用户看不到，但它决定了AI的行为模式和能力边界。每次API调用它都会被发送一遍。
### System Prompt决定三件事
- **角色一致性**：多轮对话中AI人设不会跑偏。
- **行为边界**：哪些问题拒答、哪些格式强制遵守。
- **输出质量**：给模型足够的上下文约束减少发散。
### 核心原则
职责明确，边界清晰，有具体的行为示例。

## 15、Few-shot Learning 少样本学习
Zero-shot是直接下指令零示例，Few-shot是在Prompt里塞2-5个示例让模型照着学。
### 适用场景
- **统一输出格式**：给几个想要的输出文件示例。
- **定义分类标准**：给几个分类结果的示例。
- **风格对齐**：给几个风格范文。
### 决策路径
Zero-shot → Few-shot →微调

## 16、Chain-of -Thought 思维链
### 三种用法
- **手动触发**：在Prompt里加请一步步分析或 Let's think step by step。
- **示例驱动**：在Few-shot示例里放一个有完整推理过程的范例。
- **内置推理模型**：o1、DeepSeek R1这类模型自带思维链机制，自动先想后答。

## 17、Prompt Injection 提示词注入
攻击原理：大模型处理输入时，System Prompt和用户输入在本质上都是文本。模型很难区分系统下达的真正指令和用户伪装的系统指令。
### 防护策略
- **输入检测**：用分类模型或关键词规则过滤掉明显的注入模式。
- **Prompt加固**：在System Prompt里反复强调禁止泄露系统指令、禁止角色切换。
- **输出过滤**：查模型输出是否包含System Prompt片段或敏感信息。
- **权限隔离**：模型能调用的工具和数据按用户权限严格限制，即使注入成功也拿不到关键数据。

## 18、Pre-training 预训练
预训练出来的叫基座模型。它什么都知道一点但不会好好说话，问它问题它可能续写出一篇新闻稿而不是正常回答。变成ChatGPT那样能正常对话的，还需要SFT和RLHF两步。  
这个过程需要数千张GPU跑几个月，GPT-4级别的预训练成本超过1亿美元。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/40169fd1-460c-431f-8ac1-33d8166eae6a" />

## 19、SFT (Supervised Fine-Tuning) 监督微调
人工准备几千到几万条问题+标准答案的数据对，让模型学习在收到问题时应该用什么格式、什么风格、什么逻辑来回答。训练完之后模型从文字续写机器变成了问答助手。
### 三个关键点
- **数据质量决定一切**：放进去的标准答案就是模型学到的上限，答案质量差模型不可能学好。
- **数据不需要很多**：几千条高质量问答对就能显著改善效果，不需要百万级数据量。
- **SFT的局限**：它教会模型怎么回答，但不能教会模型判断什么回答是好的。

## 20、RLHF (Reinforcement Learning from Human Feedback) 人类反馈强化学习
### 三步流程
- **收集偏好数据**：同一个问题让模型生成多个回答，人工标注员对比排序哪个更好。
- **训练奖励模型**：用排序数据训练一个专门打分的模型，让它学会人类标注员的偏好标准。
- **PPO强化学习**：用奖励模型的打分信号去优化大模型，让它学会生成得分更高的回答。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/ddc387b4-63ec-4786-a054-75c2386ee736" />

## 21、LoRA (Low-Rank Adaptation) 低秩适应
全参微调一次GPT-3.5级别的模型，要改模型所有参数，GPU费用几万到十几万。  
LoRA的做法完全不同，冻住原始模型参数不动，在旁边加两个很小的矩阵做增量训练。训练出的结果像一个补丁贴在原模型上，只改了0.1%-1%的参数，效果却能接近全参微调。  
LoRA擅长风格和格式调整，对需要模型学习全新知识领域的场景效果有限。要灌新知识还是得靠RAG。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/830cbc42-e81e-446b-a9ce-a6a44dad0308" />

## 22、Distillation 知识蒸馏
核心思路：用大模型教小模型。先用GPT-4对大量问题生成高质量回答，再拿这些回答当训练数据去训练一个小得多的模型。  
关键在于小模型不只学答案本身，还学大模型的概率分布，就是大模型对每个Token的信心程度。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/c97ba4ee-30dc-49bb-a7ef-a043f3ddf33a" />

## 23、Quantization 量化
核心操作：把模型参数的精度从高位降到低位。精度降了，但体积也缩了。  
原来每个参数用32位浮点数存储，量化到INT8就只用8位，量化到INT4只用4位。  
量化方案选型时关注两个指标：困惑度损失越小越好，推理速度提升倍数越大越好。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/80d9370d-2108-472d-9874-ddc9975c4463" />

## 24、Inference 推理
训练是一次性投入，推理是持续性支出。模型训练好之后每用一次就是一次推理，每次推理都要花钱。  
推理成本： 输入Token ✖ 输入单价 + 输出Token ✖ 输出单价 = 单次成本
### 三个指标
- **延迟Latency**：用户发出请求到收到完整回答的时间，直接影响用户体验。
- **吞吐量Throughput**：系统每秒能处理多少并发请求，直接影响能服务多少用户。
- **单次成本Cost per Query**：每次调用花多少钱，直接影响商业模型。
### 优化推理成本
量化降精度、蒸馏换小模型、Prompt精简减Token、缓存高频问答结果、批量推理提高GPU利用率。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/671fa0d9-e2f7-49a6-9bc1-3e43133f5ee1" />

## 25、Transformer
Transformer是一种基于自注意力机制（Self-Attention）的深度学习模型，主要用于自然语言处理（NLP）和计算机视觉（CV）任务。  
所有主流大模型的底层都是Transformer架构，2017 年Google提出。  
核心代价：计算量和输入长度的平方成正比。
### 架构
### 核心特性
- **并行计算**：早期的RNN架构必须一个词一个词按顺序处理，Transformer可以同时看所有词。  
- **长距离依赖**：每个词都能直接关注到文本里任何位置的其他词，不会像 RNN 那样远距离信息衰减。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/7c1eb583-e18f-4dbb-9df0-b784e378c7b3" />

## 26、Attention 注意力机制
Attention是Transformer的核心引擎。  
关键信息放在Prompt开头和结尾效果最好，放在中间容易被忽略，这叫中间迷失问题。  
### Multi-Head Attention  
是进一步升级，不是一组注意力而是多组同时计算，每组关注不同维度的信息。一组关注语义，一组关注语法结构，一组关注位置关系。多个视角同时分析，理解就更全面。

## 27、开源模型 vs 闭源模型
### 开源模型
GPT-4、Claude、Gemini这些。优势是效果上限最高、即开即用不需要运维、供应商持续升级你坐享其成。劣势是数据要传到第三方服务器、按量付费长期成本高、一旦供应商调价或停服你毫无办法。
### 闭源模型
Llama、Qwen、DeepSeek这些。优势是可以部署在自己服务器上数据不出域、可以深度微调定制、长期来看成本更可控。劣势是需要自建GPU集群和运维团队、效果可能稍弱于顶尖闭源、需要自己跟进模型升级。
<img width="640" height="640" alt="image" src="https://github.com/user-attachments/assets/c6f80799-a7e1-4296-b15c-bdc4fffeb0f3" />

## 28、多模态模型


## 29、Context Window 上下文窗口


## 30、Vector Database 向量数据库


## 31、Chunking 文档分块


## 32、Reranking 重排序


## 33、Hybrid Search 混合搜索


## 34、Knowledge Graph 知识图谱


## 35、知识库建设


## 36、 文档解析


## 37、Grounding 接地


## 38、Workflow 工作流


## 39、Multi-Agent 多智能体


## 40、Planning 规划能力


## 41、Memory 记忆机制


## 42、ReAct 推理与行动


## 43、 Tool Use 工具使用


## 44、Orchestrator 编排器


## 45、 TTS 文本转语音


## 46、ASR 语音识别


## 47、OCR 光学字符识别


## 48、Text-to-Image 文生图


## 49、NER 命名实体识别


## 50、Intent Recognition 意图识别


## 51、Precision 精确率


## 52、Recall 召回率


## 53、Bad Case 分析


## 54、数据标注


## 55、Human Evaluation 人工评测


## 56、API


## 57、Latency 延迟


## 58、Rate Limiting 限流


## 59、 灰度发布


## 60、数据飞轮
### 飞轮闭环
产品上线 → 收集用户数据 → 优化模型 → 体验提升 → 吸引更多用户 → 更多数据
### 数据收集机制
- **隐式反馈**
- **显式反馈**
- **行为数据**
- **转化数据**



