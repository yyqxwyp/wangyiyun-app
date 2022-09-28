vue2 vue3 axios html5 css3 vant组件库 es6 vuex  vue3-marquee组件

网址： http://192.168.1.9:8081/

实现了页面的绘制，vant组件库，和接口的请求绘制页面
用路由实现页面跳转，实现音乐播放和自动播放下一首，歌词显示
登录的权限判断
难点：歌词跟着音乐显示跳动的时候：
切换上下首的时候 使用watch监听，playListIndex是否发生变化来实现切换音乐
个人首页的权限判断是否登录，利用了路由守卫实现
把歌词，歌单都存储到vuex中

创建api存储数据，在组件中方便引入，以便基础域名改变时容易更改

刷新页面，歌单数据丢失的处理：

​    // 防止页面刷新 造成数据丢失 将数据保存到sessionStory中
​      sessionStorage.setItem('itemDetail', JSON.stringify(state))
//判断如果拿不到数据 就拿本地存储的数据
 // 通过props传值 判断如果获取不到就拿sessstorage中的数据
​    if ((props.playlist.creator = '')) {
​      ;(props.playlist.creator = JSON.parse(sessionStorage.getItem().playlist)), creator
​    }

底部组件是一个全局注册的组件



搜索部分 防止数据丢失保存到本地：

 // 渲染

 mounted() {

  this.keyWordList = JSON.parse(localStorage.getItem('keyWordList')) ? JSON.parse(localStorage.getItem('keyWordList')) : []

 },

历史记录去重：

   // 去重  es6 语法 set去重

​    this.keyWordList = [...new Set(this.keyWordList)]


## 网易云APP

## 1. 初始化

### 1.1 创建项目

1. 新建‘网易云案例’文件作为项目根目录，并在项目根目录运行如下命令，配置初始化包

```bash
vue create wang-app
```

2. 以下创建的组件 都需全部引入到HomeView.js中，使用组件化开发，把一个页面拆分成几个组件 再引入到一个公用的组件中

### 2. 创建主组件

### 在commponents中创建 home 文件夹 里面存放主页面的分组件

#### 2.1在home里面创建TopNav.vue组件 存放头部内容

##### 2.1.1 在TopNav.vue中

```vue
<template>
  <div class="topNav">
    <div class="topLeft">
      <svg class="icon" aria-hidden="true">
        <use xlink:href="#icon-category"></use>
      </svg>
    </div>

    <div class="topContent">
      <span @click="$router.push('/InfoUser')">我的</span>
      <span class="active">发现</span>
      <span>云村</span>
      <span>视频</span>
    </div>

    <div class="topRight">
      <svg class="icon" aria-hidden="true" @click="$router.push('/search')">
        <use xlink:href="#icon-viewlarger"></use>
      </svg>
    </div>
  </div>
</template>
```



#### 2.2 在home中创建Swpier.vue

> 轮番图制作

##### 2.2.1组件的引入和配置

1. 运行命令安装vant组件

```bash
# Vue 3 项目，安装最新版 Vant
npm i vant
```
   2.在main.js中引入配置

```js
import { createApp } from 'vue';
// 1. 引入你需要的组件
import { Button } from 'vant';
// 2. 引入组件样式
import 'vant/lib/index.css';

const app = createApp();

// 3. 注册你需要的组件
app.use(Button);
```
3. 更新vant组件的引入，集中管理第三方库的引入

  3.1在src中创建 plugins文件夹中在创建index.js 集中管理vant组件库

```js
//需要用到的组件名字
import { Button, Swipe, SwipeItem, Popup } from 'vant'
// 放入数组中
let plugins = [Swipe, SwipeItem, Button, Popup]
export default function getVant(app) {
  plugins.forEach(item => {
    return app.use(item)
  })
}
```
更新main.js

```js
import getVant from './plugins'
// 2. 引入组件样式
import 'vant/lib/index.css'

const app = createApp(App)
// 调用index。js中的函数
getVant(app)
app.use(store)
app.use(router).mount('#app')
```

##### 2.2.2 在Swpier.vue引入轮播图的组件
见vant组件官网的使用方法

