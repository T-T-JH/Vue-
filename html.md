## HTML核心

#### 标签结构

- p 	定义html中的一个段落
- body   定义html的主体
- html    定义整个html文档(开始标签,结束标签)
- a           定义一个链接
- "<h1> - <h6>"    定义HTML标题
- hr          定义水平线

#### 属性

- id	为元素指定标识符

- class   为元素指定一个或多个类名

- style    在元素上应用CSS样式

- title      为元素提供额外提示信息,在鼠标悬停时显示

- data-*  储存自定义数据

- href      指定连接url

  ```js
  <a>, <link>
  ```

  

- src        指定外部资源的URL

  ```
  <img>`, `<script>`, `<iframe>
  ```

- alt         为图像提供替代文本,图像无法显示时显示<img>

- type      指定输入控件的类型(text,password,checkbox)

  ```
  <input>,<button>
  ```

- value      指定元素初始值

  ```
  <input>,<button>,<option>
  ```

- disabled  禁用元素

- checked    指定复选框或单选按钮是否选中

  ```
  <input type="checkbox">, <input type="radio">
  ```

- placeholder  在输入框中显示提示文本

  ```
  <input>, <textarea>
  ```

- target         指定链接或表单提交的目标窗口或框架

  ```
  <a>,<form>
  ```

- readonly     使输入框只读

- required      指定输入字段为必填项

- autoplay       自动播放媒体

  ```
  <audio>,<video>
  ```

- onclick          点击元素时触发JavaScript事件

- onmouseover    鼠标悬停在元素上时触发JavaScript事件

- onchange       元素值发生变化时出发JavaScript事件

#### 表单元素

#### 语义化标签