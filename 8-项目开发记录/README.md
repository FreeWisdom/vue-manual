# 项目记录

## 1、css 基础样式集

​		浏览器支持和理解的CSS规范不同，导致渲染页面时效果不一致，会出现很多兼容性问题。有两种解决方案。

### 1.1、一种是 css reset 

```css
/* KISSY CSS Reset
理念：清除和重置是紧密不可分的
特色：1.适应中文 2.基于最新主流浏览器
维护：玉伯(lifesinger@gmail.com), 正淳(ragecarrier@gmail.com)
*/

/* 清除内外边距 */
body, h1, h2, h3, h4, h5, h6, hr, p, blockquote, /* structural elements 结构元素 */
dl, dt, dd, ul, ol, li, /* list elements 列表元素 */
pre, /* text formatting elements 文本格式元素 */
fieldset, lengend, button, input, textarea, /* form elements 表单元素 */
th, td { /* table elements 表格元素 */
    margin: 0;
    padding: 0;
}

/* 设置默认字体 */
body,
button, input, select, textarea { /* for ie */
    /*font: 12px/1 Tahoma, Helvetica, Arial, "宋体", sans-serif;*/
    font: 12px/1 Tahoma, Helvetica, Arial, "\5b8b\4f53", sans-serif; 
    /* 用 ascii 字符表示，使得在任何编码下都无问题 */
}

h1 { font-size: 18px; /* 18px / 12px = 1.5 */ }
h2 { font-size: 16px; }
h3 { font-size: 14px; }
h4, h5, h6 { font-size: 100%; }

address, cite, dfn, em, var { font-style: normal; } /* 将斜体扶正 */
code, kbd, pre, samp, tt { font-family: "Courier New", Courier, monospace; } /* 统一等宽字体 */
small { font-size: 12px; } /* 小于 12px 的中文很难阅读，让 small 正常化 */

/* 重置列表元素 */
ul, ol { list-style: none; }

/* 重置文本格式元素 */
a { text-decoration: none; }
a:hover { text-decoration: underline; }

abbr[title], acronym[title] { /* 注：1.ie6 不支持 abbr; 2.这里用了属性选择符，ie6 下无效果 */
 border-bottom: 1px dotted;
 cursor: help;
}

q:before, q:after { content: ''; }

/* 重置表单元素 */
legend { color: #000; } /* for ie6 */
fieldset, img { border: none; } /* img 搭车：让链接里的 img 无边框 */
/* 注：optgroup 无法扶正 */
button, input, select, textarea {
    font-size: 100%; /* 使得表单元素在 ie 下能继承字体大小 */
}

/* 重置表格元素 */
table {
 border-collapse: collapse;
 border-spacing: 0;
}

/* 重置 hr */
hr {
    border: none;
    height: 1px;
}
/* 让非ie浏览器默认也显示垂直滚动条，防止因滚动条引起的闪烁 */
html { overflow-y: scroll; }
```

### 1.2、一种是 [Normalize.css](http://nicolasgallagher.com/about-normalize-css/)

1. 对于普通的 **H5** 项目，我们可以到其官网下载最新的 **Normalize.css**，然后在页面中引入使用。
   * **官网地址**：https://necolas.github.io/normalize.css/

2. 对于 **Vue.js** 项目，可以先进入项目文件夹中执行如下命令安装：

```shell
npm install --save normalize.css
```

3. 然后在 **vue** 的主文件中引入即可：

```sh
import 'normalize.css/normalize.css'
```

## 2、rem和em总结

1. rem 单位翻译为像素值的时候是由 html 元素的字体大小决定的。此字体大小会被浏览器中字体大小的设置影响，除非显式的在 html 为 font-size 重写一个单位；

