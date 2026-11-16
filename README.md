# WebDesign8-Rider

<img src="./cover.png" alt="Pink Rider interactive hero cover" width="100%">

一个单文件的 Pink Rider 互动 Hero 页面：以浅粉色视觉系统、反向滚动 Marquee 和 Canvas 撕裂 Reveal 交互，把你的半身肖像切换成粉色骑手造型。

## 在线内容

- 入口文件：[\`index.html\`](./index.html)
- 页面资源：[\`pink-rider.png\`](./pink-rider.png)
- 仓库：<https://github.com/ThreshoLdnnis/WebDesign8-Rider>

## 设计亮点

### 1. Baby-pink editorial 视觉

页面使用浅粉背景、柔和径向光晕和高对比黑色字体，形成偏时装杂志的视觉基调。中间两行 Marquee 以相反方向运动：

- 浅灰：DREAM RIDE SPARKLE
- 黑色：RIDE PRETTY STAY BOLDER

人物位于文字前方，文字在人物背后穿过，形成有纵深的舞台感。

### 2. 透明人物分层

页面使用两张同构图的透明 PNG：

- front：你的原始半身抠图
- back：粉色骑手图层

两张图都通过同一个 \`drawCover\` 方法底部对齐和居中绘制，所以 Reveal 发生时，头部、肩线和身体位置可以保持一致。

### 3. 单文件交付

页面不依赖框架、构建工具或本地图片目录。HTML、CSS、JavaScript 和人物图像都集中在 \`index.html\` 中，下载后可以直接打开。

## 鼠标撕裂 Reveal 是怎么实现的

这个效果不是简单地把两张图片做淡入淡出，而是实时生成一张动态遮罩，再用 Canvas 合成两层人物。

### 第一步：记录鼠标轨迹

鼠标移动时，脚本会把浏览器坐标转换成人物区域内部坐标。如果鼠标移动很快，会在前后两个点之间插入多个中间点，保证笔刷轨迹连续：

~~~js
const step = Math.max(W, 260) * 0.04;
const count = Math.min(40, Math.ceil(distance / step));
~~~

### 第二步：生成有机笔刷印记

每个印记由 3～5 个随机重叠圆组成。每个圆拥有随机角度、偏移距离和半径，因此边缘不会是规则圆形，而是类似撕纸或干刷留下的破碎边缘。

每个印记都会记录创建时间：

~~~js
stamps.push({ x, y, t: now, lobes });
~~~

### 第三步：生成 mask 画布

渲染循环会把所有还没过期的印记画到隐藏的 \`mask\` 画布上。印记的生命周期约为 1.1 秒，并会在接近过期时缩小：

~~~js
const k = 1 - age / LIFE;
const scale = k < 0.75 ? k / 0.75 : 1;
~~~

### 第四步：擦除前景人物

主画布先绘制你的 front 图层，然后使用 \`destination-out\`：

~~~js
ctx.globalCompositeOperation = "destination-out";
ctx.drawImage(mask, 0, 0, W, H);
~~~

\`destination-out\` 会把 mask 覆盖到的区域变成透明，因此你的原始人物只会在笔刷经过的地方被擦除。

### 第五步：显示骑手图层

骑手图层先在离屏缓冲画布 \`buf\` 中绘制。通过 \`source-in\`，只保留 mask 区域内的骑手图像：

~~~js
bctx.drawImage(mask, 0, 0, W, H);
bctx.globalCompositeOperation = "source-in";
drawCover(bctx, back);
~~~

最后把 \`buf\` 合成回主画布。结果就是：

- mask 外：显示你的原始肖像
- mask 内：显示粉色骑手
- mask 边缘：保留随机重叠圆形成的撕裂形状

### 第六步：自动清理与移动端支持

渲染循环会删除超过 \`LIFE = 1100ms\` 的印记，所以效果会自然消退。移动端通过 \`matchMedia("(hover: none), (pointer: coarse)")\` 判断触摸设备；用户停留一段时间没有操作时，笔刷会自动缓慢游走。

## 运行方式

直接双击 \`index.html\` 即可打开。也可以在本地启动任意静态服务器：

~~~bash
npx serve .
~~~

## 技术栈

- HTML
- Inline CSS
- Vanilla JavaScript
- Canvas 2D Compositing
- Google Fonts / Inter Tight

## License

暂未指定 License。
