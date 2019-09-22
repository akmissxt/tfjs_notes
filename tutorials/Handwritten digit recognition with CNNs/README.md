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

## 2. 开始

### 创建一个包含JavaScript的HTML页面

**将以下代码写到命名为 `index.html` html的文件。**

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TensorFlow.js Tutorial</title>

  <!-- Import TensorFlow.js -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.0/dist/tf.min.js"></script>
  <!-- Import tfjs-vis -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-vis@1.0.2/dist/tfjs-vis.umd.min.js"></script>

  <!-- Import the data file -->
  <script src="data.js" type="module"></script>

  <!-- Import the main script file -->
  <script src="script.js" type="module"></script>

</head>

<body>
</body>
</html>

```

### 创建数据和代码的 JavaScript 文件 

1. 在上面的 HTML 文件的相同文件夹下，创建名为 data.js 的文件夹然后从[这个链接](https://storage.googleapis.com/tfjs-tutorials/mnist_data.js)拷贝内容到该文件中。

2. 在第一步的相同文件夹中，创建命名为 script.js 的文件然后把下面的代码写进去。

```javascript
console.log('Hello TensorFlow');
```

## 测试一下

现在你已经将 HTML 和 JavaScript 文件创建好了，测试一下。在浏览器中打开 index.html 文件然后打开开发工具的控制台。

如果一切正常, 应该会有两个全局变量被创建出来。`tf` 是 TensorFlow.js 库的引用, `tfvis` 是 [tfjs-vis 库](https://github.com/tensorflow/tfjs-vis) 的引用.

## 3. 加载数据

在这次的教程里你将训练一个模型去学习识别像下面这样的图片中的数字。这些图片是来自于一个叫 [MNIST](http://yann.lecun.com/exdb/mnist/)的数据集的 28x28px 的灰度图片。

![mnist 4](https://codelabs.developers.google.com/codelabs/tfjs-training-classfication/img/19dce81db67e1136.png) ![mnist 3](https://codelabs.developers.google.com/codelabs/tfjs-training-classfication/img/f7c09fea8d6636e4.png) ![mnist 8](https://codelabs.developers.google.com/codelabs/tfjs-training-classfication/img/82c05a6c7f0a18e2.png)

我们已经在一个特别的 [sprite file](https://storage.googleapis.com/learnjs-data/model-builder/mnist_images.png) (大约10MB)提供了加载这些图片的代码以便我们能专注于训练模型的部分。

随意地学习 [`data.js`](https://raw.githubusercontent.com/tensorflow/tfjs-examples/master/mnist-core/data.js) 文件去了解数据是如何被加载的。或者当你完成了这次教程，创建你自己的方法去加载数据。

提供的代码包含一个带有两个公共方法的类 `MnistData` ：

- `nextTrainBatch(batchSize)`: 从训练集中返回随机的一批图片和对应的标签。
- `nextTestBatch(batchSize)`: 从测试集中返回一批图片和它们对应的标签。

MnistData类同也做了打乱数据和规范化数据的重要步骤。

数据集一共有65000张图片，我们会使用高达55000张图片去训练模型，留下10000张图片在我们完成训练以后用来测试模型的表现。而且我们会在浏览器中进行所有这些步骤。

让我们加载数据并测试其是否已经正确加载。

**将下面的代码写到 script.js 文件中。**

```javascript
import {MnistData} from './data.js';

async function showExamples(data) {
  // Create a container in the visor
  const surface =
    tfvis.visor().surface({ name: 'Input Data Examples', tab: 'Input Data'});  

  // Get the examples
  const examples = data.nextTestBatch(20);
  const numExamples = examples.xs.shape[0];
  
  // Create a canvas element to render each example
  for (let i = 0; i < numExamples; i++) {
    const imageTensor = tf.tidy(() => {
      // Reshape the image to 28x28 px
      return examples.xs
        .slice([i, 0], [1, examples.xs.shape[1]])
        .reshape([28, 28, 1]);
    });
    
    const canvas = document.createElement('canvas');
    canvas.width = 28;
    canvas.height = 28;
    canvas.style = 'margin: 4px;';
    await tf.browser.toPixels(imageTensor, canvas);
    surface.drawArea.appendChild(canvas);

    imageTensor.dispose();
  }
}

async function run() {  
  const data = new MnistData();
  await data.load();
  await showExamples(data);
}

document.addEventListener('DOMContentLoaded', run);
```

刷新页面，过几秒后你应该看到在右边有一个数字图片的面板。

![img](https://codelabs.developers.google.com/codelabs/tfjs-training-classfication/img/b675d1a8c09ddf78.png)