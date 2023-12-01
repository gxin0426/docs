## front base

### 路线

**初级阶段：基础知识**

1. **HTML**: 学习HTML基础，理解网页结构。
   - 学习资源：W3Schools, MDN Web Docs。
2. **CSS**: 学习CSS基础，掌握页面布局和样式设计。
   - 学习资源：CSS Tricks, W3Schools。
3. **JavaScript**: 学习JavaScript基础，理解交互式网页的开发。
   - 学习资源：Eloquent JavaScript, JavaScript.info。

**中级阶段：框架和工具**

1. **响应式设计**: 学习如何创建响应式网站，适应不同设备。
2. **Bootstrap**: 学习Bootstrap框架，快速布局和样式设计。
3. **jQuery**: 了解jQuery，简化JavaScript编程。

**高级阶段：现代前端框架**

1. **React.js**: 选择一个现代前端框架深入学习。
   - 学习资源：官方文档，相关在线课程。

### 基础知识

#### html语法中都有哪些label

在HTML中，术语“label”通常指的是 `<label>` 标签，用于定义表单控件间的关联。不过，如果你是指“标签”（tag），那么HTML中有多种标签。以下是一些基本的HTML标签：

##### 文档结构标签
- `<!DOCTYPE>`: 定义文档类型。
- `<html>`: 根元素，包含整个HTML文档。
- `<head>`: 包含文档的元数据。
- `<title>`: 文档的标题。
- `<body>`: 文档的主体，包含可见内容。

##### 文本格式化标签
- `<h1>` 到 `<h6>`: 标题标签，`<h1>` 是最高级别。
- `<p>`: 段落标签。
- `<br>`: 换行标签。
- `<b>`: 粗体文本。
- `<i>`: 斜体文本。
- `<strong>`: 重要文本。
- `<em>`: 强调文本。
- `<mark>`: 标记文本。
- `<small>`: 小号文本。
- `<sub>`: 下标文本。
- `<sup>`: 上标文本。

##### 超链接和图像标签
- `<a>`: 锚标签，用于创建链接。
- `<img>`: 图像标签。

##### 列表标签
- `<ul>`: 无序列表。
- `<ol>`: 有序列表。
- `<li>`: 列表项。

##### 表格标签
- `<table>`: 表格。
- `<tr>`: 表格行。
- `<td>`: 表格单元。
- `<th>`: 表格头部单元。
- `<thead>`: 表格头部区域。
- `<tbody>`: 表格主体区域。
- `<tfoot>`: 表格底部区域。

##### 表单标签
- `<form>`: 表单。
- `<input>`: 输入框。
- `<textarea>`: 文本区域。
- `<button>`: 按钮。
- `<select>`: 下拉列表。
- `<option>`: 选项。

##### 语义化标签（HTML5）
- `<article>`: 文章。
- `<section>`: 部分。
- `<header>`: 头部区域。
- `<footer>`: 底部区域。
- `<nav>`: 导航链接区域。
- `<aside>`: 侧边栏。

#### css属性和布局都有哪些知识点

CSS（层叠样式表）用于控制网页的视觉效果和布局。它有许多属性和布局技术，可以用来设计和排列网页元素。以下是一些重要的CSS属性和布局知识点：

##### 基本样式属性
- **颜色和背景**: `color`, `background-color`, `background-image`, `background-repeat`, `background-position`, `background-size`.
- **文本格式**: `font-family`, `font-size`, `font-weight`, `text-align`, `line-height`, `text-decoration`, `text-transform`.
- **盒模型**: `width`, `height`, `padding`, `border`, `margin`.
- **列表样式**: `list-style-type`, `list-style-position`, `list-style-image`.

##### 高级样式属性
- **伪类和伪元素**: `:hover`, `:active`, `:first-child`, `::before`, `::after`.
- **透明度和渐变**: `opacity`, `linear-gradient`, `radial-gradient`.
- **变形和过渡**: `transform`, `transition`.
- **动画**: `@keyframes`, `animation`.

##### 布局技术
- **浮动布局**: 使用 `float` 和 `clear` 实现布局。
- **定位布局**: `position` (relative, absolute, fixed, sticky)。
- **Flexbox布局**: 灵活的盒状模型，适用于一维布局。
- **Grid布局**: 用于二维布局的强大系统。
- **多列布局**: `column-count`, `column-gap`, `column-rule`。

