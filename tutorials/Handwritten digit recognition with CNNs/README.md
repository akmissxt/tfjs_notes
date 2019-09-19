[TOC]

## 1. 简介

在这个教程里，我们会用卷积神经网络建立一个TensorFlow.js模型去识别手写数字。首先，我们让识别器去"看"数千张手写数字的图片和它们的标签。然后，我们会用模型没有看过的的测试数据来评估识别器的识别准确性。

这个任务被看作是分类问题，因为我们是训练模型去给输入的图片进行分配类目。我们会通过向其展示许多带有正确输出的例子来训练模型，这称为监督学习。

### 你将构建的

你将会完成一个用TensorFlow.js在浏览器里训练模型的网页。给出一张特定大小的黑白图片，他会识别出图片里出现的数字。包含一下步骤：

- 加载数据。
- 定义模型结构。
- 训练模型并监测训练的性能。
- 通过做一些预测来评估训练好的模型。

### 你将学到的

- [x] 使用 TensorFlow.js Layers API 创建卷积模型的TensorFlow.js 语法。
- [x] 在 TensorFlow.js 中规划分类问题。
- [x] 如何用 tfjs-vis 库监控浏览里的模型训练。

### 你需要的

- 最近版本的Chrome或其它现代浏览器。
- 文本编辑器，既可以是你机器本地运行的，也可以是通过网络运行的如 [Codepen](<https://codepen.io/>)  或者 [Glitch](<https://glitch.com/>) 。
- HTML、CSS、JavaScript相关知识和 Chrome 开发者工具（或者是你喜欢的浏览器开发工具）。
- 对神经网络有较好的概念理解。如果你需要介绍或复习这些，可以考虑观看 [3blue1brown的这个视频](<https://www.youtube.com/watch?v=aircAruvnKk>) 或者 [Ashi Krishnan的JS深度学习](<https://www.youtube.com/watch?v=SV-cgdobtTA>) 。

你可以学习[第一个教程](<https://codelabs.developers.google.com/codelabs/tfjs-training-regression/index.html#0>)的材料来适应一下。