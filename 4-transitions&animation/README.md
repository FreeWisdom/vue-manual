# 第四章 Vue中的动画
## 一、使用 Vue 实现基础的 CSS 过渡与动画效果
1. 数据控制css交互
    - css中书写样式；
    - data中书写样式；
    ```html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>基础的 CSS 过渡与动画效果</title>
            <script src="https://unpkg.com/vue@next"></script>
            <style>
                /* 动画 */
                @keyframes leftToRight {
                    0% {
                        transform: translate(-100px);
                    }
                    50% {
                        transform: translate(-50px);
                    }
                    0% {
                        transform: translate(0px);
                    }
                }
                .animation {
                    animation: leftToRight 3s;
                }

                /* 过度 */
                .transition {
                    transition: 3s background-color ease;
                }
                .red {
                    background-color: red;
                }
                .yellow {
                    background-color: yellow;
                }

                .transition1 {
                    transition: 3s background-color ease;
                }
            </style>
        </head>
        <body>
            <div id="root">
            </div>
            <script>
                const app = Vue.createApp({
                    data() {
                        return {
                            animate: {
                                transition: true,
                                animation: true,
                                red: true,
                                yellow: false
                            },
                            animate1: {
                                transition1: true,
                            },
                            styleObj: {
                                background: "pink"
                            }
                        }
                    },
                    methods: {
                        handleClick() {
                            this.animate.animation = !this.animate.animation;
                            this.animate.red = !this.animate.red;
                            this.animate.yellow = !this.animate.yellow;
                            this.styleObj.background === "pink" ? this.styleObj.background = "blue" : this.styleObj.background = "pink";
                        }
                    },
                    template: `
                        <div>
                            <div :class="animate">hello word!</div>
                            <div :class="animate1" :style="styleObj">hello word!</div>
                            <button @click="handleClick">切换</button>
                        </div>
                    `
                });

                app.mount("#root");
            </script>
        </body>
    </html>
    ```
## 二、使用 transition 标签实现单元素组件的过渡和动画效果（1）
1. 若`<transition>....</transition>`
    - 配合在style中书写：入场动画
        - .v-enter-from
            - 开始状态
        - .v-enter-active
            - 过渡动画
        - .v-enter-to
            - 结束状态
    - 配合在style中书写：出场动画
        - .v-leave-from
            - 开始状态
        - .v-leave-active
            - 过渡动画
        - .v-leave-to
            - 结束状态
2. 若`<transition name="seconde">....</transition>`
    - 配合在style中书写：入场动画
        - .seconde-enter-active
            - 入场动画
        - .seconde-leave-active
            - 出场动画
    ```html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>transition标签的使用</title>
            <script src="https://unpkg.com/vue@next"></script>
            <style>
                /* 过渡 */
                .v-enter-from {
                    opacity: 0;
                }
                .v-enter-active, .v-leave-active {
                    transition: opacity 3s ease-in;
                }
                .v-enter-to {
                    opacity: 1;
                }
                /*可以省略 .v-leave-from {
                    opacity: 1;
                } */
                /*可以合并 .v-leave-active {
                    transition: opacity 3s ease-out;
                } */
                .v-leave-to {
                    opacity: 0;
                }

                /* 动画 */
                @keyframes shake {
                    0% {
                        transform: translateX(100px);
                    }
                    50% {
                        transform: translateX(50px);
                    }
                    100% {
                        transform: translateX(0px);
                    }
                }
                .seconde-enter-active {
                    animation: shake 3s;
                }
                .seconde-leave-active {
                    animation: shake 3s;
                }
            </style>
        </head>
        <body>
            <div id="root">
            </div>
            <script>
                const app = Vue.createApp({
                    data() {
                        return {
                            show: false
                        }
                    },
                    methods: {
                        handleClick() {
                            this.show = !this.show;
                        }
                    },
                    template: `
                        <div>
                            <transition>
                                <div v-if="show">hello word!</div>
                            </transition>
                            <button @click="handleClick">切换</button>
                        </div>
                        <div>
                            <transition name="seconde">
                                <div v-if="show">hello word!</div>
                            </transition>
                            <button @click="handleClick">切换</button>
                        </div>
                    `
                });

                app.mount("#root");
            </script>
        </body>
    </html>
    ```
## 三、使用 transition 标签实现单元素组件的过渡和动画效果（2）
1. 若使用`<transition enter-active-class="enter" leave-active-class="leave" >...</transition>`自定义class名的语法
    - 在style中可使用该属性自动定义的class名