2. em 单位转换为像素值的时候，取决于使用它们的元素的 font-size 的大小，但是有因为继承关系，所以比较复杂；
   * **优缺点** 
     * em 可以让我们的页面更灵活，更健壮，比起到处写死的 px 值，em 似乎更有张力，改动父元素的字体大小，子元素会等比例变化，这一变化似乎预示了无限可能；
     * em 做弹性布局的缺点还在于牵一发而动全身，一旦某个节点的字体大小发生变化，那么其后代元素都得重新计算。

## 3、css BEM(`block__element--modifier`)命名规则

* BEM的意思就是块（block）、元素（element）、修饰符（modifier）

* ```
  .块__元素--修饰符{}
  ```

  - `block` 代表了更高级别的抽象或组件
  - `block__element` 代表 block 的后代，用于形成一个完整的 block 的整体
  - `block--modifier`代表 block 的不同状态或不同版本

```html
<div class="docker__item--active"></div>
```

## 4、如何设置fontasize值为10px

1. 横向纵向均缩放50%；
2. center top为缩放中心，最终缩放位置为正确位置；

```css
&__title {
    font-size: .2rem;
    transform: scale(.5, .5);
    transform-origin: center top;
}
```

## 5、♨️页面加载抖动问题

1. 在web开发中，经常会遇到这样一个问题，比如**一个宽度百分百，高度自适应的图片，在网速慢的情况下加载过程中会出现抖动的问题**（未加载图片前容器的高度为0，图片加载完成后下面的内容会被挤下去）；

2. 这种问题如果是图片有固定高度，就不会出现加载抖动。但一般情况下，为了使图片不被拉伸，高度一般设为自适应，那么为了防止加载抖动，我们需要给图片提前占个位，这里***使用css的padding-bottom百分比进行占位***；

> 示例代码如下

```css
.banner {
  height: 0;
  overflow: hidden;
  padding-bottom: 25.4%;
  &__img {
    width: 100%;
  }
}
```

3. padding-bottom实际上是提前占位了，这个***容器的高度始终是0***，高度为0还之所以能够显示内容是因为内容溢出在了padding-bottom上，这里的25.4%是图片的**高/宽**比例，切记是相对于**父元素宽度**的25.4%（此处即.banner的上一级），不是相对于自己的width。

## 6、vue/style的scoped属性(坑)

* scoped原理：https://blog.csdn.net/qq_39043923/article/details/88687046

  ```scss
  <style lang="scss" scoped>
  </style>
  ```

  1. `scoped`属性达到了css作用域约束的目的；
  2. 给HTML的DOM节点加一个不重复data属性(形如：data-v-2311c06a)来表示他的唯一性；

  3. 在每句css选择器的末尾（编译后的生成的css语句）加一个当前组件的data属性选择器（如[data-v-2311c06a]）来私有化样式；

  4. 如果组件内部包含有其他组件，只会给其他组件的最外层标签加上当前组件的data属性

## 7、 beforeEach/beforeEnter实现路由校验



## 8、`?.`optional chaining语法

* 详见：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Optional_chaining

## 9、axios普通封装/创建一次实例封装

### 9.1、普通封装

```js
import axios from 'axios'

const instance = axios.create({
  baseURL: 'https://www.fastmock.site/mock/ae8e9031947a302fed5f92425995aa19/jd',
  timeout: 10000
})

export const get = (url, params = {}) => {
  return new Promise((resolve, reject) => {
    instance.get(url, { params }).then((response) => {
      resolve(response.data)
    }, err => {
      reject(err)
    })
  })
}

export const post = (url, data = {}) => {
  return new Promise((resolve, reject) => {
    instance.post(url, data, {
      headers: {
        'Content-Type': 'application/json'
      }
    }).then((response) => {
      resolve(response.data)
    }, err => {
      reject(err)
    })
  })
}

export const patch = (url, data = {}) => {
  return new Promise((resolve, reject) => {
    instance.patch(url, data, {
      headers: {
        'Content-Type': 'application/json'
      }
    }).then(response => {
        resolve(response.data)
      }).catch(err => {
      reject(err)
    })
  })
}
```

* 使用

