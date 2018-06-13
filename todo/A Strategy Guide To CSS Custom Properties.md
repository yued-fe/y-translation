> * 原文地址：https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web

> * 译文地址：

> * 译者：

> * 校对者：


# CSS 定制属性的策略指南

**关于作者** Mike是来自澳大利亚的独立网站开发人员，曾在澳大利亚的一些最大的网站以及一些最小的社区工作...... <a href="https://www.smashingmagazine.com/author/michaelriethmuller/">关于Michael的更多信息...</a>

> 所有现代浏览器现在都支持CSS自定义属性（也称“CSS变量”），而且有人已经在项目中使用它们。这很棒，但它们与预处理器中的变量是不同，我看到很多人使用，却没有搞清楚它们真正的优势在哪里的例子。

自定义属性有很大的潜力可以改变我们编写和组织 CSS 的方式，并且在一定程度上改变JavaScript与UI组件的调用方式。我不会专注于语法和它们的工作方式（为此，我建议你阅读“<a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">现在是时候开始使用自定义属性</a>”）。相反，我想更深入地研究如何充分利用CSS自定义属性。

### 它们与预处理器中变量的相似之处？
自定义属性有点像预处理器中的变量，但还是有很大的差别。最重要也是最明显的区别是在于语法。

在SCSS中我们用美元符号来定义变量：

``` scss
$smashing-red: #d33a2c;
```
在 Less 中我们用@符号：
``` scss
smashing-red: #d33a2c;
```
自定义属性遵循类似的约定并使用--前缀的方式：
``` css
:root { --smashing-red: #d33a2c; }
.smashing-text { 
  color: var(--smashing-red);
}
```
自定义属性和预处理器中的变量最大的不同在于键值对的语法规则。自定义属性采用var()函数去取值。

另一个最明显的区别在于名称。它们之所以被称为'自定义属性'，是因为它们真的是 CSS 属性。在预处理器中，你可以在任何位置声明和使用变量，包括外部声明块，在 Media 中，甚至在选择器中也可以使用。
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
以上的大多数自定义属性示例都是无效的

自定义属性和常规CSS属性的用法是一样的。把它们当作动态属性会比变量要好。这意味着它们只能在声明块中使用，换句话说，自定义属性和选择器是强绑定的。这可以是:root选择器或任何其它有效的选择器。
``` css
:root { --smashing-red: #d33a2c; }

@media screen and (min-width: 800px) {
  .smashing-text, .cats {
    --margin-left:  1em;
  }
}
```
你可以在属性声明中的任何地方获取声明的值，这个意味着它们可以作为单个值使用，作为速记语句的一部分，甚至是在calc()方程式中使用。
``` css
.smashing-text, .cats {
  color: var(--smashing-red);
  margin: 0 var(--margin-horizontal);
  padding: calc(var(--margin-horizontal) / 2)
}
```
但是，它们不能用于media查询或选择器，包括:nth-child()。

关于语法和自定义属性的工作原理可能还有很多，比如如何使用fallback值，以及是否可以将变量分配给其他变量（是），本文介绍的基础知识应该已经足以让大家理解其中的概念。更多关于自定义属性工作方式的详细信息，可以阅读由Serg Hospodarets编写的“<a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/"> 现在开始使用自定义属性 </a>”。

### 动态与静态

抛开样式差异，预处理器和自定义属性中变量之间最大的差别是它们的作用范围。我们可以将变量根据作用域分为静态和动态两个部分。预处理器中的变量是静态的，而自定义属性是动态的。

在CSS中，静态意味着你可以在编译过程中的不同点更新变量的值，但是这不能改变它之前的值

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
一旦编译了这个CSS，这个变量就消失了。而这意味着我们在读取一个.scss文件并输出的时候，不需要关心，HTML浏览器或其它的输入。这些在自定义属性中都不是问题。

