### html基础

![](frontend2/grumpy-cat-small.png)

开始标签

结束标签

内容

元素

元素中可以有属性

```html
<p class="editor-note"> my cat </p>
```

空元素: 不包含任何内容

```html
<img src="/opt/img.img" alt="test img" /> #两个属性
```

```html
文档类型
<!doctype html>
html元素 包含整个页面所有内容
<html lang="en-US">
  不向看客展示得页面成员
  <head>
    指定一些属性
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    显示在浏览器标签页上
    <title>My test page</title>
  </head>
  body中是期望用户看到的内容
  <body>
    <img src="images/firefox-icon.png" alt="My test image" />
  </body>
</html>
```

图像

```html
<img src="images/firefox-icon.png" alt="My test image" />
```

标记文本

- 标题

  ```
  <h1>主标题</h1>
  <h2>顶层标题</h2>
  <h3>子标题</h3>
  <h4>次子标题</h4>
  ```

- 段落

  ```html
  <p>这是一个段落</p>
  ```

- 列表

  - 无序列表 <ul>

  - 有序列表 <ol>

    列表的每个项目用一个列表项目(List Item) 元素<li> 包围

- 链接

  ```
  <a href="">Meee</a>
  ```

### css基础

![](frontend2/css-declaration-small.png)

选择器(Selector)

声明(Declaration)

属性(Properties)

属性值(Properties value)

```css
p {
    color: red;
    width: 500px;
    border: 1px solid black;
}
```

盒子模式

- padding
- border
- margin

![](frontend2/box-model.png)

### html部分

#### html介绍与多媒体和嵌入

