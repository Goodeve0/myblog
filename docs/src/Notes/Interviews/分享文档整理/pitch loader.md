# pitch loader

## 分享内容

面试的时候，如果你简历写了熟悉 or 了解工程化，那面试官十有八九就会问这样几个问题：你知道哪些 plugin？平时用过哪些 loader？会自定义一个 plugin/loader 嘛，思路是什么？如果问你这些问题，你会怎么答呢~

那我们也可以从面试官的问题中知道，学好 webpack，了解 plugin 和 loader 少不了。

今天主要先讲讲 loader，我对 loader 的理解就是把一个资源转换成另一个资源，先来段代码体验一下

webpack 的 loader 一般会有两个阶段：pitching 阶段和 normal 阶段。我们刚刚体验的就是 normal 阶段，

normal loader 呢就是 loader 函数本身

```jsx
function loader(content) {
  return content;
}
```

pitch loader 就是 normal loader 上的 pitch 属性函数

```jsx
module.exports = function (content) {
  return content;
};
module.exports.pitch = function (remainingRequest, precedingRequest, data) {
  console.log("hello");
};
```

它特殊的点在于我们知道 loader 是按从后到前执行，而 pitch 是从前到后执行。

给大家上一段源码解读，深入了解一下它们执行顺序的原因

loader 的加载是 webpack 在 runloader 的时候执行的，调用 loader-runner 这个库，它在执行 loader 的时候将所有的 loader 加了一个 index，这个 index 在判断当前 loader 是 normalloader 的时候会递减，所以执行顺序是倒序

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/43953d5c-dc05-4e95-bd86-57bd3e17e7b9/ddc1460c-d5ef-4983-917c-38d9ff94e698/image.png)

而当判断 loader 为 pitchLoader 的时候 index 递增，所以是正序

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/43953d5c-dc05-4e95-bd86-57bd3e17e7b9/d2fad91e-a1f0-44eb-b2d9-0c63724c782a/image.png)

也就是下图所示

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/43953d5c-dc05-4e95-bd86-57bd3e17e7b9/09a4b644-7a5f-4af3-859d-c36267cc0f4e/image.png)

需要注意的是：

pitch 会有熔断效果，这是什么意思呢，就是在这个过程中如果任何 pitch 有返回值（非 undefined），则 loader 链被阻断。webpack 会跳过后面所有的的 pitch 和 loader，直接进入上一个 loader 。

!https://cdn.nlark.com/yuque/0/2024/png/42817320/1730551371479-b92ed740-7552-4336-aa40-75be1a308d8d.png

话题说回来，pitch 方法有三个参数：

- remainingRequest：loader 链中排在自己后面的 loader 以及资源文件的绝对路径以!作为连接符组成的字符串。
- precedingRequest：loader 链中排在自己前面的 loader 的绝对路径以!作为连接符组成的字符串。
- data：每个 loader 中存放在上下文中的固定字段，可用于 pitch 给 loader 传递数据。

接下来我们来使用一下 pitch 制作一个简单的 loader

```jsx
const styleLoader = () => {};

styleLoader.pitch = function (remainingRequest) {
  const relativeRequest = remainingRequest
    .split("!")
    .map((part) => {
      // 将路径转化为相对路径
      const relativePath = this.utils.contextify(this.context, part);
      return relativePath;
    })
    .join("!");

  const script = `
    import style from "!!${relativeRequest}"
    const styleEl = document.createElement('style')
    styleEl.innerHTML = style
    document.head.appendChild(styleEl)
  `;

  return script;
};

module.exports = styleLoader;
```

## 提问环节

Q: 如果 pitch loader 封装的是异步函数，它和 normal loader 的打印顺序会是什么样的？
A:

Q:
A: 不是哦，他的意思应该是，而你说的是 loader 的另一种类型 async loader。
