---
# https://vitepress.dev/reference/default-theme-home-page
layout: home

hero:
  name: "Sampson的博客"
  text: "博客为工作日常积累"
  tagline: 网站编程内容有点杂乱，仅供个人使用，欢迎大家交流学习
  image: /person.webp
  actions:
    - theme: brand
      text: 进入学习
      link: /markdown-examples
    - theme: alt
      text: 进入简介
      link: /api-examples

features:
  - title: 前端
    details: 本人工作搞前端,技术栈有HTML、CSS、JS、Vue2、Vue3、React、ReactNative、NextJs、NuxtJs、Uniapp、ThreeJs、Electron、Echarts、微信小程序以及各个技术栈所需的组件库（等等）
  - title: 后端
    details: 偶尔接触了后端，技术栈有NodeJs、Java
  - title: 简介
    details: 热爱编程，热爱学习，热爱赚钱，往后会一直在这个博客里面加入新鲜的知识，如前后端开发小技巧，组件库封装，新知识灌入，总的来说，我的博客网站会很杂乱，但都是真知识，欢迎大家一起收藏学习

---

<style lang="less">
:root {
  --vp-home-hero-image-background-image: linear-gradient( -45deg, #bd34fe 50%, #47caff 50% );
  --vp-home-hero-image-filter: blur(72px);
}

.VPHome {
  /* height:100vh; */
}

.VPHero {
  /* height:100vh; */
  /* background-image: url('./public/bg.png'); */
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.VPHero .container {
  background-color: rgba(255, 255, 255, 0.9);
  padding: 20px;
  border-radius: 10px;
}

.VPFeatures .container {
  background-color: rgba(255, 255, 255, 0.9);
  padding: 20px;
  border-radius: 10px;
}

.VPNavBar {
 box-shadow: 1px 1px 7px #857e7e;
}

.VPImage {
  border-radius: 10px;
}
</style>

