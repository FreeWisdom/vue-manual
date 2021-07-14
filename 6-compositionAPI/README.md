# 第6章 Composition API
> 产生背景：
> 当我们的组件变得更大时，业务逻辑关注点的代码也会变长，假如想找到某个在多处代码中有频繁操作的属性进行新需求的具体操作，会在100甚至更多行代码中跳来跳去，找到属于该属性的逻辑，尤其是刚接手项目的新人来说，这会导致组件难以阅读和理解。CompositionAPI中能够配置与相同逻辑问题相关的代码，这正是我们需要的。
## 6-1 Setup 函数的使用
1. Composition API 所有代码编写之前，一定建立在`setup()`函数之上;
2. `setup(props, context){}`在**created 实例被完全初始化之前执行**；
    - 由于`setup`是在实例被初始化之前执行的，故在`setup`中的`this`上没有任何东西;
        - 故 **`setup`中不能使用`this`**；
        - 故 **`setup`中不能调用外部方法(template/mounted/methods/……)**；
        - 而 **在`setup`外部app实例的 方法/生命周期函数 中可以通过`this.$options.setup()`调用`setup`中`return{}`的所有内容**
3. `setup(){}`中必须`return{}`，其 return 的内容被暴露在外部，故可以在 template 中直接使用其 return 出来的变量、函数……;
```js
const app = Vue.createApp({
    methods: {
        test() {
            console.log(this.$options.setup());
        }
    },
    template: `
        <div @click="handleClick">{{name}}</div>
    `,
    mounted() {
        this.test();
    },
    setup(props, context) {
        return {
            name: "zhz",
            handleClick: () => {
                alert("setup");
            }
        }
    }
});

const vm = app.mount("#root");
```
## 6-2 ref, reactive, readonly, toRefs 响应式引用的用法（对标usestate）
1. 响应式引用的原理：**通过proxy对数据进行封装，当数据变化时，触发模板等内容的更新；**
    * **为什么要用对象包装？这是因为：**
      *  **JavaScript 中，像 Number 或 String 这样的基本类型是按值传递的，而不是按引用传递的，有一个包装对象围绕任何值，使我们能够安全地在整个应用程序中传递它，而不用担心在传递过程中某个地方会丢失反应。**
2. `ref()`处理基础类型的数据，如 字符串、数字；
    * `ref("xxx")`之后，将 `"zhz"` 变成 `proxy({value: "dell"})` 这样的响应式引用；
    * **`template: '<div>{{name}}</div>'`中，Vue底层会自动调用`name.value`**

3. `reactive()`处理非基础类型数据，如 数组、对象；
    * `reactive({xxx: 22})`之后，将 `{age:22}` 变成 `proxy({age:22})` 这样的响应式引用；
4. `readonly `对以上两种响应式的引用处理为只读，返回的对象为只读；
5. ref、reactive 代替了Vue3之前的 `data(){return {xxx: 33}}` 响应式数据的定义；
6. `toRefs(infoOobject)` === ``toRefs({gender: "男", like: "女"})`：让 reactive 中的对象解构，方便在 template 中调用；
    * 以上代码转换为对象 ===> `{gender: proxy({value: "男"}), like: proxy({value: 女})}；`，达到解构的目的；
```html
<script>
  const app = Vue.createApp({
    template: `
    <!--<div>{{name}}</div> 中，Vue底层会自动调用name.value-->
    <div>{{name}}：{{ageObj.age}} - {{onlyRead.day}} - {{gender}} - {{like}}</div>
    `,
    setup(props, context) {
      const { ref, reactive, readonly, toRefs } = Vue;

      let name = ref("zhz");
      let ageObj = reactive({age: 22});
      let birth = reactive({day: 6666});
      let onlyRead = readonly(birth);
      let infoOobject = reactive({gender: "男", like: "女"});
      setTimeout(() => {
        name.value = "zhe";
        ageObj.age = 99;
        onlyRead.day = "change";
      }, 2000);

      const {gender, like} = toRefs(infoOobject)
      return { name, ageObj, onlyRead, gender, like };
    }
  });

  const vm = app.mount("#root");
</script>
```
## 6-3 toRef （对标usestate）

* 当初始值不存在时使用，即使源属性当前不存在，toRef 也会返回一个可用的 ref(引用)。

```HTML
<script>
  const app = Vue.createApp({
    template: `
    <div>{{name}}---{{age}}</div>
    `,
    setup(props, context) {
      const { reactive, toRef, toRefs } = Vue;

      let ageObj = reactive({age: 22});
      let name = toRef(ageObj, "name");

      setTimeout(() => {
        name.value = "zhe";
        ageObj.age = 99;
      }, 2000);

      let { age } = toRefs(ageObj);
      return { name, age };
    }
  });

  const vm = app.mount("#root");
