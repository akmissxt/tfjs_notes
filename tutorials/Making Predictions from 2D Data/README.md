[TOC]

## 1.简介

在这个教程中你将使用一组汽车的数字数据集训练一个模型并且预测。

这个练习将会展示训练不同类型模型的常用步骤，但是会使用一个小的数据集和一个简单（浅显）的模型。主要目的是帮助你熟悉TensorFlow.js训练模型相关的基本术语、概念和语法，并且为将来的探索和学习提供垫脚石。

因为我们要使用训练一个模型来预测连续的数字，此任务有时会被称为回归任务。我们将通过向其展示一些附有正确输出的输入数据作为例子来训练模型。这种被称为监督学习。

### 你将构建的

你将会完成一个用TensorFlow.js在浏览器里训练模型的网页。给出一辆车的"Horsepower"（马力），模型会学会预测它的"Miles per Gallon" (MPG)（耗油量）。

要做到这一点，你将：

- 准备并加载训练用的数据。
- 定义模型结构。
- 训练模型并监测训练时的性能。
- 通过进行数次预测评估训练好的模型。

### 你将学到的

- [x] 机器学习的最佳实践，包括打乱数据和规范化。
- [x] 使用 [tf.layers API](<https://js.tensorflow.org/api/latest/#Layers>) 创建模型的TensorFlow.js语法。
- [x] 如何使用 [tfjs-vjs library](<https://github.com/tensorflow/tfjs-vis>) 监控在浏览器内的训练。

### 你需要的

- 最近版本的Chrome或其它现代浏览器。
- 文本编辑器，既可以是你机器本地运行的，也可以是通过网络运行的如 [Codepen](<https://codepen.io/>)  或者 [Glitch](<https://glitch.com/>) 。
- HTML、CSS、JavaScript相关知识和 Chrome 开发者工具（或者是你喜欢的浏览器开发工具）。
- 对神经网络有较好的概念理解。如果你需要介绍或复习这些，可以考虑观看 [3blue1brown的这个视频](<https://www.youtube.com/watch?v=aircAruvnKk>) 或者 [Ashi Krishnan的JS深度学习](<https://www.youtube.com/watch?v=SV-cgdobtTA>) 。

## 2.开始

#### 创建一个包含JavaScript文件的HTML页面

把下面的代码拷贝到命名为 `index.html` 的文件。

```html
<!DOCTYPE html>
<html>
<head>
  <title>TensorFlow.js Tutorial</title>

  <!-- Import TensorFlow.js -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.0/dist/tf.min.js"></script>
  <!-- Import tfjs-vis -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-vis@1.0.2/dist/tfjs-vis.umd.min.js"></script>

  <!-- Import the main script file -->
  <script src="script.js"></script>

</head>

<body>
</body>
</html>
```

#### 创建JavaScript代码文件

在上面创建的HTML文件的同一文件夹下创建一个命名为 `script.js` 的文件并把下面代码拷贝进去。

```javascript
console.log('Hello TensorFlow');
```

 #### 测试一下

现在你已经创建好了HTML文件和JavaScript文件，来测试一下。在浏览器里面打开 index.html 文件然后开启控制台。

如果一切正常，开发者工具的控制台应该会有两个全局变量已经创建并可用：

- `tf` 是 TensorFlow.js 库的引用
- `tfvis` 是tfjs-vis 库的引用

打开你浏览器的开发者工具，在控制台输出你应该看到一条 `Hello TensorFlow` 的输出信息。如果是这样，你已经准备好进入下一步了。

## 3. 加载、格式化并可视化输入数据

首先让我们加载、格式化并可视化我们想用来训练模型的数据。

我们会从一个事先准备好的JSON文件加载车的数据集，里面包含了每一辆给定的车的许多不同特性。对应这个教程，我们只会抽取关于Horsepower和Miles Per Gallon的数据。

### 添加获取数据的函数

把下面的代码添加到 `script.js`  文件。

```javascript
/**
 * Get the car data reduced to just the variables we are interested
 * and cleaned of missing data.
 */
async function getData() {
  const carsDataReq = await fetch('https://storage.googleapis.com/tfjs-tutorials/carsData.json');  
  const carsData = await carsDataReq.json();  
  const cleaned = carsData.map(car => ({
    mpg: car.Miles_per_Gallon,
    horsepower: car.Horsepower,
  }))
  .filter(car => (car.mpg != null && car.horsepower != null));
  
  return cleaned;
}

```

这段代码也会移除那些没有定义horsepower和miles per gallon属性的项。让我们在散点图中绘制这些数据来看看它是什么样子。

### 添加生成散点图的函数和监听事件

把下面的代码添加到 `script.js`  文件。

```javascript
async function run() {
  // Load and plot the original input data that we are going to train on.
  const data = await getData();
  const values = data.map(d => ({
    x: d.horsepower,
    y: d.mpg,
  }));

  tfvis.render.scatterplot(
    {name: 'Horsepower v MPG'},
    {values}, 
    {
      xLabel: 'Horsepower',
      yLabel: 'MPG',
      height: 300
    }
  );

  // More code will be added below
}

document.addEventListener('DOMContentLoaded', run);
```

当你刷新页面，你应该看到在页面左手边（右手边？）有散点图数据的面板，它看起来应该像下面这样。

![img](https://codelabs.developers.google.com/codelabs/tfjs-training-regression/img/6a7452e88f16d8e.png)