---
title: 智能对话：ChatGPT的艺术
date: 2024-04-05 11:32:22
tags:
- ChatGPT
categories: 学习笔记
mathjax: true
---

曾几何时，和机器对话仿佛是在和一堵墙进行沟通——你可能得到回声，但却难觅灵魂。然而，随着时间的推进，科技的发展，现在我们迎来了ChatGPT——一个不仅能聆听还能理解，甚至能分享笑容的虚拟伙伴。但ChatGPT的真正魅力，不仅仅在于它能给出什么样的回答，而更在于你如何与它互动。本篇文章旨在探索如何通过巧妙设计的提示（Prompts），让ChatGPT变身为你的私人顾问、故事讲述者，乃至思考伙伴。现在，让我们一起跳进这个魔法般的对话世界，看看如何使ChatGPT的对话不仅仅是交流，更是一场思维的飞跃！

<!-- more -->

## 1. Prompt Engineering：艺术与技术的融合

### 1.1 什么是Prompt Engineering？

Prompt Engineering，或称提示工程，是一种艺术和技术的结合，旨在通过精确且富有创意的提示，指导ChatGPT产生你期望的回答或动作。它不仅仅是问一个问题那么简单，而是关于如何构建那个问题，以及如何通过你的问题引导出更深层次的交流和创造性的思维。

### 1.2 提示的基本结构

一个有效的提示通常包括三个部分：角色（Role）、目标（Goal）和上下文（Context）。

- **角色（Role）**：定义ChatGPT在这次对话中扮演的角色，比如旅游顾问、编程老师或是故事讲述者。
- **目标（Goal）**：明确你想要ChatGPT实现的目标，如提供旅游建议、解答编程疑惑或讲述一个故事。
- **上下文（Context）**：提供足够的背景信息，使ChatGPT能够在适当的上下文中给出回答。

掌握这三个元素的有效结合，可以极大提升与ChatGPT的互动效率和质量。

### 1.3 实践示例

假设你希望ChatGPT帮你规划一次旅行，你可以这样构建你的提示：

```bash
# 指定角色
You are a travel consultant AI, helping customers find their perfect vacation destinations.
# 提出目标
Please provide five recommendations for family-friendly vacation spots in Europe that are suitable for a 7-day trip.
# 添加上下文
Consider factors such as cultural experiences, outdoor activities, and budget-friendly options.
```

这样的提示不仅清晰地界定了ChatGPT的角色、你的目标以及需要考虑的上下文信息，从而让AI能够提供更加精准和实用的建议。

## 2. 提升交流效率的策略

与ChatGPT的互动不仅仅限于简单的问答。它能帮助你完成复杂的任务、解答疑难问题，甚至参与创造性的项目。为了使这些交流尽可能高效，以下是一些实用策略。

### 2.1 格式化引导：精准定制你的ChatGPT体验

格式化引导是与ChatGPT进行高效交流的关键技术。通过明确指示ChatGPT的角色、目标以及上下文，你可以引导这个AI以最接近你期望的方式回应。为了让AI按照你期望的格式，你可以为他提供示例，便于它更好的理解。

假设你正在规划夏日旅行，需要ChatGPT提供一些创意建议。你可以这样构建你的提示：

```bash
# 指定角色
You are a travel agency Al, helping customers find their perfect travel destination.
# 提供目标
Give me five great travel destinations for this summer.
```

通过这种方式，你不仅定义了ChatGPT的角色（AI旅行助手），还设定了具体目标（提供旅游地点的主题建议）。这样构建的提示确实能够获得有用回答。可能我们还想知道平均温度、光照时间，可以提供输出的格式要求及相关示例。这样精心构建的提示能够大大提高获得有用和创新回答的几率。

```bash
# 提供约束， 给出输出的格式要求
Format your response like this:
Location:
Best time to visit:
Avg. temperature (Celsius):
Hours of Sunshine:
Rainy Days per Month:
# 提供示例
An example output could look like this:
Location: Seychelles
Best time to visit: Feb-Apr
Avg. temperature(Celsius): 28
Hours ofSunshine per Day: 7
Rainy Days per Month: 5.5
```

+ 示例引导输出

这种方式与上面类似，也是通过提供示例，让chatgpt模仿你的结构和风格进行输出