```js
const useLoginEffect = (showToast) => {
  const router = useRouter()
  const data = reactive({ username: '', password: '' })

  const handleLogin = async () => {
    try {
      const result = await post('/api/user/login', {
        username: data.username,
        password: data.password
      })
      if (result?.errno === 0) {
        localStorage.isLogin = true
        router.push({ name: 'Home' })
      } else {
        showToast('登陆失败')
      }
    } catch (e) {
      showToast('请求失败')
    }
  }

  const { username, password } = toRefs(data)
  return { username, password, handleLogin}
}
```

### 9.2、只创建一次实例封装



## 10、setup函数主调度功能

* setup函数的主调度功能，具体的业务实现都应摘出去封装；

```js
export default {
  name: 'Login',
  components: { Toast },
  // 职责就是告诉你，代码执行的一个流程
  setup () {
    const { show, toastMessage, showToast } = useToastEffect()
    const { username, password, handleLogin } = useLoginEffect(showToast)
    const { handleRegisterClick } = useRegisterEffect()

    return {
      username, password, show, toastMessage,
      handleLogin, handleRegisterClick,
    }
  }
}
```

## 11、♨️图片未加载完成时避免出现裂图展示的优化

* 图片资源加载完成前，图片是裂图，使用 `v-show="图片地址"` 使得加载图片前没有裂图；
* `v-show="item.imgUrl"`:其中 `item.imgUrl` 为图片地址，没有图片时也没有该元素，所以没有裂图出现；

```vue
<ShopInfo :item="item" :hideBorder="true" v-show="item.imgUrl"/>
```

## 12、HTML人民币书写用“&yen”

