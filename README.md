[合集 \- vue3代码修炼秘籍(16\)](https://github.com)[1\.答应我，在vue中不要滥用watch好吗？02\-29](https://github.com/heavenYJJ/p/18045325)[2\.一文搞懂 Vue3 defineModel 双向绑定：告别繁琐代码！02\-04](https://github.com/heavenYJJ/p/18006048)[3\.没有虚拟DOM版本的vue（Vue Vapor）01\-26](https://github.com/heavenYJJ/p/17988599)[4\.有了Composition API后，有些场景或许你不需要pinia了01\-23](https://github.com/heavenYJJ/p/17982518)[5\.你不知道的vue3：使用runWithContext实现在非 setup 期间使用inject01\-17](https://github.com/heavenYJJ/p/17970284)[6\.直接在\*.vue文件（SFC）中使用JSX/TSX渲染函数，真香!01\-12](https://github.com/heavenYJJ/p/17960753)[7\.5分钟搞定vue3函数式弹窗01\-10](https://github.com/heavenYJJ/p/17955954)[8\.看不懂来打我，vue3如何将template编译成render函数04\-12](https://github.com/heavenYJJ/p/18129909)[9\.终于搞懂了！原来 Vue 3 的 generate 是这样生成 render 函数的05\-20](https://github.com/heavenYJJ/p/18200376)[10\.涨见识了！脱离vue项目竟然也可以使用响应式API07\-25](https://github.com/heavenYJJ/p/18321516):[wgetCloud机场](https://tabijibiyori.org)[11\.70%的人都答错了的面试题，vue3的ref是如何实现响应式的？07\-29](https://github.com/heavenYJJ/p/18328847)[12\.用了组合式 (Composition) API 后代码变得更乱了，怎么办？08\-02](https://github.com/heavenYJJ/p/18337613)[13\.给我5分钟，保证教会你在vue3中动态加载远程组件08\-07](https://github.com/heavenYJJ/p/18346051)[14\.卧槽，牛逼！vue3的组件竟然还能“暂停”渲染！08\-19](https://github.com/heavenYJJ/p/18366122)[15\.牛逼！Vue3\.5的useTemplateRef让ref操作DOM更加丝滑09\-04](https://github.com/heavenYJJ/p/18395547)16\.这应该是全网最详细的Vue3\.5版本解读09\-05收起
# 前言


`Vue3.5正式版`在这两天发布了，网上已经有了不少关于Vue3\.5版本的解读文章。但是欧阳发现这些文章对3\.5中新增的功能介绍都`不是很全`，所以导致不少同学有个`错觉`，觉得Vue3\.5版本不过如此，选择跳过这个版本等下个大版本再去更新。所以欧阳写了这篇`超级详细`的Vue3\.5版本解读文章，小伙伴们可以看看在3\.5版本中有没有增加一些你期待的功能。


关注公众号：【前端欧阳】，给自己一个进阶vue的机会


# 版本号


这次的版本号是`天元突破红莲螺岩`，这是07年出的一个二次元动漫，欧阳是没看过的。在此之前我一直以为这次的版本号会叫`黑神话：悟空`，可能悟空不够二次元吧。


# 响应式


响应式相关的内容主要分为：重构响应式、响应式props支持解构、新增`onEffectCleanup`函数、新增`base watch`函数、新增`onWatcherCleanup`函数、新增`pause`和`resume`方法。


## 重构响应式


这次响应式的重构是属于Vue内部优化，对于普通开发者来说是无感的。重构后内存占用减少了56%，优化手段主要是通过`版本计数`和`双向链表数据结构`，灵感来源于[Preact signals](https://github.com)。后续欧阳会出一系列关于响应式相关的源码文章，大家可以关注一波欧阳。


## 响应式props支持解构


在3\.5中响应式props支持解构终于正式稳定了，在没有这个功能之前我们想要在js中访问prop必须要这样写：`props.name`，否则`name`将会丢失响应式。


有了响应式props解构后，在js中我们就可以直接解构出`name`来使用，比如下面这样的代码：



```


```

当`defineProps`搭配解构一起使用后，在编译时就可以将`name`处理成`props.name`。编译后简化的代码如下：



```
setup(__props) {
  console.log(__props.name);
  const __returned__ = {};
  return __returned__;
}

```

从上面的代码可以看到`console.log(name)`经过编译后变成了`console.log(__props.name)`，这样处理后`name`当然就不会丢失响应式了。


## 新增onEffectCleanup函数


在组件卸载之前或者下一次`watchEffect`回调执行之前会自动调用`onEffectCleanup`函数，有了这个函数后你就不需要在组件的`beforeUnmount`钩子函数去统一清理一些timer了。比如下面这个场景：



```
import { watchEffect, ref } from "vue";
import { onEffectCleanup } from "@vue/reactivity";

const flag = ref(true);
watchEffect(() => {
  if (flag.value) {
    const timer = setInterval(() => {
      // 做一些事情
      console.log("do something");
    }, 200);
    onEffectCleanup(() => {
      clearInterval(timer);
    });
  }
});

```

上面这个例子在`watchEffect`中会去注册一个循环调用的定时器，如果不使用`onEffectCleanup`，那么我们就需要在`beforeUnmount`钩子函数中去清理定时器。


但是有了`onEffectCleanup`后，将`clearInterval`放在他的回调中就可以了。当组件卸载时会自动执行`onEffectCleanup`传入的回调函数，也就是会执行`clearInterval`清除定时器。


还有一点值得注意的是`onEffectCleanup`函数目前没有在`vue`包中暴露出来，如果你想使用可以像我这样从`@vue/reactivity`包中导入`onEffectCleanup`函数。


## 新增base watch函数


我们之前使用的`watch`函数是和Vue组件以及生命周期一起实现的，他们是深度绑定的，所以`watch`函数代码的位置在vue源码中的`runtime-core`模块中。


但是有的场景中我们只想使用vue的响应式功能，也就是vue源码中的`reactivity`模块，比如小程序`vuemini`。为此我们不得不将`runtime-core`模块也导入到项目中，或者像`vuemini`一样去手写一个watch函数。


在3\.5版本中重构了一个`base watch`函数，这个函数的实现和vue组件没有一毛钱关系，所以他是在`reactivity`模块中。详情可以查看我之前的文章： [Vue3\.5新增的baseWatch让watch函数和Vue组件彻底分手](https://github.com)


还有一点就是这个`base watch`函数对于普通开发者来说没有什么影响，但是对于一些下游项目，比如`vuemini`来说是和受益的。


## 新增onWatcherCleanup函数


和前面的`onEffectCleanup`函数类似，在组件卸载之前或者下一次`watch`回调执行之前会自动调用`onWatcherCleanup`函数，同样有了这个函数后你就不需要在组件的`beforeUnmount`钩子函数去统一清理一些timer了。比如下面这个场景：



```
import { watch, ref, onWatcherCleanup } from "vue";

watch(flag, () => {
  const timer = setInterval(() => {
    // 做一些事情
    console.log("do something");
  }, 200);
  onWatcherCleanup(() => {
    console.log("清理定时器");
    clearInterval(timer);
  });
});

```

和`onEffectCleanup`函数不同的是我们可以从vue中import导入`onWatcherCleanup`函数。


## 新增pause和resume方法


有的场景中我们可能想在“一段时间中暂停一下”，不去执行`watch`或者`watchEffect`中的回调。等业务条件满足后再去恢复执行`watch`或者`watchEffect`中的回调。在这种场景中`pause`和`resume`方法就能派上用场啦。


下面这个是`watchEffect`的例子，代码如下：



```

  <button @click="count++">count++button>
  <button @click="runner2.pause()">暂停button>
  <button @click="runner2.resume()">恢复button>


<script setup lang="ts">
import { watchEffect } from "vue";

const count = ref(0);
const runner = watchEffect(() => {
  if (count.value > 0) {
    console.log(count.value);
  }
});
script>

```

在上面的demo中，点击`count++`按钮后理论上每次都会执行一次`watchEffect`的回调。


但是当我们点击了暂停按钮后就会执行`pause`方法进行暂停，在暂停期间`watchEffect`的回调就不会执行了。


当我们再次点击了恢复按钮后就会执行`resume`方法进行恢复，此时`watchEffect`的回调就会重新执行。


`console.log`的结果如下图：
![console](https://img2024.cnblogs.com/blog/1217259/202409/1217259-20240904215506297-1753311669.png)


从上图中可以看到`count`打印到4后就没接着打印了，因为我们执行了`pause`方法暂停了。当重新执行了`resume`方法恢复后可以看到`count`又重新开始打印了，此时从8开始打印了。


不光`watchEffect`可以执行`pause`和`resume`方法，`watch`一样也可以执行`pause`和`resume`方法。代码如下：



```
const runner = watch(count, () => {
  if (count.value > 0) {
    console.log(count.value);
  }
});

runner.pause()	// 暂停方法
runner.resume()	// 恢复方法

```

# watch的deep选项支持传入数字


在以前`deep`选项的值要么是`false`，要么是`true`，表明是否深度监听一个对象。在3\.5中`deep`选项支持传入数字了，表明监控对象的深度。


比如下面的这个demo：



```
const obj1 = ref({
  a: {
    b: 1,
    c: {
      d: 2,
      e: {
        f: 3,
      },
    },
  },
});

watch(
  obj1,
  () => {
    console.log("监听到obj1变化");
  },
  {
    deep: 3,
  }
);

function changeDeep3Obj() {
  obj1.value.a.c.d = 20;
}

function changeDeep4Obj() {
  obj1.value.a.c.e.f = 30;
}

```

在上面的例子`watch`的`deep`选项值是3，表明监听到对象的第3层。


`changeDeep3Obj`函数中就是修改对象的第3层的`d`属性，所以能够触发`watch`的回调。


而`changeDeep4Obj`函数是修改对象的第4层的`f`属性，所以不能触发`watch`的回调。


# SSR服务端渲染


服务端渲染SSR主要有这几个部分：新增`useId`函数、Lazy Hydration  懒加载水合、`data-allow-mismatch`


## 新增`useId`函数


有时我们需要生成一个随机数塞到DOM元素上，比如下面这个场景：



```

  <label :htmlFor="id">Do you like Vue3.5?label>
  <input type="checkbox" name="vue3.5" :id="id" />


<script setup lang="ts">
const id = Math.random();
script>

```

在这个场景中我们需要生成一个随机数`id`，在普通的客户端渲染中这个代码是没问题的。


但是如果这个代码是在SSR服务端渲染中那么就会报警告了，如下图：
![useId](https://img2024.cnblogs.com/blog/1217259/202409/1217259-20240904215523093-2071470948.png)


上面报错的意思是服务端和客户端生成的`id`不一样，因为服务端和客户端都执行了一次`Math.random()`生成`id`。由于`Math.random()`每次执行的结果都不同，自然服务端和客户端生成的`id`也不同。


`useId`函数的作用就是为了解决这个问题。


当然`useId`也可以用于客户端渲染的一些场景，比如在列表中我们需要一个唯一键，但是服务端又没有给我们，这时我们就可以使用`useId`给列表中的每一项生成一个唯一键。


## Lazy Hydration  懒加载水合


异步组件现在可以通过 defineAsyncComponent() API 的 hydrate 选项来控制何时进行水合。（欧阳觉得这个普通开发者用不上，所以就不细讲了）


## `data-allow-mismatch`


SSR中有的时候确实在服务端和客户端生成的html不一致，比如在DOM上面渲染当前时间，代码如下：



```

  <div>当前时间是：{{ new Date() }}div>


```

这种情况是避免不了会出现前面`useId`例子中的那种警告，此时我们可以使用`data-allow-mismatch`属性来干掉警告，代码如下：



```

  <div data-allow-mismatch>当前时间是：{{ new Date() }}div>


```

# Custom Element 自定义元素改进


这个欧阳也觉得平时大家都用不上，所以就不细讲了。


# Teleport组件新增defer延迟属性


`Teleport`组件的作用是将children中的内容传送到指定的位置去，比如下面的代码：



```
"target">
<Teleport to="#target">被传送的内容Teleport>

```

文案`被传送的内容`最终会渲染在`id="target"`的div元素中。


在之前有个限制，就是不能将放在`Teleport`组件的后面。


这个也很容易理解DOM是从上向下开始渲染的，如果先渲染到`Teleport`组件。然后就会去找id的值为`target`的元素，如果找不到当然就不能成功的将`Teleport`组件的子节点传送到`target`的位置。


在3\.5中为了解决这个问题，在`Teleport`组件上新增了一个`defer`延迟属性。


加了`defer`延迟属性后就能将`target`写在`Teleport`组件的后面，代码如下：



```
<Teleport defer to="#target">被传送的内容Teleport>
<div id="target">div>

```

`defer`延迟属性的实现也很简单，就是等这一轮渲染周期结束后再去渲染`Teleport`组件。所以就算是`target`写在`Teleport`组件的后面，等到渲染`Teleport`组件的时候`target`也已经渲染到页面上了。


# `useTemplateRef`函数


vue3中想要访问DOM和子组件可以使用ref进行模版引用，但是这个ref有一些让人迷惑的地方。


比如定义的ref变量到底是一个响应式数据还是DOM元素？


还有template中ref属性的值明明是一个字符串，比如`ref="inputEl"`，怎么就和script中同名的`inputEl`变量绑到一块了呢？


3\.5中的`useTemplateRef`函数就可以完美的解决了这些问题。


这是3\.5之前使用ref访问input输入框的例子：



```
"text" ref="inputEl" />

const inputEl = ref<HTMLInputElement>();

```

这个写法很不符合编程直觉，不知道有多少同学和欧阳一样最开始用vue3时会给`ref`属性绑定一个响应式变量。比如这样：`:ref="inputEl"`


更加要命的是这样写还不会报错，就是`inputEl`中的值一直是`undefined`。


最后一番排查后才发现`ref`属性应该是绑定的变量名称：`ref="inputEl"`


使用`useTemplateRef`函数后就好多了，代码如下：



```
"text" ref="inputRef" />

const inputEl = useTemplateRef<HTMLInputElement>("inputRef");

```

使用`useTemplateRef`函数后会返回一个ref变量，`useTemplateRef`函数传的参数是字符串`"inputRef"`。


在template中`ref`属性的值也是字符串`"inputRef"`，所以`useTemplateRef`函数的返回值就指向了DOM元素input输入框。这个比3\.5之前的体验要好很多了，详情可以查看我之前的文章： [牛逼！Vue3\.5的useTemplateRef让ref操作DOM更加丝滑](https://github.com)


# 总结


对于开发者来说Vue3\.5版本中还是新增了许多有趣的功能的，比如：`onEffectCleanup`函数、`onWatcherCleanup`函数、`pause`和`resume`方法、`watch`的`deep`选项支持传入数字、`useId`函数、`Teleport`组件新增`defer`延迟属性、`useTemplateRef`函数。


这些功能在一些特殊场景中还是很有用的，欧阳的个人看法还是得将Vue升到3\.5。


关注公众号：【前端欧阳】，给自己一个进阶vue的机会


![](https://img2024.cnblogs.com/blog/1217259/202406/1217259-20240606112202286-1547217900.jpg)


另外欧阳写了一本开源电子书[vue3编译原理揭秘](https://github.com)，看完这本书可以让你对vue编译的认知有质的提升。这本书初、中级前端能看懂，完全免费，只求一个star。