HTML 的主要工作之一就是给予文本意义（也被叫做[语义](https://developer.mozilla.org/zh-CN/docs/Glossary/Semantics)）

 HTML 元素和页面的组织方式

##### 开始

- 块级元素和内敛元素

  块级元素通常表示段落、标题等内容，而行级元素则用于文本内的小块内容，比如强调或超链接。下面的例子将展示这两个概念的具体应用：

  ```html
  <!DOCTYPE html>
  <html>
  <head>
      <title>块级元素和行级元素示例</title>
  </head>
  <body>
  
  <!-- 块级元素的例子 -->
  <div>
      <p>这是一个段落标签，它是一个块级元素，通常用于表示较大的内容块。</p>
      <h1>这是一个标题标签，也是块级元素，用于显示标题。</h1>
  </div>
  
  <!-- 行级元素的例子 -->
  <p>这里有一些文本，并且有一个<span style="color: blue;">蓝色的行级元素</span>来突出显示。</p>
  <p>如果你想要 <em>强调</em> 文本的某个部分，你可以使用<em>标签。</p>
  <p>同时，如果你想要一个文本<strong>更加突出</strong>，你可以使用<strong>标签。</p>
  <p>在文本中添加链接：<a href="https://www.example.com">这是一个超链接</a>，它也是行级元素。</p>
  
  </body>
  </html>
  ```

  - `<div>`, `<p>`, 和 `<h1>` 是块级元素，它们各自在新的行上显示，并且可以设置外边距和宽度。
  - `<span>`, `<em>`, `<strong>`, 和 `<a>` 是行级元素，它们内嵌在文本中，与其他文本流在同一行显示。通过 `<span>` 可以对文本进行样式化，而 `<em>` 和 `<strong>` 分别用于标示文本的强调和重要性。 `<a>` 用于创建超链接。

  | 原义字符 | 等价字符引用 |           单词           |
  | :------: | :----------: | :----------------------: |
  |    <     |    `&lt;`    |  less than（小于符号）   |
  |    >     |    `&gt;`    | greater than（大于符号） |
  |    "     |   `&quot;`   |  quotation mark（引号）  |
  |    '     |   `&apos;`   |    apostrophe（撇号）    |
  |    &     |   `&amp;`    |    ampersand（和号）     |

- <em>（emphasis）

- <strong> (strong importance) (如果仅仅为了获得粗体样式而不增加语义辅助，你应该使用<span>元素和一些 CSS，或者是 <b>元素)

- <b> <i> <u>的情况却有点复杂。它们出现于人们要在文本中使用粗体、斜体、下划线但 CSS 仍然不被完全支持的时期。像这样仅仅影响表象而且没有语义的元素，被称为**表象元素**（presentational elements）**并且不应该再被使用**。 只有在没有更合适的元素时，才适合使用 `<b>`、`<i>` 或 `<u>` 来表达传统上用粗体、斜体或下划线表达的意思；而通常情况下是有更合适的元素可供使用的。`<strong>`、`<em>`、`<mark>` 或 `<span>` 可能是更加合适的选择

- ```html
  <p> 我的链接: <a href="https://test.com"> test </a>。</p>
  ```

- 描述列表使用与其他列表类型不同的闭合标签—— <dl> ；此外，每一项都用 <dt>（description term）元素闭合。每个描述都用 <dd> (description definition）元素闭合。

- <sub> 下标 <sup>上标

- ``` html
  <br> 换行 <hr> 主题性中断元素
  ```

- ```html
  <iframe src="url" width="" height=""></iframe>
  <!--  <object> 元素用于嵌入多种类型的媒体，比如 Flash、PDF、Java applets、SVG 图像等 -->
  <object data="文件或资源的URL" width="" height=""></object>
  <object data="document.pdf" width="600" height="400" type="application/pdf"></object>
  
  <!-- <embed> 元素用于嵌入各种类型的媒体，如视频、音频文件、Flash 内容等。它比 <object> 元素更为简单 -->
  ```

- 响应式设计

  ```markdown
  <img
    srcset="elva-fairy-320w.jpg, elva-fairy-480w.jpg 1.5x, elva-fairy-640w.jpg 2x"
    src="elva-fairy-640w.jpg"
    alt="Elva dressed as a fairy" /> 
    
  //这段代码是 HTML 中用于响应式图像的一个例子，具体用到了 `srcset` 属性。这个属性允许开发者为不同分辨率的屏幕提供多个图像源。浏览器会根据屏幕的分辨率和设备像素比（DPR）来选择最合适的图像。
  
  ### 代码解释
  
  1. **`srcset` 属性**：这个属性列出了不同版本的图像以及它们的大小。在这个例子中：
     
     - `elva-fairy-320w.jpg`：基础版本，没有指定比例，通常用于低分辨率屏幕。
     - `elva-fairy-480w.jpg 1.5x`：这个图像是1.5倍的设备像素比，适合中等分辨率的屏幕。
     - `elva-fairy-640w.jpg 2x`：这个图像是2倍的设备像素比，适合高分辨率的屏幕。
  
     浏览器会根据设备的像素密度自动选择最合适的图像。例如，如果设备的像素比为1.5x，那么浏览器会选择 `elva-fairy-480w.jpg`。
  
  2. **`src` 属性**：这是一个回退选项，用于老旧的浏览器或者在 `srcset` 出现问题时。这里它被设置为 `elva-fairy-640w.jpg`，意味着如果浏览器不支持 `srcset`，它将默认显示这个图像。
  
  3. **`alt` 属性**：这是图像的替代文本，用于描述图像内容，特别重要的是对于视觉障碍人士使用屏幕阅读器时，或者当图像由于某种原因无法显示时。在这个例子中，替代文本是“Elva dressed as a fairy”。
  
  ### 响应式图像的重要性
  
  使用 `srcset` 可以确保在不同设备和不同屏幕分辨率上都能提供最佳的用户体验。它允许浏览器选择适合当前查看条件的图像，既可以节省带宽（不会为小屏设备加载过大的图像），又可以确保图像的清晰度。
  ```

- ```html
  <picture>
    <source media="(max-width: 799px)" srcset="elva-480w-close-portrait.jpg" />
    <source media="(min-width: 800px)" srcset="elva-800w.jpg" />
    <img src="elva-800w.jpg" alt="Chris standing up holding his daughter Elva" />
  </picture>
  <!--
  <picture>：这是一个容器元素，它包含多个 <source> 元素和一个 <img> 元素。
  <picture> 元素的主要优点是它提供了比 <img> 元素的 srcset 更大的灵活性。通过使用不同的 <source> 元素，开发者可以为不同屏幕大小和分辨率指定最合适的图像
  -->
  <picture>
    <source type="image/svg+xml" srcset="pyramid.svg" />
    <source type="image/webp" srcset="pyramid.webp" />
    <img
      src="pyramid.png"
      alt="regular pyramid built from four equilateral triangles" />
  </picture>
  
  <!--
  这段代码同样使用了 HTML 的 <picture> 元素来提供灵活的响应式图像处理，但它与您之前提到的代码的主要区别在于它是基于图像格式类型来选择不同的源文件，而不是基于视口宽度
  
  -->
  ```

- ```html
  <table> <tr>(table row) <td>(table data) <th>(table header)
  <!--
  <tr> 代表 "table row"，用来创建表格的一行。你可以在 <tr> 标签内部添加 <td> 或 <th> 标签来创建单元格。
  <td> 代表 "table data"，用来创建表格的一个数据单元格，这通常是表格的主体部分。
  <th> 代表 "table header"，用来创建一个表头单元格，通常包含列标题，它默认加粗并居中显示。
  -->
  
  scope 用法
  
  ```

### css部分

#### css基础

- 类型、类和 ID 选择器
- 层叠、优先级、继承
- 层叠层
- 盒模式
- 处理不同方向的文本
- 背景和边框
- 

#### 样式化文本

#### css排版

### js部分

#### 第一步

#### js基础

- 变量：var let
- 

#### js对象介绍

#### 异步js

#### 客户端web api

### web表单

##### - 当然，下面我会详细介绍这些表单元素的用法和每种输入类型的具体例子。

### 表单元素

1. **`<form>`**: 用于创建 HTML 表单，以便用户输入数据。这个元素包围着所有的输入元素和按钮。
   ```html
   <form action="/submit-form" method="post">
     <!-- 表单元素放在这里 -->
   </form>
   ```

2. **`<fieldset>`**: 用于将表单中相关元素分组。
   ```html
   <fieldset>
     <legend>用户信息</legend>
     <!-- 分组的表单元素 -->
   </fieldset>
   ```

3. **`<legend>`**: 与 `<fieldset>` 配合使用，为分组提供标题。
   ```html
   <fieldset>
     <legend>注册信息</legend>
     <!-- 表单元素 -->
   </fieldset>
   ```

4. **`<textarea>`**: 提供一个多行的文本输入区域。
   ```html
   <textarea name="message" rows="4" cols="50">
     在这里输入文本...
   </textarea>
   ```

5. **`<label>`**: 定义输入字段的标签，通常提高可访问性。
   ```html
   <label for="username">用户名:</label>
   <input type="text" id="username" name="username">
   ```

6. **`<button>`**: 定义一个按钮。
   ```html
   <button type="submit">提交</button>
   ```

7. **`<input>`**: 创建各种类型的输入字段。

###### 输入（Input）类型

1. **button**
   - 定义可点击的按钮，非提交按钮。
   ```html
   <input type="button" value="点击我">
   ```

2. **checkbox**
   - 定义复选框，允许用户选择零个或多个选项。
   ```html
   <label><input type="checkbox" name="interest" value="sports"> 体育</label>
   <label><input type="checkbox" name="interest" value="music"> 音乐</label>
   ```

3. **file**
   - 定义文件选择框，允许用户选择文件上传。
   ```html
   <input type="file" name="myfile">
   ```

4. **hidden**
   - 定义不可见的输入字段，通常存储不需要用户修改的数据。
   ```html
   <input type="hidden" name="userid" value="12345">
   ```

5. **image**
   - 定义图像作为提交按钮。
   ```html
   <input type="image" src="submit.gif" alt="Submit">
   ```

6. **password**
   - 定义密码字段，输入字符被掩码。
   ```html
   <input type="password" name="pwd">
   ```

7. **radio**
   - 定义单选按钮，允许用户选择一个选项。
   ```html
   <label><input type="radio" name="gender" value="male" checked> 男性</label>
   <label><input type="radio" name="gender" value="female"> 女性</label>
   ```

8. **reset**
   - 定义重置按钮，将表单中的所有字段重置为其默认值。
   ```html
   <input type="reset">
   ```

9. **submit**
   - 定义用于提交表单的按钮。
   ```html
   <input type="submit" value="提交">
   ```

10. **text**
    - 定义单行输入字段，用于输入文本。
    ```html
    <input type="text" name="firstname">
    ```

这些是 HTML 表单中常用的元素和输入类型，每种类型都适用于特定的用途。

#### web表单进阶

### React
