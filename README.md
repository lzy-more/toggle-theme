# Vue 3 + TypeScript + Vite

分支说明：切换主题色
前言
换肤功能是一项普遍的需求，尤其是在夜晚，用户更倾向于使用暗黑模式。在我负责的公司项目中，每个项目都有换肤功能的需求。
过去，我主要使用 SCSS 变量，并利用其提供的函数，如 @each、map-get来实现换肤功能。但因其使用成本高，只能适用于SCSS项目，于是后来我改用 CSS 变量来实现换肤。这样无论是基于 LESS 的 React 项目，还是基于 SCSS 的 Vue 项目，都能应用换肤功能。并且使用时只需调用var函数，降低了使用成本。
Demo地址：
https://github.com/cwjbjy/vite-vue-ts-seed/tree/feature/css
1. 一键换肤
1. 前置知识
CSS变量：声明自定义CSS属性，它包含的值可以在整个文档中重复使用。属性名需要以两个减号（--）开始，属性值则可以是任何有效的 CSS 值
--fontColor:'#fff'
Var函数：用于使用CSS变量。第一个参数为CSS变量名称，第二个可选参数作为默认值
color: var(--fontColor);
CSS属性选择器：匹配具有特定属性或属性值的元素。例如[data-theme='black']，将选择所有 data-theme 属性值为 'black' 的元素
2. 定义主题色
1. 新建src/assets/theme/theme-default.css
这里定义字体颜色与布局的背景色，更多CSS变量可根据项目的需求来定义
[data-theme='default'] {
  /* 字体 */
  --font-primary: #fff;
  --font-highlight: #434a50;
  /* 布局 */
  --background-header: #2f3542;
  --background-aside: #545c64;
  --background-main: #0678be;
}
2. 新建src/assets/theme/theme-black.css
再定义一套暗黑主题色
[data-theme='black'] {
  /* 字体 */
  --font-primary: #fff;
  --font-highlight: #434a50;
  /* 布局 */
  --background-header: #303030;
  --background-aside: #303030;
  --background-main: #393939;
}
3. 新建src/assets/theme/index.css
在index.css文件中导出全部主题色
@import './theme-default.css'; 
@import './theme-black.css';
4. 引入全局样式
在入口文件引入样式，比如我这里是main.tsx
import '@/assets/styles/theme/index.css';
3. 在html标签上增加自定义属性
修改index.html，在html标签上增加自定义属性data-theme
<html lang="en" data-theme="default"></html>
这里使用data-theme是为了被CSS属性选择器[data-theme='default']选中，也可更换为其他自定义属性，只需与CSS属性选择器对应上即可。
4. 修改CSS主题色
关键点：监听change事件，使用document.documentElement.setAttribute动态修改data-theme属性，然后CSS属性选择器将自动选择对应的css变量
<template>
  <div>
    <select name="pets" @change="handleChange">
      <option value="default">默认色</option>
      <option value="black">黑色</option>
    </select>
    <div>登录页面</div>
  </div>
</template>


<script setup lang="ts">
const handleChange = (e: Event) => {
  window.document.documentElement.setAttribute('data-theme', (e.target as HTMLSelectElement).value);
};
</script>


<style lang="scss">
body {
  color: var(--font-primary);
  background-color: var(--background-main);
}
</style>
效果图，默认色：
图片

效果图，暗黑色：

图片

5. 修改JS主题色
切换主题色，除了需要修改css样式，有时也需在js文件中修改样式，例如修改echarts的配置文件，来改变柱状图、饼图等的颜色。
1. 新建src/config/theme.js
定义图像的颜色，这里定义字体的颜色，默认情况下字体为黑色，暗黑模式下，字体为白色
const themeColor = {
  default: {
    font: '#333',
  },
  black: {
    font: '#fff',
  },
};


export default themeColor;
2. 修改vue文件
关键点：
1. 定义主题色TS类型，规定默认和暗黑两种：type ThemeTypes = 'default' | 'black';
2. 定义theme响应式变量，用来记录当前主题色：const theme = ref<ThemeTypes>('default');
3. 监听change事件，将选中的值赋给theme：theme.value = selectTheme;
4. 使用watch进行监听，如果theme改变，则重新绘制echarts图形
完整的vue文件：
<template>
  <div>
    <select name="pets" @change="handleChange">
      <option value="default">默认色</option>
      <option value="black">黑色</option>
    </select>
    <div>登录页面</div>
    <div ref="echartRef" class="myChart"></div>
  </div>
</template>


<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import themeColor from '@/config/theme';
import * as echarts from 'echarts';


type ThemeTypes = 'default' | 'black';


const echartRef = ref<HTMLDivElement | null>(null);
const theme = ref<ThemeTypes>('default');
const handleChange = (e: Event) => {
  const selectTheme = (e.target as HTMLSelectElement).value as ThemeTypes;
  theme.value = selectTheme;
  window.document.documentElement.setAttribute('data-theme', selectTheme);
};


const drawGraph = () => {
  let echartsInstance = echarts.getInstanceByDom(echartRef.value!);
  if (!echartsInstance) {
    echartsInstance = echarts.init(echartRef.value);
  }
  echartsInstance.clear();
  var option = {
    color: ['#3398DB'],
    title: {
      text: '柱状图',
      left: 'center',
      textStyle: {
        color: themeColor[theme.value].font,
      },
    },
    xAxis: [
      {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
        axisLabel: {
          show: true,
          color: themeColor[theme.value].font,
        },
        nameTextStyle: {
          color: themeColor[theme.value].font,
        },
      },
    ],
    yAxis: [
      {
        type: 'value',
        axisLabel: {
          show: true,
          color: themeColor[theme.value].font,
        },
        nameTextStyle: {
          color: themeColor[theme.value].font,
        },
      },
    ],
    series: [
      {
        name: '直接访问',
        type: 'bar',
        barWidth: '60%',
        data: [10, 52, 200, 334, 390, 330, 220],
      },
    ],
  };


  echartsInstance.setOption(option);
};
onMounted(() => {
  drawGraph();
});
watch(theme, () => {
  drawGraph();
});
</script>


<style lang="scss">
body {
  color: var(--font-primary);
  background-color: var(--background-main);
}
.myChart {
  width: 300px;
  height: 300px;
}
</style>
2. 一键变灰
在特殊的日子里，网页有整体变灰色的需求。可以使用filter 的 grayscale() 改变图像灰度，值在 0% 到 100% 之间，值为0%展示原图，值为100% 则完全转为灰度图像
body {
  filter: grayscale(1); //1相当于100%
}
结尾
本文只是介绍大概的思路，更多的功能可根据业务增加。例如将主题色theme存储到pinia上，应用到全局上；将主题色存储到localStorage上，在页面刷新时，防止主题色恢复默认。
