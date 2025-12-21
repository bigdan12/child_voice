🧒 儿童语音欺骗攻击与防御
—— 欺骗攻击建模 · 防御机制 · 真实度可控合成
构建首个面向儿童语音安全的 攻防一体化研究体系：从高保真伪造生成 → 构建检测基准 → 实现安全可控输出

🔍 背景（Motivation）
✅ 成人语音伪造与鉴别已受广泛关注（如 ASVspoof）
⚠️ 儿童语音领域仍为空白：
缺乏专门的伪造检测标准
儿童语音数据稀缺、隐私敏感，难以获取
TTS/VC 技术滥用风险加剧（如“AI绑架”诈骗）
🎯 本项目目标：建立一个 从攻击模拟到防御识别再到生成控制 的完整闭环系统，推动儿童语音合成技术的安全发展。

❗ 问题与挑战
全屏
复制
任务	当前瓶颈
儿童语音生成	数据少、发音不准 → 对齐困难 → 早停、乱断句；SFT 微调易“平滑”掉童声特征
儿童语音防伪	无公开伪造数据集，无法训练有效鉴别模型
生成可控性	高质量 ≠ 高真实感，缺乏对“是否像真孩子”的主动控制能力
🎯 目标
构建全球首个 儿童语音安全研究框架：

text
[攻击] → [防御] → [可控生成]
实现：

可复现的儿童语音伪造能力（用于测试）
首个儿童语音伪造基准库 FakeKidBank
支持“真实度调节”的安全语音生成接口
🛠️ 方案详解
1. 儿童语音生成系统（Attack Model）
问题
发音缺陷导致强制对齐失败，TTS 生成常出现 早停、跳字、乱断句
SFT 微调后童声音色“被成人化”，失去稚嫩感
解决方案
🔹 LLM（Text2Token）模块优化
年龄感建模
训练 年龄打分器（Age Scorer）：基于真实儿童语音回归听感年龄（3~5岁）
利用打分器筛选高质量“儿童风格”合成数据
联合训练文本编码与年龄特征，使语义输出更贴近儿童语境
韵律边界控制分词断句
使用 MFA + BERT 联合标注韵律边界：
MFA 提供音素级对齐
BERT 提供上下文语义理解
输出 [逗号]、[句号]、[停顿] 等韵律标签
将其作为条件输入 TTS 模型，精准控制停顿位置，提升自然度
🔹 Flow-Matching（Token2Mel）模块优化
引入 flow-encoder 解耦 speech token 与音色信息
在保留原始发音内容的同时，精确还原目标童声音色（timbre & brightness）
显著改善“早停”、“音色漂移”等问题
🎯 效果：在不放大儿童天然发音缺陷的前提下，实现高相似度、高自然度的童声克隆。

2. 儿童语音鉴别模型（Defense Model）
问题
无公开可用的“儿童语音伪造”数据集
现有鉴伪模型在儿童场景下性能骤降
做法：构建首个儿童语音伪造数据库 —— FakeKidBank

类型	来源	数量
真实儿童语音	ChildMandarin, TAL-Speech, 自采数据	50+ 小时
伪造语音（TTS）	小模型（VITS、FastSpeech）、大模型（IndexTTS、SparkTTS、CosyVoice2）	300+ 小时
伪造语音（VC）	小模型（FreeVC、DDDMVC、YourTTS）、大模型（Seed-VC、MeanVC、CosyVoice）	300+ 小时
声码器重合成	HiFi-GAN、BigVGAN、MelGAN	300+ 小时


🔧 基于该数据集训练首个儿童专用鉴伪模型：

模型架构：ECAPA-TDNN + Self-Attention Pooling
输入特征：mel-spectrogram + F0 + Jitter/Shimmer
输出：真实性得分（real/fake）
目标：EER < 10%
3. 真实度可控生成（Realness Control）
核心思想
让合法系统能“生成高质量但低真实感”的语音，防止被恶意滥用。

方法：构建 可控声码器（Realness-Controlled Vocoder）
在 HiFi-GAN 中引入 realness_condition 向量：
python
audio = vocoder(mel, realness_flag=True/False)
训练方式：使用 真实语音 + 伪造语音 联合训练
学习目标：通过 contrastive learning 区分“高真实”与“可察觉为假”的输出
全屏
复制
模式	表现
realness=True	高保真，用于演示/产品
realness=False	加入轻微 artifacts（相位扰动、高频衰减），人耳可感知“非真实”
应用价值
家长听到“孩子求救”电话时，若声音有机械感 → 提高警惕
社交平台审核自动拦截“异常真实”的合成语音
教育机器人提供“安全模式”输出