##### 响应式设计
- **视口和媒体查询**: `@media` 规则，用于创建响应不同屏幕尺寸和分辨率的设计。
- **百分比和相对单位**: 使用 `%`, `em`, `rem`, `vw`, `vh` 等。
- **响应式图片**: `srcset` 和 `<picture>` 元素。

##### CSS预处理器
- **Sass**, **Less**, 和 **Stylus**: 提供变量、嵌套、混合等高级功能。

##### 现代CSS功能
- **CSS变量**: 自定义属性，用于存储可重复使用的值。
- **CSS Grid布局增强**: 更复杂的网格布局。
- **Flexbox布局增强**: 提高一维布局的灵活性。
- **Subgrid**: Grid布局的扩展，允许网格项具有自己的子网格。

##### 性能和最佳实践
- **加载时间优化**: 压缩CSS，删除未使用的CSS。
- **可访问性**: 确保样式不影响网站的可访问性。
- **跨浏览器兼容性**: 确保样式在不同浏览器中的一致性。

这些知识点构成了CSS的核心。掌握这些基础后，你可以创建视觉上吸引人、布局合理的网页。随着技能的提升，你可以探索更多高级技术和最新的CSS功能。

#### 列出常用的js语法

JavaScript是一种功能丰富的编程语言，用于添加交互性和动态功能到网页上。以下是一些常用的JavaScript语法元素：

##### 基本元素

1. **变量声明**
   - `var`, `let`, `const`：用于声明变量。

2. **数据类型**
   - 基本类型：`Number`, `String`, `Boolean`, `undefined`, `null`
   - 复杂类型：`Object`, `Array`, `Function`

3. **运算符**
   - 算术运算符：`+`, `-`, `*`, `/`, `%`
   - 比较运算符：`==`, `===`, `!=`, `!==`, `<`, `>`, `<=`, `>=`
   - 逻辑运算符：`&&`, `||`, `!`

4. **条件语句**
   - `if`, `else if`, `else`
   - `switch`

5. **循环**
   - `for`, `while`, `do...while`
   - `for...in`, `for...of`（用于对象和数组）

6. **函数**
   - 函数声明：`function functionName() {}`
   - 箭头函数：`(params) => {}`

7. **数组方法**
   - `.push()`, `.pop()`, `.shift()`, `.unshift()`
   - `.map()`, `.filter()`, `.reduce()`
   - `.forEach()`, `.indexOf()`, `.find()`, `.sort()`

8. **对象**
   - 访问属性：`object.property`, `object['property']`
   - 修改属性：`object.property = value`

##### 高级语法

1. **模板字符串**
   - 使用反引号和`${}`插入变量：`` `Hello, ${name}!` ``

2. **解构赋值**
   - 数组：`[a, b] = [10, 20]`
   - 对象：`{a, b} = {a:10, b:20}`

3. **展开运算符**
   - 数组：`[...array1, ...array2]`
   - 对象：`{...object1, ...object2}`

4. **默认参数**
   - `function functionName(a = 5) {}`

5. **回调函数和高阶函数**
   - 函数作为参数传递给其他函数

6. **异步编程**
   - `Promise`, `async/await`

7. **事件处理**
   - `addEventListener()`, `removeEventListener()`

8. **DOM操作**
   - 选择元素：`document.querySelector()`, `document.querySelectorAll()`
   - 修改元素：`.innerHTML`, `.textContent`, `.style`
   - 创建和移除元素：`document.createElement()`, `.appendChild()`, `.removeChild()`

##### 调试和错误处理

1. **控制台输出**
   - `console.log()`, `console.error()`, `console.warn()`

2. **错误处理**
   - `try...catch`, `throw`

#### dom api有哪些

DOM（文档对象模型）API 提供了一系列方法和属性，允许JavaScript与HTML文档进行交互和操作。这些API使得可以在文档加载后动态地改变文档内容、结构和样式。以下是一些常用的DOM API：

