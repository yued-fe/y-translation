> * 原文地址：https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web

> * 译文地址：https://github.com/yued-fe/y-translation/blob/master/todo/A%20Strategy%20Guide%20To%20CSS%20Custom%20Properties.md

> * 译者：[popeyesailorman](https://github.com/popeyesailorman)

> * 校对者：[ziven27](https://github.com/ziven27) [djmaxwow](https://github.com/djmaxwow)


# CSS 自定义属性的策略指南

**关于作者** Mike 是来自澳大利亚的独立网站开发人员，曾在澳大利亚的一些大型网站和一些小型社区工作过...... <a href="https://www.smashingmagazine.com/author/michaelriethmuller/">关于Michael的更多信息...</a>

> CSS 自定义属性(也称为“CSS 变量”)，在目前所有的现代浏览器中都得到了支持，开发者已经在项目中开始使用，但是它们与预处理器中的变量不同，虽然我已经看到过很多例子，却没有搞清楚他们真正的优势在哪里。

自定义属性有很大的潜力可以改变我们编写和组织 CSS 的方式，并且在一定程度上改变 JavaScript 与 UI 组件的调用方式。我并不关心语法和它们的工作方式（为此，我建议你阅读“<a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">现在是时候开始使用自定义属性了</a>”这篇文章）。同时我想更深入地研究如何充分利用 CSS 自定义属性。

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

另一个明显的区别在于名称。它们之所以被称为"自定义属性"，是因为它们真的是 CSS 属性。在预处理器中，你可以在任何位置声明和使用变量，包括外部声明块，在媒体查询中，甚至在选择器中也可以使用，例如：
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

抛开样式差异，预处理器中的变量和自定义属性中的变量之间最大的差别是作用范围。我们可以将变量根据作用域分为静态和动态两个部分。预处理器中的变量是静态的，而自定义属性是动态的。

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
一旦编译成了 CSS，这个变量就会消失。这意味着我们在读取一个 .scss 文件并输出的时候并不需要关心 HTML、浏览器或其它输入，而自定义属性不是这样的。
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

这听起来很棒，因为你可以在媒体查询中，通过使用类似 hover 的伪类选择器甚至是 JavaScript 改变自定义属性的值。
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

我们不需要在自定义属性使用的地方去修改它，我们可以通过 CSS 去修改它的值。这意味着同一个自定义属性，可以在不同的地方，或者是上下文中有不同的值。

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

我的 SCSS 可能看起来像这样：
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

因为大多数静态变量都是全局的，所以我喜欢区分只在组件内部使用的静态变量。为此，可以在这些变量前面加上组件名，或者可以使用另一个前缀，如组件的 c 变量名或 l 变量名。您可以使用任何您想要的前缀，或者可以在全局变量前面加上前缀。无论您选择什么，区分都是很有帮助的，特别是当转换一个现有的代码来使用自定义属性时。

### 何时使用自定义属性
如果可以在组件内部使用静态变量，那么什么时候应该使用自定义属性呢? 将现有的预处理器变量转换为自定义属性通常没什么意义。毕竟，自定义属性的原因是完全不同的。当我们有 CSS 属性时，自定义属性是有意义的，尤其是在 DOM 中(尤其是动态条件)，例如 :fouces、hover、媒体查询或 JavaScript。

我猜想我们将始终使用某种形式的静态变量，尽管我们将来可能需要更少的静态变量，因为自定义属性提供了组织逻辑和代码的新方法。在此之前，我认为在大多数情况下，我们可以将预处理器变量和自定义属性组合使用。

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
这里，我创建了一个自定义属性: --button-size。这个自定义属性最初的作用域是使用 btn 类的所有按钮元素。然后，我将 btn-med 和 btn-lrg 类的按钮大小更改为 600px 以上。最后，我将这个自定义属性应用到一个位置的所有按钮元素。

### 不要太聪明过头
自定义属性的动态特性允许我们创建一些聪明而复杂的组件。

随着预处理器的推出，我们中的许多人可以使用 mixin 和自定义函数创建具有巧妙抽象的库。在有限的情况下，像这样的例子仍然有用，但大多数情况下，随着使用预处理器的时间越长，我使用的功能就越少。现在，我使用预处理器的场景几乎只在静态变量的部分。

自定义属性同样也会出现这样的状况，我期待看到更多聪明的例子。但从长远来看，可读和可维护的代码总会是更好的选择（至少在项目中是这样）。

我最近在 Free Code Camp Medium 上阅读了关于此主题的优秀文章。它是由 Bill Sourour 撰写的，叫‘<a href="https://medium.freecodecamp.org/dont-do-it-at-runtime-do-it-at-design-time-c4f59d1775e4"> Don’t Do It At Runtime. Do It At Design Time </a>’。与其解释他的观点，我更推荐你先去看一下这篇文章。

预处理变量和自定义属性之间的一个关键区别是：自定义属性在运行时工作。这意味着，在复杂性方面自定义属性是可以被接受的，因为预处理器没有自定义属性这么好的办法。

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
这产生了一个比例组件。比例组件是一系列和比率相互关联的数字。它们经常用于网页设计和开发来设置字体大小或间距。

在本例中，每个自定义属性都是使用 calc() 确定的，方法是取之前的自定义属性的值并将其乘以比率。这样做，我们可以得到下一个数字。

这意味着在运行时计算比率，您可以通过只更新 --font-size 属性的值来更改它们。例如:
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

### 修改属性值而不是变量
修改属性值而不是变量是使用自定义属性的最重要准则之一。

通常来说，不推荐修改仅有单一目的的自定义属性。虽然这很容易实现，因为这正是预处理器应该做的事情，但对于自定义属性来说这没有多大意义。

在这个例子中，示例组件上我们创建了两个使用的自定义属性。根据屏幕的尺寸变化，将自定义属性 --font-size-small 修改为 --font-size-large。

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
更好的方式是在这个组建当中只定义一个自定义属性，然后通过媒体查询或者其它的选择器改变这个属性的值。
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
最后，只通过调用这个自定义属性来获取你想要的属性值。
``` css
.example {
  font-size: var(--example-font-size);
}
```
在这个和之前的例子的例子中，我们都仅通过使用媒体查询去修改自定义属性的值。也只在一个地方通过 var（）去声明这个自定义属性，然后 CSS 的值就通过这个自定义属性自动更新了。

我们故意分离了值声明和属性声明，这样做的原因其实有很多，响应性设计就是一个最明显的例子。
### 使用自定义属性创建响应布局
响应式布局中最大的难点在于它太过于依赖媒体查询，这导致无论你多么用心的管理你的 CSS，和组件相关的样式都会变得很零散。

你很难知道哪些 CSS 属性会发生改变。CSS 自定义属性，正好可以帮助我们处理与响应式设计相关的这个逻辑关系，从而降低媒体查询的使用难度。
### 如果它改变了，这仅仅只是一个变量而已
媒体查询中属性的变化本身就是动态的，自定义属性正好弥补了 CSS 属性不具有动态性的特点。这意味着如果你想通过媒体查询来修改 CSS 属性，自定义属性是一个不错的选择。

然后，你可以将媒体查询规则、hover 的状态或者任何定义了属性值修改方式的动态选择器，都移动到文档的顶部。
### 将逻辑和设计分离
如果你的操作正确，那么逻辑与设计的分离意味着媒体查询只是用来改变自定义属性的值。而这说明与响应性设计相关的所有逻辑都应该放到于文档的顶部，并且无论我们在哪里看到 var() 声明语句，我们都能很明显的知道这个属性会发生变化。而使用传统的 CSS 方式，我们是无法觉察这一点的。

我们中的大多数人都非常擅长阅读和理解 CSS，我们需要思考在不同的状态下哪些属性发生了变化。我受够了这样的方式了。现在自定义属性帮我们把逻辑和实现链接了起来，所以我们不需要在大脑去跟踪这些变化，这在项目中真的非常有用！
###  折叠逻辑
在文档或函数顶部声明变量的想法是很早就有的方式。也是大多数语言中推荐的做法，现在我们也可以在 CSS 中完成。以这种方式编写 CSS，光从视觉角度就很容易区分顶部和之后的代码。打我要使用它们是我能很方便的找到它们。而这就是我想说的“折叠逻辑”这个概念。
在这个折叠的上方包含所有预处理器变量和自定义属性。这包含了所有的参数和自定义属性可能的变化。这样我们就很容易的知道自定义属性发生了哪些变化。

在这段折叠之后的 CSS 代码可读性也是很高的，这就和你原来写媒体查询，和其它必要的代码一样没有什么区别。

我们再来看一个关于六列 Flexbox 布局的网格系统非常简单的例子：
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
这个 --row-display 自定义属性的初始值设置为 block。当屏幕超过 600px 之后这个值会被重置为 flex。

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
我们可以很直观的知道 --row-display 是一个变量。并且它的值现在应该是 block，而不是 flex。

这是一个简单的例子，但如果我们要拓展一个填充剩余空间的列的话，flex-grow、flex-shrink 和 flex-basis 值则同样需要转换为自定义属性。你可以尝试写一下，或者点击这里看一下<a href="https://codepen.io/MadeByMike/pen/f42ce1a954af9796ae76c62a9ea801f4/">更详细的例子</a>。

### 基于主题创建自定义属性
我不推荐使用自定义属性去创建全局动态变量，也不推荐将自定义属性附加到 :root 选择器作用域下。但是每个规则都有一个例外，对于自定义属性而言，在创建主题的场景下，这就是一个例外。

有节制的使用全局自定义属性可以使主题的创建更容易。

主题化通常指的是让用户以某种方式定制UI。这可能类似于在配置文件上修改颜色。更简单的说，就像你在 Google Keep 这个应用程序中为你的笔记选择了一个颜色一样。
![](https://qidian.qpic.cn/qidian_common/349573/f09b3fb6e4d79f7ef79ecc22a2ca70b4/0)

主题化通常会有独立的样式表用与基于用户的选择来覆盖之前的样式，或者对于不同的样式有完全独立的样式表文件。这两种方法实现都比较困难，并且会对性能产生影响。

### 用全大写的方式表示全局动态属性
自定义属性是大小写敏感的，自定义属性建议是都是局部的，如果你需要使用到是全局动态属性，推荐使用全大写的方式。
``` css
:root {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
```
全大写的变量常常用于表示全局常量。对我们来说，一看到全大写的变量，则这意味着该属性是全局的，我们不应该在局部去修改它。

### 避免直接设置全局动态属性
自定义属性接受一个备选值。我们应该避免直接覆盖全局自定义属性的值并尽量与其它值分离。我们可以使用备选值来实现这一点。

上个例子我们有将 --THEME-COLOR 的值设置为 --user-theme-color。如果 --user-theme-color 未设置，则将使用备选值 #d33a2c。这样，我们无需在每次使用 --THEME-COLOR 时都提供一个备选值。

在下面的例子中你可能希望的是背景被设置为 green。但是在 ：root 作用域下的 --user-theme-color 的值并没有设置，所以最后 --THEME-COOR 值不会发生变化。

``` css
:root {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
body {
  --user-theme-color: green;
  background: var(--THEME-COLOR);
}
```
像这样间接设置全局动态属性可以防止它们在局部被覆盖，并确保用户的设置始终都从根元素继承。这个约定能避免主题参数被意外的继承。

如果我们想要将某些特定的属性暴露出去，我们可以用 * 号选择器替换 :root 选择器：
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

你可以在 “<a href="https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web#manipulating-color-with-custom-properties">使用自定义属性操纵颜色</a>”一节中看到更多详细示例。

### 使用 JavaScript 更新自定义属性
如果你想通过 JavaScript 设置自定义属性，这里有个相当简单的 API：
``` css
const elm = document.documentElement;
elm.style.setProperty('--USER-THEME-COLOR', 'tomato');
```
在这里，我设置了 --USER-THEME-COLOR 元素的值，换言之，:root 将被所有元素继承。

这并不是一个新的 API, 它仅仅只是用于更新元素样式的 JavaScript 方法。这些是内联样式，因此它们比普通 CSS 具有更高的权重。

这让局部自定义边得很容易：
``` css
.note {
  --note-color: #eaeaea;
}
.note {
  background: var(--note-color);
}
```
在这里，我设置了默认值 --note-color 并将其作用域限制在了 .note 组件下。即使在这个简单的例子中，我也将变量声明与属性声明分开。
``` css
const elm = document.querySelector('#note-uid');
elm.style.setProperty('--note-color', 'yellow');
```
然后，我定位了一个 .note 元素的实例，并仅更改该实例自定义属性 --note-color 的值。这将比默认值具有更高的权重。

你可以看<a href="https://codepen.io/MadeByMike/pen/WzbGdE?editors=0110">使用 React 的例子</a>来了解它是如何工作的。这些用户首选项可以保存在本地存储中，在更大应用中也可以保存到数据库里。

### 使用自定义属性操作颜色
除了十六进制值和已命名的颜色之外，CSS 还具有诸如 rgb() 和 hsl() 这样的颜色方法可以使用。这些允许我们为组件设置例如色调或亮度等特定的颜色属性。自定义属性可以与这些方法结合使用。
``` css
:root {
  --hue: 25;
}
body {
  background: hsl(var(--hue), 80%, 50%);
}
```
这很有用，而预处理器中提供了更多更高级的颜色方法，我们可以通过这些方法去实现颜色的亮化、变暗或去饱和等功能：
``` css
darken($base-color, 10%);
lighten($base-color, 10%);
desaturate($base-color, 20%);
```
如果浏览器自身就提供了这些方法那就更好了。“<a href="https://www.w3.org/TR/css-color-4/#funcdef-color-mod">他们即将到来</a>”，但在 CSS 原生支持拥有这些功能之前，自定义属性刚好能填补这个空缺。

我们已经看到，自定义属性不仅可以在比如 rgb()、hsl() 等颜色方法中使用，也可以在 calc() 中使用。这意味着我们可以通过乘法把一个数转换成百分比的形式，例如 calc(50 * 1%) = 50%。
``` css
:root {
  --lightness: 50;
}
body {
  background: hsl(25, 80%, calc(var(--lightness) * 1%));
}
```
我们将亮度值存储为整数的原因是：在将其转化为百分比之前可以使用 calc 方法来进行转换。举个例子，如果我想一个颜色变暗 20%，我可以将其亮度值乘以它 0.8 实现。通过自定义属性，将亮度的计算方法限制在局部作用域下，可以使我们的代码可读性更强：
``` css
:root {
  --lightness: 50;
}
body {
  --lightness: calc(var(--lightness * 0.8));
  background: hsl(25, 80%, calc(var(--lightness) * 1%));
}
```
我们甚至可以抽象出更多的计算方法，并创建类似“<a href="https://codepen.io/MadeByMike/pen/YLQWeb">基于 CSS 自定义属性的颜色拓展功能</a>”中提到的方法一样。这个例子对于大多数主题的实际情况来说可能相对复杂，但是它充分展示了动态自定义属性的能力。

### 简化主题
使用自定义属性的优点之一是能够让主题的创建更加的简单。应用程序不需要知道自定义属性是如何使用的。相反，我们通过 JavaScript 或服务器端代码来设置自定义属性的值，而这些值又直接由样式表控制。

这意味着我们进一步将逻辑与设计分离。如果你有一个专业的设计团队，设计师可以通过更新样式表来决定如何应用自定义属性，而无需更改一行 JavaScript 或后端代码。

自定义属性，将主题的复杂度移动到了 CSS 中，这种复杂性可能会对 CSS 的维护性产生负面影响，所以请可能保持主题的简单。


### 从现在就开始使用自定义属性
即使你的项目依然需要支持 IE10 和 IE11，你也可以使用自定义属性。本文中的大多数示例都与如何编写和构建 CSS 有关。但是，从可维护性的角度来说，这些好处是非常重要的，本文大多数示例只是减少了原本需要使用更复杂的代码才能实现的事情。

我使用一个名为 <a href="https://github.com/MadLittleMods/postcss-css-variables">postcss-css-variables</a> 的工具将自定义属性的大部分方法转换为相同功能的静态方法。而其他类似的工具忽略了媒体查询或复杂选择器中的自定义属性，将其当作预处理变量来处理。

这些工具不能做到的是模拟出自定义属性的实时特性。而这意味着失去了之前我们在主题和通过 JavaScript 更改属性中提到的动态特性。这在很多情况下也是可以接受的。毕竟基于不同的情况，自定义UI也不失为一种渐进式增强的方式，而默认的主题对于旧的浏览器来说也是完全可以接受的。


### 加载正确的样式表

使用 postCSS 的有很多。我通过使用 Gulp 的进程来来区分新旧浏览器的样式表。一个我创建的 Gulp 任务如下：
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
这个结果将输出两个 CSS 文件：一个具有自定义属性的(styles.css)文件，另一个用于旧浏览器(styles.no-vars.css)。我希望在 IE10 和 IE11 浏览器中使用 styles.no-vars.css 这个文件，其它浏览使用常规的 CSS 文件。

通常，我主张使用功能查询，但“<a href="https://caniuse.com/#search=%40supports"> IE11 不支持功能查询</a>”，可是我们已经大量使用了自定义属性，在这种情况下，为其提供不同样式表就显得有意义了。

通过提供不同的样式表以避免无样式内容的闪烁并不是一个简单的事情。如果不需要自定义属性中的动态特性，可以考虑仅使用适用于所有浏览器的 styles.no-vars.css 文件，而自定义属性仅作为开发工具。

如果你想充分利用自定义属性的所有动态特性，我建议使用“<a href="https://www.smashingmagazine.com/2015/08/understanding-critical-css/">关键的 CSS 技术</a>”。基于这些技术，主样式表是异步加载的，而关键的 CSS 是内联的。你的页面的 header 可能看起来像这样：
``` html
<head>
  <style> /* inlined critical CSS */ </style>
  <script> loadCSS('non-critical.css'); </script>
</head>
```
我们可以扩展这个方法，来实现基于浏览器是否支持支持自定义属性来决定加载 styles.css 文件还是 styles.no-vars.css 文件的效果。我们可以这样来实现：
``` javascript
if ( window.CSS && CSS.supports('color', 'var(--test)') ) {
  loadCSS('styles.css');
} else {
  loadCSS('styles.no-vars.css');
}
```
### 结论
如果你想要更有效地管理 CSS、在使用响应式的功能上遇到困难、想要实现类似客户端主题的效果或者只是想尝试一下自定义属性，这片文章应该可以帮你解答这些问题了。这不仅解释了 CSS 中动态变量和静态变量之间的区别，还有如下的其它的几条规则：

-  将逻辑从设计中分离;<br/>
-  如果 CSS 属性需要发生变化，可以考虑使用自定义属性;<br/>
-  仅自定义属性的值，而不是修改自定义属性本身;<br/>
-  全局变量通常是静态的。<br/>

如果理解了以上这些条例，你会发现使用自定义属性比你想象的要容易得多，甚至可能会改变你对 CSS 的处理方式。

#### 进一步阅读
“ <a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">是时候使用自定义属性了</a> ”，Serg Hospodarets 介绍的自定义属性的语法和特性的。<br/>
“ <a href="https://csswizardry.com/2016/10/pragmatic-practical-progressive-theming-with-custom-properties/">渐进式实用的CSS自定义属性 </a> ”，Harry Roberts 介绍的更多关于主题的有用信息。<br/>
“<a href="https://codepen.io/collection/naJLrB/">自定义属性集合</a>”，Mike Riethmuller 在 CodePen 上提供的大量的不同的示例。