</script>
```

## 6-4 context 参数中的 attrs/emit/slots

1. attrs：相当于 non-props属性；
2. emit：相当于 this.$emit("xxx")；
3. slots：相当于 this.$slots
   * slots.default()：是父组件插槽所传递内容的虚拟 DOM，可以 return 成真实 DOM；

```html
<script>
  const app = Vue.createApp({
    methods: {
      handleclick() {
        alert("hi");
      }
    },
    template: `
    <emit @change="handleclick"></emit>
    <attrs name="zhz"></attrs>
    <slots>slots</slots>
    `,
  });

  app.component("emit", {
    template: `<div @click="handleclick">emit</div>`,
    setup(props, context) {
      const { emit } = context; // this.$emit("xxxx")
      function handleclick() {
        emit("change");
      }
      return { handleclick }
    },
  });

  app.component("attrs", {
    template: `<div>attrs:{{name}}</div>`,
    setup(props, context) {
      const { attrs } = context; // non-props 属性
      const name = attrs.name;
      return { name };
    }
  });

  app.component("slots", {
    setup(props, context) {
      const { h } = Vue;
      const { slots } = context; // this.$slots
      console.log("slots:", slots.default()) // 虚拟 DOM
      return () => h("div", {name: "zhz"}, slots.default());
    }
  })

  const vm = app.mount("#root");
</script>
```

## 6-5 Composition API 写todolist组件

```html
<script>
  // list相关的抽离；
  const listReactive = () => {
    const { reactive } = Vue;
    const list = reactive([]);
    const handlesubmite = (value) => {
      console.log(value);
      list.push(value);
    };
    return { list, handlesubmite }
  };

  // inputValue相关的抽离；
  const inputValueReactive = () => {
    const { ref } = Vue;
    const inputValue = ref("");
    const handleInput = (e) => {
      inputValue.value = e.target.value;
    };
    return { inputValue, handleInput };
  };

  // 流程调度，数据中专
  const app = Vue.createApp({
    setup(props, context) {
      const { list, handlesubmite } = listReactive();
      const { inputValue, handleInput } = inputValueReactive();
      return {
        list, handlesubmite,
        inputValue, handleInput
      };
    },
    template: `
    <div>
      <div>
        <input :value="inputValue" @input="handleInput"/>
        <button @click="() => handlesubmite(inputValue)">提交</button>
      </div>
      <ul>
      	<li v-for="(item, index) in list" :key="index">{{item}}</li>
      </ul>
    </div>
    `,
  });

  const vm = app.mount("#root");
</script>
```

## 6-6 CompositionAPI中computed使用

```html
<script>
  // count相关的抽离；
  const countReactive = () => {
    const { ref } = Vue;
    const count = ref(99);
    const handleclick = () => {
      console.log(count.value);
      count.value += 1;
    };
    return { count, handleclick }
  };

  // 计算属性一般操作；
  const countAddFiveReactive = (count) => {
    const { reactive, computed } = Vue;
    // const template = reactive("");
    const countAddFive = computed(() => {
      console.log(count.value)
      return count.value + 5;
    });
    return { countAddFive };
  };

  // 计算属性复杂操作get/set；
  const countAddTenReactive = (count) => {
    const { computed } = Vue;
    const countAddTen = computed({
      get: () => {
        return count.value + 10
      },
      set: (param) => {
        return count.value = param - 1;
      }
    });
    return { countAddTen }
  }

  // 流程调度，数据中专
  const app = Vue.createApp({
    setup(props, context) {
      const { count, handleclick } = countReactive();
      const { countAddFive } = countAddFiveReactive(count);
      let { countAddTen } = countAddTenReactive(count);
      setTimeout(() => {
        countAddTen.value = 888;
      }, 3000)
      return {
        count, handleclick,
        countAddFive,
        countAddTen
      };
    },
    template: `
    <div>
    <div @click="handleclick">{{count}}</div>
    <div>{{countAddFive}}</div>
    <div>{{countAddTen}}</div>
      </div>
    `,
  });

  const vm = app.mount("#root");