##### 元素选择
- `document.getElementById(id)`: 通过ID选择元素。
- `document.getElementsByClassName(name)`: 通过类名选择元素。
- `document.getElementsByTagName(name)`: 通过标签名选择元素。
- `document.querySelector(selector)`: 选择第一个匹配的元素。
- `document.querySelectorAll(selector)`: 选择所有匹配的元素。

##### 元素操作
- `element.innerHTML`: 获取或设置元素内的HTML。
- `element.textContent`: 获取或设置元素内的文本内容。
- `element.style`: 用来获取或设置元素的样式。
- `element.setAttribute(name, value)`: 设置元素的属性。
- `element.getAttribute(name)`: 获取元素的属性。
- `element.removeAttribute(name)`: 移除元素的属性。

##### 创建和移除元素
- `document.createElement(tag)`: 创建新元素。
- `element.appendChild(child)`: 将子元素添加到父元素。
- `element.removeChild(child)`: 从父元素中移除子元素。
- `element.replaceChild(newChild, oldChild)`: 替换元素。
- `element.cloneNode(deep)`: 克隆元素。

##### 事件处理
- `element.addEventListener(event, handler)`: 给元素添加事件监听器。
- `element.removeEventListener(event, handler)`: 移除事件监听器。
- `event.preventDefault()`: 阻止事件的默认行为。
- `event.stopPropagation()`: 阻止事件冒泡。

##### 类和样式操作
- `element.classList.add(className)`: 添加类。
- `element.classList.remove(className)`: 移除类。
- `element.classList.toggle(className)`: 切换类。
- `element.classList.contains(className)`: 检查是否包含某个类。

##### 节点属性和遍历
- `element.childNodes`: 获取子节点列表。
- `element.firstChild`: 获取第一个子节点。
- `element.lastChild`: 获取最后一个子节点。
- `element.nextSibling`: 获取下一个兄弟节点。
- `element.previousSibling`: 获取上一个兄弟节点。
- `element.parentNode`: 获取父节点。

##### 其他常用API
- `document.createElementNS()`: 创建指定命名空间的元素。
- `document.createDocumentFragment()`: 创建一个文档片段。
- `element.getBoundingClientRect()`: 获取元素的大小及其相对于视口的位置。

这些API构成了与DOM交互的基础，使得Web开发者可以创建动态和交互式的网页。通过结合这些API，可以实现丰富的客户端功能，如动态内容更新、表单验证、动画效果等。掌握这些DOM操作是前端开发的一个重要部分。

#### js如何操作浏览器网页内的元素

在JavaScript中操作浏览器网页内的元素主要涉及到选择、修改、添加和删除元素。以下是一些基本步骤和示例代码，展示了如何进行这些操作：

##### 1. 选择元素
首先，你需要选择一个或多个你想要操作的DOM元素。

```javascript
// 通过ID选择元素
const elementById = document.getElementById('elementId');

// 通过类名选择元素
const elementsByClass = document.getElementsByClassName('className');

// 通过标签名选择元素
const elementsByTag = document.getElementsByTagName('tagName');

// 选择第一个匹配CSS选择器的元素
const firstElement = document.querySelector('.className');

// 选择所有匹配CSS选择器的元素
const allElements = document.querySelectorAll('.className');
```

### 2. 修改元素
一旦选择了元素，你可以修改其内容、样式或属性。

```javascript
// 修改元素的文本内容
elementById.textContent = '新的文本内容';

// 修改元素的HTML内容
elementById.innerHTML = '<span>新的HTML内容</span>';

// 改变元素的样式
elementById.style.color = 'blue';
elementById.style.fontSize = '16px';

// 设置或修改元素的属性
elementById.setAttribute('href', 'https://www.example.com');
```

##### 3. 添加和删除元素
你可以创建新的元素，并将它们添加到DOM中，或者从DOM中删除现有元素。

```javascript
// 创建一个新元素
const newElement = document.createElement('div');
newElement.textContent = '这是一个新元素';

// 将新元素添加到DOM
document.body.appendChild(newElement);

// 删除DOM中的元素
const parentElement = document.getElementById('parentElementId');
parentElement.removeChild(elementById);
```

##### 4. 事件处理
可以为元素添加事件监听器来处理用户交互，如点击、悬停等。