预处理器确实有一种“块级作用域”，其中变量可以在选择器，函数或mixin中临时更改。这改变了块内变量的值，但它仍然是静态的。这与块有关，而不是选择器。在下面的例子中，变量$background在.example类内部被改变。即使我们使用相同的选择器，它也会变回块级作用域之外的初始值。

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
自定义属性不同的工作。在涉及自定义属性的地方，动态范围意味着它们受到继承和级联的影响。属性与选择器绑定，如果值发生变化，就会像其他CSS属性一样影响所有匹配的DOM元素。

这听起来很棒，因为你可以在媒体查询中，通过使用类似hover的伪类选择器甚至是javascript改变自定义属性。
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

我们不需要在自定义属性的使用的地方去修改它 - 我们可以通过CSS去修改它的值。这意味着同一个自定义属性，可以在不同的地方，或者是上下文中有不同的值。

### 全局与局部

除了静态或动态之外，变量还可以是全局的或局部的。如果你写JavaScript，你可能会更了解这一点。变量既可以作用在应用程序全局环境中，也可以将其作用范围限制在特定的功能或代码块中。

CSS也是一样的。有作用于全局，有些作用于局部性。一些重要的颜色，垂直间距，排版方式，这些可能你会希望在端和网页中可以全局调用。当然也有一些局部的东西，比如，按钮组件可能具有小型和大型尺寸。你不希望这些按钮的大小适用于所有输入元素或页面上的每个元素。

这是我们在CSS中熟悉的东西。我们开发了设计系统，命名规范和JavaScript库，这些可以分离局部组件和全局组件。自定义属性给这类问题提供了新的思路。

通常 CSS 自定义属性的范围局限于我们指定的选择器中。这看起来有点像局部变量。但是，自定义属性具有继承的特性，所以在很多情况下，它们表现的更像全局变量 - 特别是在应用于:root选择器的时候。这意味着我们需要考虑如何使用它们。

大量的事例都被作用在了:root元素上，对于Demo这还说得过去，但它可能污染全局作用域，从而导致意外的继承问题。幸运的是，我们已经学到了这些教训。

### 全局变量趋于静态
可能会有一些例外，但通常来说，CSS中的大多数的全局事物也是静态的。

诸如商标颜色、字体和间距之类的大变量不会在不同的组件之间产生太大的变化。当它们发生变化时，这往往是一个全球性的品牌重塑，或者是在一个成熟产品上很少发生的其他重大变化。对于这些变量来说，它们仍然是有意义的，它们在很多地方被使用，而变量有助于保持一致性。但它们是动态的是没有意义的。这些变量的值不会以任何动态的方式变化。

因此，我强烈建议对全局(静态)变量使用预处理器。这不仅确保了它们始终是静态的，而且还可以在代码中直观地表示它们。这可以使CSS更易于阅读和维护。

### 局部静态变量是不错的（有时候）
您可能会认为，考虑到全局变量是静态的，那么通过反射，所有的局部变量都可能需要是动态的。虽然局部变量确实是动态的，但这远不如全局变量是静态的。

局部静态变量在很多情况下都是可以的。我在组件文件中使用预处理器变量，主要是作为开发人员的方便。

