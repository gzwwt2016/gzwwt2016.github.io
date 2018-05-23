---
layout: post
title: "VueJS之-“Vuex篇“（四）"
date: 2017-10-11
tag: VueJS之-“Vuex篇“（四） 
---  
### Vuex
```bash
 Vuex借鉴了Flux和Redux设计思想.

优点:
    Vue组件父子之间通过$on、$emit自定义事件来进行通信
    Vuex可以解决同级组件之间无法传递消息
```
### 执行步骤

```bash
Store: 定义仓库，包含state对象，组件通过getter从store读取数据，通过Getter

核心模块:

    1、state: 定义存储状态

    2、getter: 对数据进行过滤，获取state数据，在计算属性中获取state的值

    3、mutation: 更改 Vuex 的 store 中的state值的唯一方法是提交 mutation

    4、action: 类似于 mutation，不同在于可以包含任意异步操作，Action通过commit()方法来调用mutation中的方法在改变来改变state的值，而不是直接改变state中的值

            分发Action: store.dispatch()方法触发

    5、modules: 如果应用过大，便可以使用 modules 来分割管理，不至于 store 变得非常臃肿


模块示例详解:

一、state: 

    用来存储数据，如果在组件中获取state中的数据，可以通过两种方法来获取: 计算属性computed 和 Getters

    在组件中显示state值可以通过computed来获取:

        <template>
            <div class="box">
                {{getSideData}}
            </div>
        </template>

        方法1: 计算属性中获取

        computed: {
            getSideData () {
                return this.$store.state.sideData;            // 当getSideData的值改变就会显示出来
            }
        }


        方法2: mapState函数获取

        import { mapState } from 'vuex';        // 引用mapState()

        export default {
            name: 'side',
            computed: {
                ...mapState({    // 对象展开运算符
                    getSideData: (state) => {
                        return state.sideData
                    }
                })
            }
        }

        getters.js
            export const sideData = (state) => {
                return state.sideData;
            } 


2、Getter

    获取state中数据的第二种方法

    getters.js中定义获取state的方法: 

        export const sideData = (state) => {
            return state.sideData;
        }

    组件中调用

        1）computed方式

            computed: {
                getSideData(){
                    return this.$store.getters.sideData;        // $store.getters下找到定义的sideDap
                }
            }

        2）通过mapGetters获取

            <template>
                <div>{{ getSideData }}</div>
            </tempate>

            import { mapState, mapGetters } from 'vuex';

            computed: {
                ...mapGetters({
                    getSideData: "sideData"            // 映射到getters.js中定义的sideData, 并将数据返回给getSideData
                })
            }

3、Mutations

    用来更改Vuex中state的数据，它是同步的

    1、commit()触发Action, Action触发mutation

        定义mutations 

            export default {
                changeSideData(state, sideJson){
                    state.sideData.push(sideJson)
                }
            }

        触发
            methods:{
                addSideData(){
                    let side = {
                        id: 1,
                        name: this.name,
                        introduce: this.introduce
                    }
                    this.$store.commit('changeSideData', side);
                }
            }


    2、使用mapMutations() 来定义改变


4、Actions

    Actions提交到 Mutations中，而不能直接改变state的值

    Actions可以包含异步操作，Mutations是同步的

    1）Example

        定义actions

            export const holderSilde = ({commit}, {name, introduce}) => {    
                // {name, introduce} 这里注意一定对象传过来，这里为解析赋值写法，函数不接收第三个参数
                var side = {
                    id: 1,
                    name: name,
                    introduce: introduce
                }
                commit('changeSideData', side)
            }


        调用 分发dispatch

            methods:{
                addSideData(){
                    // 第二个为传过去的参数
                    this.$store.dispatch('holderSilde', {name: this.name, introduce: this.introduce});
                }
            }


5、Modules

    Vuex可以将Store分割到各模块，每个模块都有自己的store、mutations、actions、getters

    const moduleA = {
        state: { ... },
        mutations: { ... },
        actions: { ... },
        getters: { ... }
    }

    const moduleB = {
        state: { ... },
        mutations: { ... },
        actions: { ... }
    }

    const store = new Vuex.Store({
        modules: {
            a: moduleA,
            b: moduleB
        }
    })

    store.state.a // -> moduleA 的状态
    store.state.b // -> moduleB 的状态
```
### Example

```bash
1、App.vue:

    <template>
        <div>
            <Display></Display>
            <Increment></Increment>
        </div>
    </template>

    <script>
        import Display from './components/Display.vue'
        import Increment from './components/Increment.vue'
        import store from './vuex/store'

        export default {
            components: {
                Display,
                Increment
            },
            store           // 在根组件加入store
        }
    </script>

    <style>
    body {
        font-family: Helvetica, sans-serif;
    }
    </style>


2、Display.vue:

    <template>
        <div>
            <h3>Count is {{ counterValue }}</h3>
        </div>
    </template>

    <script>
        import { getCount } from '../vuex/getters'

        export default {
            vuex: {
                getters: {
                    counterValue: getCount
                }
            }
        }
    </script>


3、Increment.vue

    <template>
        <div>
        <button @click="increment">Increment +1</button>
        </div>
    </template>

    <script>
        import { incrementCounter } from '../vuex/action'

        export default {

            vuex: {
                actions: {
                    increment: incrementCounter
                }
            }

        }
    </script>


4、main.js

    import Vue from 'vue' 
    import App from './App.vue'

    new Vue({
        el: 'body',
        components: { 
            App
        }
    })


5、store.js

    import Vue from 'vue'
    import Vuex from 'vuex'

    // 引用vuex
    Vue.use(Vuex);

    // 启动时的初始状态
    const state = {
        // 放置初始化状态
        count: 0
    }

    const mutations = {
        // 放置要变更的函数
        INCREMENT(state, amount){
            state.count = state.count + amount;
        }
    }

    export default new Vuex.Store({
        state,
        mutations
    })


6、action.js

    // action 会收到 store 作为它的第一个参数
    // 既然我们只对事件的分发（dispatch 对象）感兴趣。（state 也可以作为可选项放入）
    // 我们可以利用 ES6 的解构（destructuring）功能来简化对参数的导入
    export const incrementCounter = function ({ dispatch, state }) {
        console.log(dispatch, state)
        dispatch('INCREMENT', 1)
    }


7、getter.js

    // 这个 getter 函数会返回 count 的值
    // 在 ES6 里你可以写成: 
    // export const getCount = state => state.count
    export function getCount(state){
        return state.count;
    }
```

### 参考资料
> https://github.com/dai-siki/vue-image-crop-upload /]( https://github.com/dai-siki/vue-image-crop-upload /)头像上传组件<br>
> https://github.com/opendigg/awesome-github-vue?f=tt&hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io /]( https://github.com/opendigg/awesome-github-vue?f=tt&hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io /)// vue项目汇总

版权归属，汪文涛的博客：[http://gzwwt2016.github.io](http://gzwwt2016.github.io) 谢谢！