</script>
```

## 6-7 CompositionAPI中watch使用

1. watch 具备一定的惰性，第一次进入页面侦听器不会执行，只有在数据变化的时候才执行；

2. 参数可以拿到当前值和原始值；

3. reactive和ref，分别处理的数据，监听的方式不同，一个监听器可以监听多个数据；

4. watch第三个参数，immediate可以控制watch变成非惰性的，与watchEffect保持一致；

   ```html
   <script>
     // nameReactive相关抽离，reactive的watch方法；
     const nameReactive = () => {
       const { reactive, toRefs, watch } = Vue;
       const nameObj = reactive({
         chName: "ZHZ",
         enName: "THALES",
       })
       watch(
         [() => nameObj.chName, () => nameObj.enName],
         ([curChName, curEnName], [preChName, preEnName]) => {
           console.log(curChName, curEnName, preChName, preEnName)
         },
         { immediate: true }
       )
       const { chName, enName } = toRefs(nameObj);
   
       return { chName, enName };
     };
   
     // ageReactive相关抽离，ref的watch方法；
     const ageReactive = () => {
       const { ref, toRefs, watch } = Vue;
       const age = ref(00);
       watch(age, (cur, pre) => {
         console.log(cur, pre)
       })
   
       return { age };
     };
   
     // 流程调度，数据中专
     const app = Vue.createApp({
       setup(props, context) {
         const { chName, enName } = nameReactive();
         const { age } = ageReactive();
         return {
           chName, enName, age
         };
       },
       template: `
       <div>
       <div>姓名：<input v-model="chName" />--<span>{{chName}}</span></div>
       <div>姓名：<input v-model="enName" /></div>
         </div>
       <div>
       <div>年龄：<input v-model="age" />--<span>{{age}}</span></div>
       <div>年龄：<span>{{age}}</span></div>
         </div>
       `,
     });
   
     const vm = app.mount("#root");
   </script>
   ```

## 6-8 CompositionAPI中watchEffect使用

1. watchEffect无惰性，立即执行，页面加载的第一次也会执行；

2. 不需要传递要侦听的内容，自动会感知watchEffect中数据对外界的代码依赖；

3. 不需要传递很多参数，只需要一个回调函数；

4. watchEffect不能获取数据改变之前的原始值，只能获取当前改变后的值；

5. 可用于ajax等异步请求，纯函数中的异步请求，使输出变得不确定性，故而产生副作用，此处effect更形象；

   ```html
   <script>
     const nameReactive = () => {
       const { reactive, toRefs, watchEffect } = Vue;
       const nameObj = reactive({
         chName: "ZHZ",
         enName: "THALES",
       });
       const { chName, enName } = toRefs(nameObj);
       const stopWatchName = watchEffect(() => {
         console.log(chName.value, enName.value)
       });
       // 3秒后监听失效
       setTimeout(() => {
         stopWatchName();
       }, 3000);
       return { chName, enName };
     };
   
     // 流程调度，数据中专
     const app = Vue.createApp({
       setup(props, context) {
         const { chName, enName } = nameReactive();
         return {
           chName, enName
         };
       },
       template: `
       <div>
       <div>姓名：<input v-model="chName" />--<span>{{chName}}</span></div>
       <div>姓名：<input v-model="enName" /></div>
         </div>
       `,
     });
   
     const vm = app.mount("#root");
   </script>
   ```

## 6-9 CompositionAPI中生命周期新写法

```html
<script>
    const app = Vue.createApp({
        // 由于setup 是在 beforeCreate到created 之间执行的，故无 beforeCreate/created
        // beforeMount => onBeforeMount
        // mounted => onMounted
        // beforeUpdate => onBeforeUpdate
        // beforeUnmount => onBeforeUnmount
        // unmouted => onUnmounted
        setup() {
            const {
                ref, onBeforeMount, onMounted, onBeforeUpdate, onUpdated,
                onRenderTracked, onRenderTriggered
            } = Vue;
            const name = ref('dell')
            onBeforeMount(() => {
                console.log('onBeforeMount')
            })
            onMounted(() => {
                console.log('onMounted')
            })
            onBeforeUpdate(() => {
                console.log('onBeforeUpdate')
            })
            onUpdated(() => {
                console.log('onUpdated')
            })
            // 每次渲染后重新收集响应式依赖
            onRenderTracked(() => {
                console.log('onRenderTracked')
            })
            // 每次触发页面重新渲染时自动执行
            onRenderTriggered(() => {
                console.log('onRenderTriggered')
            })
            const handleClick = () => {
                name.value = 'lee'
            }
            return { name, handleClick }
        },
        template: `
      <div @click="handleClick">
        {{name}}
      </div>
    `,
    });

    const vm = app.mount('#root');
</script>
```

## 6-10 CompositionAPI中provide+inject新写法

```html
<script>
    // provide, inject
    const app = Vue.createApp({
        setup() {
            const { provide, ref, readonly } = Vue;
            const name = ref('dell');
            provide('name', readonly(name));
            provide('changeName', (value) => {
                name.value = value;
            });
            return {}
        },
        template: `
      <div>
        <child />
      </div>
    `,
    });

    app.component('child', {
        setup() {
            const { inject } = Vue;
            const name = inject('name');
            const changeName = inject('changeName');
            const handleClick = () => {
                changeName('lee');
            }
            return { name, handleClick }
        },
        template: '<div @click="handleClick">{{name}}</div>'
    })

    const vm = app.mount('#root');
</script>
```

## 6-11 CompositionAPI中ref获取dom新写法

```html
<script>
    // CompositionAPI 的语法下，获取真实的 DOM 元素节点
    const app = Vue.createApp({
        setup() {
            const { ref, onMounted } = Vue;
            const hello = ref(null);
            onMounted(() => {
                console.log(hello.value);
            })
            return { hello }
        },
        template: `
      <div>
        <div ref="hello">hello world</div>
      </div>
    `,
    });

    const vm = app.mount('#root');
</script>
```