```bash
# 提供目标
Write a tweet that explains the core idea behind ChatGPT. Use a similar tone & structure as i do in my regular tweets. But don't use the content.
# 提供示例
Here's an example:
ChatGPT and all those Al tools + projects that build up on them (e.g., AutoGPT) are all amazing and truly powerful.
And learning how to use them in an efficient way is indeed important & useful.
But all those "You must learn this tool", "Top 20 Al tools you must not miss" or "You're missing out if you're not doing XYZ' etc posts are just ridiculous.
```

```bash
Write a tweet that explains the core idea behind ChatGPT. Use a similar tone & structure as l do in my regular tweets. But don't use the content.

Here are two example tweets:
1) ChatGPT and all those Al tools + projects that build up on them (e.g., AutoGPT) are all
amazing and truly powerful.
And learning how to use them in an efficient way is indeed important & useful.
But all those "You must learn this tool", "Top 20 Al tools you must not miss" or "You're missing out if you're not doing XYZ etc. posts are just ridiculous.

2) Writing this book was quite a journey and a lot of fun! 
I took great care to pick understandable and helpful examples, include many practice
exercises and explain all core concepts in-depth.
The book covers the latest versions of React and React Router(6.4+)
```

+ 格式化输出

```python
"""
understanding their basic principles can help dispel fears and demystify these powerful Al
technologies. As we continue to explore and develop Al, it's crucial to remain curious and learn about
the advancements in this ever-evolving field, embracing the potential benefits that responsible Al
can bring to our lives.
"""
Summarize the above article, starting like this:
# 指定输出格式要求
The key takeaways ofthe article are:
-
```

### 2.2 Ask-before-Answer Prompt（先问后答）：深化理解与探索

先问后答提示是一种高级的互动策略，旨在通过首先让ChatGPT提出问题来深化对话的理解和探索。这种方法特别适用于复杂的话题或当你希望ChatGPT帮助你更全面地思考问题时。

#### 什么是先问后答提示？

先问后答提示是一种提示策略，其中用户首先引导ChatGPT提出一系列相关问题，然后再回答这些问题。这种方法促使ChatGPT在提供解决方案之前，进行更深入的思考和探索，有助于揭示问题的不同方面和潜在的解决策略。

#### 为什么使用先问后答提示？

- **促进深度理解**：通过要求ChatGPT提问，你可以评估它对话题的理解程度，同时鼓励它探索问题的更多维度。
- **发掘新视角**：ChatGPT提出的问题可能揭示你未曾考虑过的观点或信息，从而丰富了对话题的理解。
- **增加互动性**：这种提示方式增加了对话的互动性，使聊天体验更加动态和有趣。

#### 应用示例

假设你想要chatgpt辅助你写一篇博客。你可以使用先问后答提示，如下：

  ```bash
# 指定扮演角色
You are an experienced travel blogger with a focus on destinations for nature & activity lovers.
# 提供目标
Your goal is to write a blog post about snorkeling at Champagne Beach on Dominica.
# 提供约束及额外要求和信息
The post should be no longer than 750 characters and target a broad audience that goes beyond divers or nature enthusiasts.
# 让GPT询问我相关问题从而达到完善Prompt的目标
Before answering,l want you to first ask for any extra information that helps you produce a better
answer. lf you got no questions, please provide the answer instead.
  ```

通过使用先问后答提示，你可以激发更深层次的对话和探索，无论是在学术研究、创意思考还是解决实际问题中，都能发挥出意想不到的效果。希望这段内容能帮助你更有效地使用这种策略。

### 2.3 Perspective Prompting （视角引导法）：扩展思维视野

视角引导法是一种通过改变提问视角来引发不同回答的技巧，旨在通过引入新的视角或假设条件来探索问题的多个方面。这种方法特别适合于促进创造性思维、解决问题和深入理解复杂概念。

#### 什么是视角引导法？

视角引导法要求在构建提示时，明确指出从哪个特定视角来探讨问题或话题。这可以是从不同人的视角，如专家、历史人物，或者是从非人类角度，如物品、概念等。通过这种方式，可以挖掘出通常不会考虑到的洞见和解答。

#### 视角引导法的优势

- **促进深入理解**：通过考虑不同的视角，可以更全面地理解问题的各个维度和影响因素。
- **激发创新思维**：不同的视角可能会激发出创新的解决方案和想法。
- **增强对话的丰富性**：引入新的视角可以使对话更加多样化和丰富，提高互动的趣味性。

![](image-20240403112130598.png)

### 2.4 情感提示

情感提示是一种特别的对话策略，旨在通过引入情感元素来丰富与ChatGPT的互动。这种方法适用于需要表达或理解情感、建立更有人情味的交流环境时。