![](https://qidian.qpic.cn/qidian_common/349573/27d2d95901c158e29a5fc4dc7ada27d2/0)

我的scss可能看起来像这样：
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
显然，如果我多次使用变量，或者从size变量中派生空白和填充值，这个示例将更有意义。然而，快速原型化不同尺寸的能力可能是一个充分的理由。

因为大多数静态变量都是全局的，所以我喜欢区分只在组件内部使用的静态变量。为此，可以在这些变量前面加上组件名，或者可以使用另一个前缀，如组件的c变量名或本地的l变量名。您可以使用任何您想要的前缀，或者可以在全局变量前面加上前缀。无论您选择什么，区分都是很有帮助的，特别是当转换一个现有的代码基来使用自定义属性时。

### 何时使用自定义属性
如果可以在组件内部使用静态变量，那么什么时候应该使用自定义属性呢?将现有的预处理器变量转换为自定义属性通常没什么意义。毕竟，自定义属性的原因是完全不同的。当我们有CSS属性时，自定义属性是有意义的，尤其是在DOM中(尤其是动态条件)，例如:焦点、悬停、媒体查询或JavaScript。

我怀疑我们将始终使用某种形式的静态变量，尽管我们将来可能需要更少的静态变量，因为自定义属性提供了组织逻辑和代码的新方法。在此之前，我认为在大多数情况下，我们将使用预处理器变量和自定义属性的组合。

知道我们可以为自定义属性分配静态变量是有帮助的。无论它们是全局的还是局部的，在许多情况下，将静态变量转换为局部动态自定义属性都是有意义的。

>注意:您知道$var是自定义属性的有效值吗?Sass的最新版本认识到了这一点，因此我们需要插入分配给自定义属性的变量，如:#{$var}。这告诉Sass您希望输出变量的值，而不是样式表中的$var。这只适用于自定义属性等情况，其中变量名也可以是有效的CSS。

如果我们以上面的按钮示例为例，决定所有的按钮都应该使用移动设备上的小变化，而不考虑HTML中应用的类，这是一种更动态的情况。为此，我们应该使用自定义属性。

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
这里，我创建了一个自定义属性:—button-size。这个自定义属性最初的作用域是使用btn类的所有按钮元素。然后，我将btn-med和btn-lrg类的按钮大小更改为600px以上。最后，我将这个自定义属性应用到一个位置的所有按钮元素。

### 不要太聪明
自定义属性的动态特性允许我们创建一些聪明而复杂的组件。

随着预处理器的推出，我们中的许多人可以使用mixin和自定义函数创建具有巧妙抽象的库。在有限的情况下，像这样的例子仍然有用，但大多数情况下，随着使用预处理器的时间越长，我使用的功能就越少。现在，我使用预处理器的场景几乎只在静态变量的部分。

自定义属性同样也会出现这样的状况，我期待看到更多聪明的例子。但从长远来看，可读和可维护的代码总是会胜过聪明的抽象（至少在项目中是这样）。

我最近在Free Code Camp Medium上阅读了关于此主题的优秀文章。它是由Bill Sourour撰写的，它叫“<a href="https://medium.freecodecamp.org/dont-do-it-at-runtime-do-it-at-design-time-c4f59d1775e4">Don’t Do It At Runtime. Do It At Design Time</a>“与其解释他的观点，我更推荐你先去看一下。

预处理变量和自定义属性之间的一个关键区别是自定义属性在运行时工作。这意味着，在复杂性方面，可能是可以被接受的东西，因为预处理器可能不是具有自定义属性的好主意。

最近的一个例子是这样的:
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

在本例中，每个自定义属性都是使用calc()确定的，方法是取之前的自定义属性的值并将其乘以比率。这样做，我们可以得到下一个数字。

这意味着在运行时计算比率，您可以通过只更新-font- size属性的值来更改它们。例如:
``` css
@media screen and (min-width: 800px) {
  :root {
    --font-scale: 1.33;
  }
}
```
如果你想改变比例，这比再次计算所有的值要聪明，简洁和快得多。这也是我在开发过程中不会做的事情。

虽然上面的例子对于原型制作很有用，但在开发中我更喜欢看到类似这样的东西：
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
与Bill的文章中的例子类似，我发现查看实际值的含义很有帮助。我们阅读代码的次数比我们编写的次数多得多，而且字体等全局参数在生产中很少变化。

上面的例子仍然不完美。它违反了以前的规则，即全局价值应该是静态的。我更愿意使用预处理器变量，并使用前面演示的技术将它们转换为局部动态自定义属性。

避免使用一个自定义属性到另一个自定义属性的情况也很重要。当我们命名这样的属性时，会发生这种情况。

### 更改值不是变量
更改值而不是变量是有效使用自定义属性的最重要准则之一。

作为一般规则，你绝对不应该更改用于任何单一用途的自定义属性。这很容易做到，因为这正是我们如何处理预处理器的事情，但对于自定义属性来说没有多大意义。

在这个例子中，我们有两个在示例组件上使用的自定义属性。根据屏幕大小，我将从使用值切换--font-size-small到--font-size-large。

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
做到这一点的更好方法是定义一个定制到组件的自定义属性。然后使用media查询或任何其他选择器更改其值。
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
在此示例及其之前的其他示例中，媒体查询仅用于更改自定义属性的值。你可能还会注意到只有在var()的地方使用了该语句，并且更新了常规CSS属性。

变量声明和属性声明之间的这种分离是有意的。这有很多原因，但在考虑响应式设计时，好处最为明显。
### 用自定义属性创建响应布局
响应式有一个最大的问题是它太以来于媒体查询了，这导致。无论你如何组织CSS，与特定组件相关的样式，在样式表中变得非常的零碎。

很难知道哪些CSS属性会发上改变。然而，CSS自定义属性可以帮助我们组织一些与响应式设计相关的逻辑关系，降低媒体查询的难度。
### 如果它改变了，这是一个变量
使用媒体查询更改的属性本质上是动态的，自定义属性提供了在CSS中表达动态值的方法。这意味着如果你试图通过媒体查询来修改CSS属性，则应该选择自定义属性。

然后，就可以将其连同所有媒体规则、悬停状态或任何定义了值改变方式的动态选择器一起移动到文档的顶部。
### 基于设计分离逻辑
在正确的情况下，逻辑与设计的分离意味着媒体查询只是用来改变自定义属性的值。这意味着与响应性设计相关的所有逻辑都应该位于文档的顶部，并且无论我们在CSS中看到什么var()语句，我们都会立即知道这个属性会发生变化。使用传统的CSS编写方法，我们无法一眼就知道这一点。

我们中的许多人都非常擅长阅读和解释CSS，同时在头脑中跟踪哪些属性在不同的情况下发生了变化。我受够了，我不想再这样了!自定义属性现在提供了逻辑和实现之间的链接，所以我们不需要跟踪它，这非常有用!
###  逻辑折叠
在文档或函数顶部声明变量的想法并不是一个新想法。这是我们在大多数语言中所做的事情，现在我们也可以在CSS中完成。以这种方式编写CSS会在文档顶部和下面的CSS之间创建清晰的视觉区别。当我谈论他们时，我需要一种区分这些部分的方式，而“逻辑折叠”这个概念是我开始使用的一种隐喻。
在fold上方包含所有预处理器变量和自定义属性。这包括自定义属性可能具有的所有不同值。跟踪自定义属性如何更改应该很容易。

在这段代码之后的CSS可读性是很高，这很像，在媒体查询之前的CSS代码，和其它复杂的CSS样式。

让我们来看一看一个六列Flexbox网格系统的例子
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
该--row-display自定义属性的初始设置为block。600px以上的显示模式设置为flex。

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
我们很直观的可以知道--row-display是一个变量。默认值是block，所以应该忽略flex的值。

这个例子是非常简单的，但如果我们要拓展一个填充剩余空间的列的话，flex-grow，flex-shrink和flex-basis值则需要转换为自定义属性。你在这里看一个更详细的例子<a href="https://codepen.io/MadeByMike/pen/f42ce1a954af9796ae76c62a9ea801f4/">点击查看</a>。

### 用于主题的自定义属性
我不推荐为全局动态变量创建自定义属性，也不推荐将自定义属性附加到:root选择器作用域下。但是每个规则都有一个例外，对于自定义属性，它应该是一个主题。

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

上面的例子将--THEME-COLOR的值设置为--user-theme-color。如果--user-theme-color未设置，则将使用#d33a2c。这样，我们无需在每次使用时提供--THEME-COLOR。

你可能会在下面的例子中期望背景将被设置为green。但是，--user-theme-color根元素的值尚未设置，所以值--THEME-COLOR没有改变。

``` css
:root {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
body {
  --user-theme-color: green;
  background: var(--THEME-COLOR);
}
```
像这样间接设置全局动态属性可以防止它们在局部被覆盖，并确保用户的设置始终都从根元素继承。这个约定能保护主题值被避免意外的继承。

如果我们想要将某些特定的属性暴露出去，我们可以用:root选择器替换选择*器：
``` css
* {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
body {
  --user-theme-color: green;
  background: var(--THEME-COLOR);
}
```
现在--THEME-COLOR在每个元素中的值都会重新计算，因此可以使用--user-theme-color这个局部值。换句话说，这个例子中的背景颜色是green。

你可以在 <a href="https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web#manipulating-color-with-custom-properties>使用自定义属性操纵颜色</a>一节中看到此模式的更多详细示例。

### 使用JAVASCRIPT更新自定义属性
如果你想使用JavaScript设置自定义属性，那么就有一个相当简单的API，它看起来像这样：
``` css
const elm = document.documentElement;
elm.style.setProperty('--USER-THEME-COLOR', 'tomato');
```
在这里，我设置了--USER-THEME-COLOR文档元素的值，换言之，:root将被所有元素继承。

这不是一个新的API; 它是用于更新元素样式的JavaScript方法。这些是内联样式，因此它们比普通CSS具有更高的特异性。

这意味着应用局部定制很容易：
``` css
.note {
  --note-color: #eaeaea;
}
.note {
  background: var(--note-color);
}
```
在这里，我设置了默认值--note-color并将其范围设置为.note组件。我将变量声明与属性声明分开，即使在这个简单的例子中。
``` css
const elm = document.querySelector('#note-uid');
elm.style.setProperty('--note-color', 'yellow');
```
然后，我定位一个.note元素的特定实例，并仅更改该元素的--note-color自定义属性的值。这将比默认值具有更高的特异性。

你可以看到这个例子<a href="https://codepen.io/MadeByMike/pen/WzbGdE?editors=0110">使用React</a>如何工作。这些用户首选项可以保存在布局存储中，或者也可以保存在数据库中，也可以保存在更大的应用程序中。

### 使用自定义属性操作颜色
除了十六进制值和已命名的颜色之外，CSS还具有诸如rgb()和的 颜色功能hsl()。这些允许我们为组件设置独立的颜色，例如色调或亮度。自定义属性可以与颜色函数结合使用。
``` css
:root {
  --hue: 25;
}
body {
  background: hsl(var(--hue), 80%, 50%);
}
```
这很有用，但预处理器中最广泛使用的一些功能是高级颜色功能，它允许我们使用亮化，变暗或去饱和等功能来操作颜色：
``` css
darken($base-color, 10%);
lighten($base-color, 10%);
desaturate($base-color, 20%);
```
在浏览器中使用这些功能会很有用。<a href="https://www.w3.org/TR/css-color-4/#funcdef-color-mod">他们即将到来</a>，但直到我们在CSS中拥有原生色彩修改功能之后，自定义属性才能填补这一空白。

我们已经看到，自定义属性可以在现有的颜色函数中使用rgb()，hsl()但也可以在其中使用calc()。这意味着我们可以通过乘以实数将其转换为百分比，例如calc(50 * 1%) = 50%。
``` css
:root {
  --lightness: 50;
}
body {
  background: hsl(25, 80%, calc(var(--lightness) * 1%));
}
```
我们希望将亮度值存储为实数的原因是，我们可以calc在将其转换为百分比之前对其进行处理。例如，如果我想让颜色变暗20%，我可以乘以它的亮度0.8。我们可以通过将亮度计算分离为布局范围的自定义属性来使其更容易阅读：
``` css
:root {
  --lightness: 50;
}
body {
  --lightness: calc(var(--lightness * 0.8));
  background: hsl(25, 80%, calc(var(--lightness) * 1%));
}
```
我们甚至可以抽象出更多的计算结果，并<a href="https://codepen.io/MadeByMike/pen/YLQWeb">基于CSS定义属性的颜色修改功能</a>。这个例子对于大多数主题的实际情况来说可能太复杂，但它展示了动态自定义属性的全部功能。

### 简化主题
使用自定义属性的优点之一是能够简化主题。应用程序不需要知道如何使用自定义属性。相反，我们使用JavaScript或服务器端代码来设置自定义属性的值。这些值的使用方式由样式表决定。

这意味着我们可以再次将逻辑与设计分开。如果你有技术设计团队，作者可以更新样式表并决定如何应用自定义属性，而无需更改一行JavaScript或后端代码。

自定义属性还允许将一些复杂的主题移动到CSS中，并且这种复杂性可能会对CSS的可维护性产生负面影响，所以请记住尽可能保持简单。


### 快快使用自定义属性
即使您支持IE10和11，你也可以在今天开始使用自定义属性。本文中的大多数示例都与如何编写和构造CSS有关。但是，从可维护性的角度来说，这些好处是非常重要的，大多数示例只减少了使用更复杂的代码所能做的事情。

我使用一个名为<a href="https://github.com/MadLittleMods/postcss-css-variables">postcss-css-variables</a>的工具将自定义属性的大部分特性转换为相同代码的静态表示。其他类似的工具忽略媒体查询或复杂选择器中的自定义属性，将自定义属性处理得更像预处理变量。

这些工具不能模仿自定义属性的运行时特性。这意味着没有动态特性，比如使用JavaScript进行主题化或更改属性。这在很多情况下都是可以的。根据不同的情况，UI定制可能被认为是一种渐进式的增强，默认的主题对于旧的浏览器来说是完全可以接受的。


### 加载正确的样式表

有很多方法可以使用postCSS。我使用gulp过程为更新和旧的浏览器编译单独的样式表。我的gulp任务的简化版本如下:
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
这将产生两个CSS文件:一个常规文件具有自定义属性(styles.css)，另一个用于旧浏览器(styles.no . var . CSS)。我想要IE10和11的样式。css和其他浏览器获取常规的css文件。

通常，我主张使用功能查询，但<a href="https://caniuse.com/#search=%40supports">IE11不支持功能查询</a>，然而我们已经大量使用自定义属性，在这种情况下，提供不同样式表是有意义的。

智能地服务于不同的样式表和避免一闪而过的无样式的内容不是一项简单的任务。如果不需要定制属性的动态特性，可以考虑使用所有的浏览器样式。css和使用自定义属性作为开发工具。

如果你想充分利用自定义属性的所有动态特性，我建议使用<a href="https://www.smashingmagazine.com/2015/08/understanding-critical-css/">关键的CSS技术</a>。按照这些技术，主样式表是异步加载的，而关键的CSS则是内联的。你的页眉可能看起来像这样:
``` html
<head>
  <style> /* inlined critical CSS */ </style>
  <script> loadCSS('non-critical.css'); </script>
</head>
```
我们可以扩展它来加载styles.css或者styles.no-vars.css取决于浏览器是否支持自定义属性。我们可以检测到这样的支持：
``` javascript
if ( window.CSS && CSS.supports('color', 'var(--test)') ) {
  loadCSS('styles.css');
} else {
  loadCSS('styles.no-vars.css');
}
```
### 结论
如果您一直在努力有效地组织CSS、处理响应组件有困难、想要实现客户端主题、或者只是想要正确地使用自定义属性，那么本指南应该告诉您所有您需要了解的内容。

这可以归结为理解CSS中的动态变量和静态变量之间的区别，以及一些简单的规则:

01  单独的逻辑与设计;<br/>
02  如果CSS属性发生变化，考虑使用自定义属性;<br/>
03  更改自定义属性的值，而不是使用哪个自定义属性;<br/>
04  全局变量通常是静态的。<br/>
如果遵循这些约定，您将发现使用自定义属性比您想象的要简单得多。这甚至可能改变您一般使用CSS的方式。

#### 进一步阅读
“ <a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">It’s Time To Start Using Custom Properties</a> ”，Serg Hospodarets自定义属性的语法和特性的一般介绍。<br/>
“ <a href="https://csswizardry.com/2016/10/pragmatic-practical-progressive-theming-with-custom-properties/">Pragmatic, Practical, And Progressive Theming With Custom Properties</a> ”，Harry Roberts 关于主题的更多有用信息。<br/>
<a href="https://codepen.io/collection/naJLrB/">自定义属性集合</a>，CodePen上的Mike Riethmuller 
你可以尝试多个不同的示例。