2. 使用 https://animate.style/ 动画库
    - 引入动画库
        ```html
        <head>
            <link
                rel="stylesheet"
                href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css"
            />
        </head>
        ```
    - 自定义class名可以直接调用动画库的class生成动画
        ```html
        <h1 class="animate__animated animate__bounce">An animated element</h1>
        ```
    ```html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>transition标签的使用</title>
            <script src="https://unpkg.com/vue@next"></script>
            <link
                rel="stylesheet"
                href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css"
            />
            <style>
                /* 过渡 */
                .v-enter-from {
                    opacity: 0;
                }
                .enter, .leave {
                    transition: opacity 3s ease-in;
                }
                .v-enter-to {
                    opacity: 1;
                }
                /*可以省略 .v-leave-from {
                    opacity: 1;
                } */
                /*可以合并 .v-leave-active {
                    transition: opacity 3s ease-out;
                } */
                .v-leave-to {
                    opacity: 0;
                }

            </style>
        </head>
        <body>
            <div id="root">
            </div>
            <script>
                const app = Vue.createApp({
                    data() {
                        return {
                            show: false
                        }
                    },
                    methods: {
                        handleClick() {
                            this.show = !this.show;
                        }
                    },
                    template: `
                        <div>
                            <transition
                                enter-active-class="enter"
                                leave-active-class="leave"
                            >
                                <div v-show="show">hello word!</div>
                            </transition>
                            <button @click="handleClick">切换</button>
                        </div>
                        <div>
                            <transition
                                enter-active-class="animate__animated animate__bounce"
                                leave-active-class="animate__animated animate__flipInY"
                            >
                                <div v-show="show">hello word!</div>
                            </transition>
                            <button @click="handleClick">切换</button>
                        </div>
                    `
                });

                app.mount("#root");
            </script>
        </body>
    </html>
    ```
## 四、使用 transition 标签实现单元素组件的过渡和动画效果（3）
1. 过渡 + 动画同时使用时，对过程时间长短的控制方法
    1. `<transition type="animation/transition">...</transition>`:
        - 同时使用动画和过渡时，使用type确定以其中一个的结束时间为准；
    2. `<transition :duration="1000">...</transition>`:
        - 行内可以固定动画或过渡的开始到结束时间，毫秒为单位；
2. `<transition :css="false">...</transition>`:
    - 标签行内去除css控制效果的方法；
3. transition标签中使用钩子控制动画
    - 当只用 JavaScript 过渡的时候，在 enter 和 leave 中必须使用 done 进行回调。否则，它们将被同步调用，过渡会立即完成。
    - 推荐对于仅使用 JavaScript 过渡的元素添加 v-bind:css="false"，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响。
    ```html
    <transition
        v-on:before-enter="beforeEnter" 
        v-on:enter="enter"
        v-on:after-enter="afterEnter"
        v-on:enter-cancelled="enterCancelled"

        v-on:before-leave="beforeLeave"
        v-on:leave="leave"
        v-on:after-leave="afterLeave"
        v-on:leave-cancelled="leaveCancelled"  
    >
        <!-- ... -->
    </transition>
    ```
## 五、组件和元素切换动画的实现
1. appear 设置节点在初始渲染的过渡
    - 无论是 appear attribute 还是 v-on:appear 钩子都会生成初始渲染过渡。
    ```html
    <transition appear>
    <!-- ... -->
    </transition>
    ```
2. 过渡模式
    - 在“on”按钮和“off”按钮的过渡中，两个按钮都被重绘了，一个离开过渡的时候另一个开始进入过渡。这是 <transition> 的默认行为 - 进入和离开同时发生。
    - 同时生效的进入和离开的过渡不能满足所有要求，所以 Vue 提供了过渡模式
        - in-out：新元素先进行过渡，完成之后当前元素过渡离开。
        - out-in：当前元素先进行过渡，完成之后新元素过渡进入。
    ```html
    <transition name="fade" mode="out-in">
        <!-- ... the buttons ... -->
    </transition>
    ```
3. 多个单元素标签之间的切换过渡
    - 当有相同标签名的元素切换时，需要通过 key attribute 设置唯一的值来标记以让 Vue 区分它们，否则 Vue 为了效率只会替换相同标签内部的内容。即使在技术上没有必要，给在 <transition> 组件中的多个元素设置 key 是一个更好的实践。
    ```html
    <transition>
        <button v-if="docState === 'saved'" key="saved">
            Edit
        </button>
        <button v-if="docState === 'edited'" key="edited">
            Save
        </button>
        <button v-if="docState === 'editing'" key="editing">
            Cancel
        </button>
    </transition>
    ```
    可重写为：
    ```html
    <transition>
        <button v-bind:key="docState">
            {{ buttonMessage }}
        </button>
    </transition>
    ```
    ```js
    // ...
    computed: {
        buttonMessage: function () {
            switch (this.docState) {
            case 'saved': return 'Edit'
            case 'edited': return 'Save'
            case 'editing': return 'Cancel'
            }
        }
    }
    ```
