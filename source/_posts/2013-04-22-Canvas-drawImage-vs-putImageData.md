---
title: 使用drawImage和putImageData缓存Canvas中间数据
---

## API
```js
void drawImage(Object image, float dx, float dy[float dw, float, dh]);
void drawImage(Object image, float sx, float sy, float sw, float sh, float dx, float dy, float dw, float dh);

ImageData getImageData(float sx, float sy, float sw, float sh);
void putImageData(ImageData imageData, float dx, float dy[, float dirtyX, float dirtyY, float dirtyWidth, float dirtyHeight]);
```
drawImage的第一个参数可以是HTMLImageElement（绘图），HTMLCanvasElement（复制）或者HTMLVideoElement（截屏）。  
## 缓存Canvas中间数据
如果我们需要在一个Canvas上画很多图片，其中一部分是固定（或短期内固定）存在的，那么最好的办法是将他们绘制完毕之后，将他们保存起来，等下一次需要绘制这些图片的时候，一次性绘制缓存结果。这样可以减少渲染次数，以提高渲染效率。  
首先想到的是getImageData和putImageData。所以做了如下测试：
```js
var repeat_times = 10000;

console.time('drawImage');
for(var i = 0; i < repeat_times; i++){
  context.drawImage(image, 0, 0);
}
console.timeEnd('drawImage');

console.time('putImageData');
context.drawImage(image, 0, 0);
var data = context.getImageData(0, 0, image.width, image.height);
for(var i = 0; i < repeat_times; i++){
  context.putImageData(data, 0, 0);
}
console.timeEnd('putImageData');
```
简单起见，这里我们仅仅绘制一张图片。打印出来的结果完全出乎意料，putImageData非但没有更快，反而比drawImage慢600倍左右。至于为什么会这么慢，我还不知道答案。总之putImageData远远不是我想象中的那么高效。  
后来才考虑到可以将Canvas中的数据缓存到某个隐藏Canvas中去，然后再调用DrawImage方法。
```js
console.time('drawCanvas');
var buffer = document.createElement('canvas');
buffer.width = canvas.width;
buffer.height = canvas.height;
context.drawImage(image, 0, 0);
for(var i = 0; i < repeat_times; i++){
  context.drawImage(buffer, 0, 0);
}
console.timeEnd('drawCanvas');
```
经过测试以后发现比drawImage方法慢了一倍，相对于10000次循环，这点性能损失可以忽略不计，因为我们仅仅绘制一张图片。图片的数目每增加1，drawImage绘图次数增加10000，而drawCanvas绘图次数仅仅增加1。  

## 小结
`putImageData`的优点在于能够像素级别操作图像，但是因其效率很低，并不适合做Canvas的数据缓存，而`drawImage(canvas, 0, 0)`较为适合。