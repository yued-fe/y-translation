> * 原文地址：https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web

> * 译文地址：https://github.com/yued-fe/y-translation/blob/master/todo/A%20Strategy%20Guide%20To%20CSS%20Custom%20Properties.md

> * 译者：[popeyesailorman](https://github.com/popeyesailorman)

> * 校对者：[ziven27](https://github.com/ziven27) [djmaxwow](https://github.com/djmaxwow)


# CSS 自定义属性的策略指南

**关于作者** Mike 是来自澳大利亚的独立网站开发人员，曾在澳大利亚的一些大型网站以及一些小型社区工作过...... <a href="https://www.smashingmagazine.com/author/michaelriethmuller/">关于Michael的更多信息...</a>

> CSS 自定义属性(也称为“CSS 变量”)，在目前所有的现代浏览器中都得到了支持，开发者已经在项目中开始使用，但是它们与预处理器中的变量不同，虽然我已经看到了很多例子，却没有搞清楚他们真正的优势在哪里。

自定义属性有很大的潜力可以改变我们编写和组织 CSS 的方式，并且在一定程度上改变 JavaScript 与 UI 组件的调用方式。我并不关心语法和它们的工作方式（为此，我建议你阅读“<a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">现在是时候开始使用自定义属性了</a>”）这篇文章。同时我想更深入地研究如何充分利用 CSS 自定义属性。

### 自定义属性与预处理器中的变量有何相似之处?
自定义属性有点像预处理器中的变量，但还是有很大的差别。最重要也是最明显的区别是在于语法。

在 SCSS 中我们用 $ 符号来定义变量：

``` scss
$smashing-red: #d33a2c;
```
在 Less 中我们用 @ 符号：
``` scss
@smashing-red: #d33a2c;
```
自定义属性遵循类似的约定并使用 -- 前缀的方式：
``` css
:root { --smashing-red: #d33a2c; }
.smashing-text { 
  color: var(--smashing-red);
}
```
自定义属性和预处理器中的变量最大的不同在于"键值对"的语法规则。自定义属性采用 var() 函数去取值。

另一个最明显的区别在于名称。它们之所以被称为"自定义属性"，是因为它们真的是 CSS 属性。在预处理器中，你可以在任何位置声明和使用变量，包括外部声明块，在 媒体查询中，甚至在选择器中也可以使用。
``` scss
$breakpoint: 800px;
$smashing-red: #d33a2c;
$smashing-things: ".smashing-text, .cats";

@media screen and (min-width: $breakpoint) {
  #{$smashing-things} {
    color: $smashing-red;
  }
}
```
而使用自定义属性，上面的大多数示例都是无效的。

自定义属性和常规 CSS 属性的用法是一样的。把它们当作动态属性会比变量要好。这意味着它们只能在声明块中使用，换句话说，自定义属性和选择器是强绑定的。这可以是 :root 选择器或任何其它有效的选择器。
``` css
:root { --smashing-red: #d33a2c; }

@media screen and (min-width: 800px) {
  .smashing-text, .cats {
    --margin-left:  1em;
  }
}
```
你可以在属性声明中的任何地方获取变量声明的值，这个意味着它们可以作为单个值使用，作为一个简写语句的一部分，甚至是在 calc() 方程式中使用。
``` css
.smashing-text, .cats {
  color: var(--smashing-red);
  margin: 0 var(--margin-horizontal);
  padding: calc(var(--margin-horizontal) / 2)
}
```
但是，它们不能用于媒体查询或选择器，包括 :nth-child()。

关于语法和自定义属性的工作原理可能还有很多，比如如何使用 fallback 值，以及是否可以将变量分配给其他变量（是），本文介绍的基础知识应该已经足以让大家理解其中的概念。更多关于自定义属性工作方式的详细信息，可以阅读由 Serg Hospodarets 编写的“<a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/"> 现在开始使用自定义属性吧 </a>”这篇文章。

### 动态与静态

抛开样式差异，预处理器和自定义属性中变量之间最大的差别是它们的作用范围。我们可以将变量根据作用域分为静态和动态两个部分。预处理器中的变量是静态的，而自定义属性是动态的。

在 CSS 中，静态意味着你可以在编译过程中更新变量的值，但是这不能改变它之前的值。

``` scss
$background: blue;
.blue {
  background: $background;
}
$background: red;
.red {
  background: $background;
}
```
结果是：
``` scss
.blue {
  background: blue;
}
.red {
  background: red;
}
```
一旦编译了成了 CSS，这个变量就会消失。这意味着我们在读取一个 .scss 文件并输出的时候并不需要关心 HTML、浏览器或其它输入，而自定制属性不是这样的。
预处理器确实有一种“块级作用域”，其中变量可以在选择器，函数或 mixin 中临时更改。这改变了块内变量的值，但它仍然是静态的。这与块有关，而不是选择器。在下面的例子中，变量 $background 在 .example 类内部被改变。即使我们使用相同的选择器，它也会变回块级作用域之外的初始值。

``` scss
$background: red;
.example {
  $background: blue;
  background: $background;
}

.example {
  background: $background;
}
```
编译后：
``` scss
.example {
  background: blue;
}
.example {
  background: red;
}
```
自定义属性不同于预处理器。在涉及自定义属性的地方，动态范围意味着它们受到继承和级联的影响。属性与选择器绑定，如果值发生变化，就会像其他 CSS 属性一样影响所有匹配的 DOM 元素。

这听起来很棒，因为你可以在媒体查询中，通过使用类似 hover 的伪类选择器甚至是 javascript 改变自定义属性。
``` css
a {
  --link-color: black;
}
a:hover,
a:focus {
  --link-color: tomato;
}
@media screen and (min-width: 600px) {
  a {
    --link-color: blue;
  }
}

a {
  color: var(--link-color);
}
```

我们不需要在自定义属性使用的地方去修改它 - 我们可以通过 CSS 去修改它的值。这意味着同一个自定义属性，可以在不同的地方，或者是上下文中有不同的值。

### 全局变量与局部变量

除了静态或动态之外，变量还可以是全局的或局部的。如果你写 JavaScript，你可能会更了解这一点。变量既可以作用在应用程序全局环境中，也可以将其作用范围限制在特定的功能或代码块中。

CSS 也一样。有全局的，也有局部的。品牌颜色、垂直间距、排版方式，这些你可能会希望都能在 app 端和网页中全局调用。当然也有一些局部的东西，比如，按钮组件可能具有大小尺寸。你不希望这些按钮的大小适用于所有输入元素或页面上的每个元素。

这是我们在 CSS 中熟悉的东西。我们开发了设计系统、命名规范和 JavaScript 库，这些可以分离局部组件和全局组件。自定义属性给这类问题提供了新的思路。

通常 CSS 自定义属性的范围局限于我们指定的选择器中。这看起来有点像局部变量。但是，自定义属性具有继承的特性，所以在很多情况下，它们表现的更像全局变量 - 特别是在应用于 :root 选择器的时候。这意味着我们需要考虑如何使用它们。

大量的示例都表示将自定义属性应用到 :root 元素上，对于 Demo 这还说得过去，但它可能会污染全局作用域，从而导致意外的继承问题。幸运的是，我们已经吸取了教训。

### 全局变量趋于静态
可能会有一些例外，但通常来说，CSS 中的大多数的全局变量也是静态的。

比如品牌颜色、字体和间距之类的变量不会在不同的组件之间产生太大的变化。当它们发生变化时，这往往是一个全局性的品牌重塑，或者是在一个成熟产品上很少发生的其他重大变化。对于这些变量来说它们仍然是有意义的，它们在很多地方被使用，而变量有助于保持一致性。但它们是动态的是没有意义的。这些变量的值不会以任何动态的方式变化。

因此，我强烈建议对全局(静态)变量使用预处理器。这不仅确保了它们始终是静态的，而且还可以在代码中直观地表示它们。这可以使 CSS 更易于阅读和维护。

### 局部静态变量也不是不可以使用
您可能会认为，考虑到全局变量是静态的，那么相反的，所有的局部变量都可能需要是动态的。虽然局部变量确实是动态的，但这远不如全局变量是静态的。

局部静态变量在很多情况下都是的。我在组件文件中使用预处理器变量，主要是作为开发人员的方便。

![](https://qidian.qpic.cn/qidian_common/349573/27d2d95901c158e29a5fc4dc7ada27d2/0)

我的 scss 可能看起来像这样：
``` scss
$button-sml: 1em;
$button-med: 1.5em;
$button-lrg: 2em;

.btn {
  // Visual styles
}

.btn-sml {
  font-size: $button-sml;
}

.btn-med {
  font-size: $button-med;
}

.btn-lrg {
  font-size: $button-lrg;
}
```
显然，如果我多次使用变量或从变量计算得到 margin 或 padding 值，这个示例将更有意义。然而，快速原型化不同尺寸的能力可能是一个充分的理由。

因为大多数静态变量都是全局的，所以我喜欢区分只在组件内部使用的静态变量。为此，可以在这些变量前面加上组件名，或者可以使用另一个前缀，如组件的 c 变量名或 l 变量名。您可以使用任何您想要的前缀，或者可以在全局变量前面加上前缀。无论您选择什么，区分都是很有帮助的，特别是当转换一个现有的代码基来使用自定义属性时。

### 何时使用自定义属性
如果可以在组件内部使用静态变量，那么什么时候应该使用自定义属性呢? 将现有的预处理器变量转换为自定义属性通常没什么意义。毕竟，自定义属性的原因是完全不同的。当我们有 CSS 属性时，自定义属性是有意义的，尤其是在 DOM 中(尤其是动态条件)，例如:fouces、hover、媒体查询或 JavaScript。

我猜想我们将始终使用某种形式的静态变量，尽管我们将来可能需要更少的静态变量，因为自定义属性提供了组织逻辑和代码的新方法。在此之前，我认为在大多数情况下，我们将使用预处理器变量和自定义属性的组合。

我们可以为自定义属性分配静态变量。无论它们是全局的还是局部的，在许多情况下，将静态变量转换为局部动态自定义属性都是有意义的。

>注意:您知道 $var 是自定义属性的有效值吗? Sass 的最新版本认识到了这一点，因此我们需要插入分配给自定义属性的变量，如: #{$var}。这告诉 Sass 您希望输出变量的值，而不是样式表中的 $var。这只适用于自定义属性等情况，其中变量名也可以是有效的 CSS。

如果我们以上面的按钮示例为例，决定所有的按钮都应该使用移动设备上的小变化，而不考虑 HTML 中应用的类，这是一种更动态的情况。为此，我们应该使用自定义属性。

``` css
$button-sml: 1em;
$button-med: 1.5em;
$button-lrg: 2em;

.btn {
  --button-size: #{$button-sml};
}

@media screen and (min-width: 600px) {
  .btn-med {
    --button-size: #{$button-med};
  }
  .btn-lrg {
    --button-size: #{$button-lrg};
  }
}

.btn {
  font-size: var(--button-size);
}
```
这里，我创建了一个自定义属性: —-button-size。这个自定义属性最初的作用域是使用 btn 类的所有按钮元素。然后，我将 btn-med 和 btn-lrg 类的按钮大小更改为 600px 以上。最后，我将这个自定义属性应用到一个位置的所有按钮元素。

### 不要太聪明过头
自定义属性的动态特性允许我们创建一些聪明而复杂的组件。

随着预处理器的推出，我们中的许多人可以使用 mixin 和自定义函数创建具有巧妙抽象的库。在有限的情况下，像这样的例子仍然有用，但大多数情况下，随着使用预处理器的时间越长，我使用的功能就越少。现在，我使用预处理器的场景几乎只在静态变量的部分。

自定义属性同样也会出现这样的状况，我期待看到更多聪明的例子。但从长远来看，可读和可维护的代码总会是更好的选择（至少在项目中是这样）。

我最近在 Free Code Camp Medium 上阅读了关于此主题的优秀文章。它是由 Bill Sourou r 撰写的，它叫‘<a href="https://medium.freecodecamp.org/dont-do-it-at-runtime-do-it-at-design-time-c4f59d1775e4"> Don’t Do It At Runtime. Do It At Design Time </a>’与其解释他的观点，我更推荐你先去看一下这篇文章。

预处理变量和自定义属性之间的一个关键区别是:自定义属性在运行时工作。这意味着，在复杂性方面是自定义属性是可以被接受的，因为预处理器没有自定义属性这么好的办法。

最近我常用来举例说明的一个例子是：
``` css
:root {
  --font-scale: 1.2;
  --font-size-1: calc(var(--font-scale) * var(--font-size-2));
  --font-size-2: calc(var(--font-scale) * var(--font-size-3)); 
  --font-size-3: calc(var(--font-scale) * var(--font-size-4));   
  --font-size-4: 1rem;     
}
```
这产生了一个比例组件。比例组件是一系列和比率相互关联的数字。它们经常用于网页设计和开发以设置字体大小或间距。

在本例中，每个自定义属性都是使用 calc() 确定的，方法是取之前的自定义属性的值并将其乘以比率。这样做，我们可以得到下一个数字。

这意味着在运行时计算比率，您可以通过只更新 -font-size 属性的值来更改它们。例如:
``` css
@media screen and (min-width: 800px) {
  :root {
    --font-scale: 1.33;
  }
}
```
如果你想改变比例，这比再次计算所有的值要聪明、简洁和快得多。这也是我在开发过程中不会做的事情。

虽然上面的例子对于原型设计很有用，但在开发中我更喜欢看到类似这样的东西：
``` css
:root {
  --font-size-1: 1.728rem;
  --font-size-2: 1.44rem;
  --font-size-3: 1.2em;
  --font-size-4: 1em;
}

@media screen and (min-width: 800px) {
  :root {
    --font-size-1: 2.369rem; 
    --font-size-2: 1.777rem;     
    --font-size-3: 1.333rem; 
    --font-size-4: 1rem;     
  }
}
```
与 Bill 的文章中的例子类似，我发现查看实际值的含义很有帮助。我们阅读代码的次数比我们编写的次数多得多，而且字体等全局参数在项目中很少变化。

上面的例子仍然不完美。它违反了以前的规则，即全局变量应该是静态的。我更愿意使用预处理器变量，并使用前面演示的例子将它们转换为局部动态自定义属性。

避免使用一个自定义属性到另一个自定义属性的情况也很重要。当我们命名这样的属性时，会发生这种情况。

### 更改值不是变量
更改值而不是变量是有效使用自定义属性的最重要准则之一。

作为一般规则，你不应该更改用于任何单一用途的自定义属性。这很容易做到，因为这正是我们如何处理预处理器的事情，但对于自定义属性来说没有多大意义。

在这个例子中，我们有两个在示例组件上使用的自定义属性。根据屏幕大小，我将从 --font-size-small 切换到 --font-size-large。

``` css
:root {
  --font-size-small: 1.2em;
  --font-size-large: 2em;            
}
.example {
  font-size: var(--font-size-small);
}
@media screen and (min-width: 800px) {
  .example {
    font-size: var(--font-size-large);
  }
}
```
做到这一点的更好方法是定义一个定制到组件的自定义属性。然后使用媒体查询或任何其他选择器更改其值。
``` css
.example {
  --example-font-size: 1.2em;
}
@media screen and (min-width: 800px) {                             
  .example {
    --example-font-size: 2em;            
  }
}
```
最后，在一个地方，我使用这个自定义属性的值：
``` css
.example {
  font-size: var(--example-font-size);
}
```
在此示例及其之前的其他示例中，媒体查询仅用于更改自定义属性的值。你可能还会注意到只有在 var() 的地方使用了该语句，并且更新了常规 CSS 属性。

变量声明和属性声明之间的这种分离是故意而为之。原因有很多，但是当考虑响应性设计时，好处是显而易见的。
### 用自定义属性创建响应布局
响应式有一个最大的问题是它太依赖于媒体查询了，这导致无论你如何组织 CSS 与特定组件相关的样式，在样式表中变得非常的零碎。

很难知道哪些 CSS 属性会发生改变。然而，CSS 自定义属性可以帮助我们组织一些与响应式设计相关的逻辑关系，降低媒体查询的难度。
### 如果它改变了，这是一个变量
使用媒体查询更改属性本质上是动态的，自定义属性提供了在 CSS 中表达动态值的方法。这意味着如果你试图通过媒体查询来修改 CSS 属性，则应该选择自定义属性。

然后，就可以将其连同所有媒体规则、hover或任何定义值的动态选择器一起移动到文档的顶部。
### 基于设计分离逻辑
在正确的情况下，逻辑与设计的分离意味着媒体查询只是用来改变自定义属性的值。这意味着与响应性设计相关的所有逻辑都应该位于文档的顶部，并且无论我们在 CSS 中看到什么 var() 语句，我们都会立即知道这个属性会发生变化。而使用传统的 CSS 编写方法，我们无法一眼就知道这一点。

我们中的许多人都非常擅长阅读和解释 CSS，同时在头脑中跟踪哪些属性在不同的情况下发生了变化。我受够了，我不想再这样了!自定义属性现在提供了逻辑和实现之间的连接，所以我们不需要跟踪它，在项目中真的非常有用！
###  逻辑折叠
在文档或函数顶部声明变量的想法并不是一个新想法。这是我们在大多数语言中所做的事情，现在我们也可以在 CSS 中完成。以这种方式编写 CSS 会在文档顶部和下面的  CSS 之间创建清晰的视觉区别。当我谈论他们时，我需要一种区分这些部分的方式，而“逻辑折叠”这个概念是我开始使用的一种隐喻。
折叠上方包含所有预处理器变量和自定义属性。这包括自定义属性可能具有的所有不同值。跟踪自定义属性如何更改应该很容易。

在这段代码之后的 CSS 可读性是很高，这很像在媒体查询之前的 CSS 代码和其它复杂的 CSS 样式。

让我们来看一看一个六列 Flexbox 网格系统的例子：
``` css
.row {
  --row-display: block;
}
@media screen and (min-width: 600px) {
  .row {
    --row-display: flex;
  }
}
```
该 --row-display 自定义属性的初始设置为 block。600px 以上的显示模式设置为 flex。

在折叠区域下面的代码可能看起来像这样：
``` css
.row {
  display: var(--row-display);
  flex-direction: row;
  flex-wrap: nowrap;
}
.col-1, .col-2, .col-3,
.col-4, .col-5, .col-6 {
  flex-grow: 0;
  flex-shrink: 0;
}
.col-1 { flex-basis: 16.66%; }
.col-2 { flex-basis: 33.33%; }
.col-3 { flex-basis: 50%; }
.col-4 { flex-basis: 66.66%; }
.col-5 { flex-basis: 83.33%; }
.col-6 { flex-basis: 100%; }
```
我们很直观的可以知道 --row-display 是一个变量。默认值是 block，所以应该忽略 flex 的值。

这个例子是非常简单的，但如果我们要拓展一个填充剩余空间的列的话，flex-grow、flex-shrink 和 flex-basis 值则需要转换为自定义属性。你在这里看一个更详细的例子<a href="https://codepen.io/MadeByMike/pen/f42ce1a954af9796ae76c62a9ea801f4/">点击查看</a>。

### 用于主题的自定义属性
我不推荐为全局动态变量创建自定义属性，也不推荐将自定义属性附加到 :root 选择器作用域下。但是每个规则都有一个例外，对于自定义属性，它应该是一个主题。

全局自定义属性的有限使用可以使整体变得更容易。

主题化通常涉及编译单独的样式表以覆盖用户首选项的默认值，或为每个用户编译不同的样式表。这两者都相对困难并且可能对性能有影响。
<img src="blob:https://maxiang.io/dfc91a90-8a30-4d3e-a99e-2b499836724a"/>

使用自定义属性，我们不需要编译不同的样式表; 我们只需要根据用户的喜好更新属性。由于它们是继承的关系，因此如果我们在根元素上执行此操作，则可以在我们的应用程序的任何位置使用它们。

### 资本化全局动态属性
自定义属性区分大小写，因为大多数自定义属性都是局部的，如果你使用的是全局动态属性，则可以将它们大写。
``` css
:root {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
```
变量的大写常常表示全局常量。对我们来说，这意味着该属性被设置在应用程序的其他地方，并且我们可能不应该在布局更改它。

### 避免直接设置全局动态属性
自定义属性接受回退值。避免直接覆盖全局自定义属性的值并保持用户值分离可能是有用的。我们可以使用回退值来执行此操作。

上面的例子将 --THEME-COLOR 的值设置为 --user-theme-color。如果 --user-theme-color 未设置，则将使用 #d33a2c。这样，我们无需在每次使用时提供  --THEME-COLOR。

你可能会在下面的例子中期望背景将被设置为green。但是，--user-theme-color 根元素的值尚未设置，所以值 --THEME-COLOR 没有改变。

``` css
:root {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
body {
  --user-theme-color: green;
  background: var(--THEME-COLOR);
}
```
像这样间接设置全局动态属性可以防止它们在局部被覆盖，并确保用户的设置始终都从根元素继承。这个约定能保护主题值被意外的继承。

如果我们想要将某些特定的属性暴露出去，我们可以用 :root 选择器替换 * 号选择器：
``` css
* {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
body {
  --user-theme-color: green;
  background: var(--THEME-COLOR);
}
```
现在 --THEME-COLOR 在每个元素中的值都会重新计算，因此可以使用 --user-theme-color 这个局部变量。换句话说，这个例子中的背景颜色是 green。

你可以在 “<a href="https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web#manipulating-color-with-custom-properties>使用自定义属性操纵颜色</a>”一节中看到此模式的更多详细示例。

### 使用 JavaScript 更新自定义属性
如果你想使用 JavaScript 设置自定义属性，那么就有一个相当简单的 API，它是这样的：
``` css
const elm = document.documentElement;
elm.style.setProperty('--USER-THEME-COLOR', 'tomato');
```
在这里，我设置了 --USER-THEME-COLOR 文档元素的值，换言之，:root 将被所有元素继承。

这不是一个新的 API, 它是用于更新元素样式的 JavaScript 方法。这些是内联样式，因此它们比普通 CSS 具有更高的特异性。

这意味着应用局部定制很容易：
``` css
.note {
  --note-color: #eaeaea;
}
.note {
  background: var(--note-color);
}
```
在这里，我设置了默认值 --note-color 并将其范围设置为 .note 组件。我将变量声明与属性声明分开，即使在这个简单的例子中。
``` css
const elm = document.querySelector('#note-uid');
elm.style.setProperty('--note-color', 'yellow');
```
然后，我定位一个 .note 元素的特定实例，并仅更改该元素的 --note-color 自定义属性的值。这将比默认值具有更高的特异性。

你可以看到这个例子“<a href="https://codepen.io/MadeByMike/pen/WzbGdE?editors=0110">使用 React </a>”如何工作。这些用户首选项可以保存在布局存储中，或者也可以保存在数据库中，也可以保存在更大的应用程序中。

### 使用自定义属性操作颜色
除了十六进制值和已命名的颜色之外，CSS 还具有诸如 rgb() 和的 颜色功能 hsl()。这些允许我们为组件设置独立的颜色，例如色调或亮度。自定义属性可以与颜色函数结合使用。
``` css
:root {
  --hue: 25;
}
body {
  background: hsl(var(--hue), 80%, 50%);
}
```
这很有用，但预处理器中最广泛使用的一些功能是高级颜色功能，它允许我们使用亮化、变暗或去饱和等功能来操作颜色：
``` css
darken($base-color, 10%);
lighten($base-color, 10%);
desaturate($base-color, 20%);
```
在浏览器中使用这些功能会很有用。“<a href="https://www.w3.org/TR/css-color-4/#funcdef-color-mod">他们即将到来</a>”，但直到我们在 CSS 中拥有原生色彩修改功能之后，自定义属性才能填补这一空白。

我们已经看到，自定义属性可以在现有的颜色函数中使用 rgb()、hsl() 但也可以在其中使用 calc()。这意味着我们可以通过乘以实数将其转换为百分比，例如calc(50 * 1%) = 50%。
``` css
:root {
  --lightness: 50;
}
body {
  background: hsl(25, 80%, calc(var(--lightness) * 1%));
}
```
我们希望将亮度值存储为实数的原因是，我们可以 calc 在将其转换为百分比之前对其进行处理。例如，如果我想让颜色变暗 20%，我可以乘以它的亮度 0.8。我们可以通过将亮度计算分离为布局范围的自定义属性来使其更容易阅读：
``` css
:root {
  --lightness: 50;
}
body {
  --lightness: calc(var(--lightness * 0.8));
  background: hsl(25, 80%, calc(var(--lightness) * 1%));
}
```
我们甚至可以抽象出更多的计算结果，并“<a href="https://codepen.io/MadeByMike/pen/YLQWeb">基于 CSS 定义属性的颜色修改功能</a>”。这个例子对于大多数主题的实际情况来说可能太复杂，但它展示了动态自定义属性的全部功能。

### 简化主题
使用自定义属性的优点之一是能够简化主题。应用程序不需要知道如何使用自定义属性。相反，我们使用 JavaScript 或服务器端代码来设置自定义属性的值。这些值的使用方式由样式表决定。

这意味着我们可以再次将逻辑与设计分开。如果你有技术设计团队，作者可以更新样式表并决定如何应用自定义属性，而无需更改一行 JavaScript 或后端代码。

自定义属性还允许将一些复杂的主题移动到 CSS 中，并且这种复杂性可能会对 CSS 的可维护性产生负面影响，所以请记住尽可能保持简单。


### 从今天开始使用自定义属性吧！
即使您支持 IE10 和 11，你也可以在今天开始使用自定义属性。本文中的大多数示例都与如何编写和构造 CSS 有关。但是，从可维护性的角度来说，这些好处是非常重要的，大多数示例只减少了使用更复杂的代码所能做的事情。

我使用一个名为‘<a href="https://github.com/MadLittleMods/postcss-css-variables">postcss-css-variables</a>’的工具将自定义属性的大部分特性转换为相同代码的静态表示。其他类似的工具忽略媒体查询或复杂选择器中的自定义属性，将自定义属性处理得更像预处理变量。

这些工具不能模仿自定义属性的运行时特性。这意味着没有动态特性，比如使用 JavaScript 进行主题化或更改属性。这在很多情况下都是可以的。根据不同的情况，UI  定制可能被认为是一种渐进式的增强，默认的主题对于旧的浏览器来说是完全可以接受的。


### 加载正确的样式表

有很多方法可以使用 postCSS。我使用 gulp 过程为更新和旧的浏览器编译单独的样式表。我的 gulp 任务的简化版本如下:
``` javascript
import gulp from "gulp";
import sass from "gulp-sass";
import postcss from "gulp-postcss";
import rename from "gulp-rename";
import cssvariables from "postcss-css-variables";
import autoprefixer from "autoprefixer";
import cssnano from "cssnano";

gulp.task("css-no-vars", () =>
  gulp
    .src("./src/css/*.scss")
    .pipe(sass().on("error", sass.logError))
    .pipe(postcss([cssvariables(), cssnano()]))
    .pipe(rename({ extname: ".no-vars.css" }))
    .pipe(gulp.dest("./dist/css"))
);

gulp.task("css", () =>
  gulp
    .src("./src/css/*.scss")
    .pipe(sass().on("error", sass.logError))
    .pipe(postcss([cssnano()]))
    .pipe(rename({ extname: ".css" }))
    .pipe(gulp.dest("./dist/css"))
);
```
这将产生两个 CSS 文件:一个常规文件具有自定义属性(styles.css)，另一个用于旧浏览器(styles.no . var . CSS)。我想要 IE10 和 11 的样式。CSS 和其他浏览器获取常规的 CSS 文件。

通常，我主张使用功能查询，但“<a href="https://caniuse.com/#search=%40supports"> IE11 不支持功能查询</a>”，然而我们已经大量使用自定义属性，在这种情况下，提供不同样式表是有意义的。

智能地服务于不同的样式表和避免一闪而过的无样式的内容不是一项简单的任务。如果不需要定制属性的动态特性，可以考虑使用所有的浏览器样式。CSS 和使用自定义属性作为开发工具。

如果你想充分利用自定义属性的所有动态特性，我建议使用“<a href="https://www.smashingmagazine.com/2015/08/understanding-critical-css/">关键的 CSS 技术</a>”。按照这些技术，主样式表是异步加载的，而关键的 CSS 则是内联的。你的页眉可能看起来像这样:
``` html
<head>
  <style> /* inlined critical CSS */ </style>
  <script> loadCSS('non-critical.css'); </script>
</head>
```
我们可以扩展它来加载 styles.css 或者 styles.no-vars.css 取决于浏览器是否支持自定义属性。我们这样检测：
``` javascript
if ( window.CSS && CSS.supports('color', 'var(--test)') ) {
  loadCSS('styles.css');
} else {
  loadCSS('styles.no-vars.css');
}
```
### 结论
如果你一直在努力有效地组织 CSS，使用响应式组件很困难，想要实现客户端主题，或者只想顺利地使用自定义属性，那么本指南应该可以告诉你所需要知道的一切了。
这总结为了解 CSS 中动态变量和静态变量之间的区别以及一些简单的规则:

-  单独的逻辑与设计;<br/>
-  如果 CSS 属性发生变化，考虑使用自定义属性;<br/>
-  更改自定义属性的值，而不是使用哪个自定义属性;<br/>
-  全局变量通常是静态的。<br/>

如果遵守以上这些条例，你会发现使用自定义属性比你想象的要容易得多，甚至可能会改变你对 CSS 的处理方式。

#### 进一步阅读
“ <a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">是时候使用自定义属性了</a> ”，Serg Hospodarets自定义属性的语法和特性的一般介绍。<br/>
“ <a href="https://csswizardry.com/2016/10/pragmatic-practical-progressive-theming-with-custom-properties/">渐进式实用的CSS自定义属性 </a> ”，Harry Roberts 关于主题的更多有用信息。<br/>
“<a href="https://codepen.io/collection/naJLrB/">自定义属性集合</a>”，CodePen上的Mike Riethmuller 
你可以尝试多个不同的示例。
