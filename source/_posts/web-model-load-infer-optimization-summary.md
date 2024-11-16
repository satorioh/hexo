---
title: Next.js + Tensorflow.js的模型加载与推理优化小结
date: 2024-04-30 11:36:24
categories: CV
tags:
  - 目标检测
  - yolo
  - tensorflow
  - nextjs
permalink: web-model-load-infer-optimization-summary
---

在上一篇[《启用 WebGPU 加速 Web 端模型推理》](https://roubin.me/enable-webgpu-accelerate-model-inference/)中提到“可以将 tensorflow.js 和现代化的前端框架结合”，基于这个想法，我继续展开迭代，将代码迁移到 Next.js 14 框架中，并针对模型在浏览器端遇到的一些加载和推理问题，进行了一番优化，整个过程大致包含如下步骤：<!--more-->

- 将 tensorflow.js 与 Next.js 整合

- 基于 IndexedDB 的模型文件缓存

- 针对 webgl backend 的内存管理

- 针对 webgl backend 的模型 warm up

- 基于 webgl backend 的推理速度对比

### 一、与 Next.js 的整合
使用的版本：Next.js 14.2.3，Tensorflow.js 4.18.0

这部分其实是比较简单的，主要是将原本的vanilla js转换成react代码，因为绝大部分代码都需要跑在client side，所以并不涉及RSC，加之原始代码量也不多，所以没花多少功夫。值得留意的一点是web worker在Next.js中的写法略有不同，代码如下：
```javascript
// page.tsx
import { useEffect, useRef, useCallback } from "react";

export default function Index() {
  const workerRef = useRef<Worker>();

  useEffect(() => {
    workerRef.current = new Worker(new URL("../worker.ts", import.meta.url));
    workerRef.current.onmessage = (event: MessageEvent<number>) =>
      alert(`WebWorker Response => ${event.data}`);
    return () => {
      workerRef.current?.terminate();
    };
  }, []);

  const handleWork = useCallback(async () => {
    workerRef.current?.postMessage(100000);
  }, []);

  return (
    <>
      <p>Do work in a WebWorker!</p>
      <button onClick={handleWork}>Calculate PI</button>
    </>
  );
}
```
```javascript
// worker.ts
import pi from "./utils/pi";

addEventListener("message", (event: MessageEvent<number>) => {
  postMessage(pi(event.data));
});

```

### 二、使用 IndexedDB
#### 1.缓存模型文件
IndexedDB 是一种浏览器内建的数据库，拥有比 localStorage 大得多的容量，还支持索引和事务，可用于离线数据的存储，并且浏览器的支持度也不错：

![next_web_indexeddb](https://roubin.me/images/next_web_indexeddb.png)

因为模型文件通常比较大，使用 IndexedDB 缓存到客户端，可以有效提升页面二次加载的速度。

tensorflow.js 的`model.save`方法支持传入一个 IndexedDB url，会自动将下载的模型文件储存到客户端的IndexedDB中，然后可以通过`loadGraphModel`加载刚才的 IndexedDB url来初始化模型，代码如下：
```javascript
const IDB_URL = "indexeddb://rps-model";

// 模型保存到IndexedDB
await model.save(IDB_URL);

// 从IndexedDB加载模型
const model = await tf.loadLayersModel(IDB_URL);
```

保存后的模型数据如下图：`tensorflowjs`相当于一个db，其中包含2个table（model_info_store和models_store）
![next_web_indexeddb_saved](https://roubin.me/images/next_web_indexeddb_saved.png)

之后就可以通过如下代码，来判断本地是否已经保存了模型，从而避免重复下载
```javascript
const modelInfo = await tf.io.listModels(); // 列出客户端已经保存的模型

if (modelInfo[IDB_URL]) {// 已经存在模型文件
    ...
}
```

#### 2.更新模型文件
在缓存了模型文件后，很自然的会想到一个问题：后续该如何更新呢？

由于我的线上模型文件存储在AWS S3桶中，这里采用的策略是：通过比对本地模型的saved time和线上模型的lastUpdate time来判断是否更新，当然更严谨的方式，我觉得还是应该使用version来控制。使用saved time的好处是，tf在保存模型时自动会添加这个字段，不需要手动操作了。

对于从S3桶中获取lastUpdate time，考虑到当前请求是无状态的，其实是有几种方式可以选择的：
- next.js client side -> S3
- next.js server side -> S3 -> client side
- backend -> S3 -> client side

最终我选择了第3种，原因如下：
- 如果选择第1种方式，势必需要在client端加载s3-sdk，以及配置访问密钥等，相对来说不够安全，而且，我也不想把client搞的太重
- 如果选择第2种方式，后续如果请求要改为有状态的（比如鉴权）会有点麻烦
- 因为之前在AWS EC2上用Django构建了一个后端服务，集成了S3的访问，所以这里直接加个接口会更方便

具体实现如下：

Django添加的查询lastUpdateTime接口：
```python
def get_model_last_update_time(model_name, config_file='model.json'):
    """
    获取模型model.json文件的上次更新时间:'Key': 'tfjs/model/rps/model.json'
    :param model_name: 模型名称
    :param config_file: 模型配置文件名称
    :return: "2024-04-28T06:14:21Z"
    """
    key = f'{settings.S3_WEBML_FOLDER}/model/{model_name}/{config_file}'
    logger.info(f"get model last update time -> {key}")
    try:
        objects = s3.list_objects_v2(Bucket=settings.S3_SIG_BUCKET_NAME)
        if 'Contents' in objects:
            obj = [content for content in objects['Contents'] if content['Key'] == key]
            return obj[0]['LastModified']
        else:
            logger.error(f"get model last update time error -> No objects in the bucket")
            return None
    except Exception as e:
        logger.error(f"get model last update time error -> {e}")
        return None
```
client side拿到lastUpdateTime后进行比对
```javascript
async function isModelLatest(dateSaved: Date) {
  const lastUpdateTime = await getModelLastUpdateTime();
  if (!lastUpdateTime) return true;
  console.log(
    `model lastUpdateTime: ${lastUpdateTime.toLocaleString()}, savedTime: ${dateSaved.toLocaleString()}`,
  );
  return dateSaved.getTime() >= lastUpdateTime.getTime();
}
```
这样当线上模型更新后，本地也能拉取到，唯一不好的是多了一次请求

### 三、启用 webgl backend
#### 1.Next.js + tf wasm backend的问题
这里也算是因祸得福，之前看一些文章讲wasm backend在启用SIMD+multi threads后，性能要优于webgl backend，以及后续webgpu会逐步取代webgl，所以一直没太关注webgl

但这次在整合Next.js时，我发现无法启用wasm backend的SIMD+multi threads，原因是根据 [tf official readme for JS Minification](https://github.com/tensorflow/tfjs/tree/master/tfjs-backend-wasm) 这一节的说明：在基于bundlers系统（webpack）打包压缩时，需要将terserPlugin中的`typeofs` compress option关闭，才能启用SIMD+multi threads，不然会报错（实际过程中我也遇到了）
![wasm_limit](https://roubin.me/images/next_web_wasm_limit.png)

但尴尬的是Next.js目前还不支持自定义terser option，详情: [Feature Request: Support for custom terser options](https://github.com/vercel/next.js/discussions/24275)，所以这次就使用了webgl backend，结果比预想要好的多（测试数据稍后附上）

#### 2.针对 webgl backend 的内存管理
在启用 webgl backend后，尝试运行，发现有内存泄露，浏览器给出了warning:`High memory usage in GPU: 1029.91 MB, most likely due to a memory leak`，查询[tf官方文档](https://www.tensorflow.org/js/guide/platform_environment?hl=zh-cn)得知对于webgl backend需要手动管理内存
![memory_leak](https://roubin.me/images/next_web_memory_leak.png)

在模型推理过程中，GPU memory中存放着tensor数据，可以通过`tf.memory()`来查看具体的tensor数量：
```javascript
console.log("numTensors: " + tf.memory().numTensors);
```
我分别打印了推理中和推理后的tensor数量（如下图），可以看到都在不断增长
![memory_increas](https://roubin.me/images/next_web_memory_increase.png)

tf官方提供了多种内存清理的方法，这里我用到的是`tf.tidy`和`tf.dispose`
##### （1）tf.tidy
针对同步代码中创建的tensor，使用`tf.tidy`包裹后，它会自动做清理
```javascript
return tf.tidy(() => {
      const tf_img = tf.browser.fromPixels(input);
      const inputs = tf_img.div(255.0).expandDims().toFloat();
      const outputs = model?.predict(inputs);
      console.log("numTensors (in predict): " + tf.memory().numTensors);
      if (outputs instanceof tf.Tensor) {
        return outputs.dataSync();
      }
    });
```
这里存在的问题是：被`tf.tidy`包裹的代码，只能返回同步数据（如下图）
![tf_tidy](https://roubin.me/images/next_web_tf_tidy.png)

而同步数据在推理过程中，势必存在性能问题，虽然tensor数量被控制住了，但是浏览器给出了warning（如下图），提示使用async api
![memory_sync](https://roubin.me/images/next_web_memory_sync.png)

那如何针对异步数据，进行内存清理？这里可以结合使用`tf.dispose`
##### （2）tf.tidy + tf.dispose
核心代码如下：
```javascript
// 推理中优化
return tf.tidy(() => {
      const tf_img = tf.browser.fromPixels(input);
      const inputs = tf_img.div(255.0).expandDims().toFloat();
      const outputs = model?.predict(inputs);
      console.log("numTensors (in predict): " + tf.memory().numTensors);
      if (outputs instanceof tf.Tensor) {
        return outputs;
      }
});
```
```javascript
// 推理后优化
addEventListener("message", async (event: MessageEvent) => {
  const { input, startTime } = event.data;
  const predict = await run_model(input);
  if (predict) {
    const result = await predict.data();
    postMessage({ type: "modelResult", result, startTime });
    tf.dispose(predict);
  }
  console.log("numTensors (outside predict): " + tf.memory().numTensors);
});
```
推理中，使用`tf.tidy`包裹同步代码自动清理，推理后，使用`tf.dispose`手动清理，这样即保证了异步性能，又控制住了GPU内存（如下图）
![memory_async](https://roubin.me/images/next_web_memory_async.png)

### 四、针对 webgl backend 的模型warm up
测试中遇到的另一个问题是，在推理开始时，会有短暂的几秒延迟，之前使用webgpu并没有这样的现象，查询[tf官方文档](https://www.tensorflow.org/js/guide/platform_environment?hl=zh-cn)后得知webgl需要做model warm up（如下图）
![model_warm_up](https://roubin.me/images/next_web_model_warm_up.png)
warm up就是让模型“热热身”，再开始正式推理，实现代码如下：
```javascript
function warmupModel(model: tf.GraphModel) {
    console.log("Warmup model before using real data");
    const inputShape = model.inputs[0].shape;
    if (inputShape) {
        const warmInput = tf.zeros(inputShape);
        const warmupResult = model.predict(warmInput);
        tf.dispose([warmupResult, warmInput]);
    }
}
```

### 五、基于 webgl backend 的推理速度对比
这次也顺带在PC和mobile上，测了下webgl backend的推理速度，结合之前webgpu和wasm的测试数据，汇总如下：

PC端：
![pc_data](https://roubin.me/images/next_web_pc_data.png)
![pc_chart](https://roubin.me/images/next_web_pc_chart.png)

mobile端：
![mobile_data](https://roubin.me/images/next_web_mobile_data.png)
![mobile_chart](https://roubin.me/images/next_web_mobile_chart.png)

总体感觉就是：比wasm有优势，和webgpu差距其实并不大，在某些低端设备上还更快，而且考虑到浏览器兼容性，webgl会是更好的选择

### 六、小结
在将Tensorflow.js和Next.js整合与优化后，整体Web ML框架更趋完善合理，同时也为后续在web端构建AI原生应用提供了便利；另外也看到了webgl的优势，在webgpu还没普及开之前，它也是一种理想的选择。

**完整代码：**[这里](https://github.com/satorioh/next_web_ai)

**演示地址：**[Next Web ML](https://next.regulusai.top/)

> 版权声明：本文为博主原创文章，转载请注明作者和出处
> 作者：CV肉饼王
> 链接：https://roubin.me/web-model-load-infer-optimization-summary/

参考文章：

[tf wasm backend readme](https://github.com/tensorflow/tfjs/tree/master/tfjs-backend-wasm)

[tensorflow 平台和环境](https://www.tensorflow.org/js/guide/platform_environment?hl=zh-cn)

[tensorflow.js api](https://js.tensorflow.org/api/latest/)

[Object detection in the image using TensorFlow in NextJS](https://medium.com/@neerajvageele451/object-detection-in-the-image-using-tensorflow-in-nextjs-3577e64280bf)

[Hand pose detection with TensorFlow.js and Next.js](https://medium.com/@felix.p.lindgren/hand-pose-detection-with-tensorflow-js-and-next-js-b87038c58918)

[Client-Side AI with Transformers.Js, Next.js, and Web Worker Threads](https://medium.com/techhappily/client-side-ai-with-transformers-js-next-js-and-web-worker-threads-259f6d955918)

[浏览器数据库 IndexedDB 入门教程](https://www.ruanyifeng.com/blog/2018/07/indexeddb.html)