```javascript
// 为元素添加点击事件监听器
elementById.addEventListener('click', function() {
    alert('元素被点击了!');
});

// 移除事件监听器
elementById.removeEventListener('click', functionName);
```

##### 5. 类和样式操作
可以通过classList API来添加、删除、切换或检查元素的类。

```javascript
// 添加类
elementById.classList.add('new-class');

// 移除类
elementById.classList.remove('existing-class');

// 切换类
elementById.classList.toggle('toggle-class');

// 检查元素是否包含某个类
if (elementById.classList.contains('some-class')) {
    // 执行操作
}
```

##### 6. 表单处理
对于表单元素，可以通过JavaScript读取或设置其值。

```javascript
// 读取输入框的值
const inputValue = document.getElementById('inputElement').value;

// 设置输入框的值
document.getElementById('inputElement').value = '新值';
```

通过这些基本操作，你可以在浏览器中动态地操作网页元素，创建丰富的用户交互体验。实际上，这只是JavaScript操作DOM的表面，还有更多高级技术和方法等待你去探索。

JavaScript对DOM的操作确实非常广泛和深入，除了基本的选择、修改、添加和删除DOM元素之外，还有一些更高级的用法和技术。这些高级技术使得JavaScript在前端开发中变得更加强大和灵活。以下是一些高级用法：

##### 1. 事件委托
事件委托是一种事件处理模式，在这种模式下，你不必在每个子元素上单独设置事件监听器。相反，你可以在父元素上设置单个监听器，然后根据事件的目标元素（`event.target`）来决定如何响应。

```javascript
document.getElementById('parentElement').addEventListener('click', function(event) {
    if (event.target.tagName === 'LI') {
        // 处理列表项的点击事件
    }
});
```

##### 2. 使用 DocumentFragment 提高性能
`DocumentFragment` 是一个轻量的DOM节点，可以作为其他DOM节点的临时容器。在`DocumentFragment`中进行DOM操作不会触发页面重绘，所以它可以用来提高性能，尤其是在处理大量DOM操作时。

```javascript
const frag = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const el = document.createElement('div');
    el.textContent = `Element ${i}`;
    frag.appendChild(el);
}
document.body.appendChild(frag);
```

##### 3. 自定义事件和触发
JavaScript允许创建自定义事件，并在需要时触发这些事件。这可以在组件间传递消息或实现复杂的事件处理逻辑。

```javascript
// 创建一个自定义事件
const myEvent = new CustomEvent('myCustomEvent', { detail: { some: 'data' } });

// 触发事件
document.dispatchEvent(myEvent);
```

##### 4. MutationObserver
`MutationObserver` API用于监视DOM变化，如元素添加、删除或属性变化。这对于创建响应DOM变化的动态应用程序非常有用。

```javascript
const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
        console.log(mutation);
    });
});

observer.observe(document.body, { attributes: true, childList: true, subtree: true });
```

##### 5. Shadow DOM
Shadow DOM允许在页面上创建封装的DOM和样式，而不影响页面的其余部分。这是Web组件技术的一个关键部分。

```javascript
const shadowHost = document.getElementById('shadowHost');
const shadowRoot = shadowHost.attachShadow({ mode: 'open' });
shadowRoot.innerHTML = `<style> p { color: red; } </style> <p>Hello Shadow DOM</p>`;
```

##### 6. 动画
使用JavaScript的`requestAnimationFrame`进行流畅的动画和视觉变化控制。

```javascript
function animate() {
    // 动画代码
    requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

##### 7. 异步DOM更新
利用`async`和`await`，或者`Promise`来处理异步DOM更新。

```javascript
async function fetchDataAndUpdateDOM() {
    const data = await fetchData();
    document.getElementById('output').textContent = data;
}
```

##### 8. 模块化和组件化
利用ES6模块或现代框架（如React, Vue, Angular）进行组件化开发，使代码更加模块化和重用。

```javascript
// 在一个模块中导出
export function myFunction() {
    // 功能代码
}

// 在另一个模块中导入
import { myFunction } from './myModule.js';
```

这些高级技术和方法不仅增强了JavaScript对DOM的控制能力，还极大地提高了前端开发的效率和质量。随着你对这些高级概念的掌握，你将能够创建更复杂和功能更强大的Web应用程序。