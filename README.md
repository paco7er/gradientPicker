# gradientPicker
基于vue3开发的渐变色组件

## 使用方法
1.单击渐变色条可以在对应位置新增一种颜色

2.右击颜色点可以删除一种颜色（必须剩余两种颜色）

3.可拖动颜色点，更改颜色位置

4.可通过表格新增/删除颜色点

5.可通过表格选取颜色，移动颜色位置
## 使用代码
```js
<GradientPicker :modelValue="colorBar"@update:modelValue="val => colorBar = val"/>
colorBar: [
  { color: '#FF0000', position: 0 },
  { color: '#00FF00', position: 100 }
],
```
## 使用效果
![image](https://github.com/user-attachments/assets/92c39f0c-ae95-4096-b616-0c6e1c05a84a)
