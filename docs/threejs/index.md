---
Author: pony@diynova.com
Date: 2021-11-25 23:50:41
LastEditors: pony@diynova.com
LastEditTime: 2022-02-16 00:41:45
FilePath: /notes/docs/threejs/index.md
Description: 
---

# Three.js
# [Three JS DOC](http://www.hewebgl.com/article/getarticle/50)

## 三大组件

### 场景 scene

> 场景是所有物体的容器，如果要显示一个苹果，就需要将苹果对象加入场景中。

```javascript
var scene = new THREE.Scene();
```

### 相机 camera

> 相机决定了场景中那个角度的景色会显示出来,场景只有一种，但是相机却又很多种。和现实中一样，不同的相机确定了呈相的各个方面。比如有的相机适合人像，有的相机适合风景，专业的摄影师根据实际用途不一样，选择不同的相机。对程序员来说，只要设置不同的相机参数，就能够让相机产生不一样的效果.

```js
var camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000) // 透视相机
```

### 渲染器 renderer

> 渲染器决定了渲染的结果应该画在页面的什么元素上面，并且以怎样的方式来绘制。

```js
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);    // 设置渲染器的大小为窗口的内宽度，也就是内容区的宽度
document.body.appendChild(renderer.domElement);
```

### 4.添加物体到场景中

```js
var geometry = new THREE.CubeGeometry(1,1,1); 
var material = new THREE.MeshBasicMaterial({color: 0x00ff00});
var cube = new THREE.Mesh(geometry, material); 
scene.add(cube);
```

### 5. 渲染
```js
renderer.render(scene, camera);
render( scene, camera, renderTarget, forceClear )

// scene：前面定义的场景
// camera：前面定义的相机
// renderTarget：渲染的目标，默认是渲染到前面定义的render变量中
// forceClear：每次绘制之前都将画布的内容给清除，即使自动清除标志autoClear为false，也会清除。
```

### 6、渲染循环
渲染有两种方式：实时渲染和离线渲染.

> 离线渲染，想想《西游降魔篇》中最后的佛主，他肯定不是真的，是电脑渲染出来的，其画面质量是很高的，它是事先渲染好一帧一帧的图片，然后再把图片拼接成电影的。这就是离线渲染。如果不事先处理好一帧一帧的图片，那么电影播放得会很卡。CPU和GPU根本没有能力在播放的时候渲染出这种高质量的图片。

> 实时渲染：就是需要不停的对画面进行渲染，即使画面中什么也没有改变，也需要重新渲染。下面就是一个渲染循环：

```js
function render() {
    cube.rotation.x += 0.1;
    cube.rotation.y += 0.1;
    renderer.render(scene, camera);
    requestAnimationFrame(render); //其中一个重要的函数是requestAnimationFrame，这个函数就是让浏览器去执行一次参数中的函数，这样通过上面render中调用requestAnimationFrame()函数，requestAnimationFrame()函数又让render()再执行一次，就形成了我们通常所说的游戏循环了。
}
```

```
Parameters是一个定义材质外观的对象，它包含多个属性来定义材质，这些属性是：

Color：线条的颜色，用16进制来表示，默认的颜色是白色。

Linewidth：线条的宽度，默认时候1个单位宽度。

Linecap：线条两端的外观，默认是圆角端点，当线条较粗的时候才看得出效果，如果线条很细，那么你几乎看不出效果了。

Linejoin：两个线条的连接点处的外观，默认是“round”，表示圆角。

VertexColors：定义线条材质是否使用顶点颜色，这是一个boolean值。意思是，线条各部分的颜色会根据顶点的颜色来进行插值。（如果关于插值不是很明白，可以QQ问我，QQ在前言中你一定能够找到，嘿嘿，虽然没有明确写出）。

Fog：定义材质的颜色是否受全局雾效的影响。
```