* 网页上的人民币标识[￥](https://link.jianshu.com/?t=http://en.wikipedia.org/wiki/¥)请统一使用转义字符（`&#165;` 或者 `&yen;`）。

* 直接写中文字符￥无论你用全角还是半角都是不规范的写法；

## 13、watchEffect()在tab切换同时请求数据时的妙用

1. Effect无惰性，立即执行，页面加载的第一次也会执行；

2. 不需要传递要侦听的内容，自动会感知watchEffect中数据对外界的代码依赖；

3. 不需要传递很多参数，只需要一个回调函数；

4. watchEffect不能获取数据改变之前的原始值，只能获取当前改变后的值；

5. 可用于ajax等异步请求，纯函数中的异步请求，使输出变得不确定性，故而产生副作用，此处effect更形象；

```js
// 列表内容相关的逻辑
import { reactive,toRefs, watchEffect } from 'vue'

const useCurrentListEffect = (currentTab, shopId) => {
  const content = reactive({ list: [] })
  const getContentData = async () => {
    const result = await get(`/api/shop/${shopId}/products`, {
      tab: currentTab.value
    })
    if(result?.errno === 0 && result?.data?.length) {
      content.list = result.data;
    }
  }
  watchEffect(() => { getContentData() })
  const { list } = toRefs(content)
  return { list }
}
```

## 14、物理像素/DPR/设备尺寸/ppi/分辨率----解析

1. **设备物理像素：**指的是手机屏幕上横向（或纵向）上共有多少个像素点，以[iphone8](http://detail.zol.com.cn/395/394162/param.shtml)为例，iphone8的分辨率为1334x750像素，1334为纵向的设备物理像素，750为横向的设备物理像素。设备的物理像素是设备决定的，不可改变；

2. **DPR（Device Pixel Ratio）：**设备像素比，指的是设备一逻辑像素点内能显示多少个物理像素，现在市面上的手机绝大部分DPR等于2或3，iphone6/7/8的DPR都为2（1个逻辑像素需要2个物理像素显示），iphone6/7/8 Plus的DPR都为3，这就是前端开发需要使用到2、3倍图的原因。设备的DPR是设备决定的，不可改变；

   ```js
   // js获取设备DPR
   window.devicePixelRatio
   ```

3. **设备尺寸：**设备屏幕对角线之间的距离，用英寸表示，例如iphone6/7/8都为4.7英寸；

4. **[PPI（Pixels Per Inch）](https://baike.baidu.com/item/PPI/7922913?fr=aladdin)：**每英寸范围内所拥有的像素数，PPI越高，每英寸内拥有的像素也越高，显示的内容也更加清晰细致；

5. **屏幕分辨率：**是指纵横向上的像素点数，单位是px。分辨率160×128的意思是水平方向含有像素数为160个，垂直方向像素数128个；

## 15、为何设备的dpr(device pixel ratio)越大，1px线越粗？

* iphone6/7/8的DPR都为2
  * 1px = 4(2*2)物理点
* iphone6/7/8 Plus的DPR都为3
  * 1px = 9(3*3)物理点

1. **故css中的px为相对单位而非绝对单位；**
2. **故dpr(device pixel ratio)越大，1px线越粗；**

## 16、♨️(移动端适配①)dpr不同时，移动端如何实现1px边框？

1. css为最佳实现方案
   * 问题1：如何得知设备的dpr值；
     * 解决：媒体查询，查像素密度；
   * 问题2：会影响原本样式；
     * 解决：伪元素，脱离原本元素；

> 基于 Stylus 的 border1px 的封装：https://gitee.com/shocked/felixlu-course-gp21/blob/master/Vue.js/maoyan/src/assets/stylus/border.styl

```stylus
border1px(width=1px, color=#ccc, style=solid, radius=0)
  position relative
  &::after
    position absolute
    // pointer-events none 解决点击事件无效
    // 详见：https://www.zhangxinxu.com/wordpress/2011/12/css3-pointer-events-none-javascript/
    pointer-events none
    top 0
    left 0
    bottom 0
    right 0
    z-index 999
    content ''
    border-width width
    border-color color
    border-style style
    border-radius radius

    @media (max--moz-device-pixel-ratio: 1.49),(-webkit-max-device-pixel-ratio: 1.49),(max-device-pixel-ratio: 1.49),(max-resolution: 143dpi),(max-resolution: 1.49dppx)
      width 100%
      height 100%
      border-radius radius
      transform scale(1)

    @media (min--moz-device-pixel-ratio: 1.5) and (max--moz-device-pixel-ratio: 2.49),(-webkit-min-device-pixel-ratio: 1.5) and (-webkit-max-device-pixel-ratio: 2.49),(min-device-pixel-ratio: 1.5) and (max-device-pixel-ratio: 2.49),(min-resolution: 144dpi) and (max-resolution: 239dpi),(min-resolution: 1.5dppx) and (max-resolution: 2.49dppx)
      width 200%
      height 200%
      border-radius radius * 2
      transform scale(0.5)

    @media (min--moz-device-pixel-ratio: 2.5), (-webkit-min-device-pixel-ratio: 2.5), (min-device-pixel-ratio: 2.5),(min-resolution: 240dpi), (min-resolution: 2.5dppx)
      width 300%
      height 300%
      border-radius radius * 3
      transform scale(0.333333)

    transform-origin 0 0
```

> border1px 的使用：

```stylus
li:last-child
	width .44rem
	border1px(0 0 0 1px)
	font-size .2rem
	line-height .44rem
	text-align center
	color #cd4c42
```

2. viewport不是所有浏览器都支持

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scaleable=no">
```

## 17、♨️(移动端适配②)移动端 -webkit-line-clamp 封装单行/多行文本溢出

> 基于 Stylus 的 ellipsis 的封装：

**`-webkit-line-clamp`**：**css3新增属性，不是w3c规范，却被chrome和safari接受**

```stylus
ellipsis($width = 100%, $line-clamp = 1)
  width $width
  display -webkit-box
  -webkit-line-clamp $line-clamp
  -webkit-box-orient vertical
  overflow hidden
```

> ellipsis 的使用：

```stylus
@import '~@/assets/stylus/ellipsis.styl'

h1
  // 第二个参数控制行数
	ellipsis(100%, 2)
```

## 18、Vue-cli 二次配置 跨域

* devserver-proxy的文档：https://cli.vuejs.org/zh/config/#devserver-proxy
* vue.config中chainWebpack配置文档：https://cli.vuejs.org/zh/guide/webpack.html#%E9%93%BE%E5%BC%8F%E6%93%8D%E4%BD%9C-%E9%AB%98%E7%BA%A7
* 了解 [webpack-chain 的 API](https://github.com/mozilla-neutrino/webpack-chain#getting-started)：
  * `set(key, value)`：https://github.com/Yatoo2018/webpack-chain/tree/zh-cmn-Hans#chainedmap

> Vue.config.js

```js
const path = require('path')

module.exports = {
  chainWebpack: config => {
    config.resolve.alias
      .set('@', path.join(__dirname, './src'))
  },

  devServer: {
    proxy: {
      '/mmdb': {
        target: 'https://wx.maoyan.com',
        changeOrigin: true
      },
      '/api': {
        target: 'http://localhost:9000',
        changeOrigin: true,
        // pathRewrite: {
        //   '^/api': ''
        // }
      },
      '/ajax': {
        target: 'https://m.maoyan.com',
        changeOrigin: true
      }
    }
  }
}
```

1. `resolve.alias`的作用：

   * 在 vue.js 中路径`./src`（项目下的src文件夹）可以替换为`@`，如下：

     ```js
     import config from '@/utils/config.js'
     ```

     ```stylus
     @import '~@/assets/stylus/border.styl'
     ```

2. `devServer`的作用：
   
   * 在localhost本地拿到线上地址；

## 19、♨️♨️(移动端适配③)移动端适配方案

### 19.1、百分比方案

* 原理：
  * 使用 **百分比%** 定义 **宽度**，**高度** 用**`px`**固定，根据可视区域实时尺寸进行调整，尽可能适应各种分辨率，通常使用`max-width`/`min-width`控制尺寸范围过大或者过小。
* 优
  * 原理简单，不存在兼容性问题；
* 缺
  * 如果屏幕尺度跨度太大，相对设计稿过大或者过小的屏幕不能正常显示，在大屏手机或横竖屏切换场景下可能会导致页面元素被拉伸变形，字体大小无法随屏幕大小发生变化；
  * 设置盒模型的不同属性时，其百分比设置的参考元素不唯一，容易使布局问题变得复杂

### 19.2、rem方案

* 原理：
  * **rem**是相对长度单位，rem方案中的样式设计为相对于**根元素**`font-size`计算值的倍数。
  * 根据 **屏幕宽度** 设置`html`标签的`font-size`，在布局时使用 **rem** 单位布局，达到自适应的目的，是 **弹性布局** 的一种实现方式。
* 过程：
  * **实现过程：** 
    *  首先获取文档根元素和设备`dpr`，设置 **rem**；
    * 在`html`文档加载和解析完成后调整`body`字体大小；
    *  在页面**缩放 / 回退 / 前进**的时候， 获取元素的内部宽度 (不包括垂直滚动条，边框和外边距)，重新调整 **rem** 大小。

### 19.1、**流式布局（百分比布局、rem布局）**

* 实现方式：
  * 宽度百分百 + 高度固定；
* 特点：
  * 手机越大，内容展示越多，适用于list 列表页；
  * 虽然使用的是rem单位，但是html的fantsize是固定的；
    * 所以高度固定的区域，其大小是固定，不随着屏幕的放大缩小而变化。

### 19.2、**等比缩放布局（rem布局）**

#### 19.2.1、方案一：vw(viewpoint width:1vw=视窗宽度的1%)+rem

>  核心原理：将`html`中`fontsize`值由`100px`设为`设计稿参照系vw`
>
> >  1vw =  (window.innerWith * 1%) px = 设计稿参照系px
> >
> > 100px = 100px/设计稿参照系px = 设计稿参照系vw
> >
> > ```css
> > html {
> >       /* fontsize: 100px */
> >       fontsize: 设计稿参照系vw
> > }
> > /*页面其他地方设置，依赖 html.fontsize 为 100px 的 如：0.44rem 值*/
> > ```
> >
> > 为了更好理解，如下举例：
>
> * 若设计稿参照系为Iphone 6/7/8--375*667
>
> >  按照上面的提到的方法，则html中`fontsize: 26.6666667vw` （100px / (375 * 1%) = 26.6666667）
>
> >  若页面中有元素高度为 0.44rem 换算成 px 如下推导：
>
> > > 0.44rem * 26.6666667vw = 11.7333333vm
>
> > > 最终换算为：11.7333333vm * 3.75px = 44px
>
> * 若使用iphone 5/se--320*568打开上述以Iphone 6/7/8为设计稿开发的页面
>
> > > 最终换算为：11.7333333vm * 3.2px = 37.5466666px
>
> > 相当于做了等比缩小。

* 若**设计稿**参照系为：**iphone 5/se--------320*568**

  * 将html.fontsize从`100px`改为`31.25vw`

  > 1vw = (window.innerWith * 1%) px= 3.2px
  >
  > 100px = 100/3.2px = 31.25vw
  >
  > ```css
  > html {
  >   /* fontsize: 100px */
  >   fontsize: 31.25vw
  > }
  > ```

* 若**设计稿**参照系为：**Iphone 6/7/8--------375*667**

  * 将html.fontsize从`100px`改为`26.6666667vw`

  > 1vw = (window.innerWith * 1%) px= 3.75px
  >
  > 100px = 100/3.75px = 26.6666667vw
  >
  > ```css
  > html {
  >   /* fontsize: 100px */
  >   fontsize: 26.6666667vw
  > }
  > ```

* 若**设计稿**参照系为：**Iphone 6/7/8 plus--------414*736**

  * 将html.fontsize从`100px`改为`26.6666667vw`

    > 1vw =  (window.innerWith * 1%) px= 4.14px
    >
    > 100px = 100/4.14px = 24.1545894vw
    >
    > ```css
    > html {
    >   /* fontsize: 100px */
    >   fontsize: 24.1545894vw
    > }
    > ```

* 若**设计稿**参照系为：**Iphone x--------375*812**

  * 宽度都为375，故同 Iphone 6/7/8 处理方式

#### 19.2.2、方案二：js获取屏幕宽度 + rem

> public/index.html添加script
>
> * ⚠️ 并不是完全的等比缩放，

```js
var width = document.documentElement.clientWidth || document.body.clientWidth;
var ratio = width / 375;
var fontSize = 100 * ratio;
document.getElementsByTagName('html')[0].style['font-size'] = fontSize + 'px';
```

#### 19.2.2、方案二：手淘flexible.js

* flexible.js记得停止维护了，其作者也建议利用vh，vw适配。

### 19.3、如何使“首页面等比缩放布局”&“列表页流式布局”

1. 问题分析，即，使不同页面html展示不同fontsize值：

   * 首页面___等比缩放布局

     ```css
     html {
       fontsize: 26.6666667vw
     }
     ```

   * 列表页___流式布局

     ```css
     html {
       fontsize: 100px
     }
     ```

2. 关键问题，如何穿透vue-scoped的约束？

   * 重新创建一个没有 scoped 的 style 标签，即可覆盖当前页面的 html 的 fontsize 值；
   * 从而使当前页面由“等比缩放布局”变成“流式布局”；

   ```html
   <style lang="stylus" scoped>
   	…………
   </style>
   
   <style lang="stylus">
   	html {
       fontsize: 100px
     }
   </style>
   ```

## 20、移动端input光标太大

```scss
input {
		margin-top: .12rem;
    line-height: .22rem;
  	font-size: .16rem;
}
```

## 21、css实现的小图标与开发时不一致

* 尽量使用iconfront或图片

## 22、发布经常出的问题

* 后端发布文件名不同需要在前端重新配置`publicPath:'/xxx'`