4. 多个单组件之间的切换过渡
    - 多个组件的过渡简单很多 - 我们不需要使用 key attribute。相反，我们只需要使用动态组件：
    ```html
    <transition name="component-fade" mode="out-in">
        <component v-bind:is="view"></component>
    </transition>
    ```
    ```js
    new Vue({
        el: '#transition-components-demo',
        data: {
            view: 'v-a'
        },
        components: {
            'v-a': {
            template: '<div>Component A</div>'
            },
            'v-b': {
            template: '<div>Component B</div>'
            }
        }
    })
    ```
    ```css
    .component-fade-enter-active, .component-fade-leave-active {
        transition: opacity .3s ease;
    }
    .component-fade-enter, .component-fade-leave-to
        /* .component-fade-leave-active for below version 2.1.8 */ {
        opacity: 0;
    }
    ```
## 六、列表动画
1. 使用 <transition-group> 组件，列表的进入/离开过渡
    ```html
    <div id="list-demo" class="demo">
        <button v-on:click="add">Add</button>
        <button v-on:click="remove">Remove</button>
        <transition-group name="list" tag="p">
            <span v-for="item in items" v-bind:key="item" class="list-item">
            {{ item }}
            </span>
        </transition-group>
    </div>
    ```
    ```js
    new Vue({
        el: '#list-demo',
        data: {
            items: [1,2,3,4,5,6,7,8,9],
            nextNum: 10
        },
        methods: {
            randomIndex: function () {
            return Math.floor(Math.random() * this.items.length)
            },
            add: function () {
            this.items.splice(this.randomIndex(), 0, this.nextNum++)
            },
            remove: function () {
            this.items.splice(this.randomIndex(), 1)
            },
        }
    })
    ```
    ```css
    .list-item {
        display: inline-block;
        margin-right: 10px;
    }
    .list-enter-active, .list-leave-active {
        transition: all 1s;
    }
    .list-enter, .list-leave-to
    /* .list-leave-active for below version 2.1.8 */ {
        opacity: 0;
        transform: translateY(30px);
    }
    ```
2、使用 <transition-group> 组件，列表的排序过渡
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>
<div id="list-complete-demo" class="demo">
    <button v-on:click="shuffle">Shuffle</button>
    <button v-on:click="add">Add</button>
    <button v-on:click="remove">Remove</button>
    <transition-group name="list-complete" tag="p">
        <span
        v-for="item in items"
        v-bind:key="item"
        class="list-complete-item"
        >
        {{ item }}
        </span>
    </transition-group>
</div>
```

```js
new Vue({
    el: '#list-complete-demo',
    data: {
        items: [1,2,3,4,5,6,7,8,9],
        nextNum: 10
    },
    methods: {
        randomIndex: function () {
            return Math.floor(Math.random() * this.items.length)
        },
        add: function () {
            this.items.splice(this.randomIndex(), 0, this.nextNum++)
        },
        remove: function () {
            this.items.splice(this.randomIndex(), 1)
        },
        shuffle: function () {
            this.items = _.shuffle(this.items)
        }
    }
})
```

```css
.list-complete-item {
    transition: all 1s;
    display: inline-block;
    margin-right: 10px;
}
.list-complete-enter, .list-complete-leave-to
/* .list-complete-leave-active for below version 2.1.8 */ {
    opacity: 0;
    transform: translateY(30px);
}
.list-complete-leave-active {
    position: absolute;
}
```

## 七、状态动画
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>状态动画</title>
        <script src="https://unpkg.com/vue@next"></script>
    </head>
    <body>
        <div id="root">
        </div>
        <script>
            const app = Vue.createApp({
                data() {
                    return {
                        number: 1,
                        animateNumber: 0
                    }
                },
                methods: {
                    handleClick() {
                        this.number = 10;
                        if(this.animateNumber < 10) {
                            let animation = setInterval(() => {
                                this.animateNumber += 1;
                                if(this.animateNumber === 10) {
                                    clearInterval(animation);
                                }
                            }, 1000)
                        }   
                    }
                },
                template: `
                    <div>
                        <div>{{animateNumber}}</div>
                        <button @click="handleClick">增加</button>
                    </div>
                `
            });

            app.mount("#root");
        </script>
    </body>
</html>
```