1. 获取轮播图的样式及数据

通过axios获取数据 在组件中先安装在引入该组件

   1.1安装axios

```bash
npm i axios
```

   1.2接口地址

`type`:资源类型,对应以下类型,默认为 0 即PC
0: pc

1: android

2: iphone

3: ipad


**接口地址 :** `/banner`

**调用例子 :** `/banner`, `/banner?type=2`



##### 2.2.3在swiper.vue中使用axios获取数据

```vue
import axios from 'axios'
// reactive也是vue3的用法
import { reactive, onMounted } from 'vue'
export default {
  // setup 是vue3的用法
  setup() {
    // vue3的响应式对象reactive
    const state = reactive({
      images: []
    })
    // 方法 获取数据
    onMounted(async () => {
   axios.get('域名地址'）.then((res)=>{
      state.images = res.data.banners
      console.log(res)
    })
})
    return { state }
  }
}
```


##### 2.2.4更新域名填写方式，为了让域名改变后更方便在代码中更改
1.4.1将基础域名提取出来
在src创建request文件夹  里面新建index.js
创建一个实例对象service
index.js代码

```js
import axios from 'axios'
let service = axios.create({
  baseURL: 'http://localhost:3000/',
  timeout: 3000
})
export default service
```

request----API----home.js

```js
import service from '..'
// 获取首页轮播图的数据
export function getBanner() {
  return service({
    method: 'get',
    url: 'banner?type=2'
  })
}
............
```

更新swiper.vue代码

```vue
import axios from 'axios'
// reactive也是vue3的用法
import { reactive, onMounted } from 'vue'
//引入home.js中的getBanner方法
import { getBanner } from '@/request/API/home'
export default {
  // setup 是vue3的用法
  setup() {
    // vue3的响应式对象reactive
    const state = reactive({
      images: []
    })
    // 方法 获取数据
    onMounted(async () => {
      let res = await getBanner()
      state.images = res.data.banners
      console.log(res)
    })
    return { state }
  }
}
```



#### 2.3创建IconList.vue引入图标组件
比较简单自己写

#### 2.4创建MusicList.vue发现好歌单组件

1.接口地址

说明 : 调用此接口 , 可获取推荐歌单

**可选参数 :** `limit`: 取出数量 , 默认为 30 (不支持 offset)

**接口地址 :** `/personalized`

**调用例子 :** `/personalized?limit=1`



##### 2.4.1更新文件

home.js添加

```js
// 获取发现好歌单
export function getMusicList() {
  return service({
    method: 'get',
    url: '/personalized?limit=10'
  })
}
```

在MusicList.vue中

```vue
import { getMusicList } from '@/request/API/home'
import { reactive, onMounted } from 'vue'
export default {
  // vue2写法
  // data() {
  //   return {
//定义一个数组存放音乐列表
  //     musicList: []
  //   }
  // },
  // methods: {
  //   async getGnedan() {
  //     let res = await getMusicList()
  //     // console.log(res)
  //     this.musicList = res.data.result
  //   },
//处理播放量
  //   changeCount: function (num) {
  //     if (num >= 100000000) {
  //       return (num / 100000000).toFixed(1) + '亿'
  //     } else if (num >= 10000) {
  //       return (num / 10000).toFixed(1) + '万'
  //     }
  //   }
  // },
  // mounted() {
  //   this.getGnedan()
  // }

  // vue3写法
  setup() {
    const state = reactive({
      musicList: []
    })
    function changeCount(num) {
      if (num >= 100000000) {
        return (num / 100000000).toFixed(1) + '亿'
      } else if (num >= 10000) {
        return (num / 10000).toFixed(1) + '万'
      }
    }
    onMounted(async () => {
      let res = await getMusicList()
      console.log(res)
      state.musicList = res.data.result
    })
    return { state, changeCount }
  }
}
</script>
```
数据获取之后利用vant组件轮播图那的，循环渲染到页面上

##### 2.4.2路由跳转

点击歌单
在view中新建itemMusic.vue

在router的index.js添加路由