#### 什么是情感提示？

情感提示涉及到在提示中明确表达情绪或情感状态，或者要求ChatGPT从特定的情感视角回答问题。这种策略的目的是使对话更加生动、真实，同时探索和理解情感对人类决策和认知的影响。

#### 应用示例

假设你想要chatgpt辅助你给同事写一封邮件。你只是想表达对其工作的不满，并不是不尊重他本人，因此，你可以指示gpt按照特定的情感要求进行回答，如下：

```python
# 定义目标
Write an email response to the following customer complaint:
"Your products are garbage. The new chair broke 1 day after it was shipped. You'll hear from my lawyer."
# 要求考虑情感
when drafting the response, take the sentiment and tone of the customer into account.
```

### 2.5 其他辅助策略

ChatGPT 的一大优点是能够进行迭代式对话。如果初次回答未能满足你的需求，可以继续提问或提供更多上下文，逐步引导 ChatGPT 给出更符合预期的答案。

+ **问题分割**；让其帮助你将一个问题分割成几个问题，并逐步解决

```python
# 定义角色
You are an experienced online blogger and digital marketer. ln addition, you have a deep expertise in machine learning, artificial intelligence and specifically GPT models as well as large language models (LLMs)

# 博客是关于什么的，及受众是谁
Help me create a blog post about the internal workings of ChatGPT by providing a list of important keywords.

The blog post should describe which models get used internally by ChatGPT. lt should explain how those models were trained. All key concepts should be explained for a non-expert target audience.
# 进行问题分割
Split this task into multiple steps and then start with the first step.
```

+ **GPT生成指令**；如果你不知道如何生成生成好的prompt，你也可以让他帮你生成，给他一个期望输出，让其反推。

```python 
You are ChatGPT - an advanced Al, aiming to help users generate content.

Your goal is to write a prompt that could be used by ChatGPT users.

Given the below example output, which prompt would've yielded a similar output?

Example output:
'Al will not entirely replace web developers, but it will certainly transform the field. As Al and automation technologies continue to advance, we can expect them to handle more routine and repetitive tasks, such as basic coding and layout design. This will enable web developers to focus more on creative and strategic aspects of their work.'
```

+ 有意义的额外补充

![](/image-20240405205133565.png)

### 小试牛刀

理论部分已经讲完了，何不来小试牛刀一下呢，自己构思一下如何完成这小任务

![](/image-20240403112629682.png)

## 3. 超越常规的使用方法

ChatGPT不仅是一个强大的信息查询工具，它还可以成为你的创造伙伴。无论是编写代码、撰写文章，还是创作音乐，通过精心设计的提示，ChatGPT能够在各种创造性任务中发挥巨大作用。

+ 搜索引擎
+ 总结文本信息
+ 翻译 & 调整语气
+ 校对 & 增强文本
+ 编写代码脚本
+ 创造性写作

### 3.1 实践示例

假设我希望ChatGPT帮我完成绩点转换的工作，可以按照如下的工作流实现：

1. 上传绩点换算规则，确保其准确的理解规则

   ![](/image-20240406180741318.png)

   检查，发现其确实正确的理解了换算的标准

   ![](/image-20240406182333823.png)

2. 给他自己的成绩单，让其生成一个excel。结果发现他做的还不错

   ![](/image-20240407094254076.png)

3. 最后，便是绩点转换；还是只需将文件传输给他，提出自己的要求，静静的等待

![](/image-20240406182907025.png)

4. 这样，一个符合我们要求的excel文件就得到了，是不是很简单！

![](/image-20240406183057213.png)

试想，这样一个工作如果自己按部就班做，光是创建excel成绩单就得很久。无论从事什么工作，ChatGPT可以作为一个卓越的助手，帮助提高工作效率。

## 结语

ChatGPT的出现开启了人类与机器交流的新篇章。通过掌握提示工程的艺术与科学，我们不仅能够提高与ChatGPT的互动效率，还能开启更多创新和创造的可能性。随着技术的不断进步，我们有理由相信，未来的对话将更加智能、个性化和有趣。

希望本文能帮助你更好地理解和使用ChatGPT，开启一段充满创新和乐趣的对话之旅。记住，与ChatGPT的每一次对话，都是一次探索未知的机会。

## Reference

1. https://www.bilibili.com/video/BV13s4y1M7TJ/?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click&vd_source=7fed059ba9905c28f73d20ac51cfd427