```js
  //歌单列表的路由
  {
    path: '/itemMusic',
    name: 'ItemMusic',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "itemMusic" */ '../views/itemMusic.vue')
  },
```



在MusicList.vue添加路由跳转功能

```vue
<router-link :to='{path:''/itemMusic'',query:{id:item.id}}'>

</router-link>
```



### 3、歌曲详情页



歌曲详情页的接口地址

说明 : 歌单能看到歌单名字 , 但看不到具体歌单内容 , 调用此接口 , 传入歌单 id, 可 以获取对应歌单内的所有的音乐，但是返回的trackIds是完整的，tracks 则是不完整的，可拿全部 trackIds 请求一次 song/detail 接口获取所有歌曲的详情 (https://github.com/Binaryify/NeteaseCloudMusicApi/issues/452)

必选参数 : id : 歌单 id

可选参数 : s : 歌单最近的 s 个收藏者

接口地址 : /playlist/detail

调用例子 : /playlist/detail?id=24381616



接口地址 在API中新建item.js,存放歌单详情页的接口

```js
import service from '..'
// 获取歌单详情页的
export function getMusicItemList(data) {
  return service({
    method: 'get',
    url: `/playlist/detail?id=${data}`
  })
}
```



#### 3.1详情页面的渲染

>  ItemMusic.vue又分别为两个子组件组成：itemMusicTop和itemMusicList.vue

跳转页面为 ItemMusic.vue中渲染页面
歌曲详情页获取

```vue
<template>
  <!-- 父向子传值 -->
  <itemMusicTop :playlist="state.playlist"></itemMusicTop>
</template>

<script>
import { useRoute } from 'vue-router'
import { onMounted, reactive } from 'vue'
// getMusicItemList歌曲详情页   getItemList歌单列表
import { getMusicItemList} from '@/request/API/item'
import itemMusicTop from '@/components/item/itemMusicTop.vue'

export default {
  setup() {
    const state = reactive({
      playlist: {},
   
    })
    onMounted(async () => {
      // console.log(useRoute())
      let id = useRoute().query.id
      // console.log(id)
      // 获取歌单详情
      let res = await getMusicItemList(id)
      // console.log(res)
      state.playlist = res.data.playlist
    })
    return { state }
  },
  components: {
    itemMusicTop,

  }
}
</script>
```



##### 3.1.1itemMusicTop歌曲详情页

```vue
<script>
export default {
  setup(props) {
    // console.log(props)
    // 通过props传值 判断如果获取不到就拿sessstorage中的数据
    if ((props.playlist.creator = '')) {
      ;(props.playlist.creator = JSON.parse(sessionStorage.getItem().playlist)), creator
    }
  },
  props: ['playlist']
}
</script>
```

> 
>
> 箭头添加事件绑定返回：@click='$router.go(-1)'
>
> 



#### 3.2歌单列表的渲染 itemMusicList.vue歌单列表

歌单列表的接口地址

接口地址：/playlist/track/all
调用例子：、playlist/track/all?id=12e3&limit=10&offset=1

然后步骤雷同上面



### 4、底部组件 

> 在item中建FooterMusic.vue

#### 4.1歌曲因该存储位置

> 在vuex中即是store中的index.js

```js
export default createStore({
  state: {
    playList: [
      // 进入app播放的默认值

      {
        al: {
          id: 90295968,
          name: '普通朋友',
          pic: 109951165037028590,
          picUrl: 'https://p2.music.126.net/o8ityMCEDxSSwhPpDbXrag==/109951165037028587.jpg',
          pic_str: '109951165037028587'
        },
        id: 1452979629,
        name: '普通朋友',
        ar: [{ name: 'phyoabx' }]
      }
    ],
    playList
      Index: 0, //默认下标为0
```

####  4.2 FooterMusic.vue

```vue
// mapState vuex中的辅助函数
import { mapState } from 'vuex'
export default {

  computed: {
    // 通过这个辅助函数进行结构
    ...mapState(['playList', 'playListIndex', 'isbtnShow', 'detailShow'])
  },
}
```



#### 4.3 实现点击按钮切换按钮功能

在store的index.js

```js
//定义一个布尔值判断是否播放
    isbtnShow: true, //暂停按钮的形式
 mutations: {
//定义一个方法进行传值
    updateIsbtnShow: function (state, value) {
      state.isbtnShow = value
    },
```

##### 4.3.1点击播放按钮，播放音乐

添加audio标签

```vue
//暂停按钮
<svg class="icon" aria-hidden="true" v-if="isbtnShow" @click="play">
        <use xlink:href="#icon-bofang"></use>
      </svg>
//播放按钮
      <svg class="icon" aria-hidden="true" v-else @click="play">
        <use xlink:href="#icon-zanting"></use>
      </svg>
//通过ref属性进行传值
    <audio ref="audio" :src="`https://music.163.com/song/media/outer/url?id=${playList[playListIndex].id}.mp3 `"></audio>
。。。。。
<script>
import { mapMutations, mapState } from 'vuex'
export default{
  methods: {
    play: function () {
      // console.log(111)
      if (this.$refs.audio.paused) {
        // 播放
        this.$refs.audio.play()
        this.updateIsbtnShow(false)
      
      } else {
        // 暂停
        this.$refs.audio.pause()
        this.updateIsbtnShow(true)
  
      }
    },
//对vuex中的方法进行结构
    ...mapMutations(['updateIsbtnShow'])
}
</script>

```


##### 4.3.2点击另外一首歌 自动播放
使用watch监听下标是否发生改变
在FooterMusic.vue

```vue
  // watch监听playListIndex是否更改
  watch: {
    playListIndex: function () {
      // 监听下标，发生改变 自动播放点击的那一首歌
      this.$refs.audio.autoplay = true
      if (this.$refs.audio.paused) {
        // 如果是暂停状态 图标改变
        this.updateIsbtnShow(false)
      }
    },
    //监听播放列表
    playList: function () {
      if (this.isbtnShow) {
        this.$refs.audio.autoplay = true
        this.updateIsbtnShow(false)
      }
    }
  }
```



### 5、歌词详情页

为底部组件添加点击事件，是一个全局组件 所以要在vuex中定义一个布尔  detailShow：false
使用vant组件。。。。。弹出层

弹出层由一个子组件向其组成叫：MusicDetail.vue

#### 5.1在FooterMusic.vue中

```vue
<template>
   <!-- 弹出层vant           :isbtnShow="isbtnShow"  按钮是否发生改变 -->
    <van-popup v-model:show="detailShow" position="right" :style="{ height: '100%', width: '100%' }">
      <MusicDetail :musicList="playList[playListIndex]" :play="play" :isbtnShow="isbtnShow" :addDuration="addDuration"></MusicDetail>
    </van-popup>
</template>
```

> 将组件内容写在 MusicDetail.vue中

#### 5.2头部

#### 5.3磁盘部分

MusicDetail.vue

```vue
  <!-- 中间磁盘部分 -->
  <div class="detailCounent" v-show="!isLyricShow">
      
      <!-- //:class="{ img_needle_active: !isbtnShow 播放状态显示该动态-->
    <img src="@/assets/needle-ab.png" alt="" class="img_needle" :class="{ img_needle_active: !isbtnShow }" />
    <img src="@/assets/cd.png" alt="" class="img_cd" />
    <img :src="musicList.al.picUrl" alt="" class="img_ar" :class="{ img_ar_active: !isbtnShow, img_ar_paused: isbtnShow }" @click="isLyricShow = true" />
  </div>

//主要展示的是点击音乐播放按钮，磁盘和磁针会发生的动画
<style>
    .detailCounent {
      // 以下是css3的动画属性
   .img_needle_active {
    width: 2rem;
    height: 3rem;
    position: absolute;
    left: 46%;
    transform-origin: 0 0;
    transform: rotate(0deg);
    transition: all 2s;
  }
  .img_ar_active {
      //控制动画是启动还是暂停
    animation-play-state: running;
  }
  .img_ar_paused {
    animation-play-state: paused;
  }
  @keyframes rotate_ar {
    0% {
      transform: rotateZ(0deg);
    }
    100% {
      transform: rotateZ(360deg);
    }
  }
}
</style>
```



#### 5.4歌词部分



##### 5.4.1接口部分

说明 : 调用此接口 , 传入音乐 id 可获得对应音乐的歌词 ( 不需要登录 )

**必选参数 :** `id`: 音乐 id

**接口地址 :** `/lyric`

**调用例子 :** `/lyric?id=33894312`



##### 5.4.2存储歌词

异步请求

在store

```js
  actions: {
    getLyric: async function (context, value) {
      let res = await getMusicLyric(value)
      // console.log(res)
    
    },
  }
```



点击底部组件的播放按钮就要触发getLyric函数

FooterMusic.vue

```vue
  updated() {
    this.$store.dispatch('getLyric', this.playList[this.playListIndex].id)
    
  },
```

watch监听歌词滚动和切换上下首歌曲

```vue
watch: {
    currentTime: function (newValue) {
      let p = document.querySelector('p.active')
      // console.log([p])
      // 歌词显示部分，监听offsetTop发生变化之后，滚动条也跟着变化
      if (p) {
        if (p.offsetTop > 300) {
          this.$refs.musicLyric.scrollTop = p.offsetTop - 280
        }
      }
      // 判断进度条完了之后 切换下一首歌曲
      if (newValue === this.duration) {
        // 判断时最后一首 下一首就切换到第一首
        if (this.playListIndex === this.playList.length - 1) {
          this.updatePlayListIndex(0)
          this.play()
        } else {
          // 不是最后一首 就切换下一首
          this.updatePlayListIndex(this.playListIndex + 1)
        }
      }
    }
  }
```

进度条跟着歌曲进行而移动

```vue
 <!-- 进度条 -->
    <div class="footerCountent">
        //min max都是range里面的属性
      <input type="range" class="range" min="0" :max="duration" v-model="currentTime" step="0.05" />
    </div>
```



### 6、搜索框 Search.vue



#### 6.1历史记录的保存/去重/垃圾桶点击删除/长度限制

 Search.vue

```vue
  methods: {
    // 搜索历史记录本地储存
    enterKey: async function () {
      if (this.searchKey !== '') {
        this.keyWordList.unshift(this.searchKey)
        // 去重  es6 语法 set去重
        this.keyWordList = [...new Set(this.keyWordList)]
        // 固定长度
        if (this.keyWordList.length > 5) {
          this.keyWordList.splice(this.keyWordList.length - 1, 1)
        }
        // 保存到本地
        localStorage.setItem('keyWordList', JSON.stringify(this.keyWordList))

        let res = await getSearchMusic(this.searchKey)
        // 打印的是相关歌手的歌曲
        console.log(res)
        // 歌曲赋值
        this.searchList = res.data.result.songs
        this.searchKey = ''
      }
    },
    // 给垃圾桶加点击删除事件
    delHistory: function () {
      localStorage.removeItem('keyWordList')
      this.keyWordList = []
    },
    searchHistory: async function (item) {
      let res = await getSearchMusic(item)
      // 打印的是相关歌手的歌曲
      console.log(res)
      // 歌曲赋值
      this.searchList = res.data.result.songs
    },
    updateIndex: function (item) {
      this.$store.commit('pushPlayList', item)
      this.$store.commit('updatePlayListIndex', this.$store.state.playList.length - 1)
    }
  }
```

#### 6.2搜索之后回车返回记录

##### 6.2.1接口

```js
说明 : 调用此接口 , 传入搜索关键词可获得搜索建议 , 搜索结果同时包含单曲 , 歌手 , 歌单 ,mv 信息

必选参数 : keywords : 关键词

可选参数 : type : 如果传 'mobile' 则返回移动端数据

接口地址 : /search/suggest

调用例子 : /search/suggest?keywords= 海阔天空 /search/suggest?keywords= 海阔天空&type=mobile
```

### 7 登陆个人中心页面

router的jindex.js权限判断

```js
 // 路由守卫 权限判断
    beforeEnter: (to, from, next) => {
      if (store.state.isLogin || store.state.token || localStorage.getItem('token')) {
        next()
      } else {
        next('/login')
      }
    },
```

