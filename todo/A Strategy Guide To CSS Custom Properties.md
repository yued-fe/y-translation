# CSS定制属性的策略指南

**关于作者** Mike是来自澳大利亚的独立网站开发人员，曾在澳大利亚的一些最大的网站以及一些最小的社区工作...... <a href="">关于Michael的更多信息...</a>

-------------------
> 所有现代浏览器现在都支持CSS自定义属性（有时称为“CSS变量”），并且人们开始在项目中使用它们。这很棒，但它们与预处理器中的变量不同，我已经看到许多人使用它们的例子，而没有考虑它们提供的优点。

自定义属性有很大的潜力可以改变我们编写和构造CSS的方式，并且在更小程度上改变我们如何使用JavaScript与UI组件交互。我不会专注于语法和它们的工作方式（为此，我建议你阅读“ <a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">现在是时候开始使用自定义属性</a> ”）。相反，我想深入了解如何充分利用CSS自定义属性。

### 它们如何与预处理器中的变量相似？
自定义属性有点像预处理器中的变量，但有一些重要的区别。第一个也是最明显的区别是语法。

随着SCSS我们用一个美元符号来表示一个变量：

``` scss
$smashing-red: #d33a2c;
```
至少我们使用一个@符号：
``` scss
smashing-red: #d33a2c;
```
自定义属性遵循类似的约定并使用--前缀：
``` css
:root { --smashing-red: #d33a2c; }
.smashing-text { 
  color: var(--smashing-red);
}
```
自定义属性和预处理器中的变量之间的一个重要区别是自定义属性具有用于分配值和检索该值的不同语法。当检索自定义属性的值时，我们使用该var()函数。

下一个最明显的区别在于名称。它们被称为'自定义属性'，因为它们确实是CSS属性。在预处理器中，你可以在任何位置声明和使用变量，包括外部声明块，media规则或甚至作为选择器的一部分。
``` css
$breakpoint: 800px;
$smashing-red: #d33a2c;
$smashing-things: ".smashing-text, .cats";

@media screen and (min-width: $breakpoint) {
  #{$smashing-things} {
    color: $smashing-red;
  }
}
```
使用自定义属性，上面的大多数示例都是无效的。

自定义属性与可以用作常规CSS属性的位置具有相同的规则。把它们看作动态属性比变量要好得多。这意味着它们只能在声明块中使用，或者换句话说，自定义属性绑定到选择器。这可以是:root选择器或任何其他有效的选择器。
``` css
:root { --smashing-red: #d33a2c; }

@media screen and (min-width: 800px) {
  .smashing-text, .cats {
    --margin-left:  1em;
  }
}
```
你可以在任何情况下检索自定义属性的值，否则将在属性声明中使用值。这意味着它们可以作为单个值使用，作为速记语句的一部分，甚至可以在calc()方程式中使用。
``` css
.smashing-text, .cats {
  color: var(--smashing-red);
  margin: 0 var(--margin-horizontal);
  padding: calc(var(--margin-horizontal) / 2)
}
```
但是，它们不能用于media查询或选择器，包括:nth-child()。

关于语法和自定义属性的工作原理可能还有很多，比如如何使用fallback值，以及是否可以将变量分配给其他变量（是），但此基本介绍应足以理解其余部分本文中的概念。有关自定义属性工作方式的详细信息，可以阅读由Serg Hospodarets编写的“<a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/"> 现在开始使用自定义属性 </a>”。

### 动态与静态

抛开样式差异，预处理器和自定义属性中变量之间最显着的区别是它们的作用范围。我们可以将变量指定为静态或动态范围。预处理器中的变量是静态的，而自定义属性是动态的。

在涉及CSS的地方，静态意味着你可以在编译过程中的不同点更新变量的值，但是这不能改变它之前的代码的值

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
一旦编译了这个CSS，变量就消失了。这意味着我们可能会读取一个.scss文件并确定其输出而不知道任何有关HTML的内容，浏览器或其他输入的情况下读取文件并确定其输出。自定义属性则不是这种情况。

预处理器确实有一种“块范围”，其中变量可以在选择器，函数或mixin中临时更改。这改变了块内变量的值，但它仍然是静态的。这与块有关，而不是选择器。在下面的例子中，变量$background在.example类内部被改变。即使我们使用相同的选择器，它也会变回块外的初始值。

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
这将导致：
``` scss
.example {
  background: blue;
}
.example {
  background: red;
}
```
定制属性的工作方式不同 在涉及定制属性的地方，动态范围意味着它们受继承和级联影响。该属性与选择器绑定，如果该值发生更改，则会像所有其他CSS属性一样影响所有匹配的DOM元素。

这很棒，因为你可以使用伪选择器（例如悬停）或甚至JavaScript来更改media查询中的自定义属性的值。

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

我们不必更改自定义属性的使用位置 - 我们使用CSS更改自定义属性的值。这意味着使用相同的自定义属性，我们可以在同一页面的不同位置或上下文中具有不同的值。

### 全局与局部

除了静态或动态之外，变量还可以是全局的或局部的。如果你写JavaScript，你会熟悉这一点。变量可以应用于应用程序中的所有内容，也可以将其范围限制为特定的功能或代码块。

CSS是相似的。我们有一些东西适用于全局，有些东西更具局部性。品牌颜色，垂直间距和排版都是你可能希望在全局范围内一致应用于你的网站或应用程序的所有示例。我们也有局部的东西。例如，按钮组件可能具有小型和大型尺寸。你不希望这些按钮的大小适用于所有输入元素或页面上的每个元素。

这是我们在CSS中熟悉的东西。我们开发了设计系统，命名约定和JavaScript库，这些都有助于隔离局部组件和全局设计元素。自定义属性为处理这个老问题提供了新的选项。

默认情况下，CSS自定义属性的范围局限于我们将其应用于的特定选择器。所以他们有点像局部变量。但是，自定义属性也是继承的，所以在很多情况下，它们的行为就像全局变量 - 特别是在应用于:root选择器时。这意味着我们需要考虑如何使用它们。

这么多示例显示了应用于:root元素的自定义属性，虽然这对于演示很适用，但它可能导致全局范围变得混乱，并导致意外的继承问题。幸运的是，我们已经学到了这些教训。

### 全局变量趋于静态
有一些小例外，但一般来说，CSS中的大多数全局事物也是静态的。

像品牌颜色，排版和间距这样的全局变量，从一个组件到另一个组件的变化不大。当他们改变时，这往往是一个全局品牌重塑或其他一些成熟产品上很少发生的重大变化。这些东西仍然是有意义的变量，它们在很多地方都有使用，变量有助于保持一致性。但是他们的动态是没有意义的。这些变量的值不会以任何动态方式变化。

出于这个原因，我强烈建议为全局（静态）变量使用预处理器。这不仅确保它们始终是静态的，而且它在代码中直观地表示它们。这可以使CSS更加易读易维护。

### 局部静态变量可以（有时）
你可能会认为，鉴于全局变量的强势立场是静态的，通过反思，所有的局部变量可能都需要是动态的。虽然局部变量确实倾向于是动态的，但这远没有全局变量趋于静态那样强烈。

在许多情况下，局部静态变量是完全可以的。我主要是为了开发人员的便利，在组件文件中使用预处理器变量。

考虑具有多种尺寸变化的按钮组件的经典示例。

![Alt text](./0EC9CEDB-10EE-4FEC-9E67-CBCB1C72A600.png)

我scss可能看起来像这样：
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
显然，如果我多次使用变量或从大小变量中派生边距和填充值，这个示例会更有意义。但是，快速制作不同尺寸原型的能力可能是一个充分的理由。

由于大多数静态变量是全局的，我喜欢区分只在组件内部使用的静态变量。为此，你可以使用组件名称为这些变量加上前缀，或者可以使用其他前缀，例如c-variable-name组件或l-variable-name局部。你可以使用任何你想要的前缀，或者你可以前缀全局变量。无论你选择什么，特别是如果将现有代码库转换为使用自定义属性，区分是有帮助的。

### 何时使用自定义属性
如果可以在组件内使用静态变量，我们应该何时使用自定义属性？将现有的预处理器变量转换为自定义属性通常没什么意义。毕竟，定制属性的原因完全不同。自定义属性意义的时候，我们有改变相对于DOM中的条件CSS属性-尤其是一个动态的条件，例如:focus，:hovermedia查询或使用JavaScript。

我怀疑我们总是会使用某种形式的静态变量，尽管将来我们可能需要更少的静态变量，因为自定义属性提供了组织逻辑和代码的新方法。在此之前，我认为在大多数情况下，我们将使用预处理器变量和自定义属性的组合。

知道我们可以将静态变量分配给自定义属性很有帮助。无论它们是全局的还是局部的，在很多情况下将静态变量转换为局部动态定制属性是有意义的。

>注意：你是否知道这$var是自定义属性的有效值？Sass的最新版本认识到了这一点，因此我们需要插入分配给自定义属性的变量，如下所示：#{$var}。这告诉Sass你想输出变量的值，而不是仅仅$var在样式表中。这只适用于自定义属性等情况，其中变量名称也可以是有效的CSS。

如果我们采用上面的按钮示例，并确定所有按钮都应该使用移动设备上的小变化，而不管HTML中应用的类是什么，那么现在就是一种更加动态的情况。为此，我们应该使用自定义属性。

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
在这里我创建一个自定义属性：--button-size。这个自定义属性最初的作用域是使用btn该类的所有按钮元素。然后我改变了--button-size600px以上的类的值btn-med和btn-lrg。最后，我将这个自定义属性应用于一个地方的所有按钮元素。

### 不要太聪明
定制属性的动态特性使我们能够创建一些聪明而复杂的组件。

随着预处理器的推出，我们中的许多人使用mixin和自定义函数创建了具有巧妙抽象的库。在有限的情况下，像这样的例子今天仍然有用，但大多数情况下，我使用预处理器的时间越长，我使用的功能就越少。今天，我几乎只用静态变量来使用预处理器。

自定义属性不会（也不应该）免于这种类型的实验，我期待看到许多聪明的例子。但从长远来看，可读和可维护的代码总是会赢得聪明人的喜爱（至少在项目中是这样）。

我最近在Free Code Camp Medium上阅读了关于此主题的优秀文章。它是由Bill Sourour撰写的，它叫“<a href="https://medium.freecodecamp.org/dont-do-it-at-runtime-do-it-at-design-time-c4f59d1775e4">Don’t Do It At Runtime. Do It At Design Time</a>“与其解释他的观点，我愿意你去阅读它。

预处理器变量和定制属性之间的一个主要区别是定制属性在运行时工作。这意味着在复杂性方面可能被边界接受的事情，对于自定义属性，预处理器可能不是一个好主意。

最近我用来举例说明的一个例子是：
``` css
:root {
  --font-scale: 1.2;
  --font-size-1: calc(var(--font-scale) * var(--font-size-2));
  --font-size-2: calc(var(--font-scale) * var(--font-size-3)); 
  --font-size-3: calc(var(--font-scale) * var(--font-size-4));   
  --font-size-4: 1rem;     
}
```
这产生模块化的比例。模块化标度是一系列使用比率相互关联的数字。它们经常用于网页设计和开发以设置字体大小或间距。

在这个例子中，每个自定义属性都是calc()通过采用前一个自定义属性的值并将其乘以比率来确定的。这样做，我们可以得到下一个数字。

这意味着比率是在运行时计算的，你可以通过仅更新--font-scale属性的值来更改它们。例如：
``` css
@media screen and (min-width: 800px) {
  :root {
    --font-scale: 1.33;
  }
}
```
如果你想改变比例，这比再次计算所有的值要聪明，简洁和快得多。这也是我在生产代码中不会做的事情。

虽然上面的例子对于原型制作很有用，但在制作中我更喜欢看到类似这样的东西：
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
与Bill的文章中的例子类似，我发现查看实际值的含义很有帮助。我们阅读代码的次数比我们编写的次数多得多，而且字体大小等全局值在生产中很少变化。

上面的例子仍然不完美。它违反了以前的规则，即全局价值应该是静态的。我更愿意使用预处理器变量，并使用前面演示的技术将它们转换为局部动态定制属性。

避免使用一个定制属性到另一个定制属性的情况也很重要。当我们命名这样的属性时，会发生这种情况。

### 更改值不是变量
更改值不是变量是有效使用自定义属性的最重要策略之一。

作为一般规则，你绝对不应该更改用于任何单一用途的自定义属性。这很容易做到，因为这正是我们如何处理预处理器的事情，但对于定制属性来说没有多大意义。

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
做到这一点的更好方法是定义一个定制到组件的定制属性。然后使用media查询或任何其他选择器更改其值。
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
最后，在一个地方，我使用这个定制属性的值：
``` css
.example {
  font-size: var(--example-font-size);
}
```
在此示例及其之前的其他示例中，media查询仅用于更改自定义属性的值。你可能还会注意到只有一个地方var()使用了该语句，并且更新了常规CSS属性。

变量声明和属性声明之间的这种分离是有意的。这有很多原因，但在考虑响应式设计时，好处最为明显。
### 自定义属性的自适应设计
响应式设计在很大程度上依赖于media查询时遇到的困难之一是，无论你如何组织CSS，与特定组件相关的样式都会在样式表中变得碎片化。

知道哪些CSS属性会改变是非常困难的。尽管如此，CSS自定义属性可以帮助我们组织一些与响应式设计相关的逻辑，并且使media查询的工作更容易。
### 如果它改变了，这是一个变量
使用media查询更改的属性本质上是动态的，自定义属性提供了在CSS中表达动态值的方法。这意味着如果你正在使用media查询来更改任何CSS属性，则应该将此值放入自定义属性中。

然后，你可以将其与所有media规则，悬停状态或定义值更改方式的任何动态选择器一起移动到文档的顶部。
### 独立于设计的逻辑
如果正确完成，逻辑和设计的分离意味着media查询仅用于更改自定义属性的值。这意味着与响应式设计相关的所有逻辑应该位于文档的顶部，无论我们var()在CSS中看到什么声明，我们都会立即知道该属性发生了变化。用传统的编写CSS的方法，一眼就无法知道这一点。

我们中的很多人都非常善于阅读和解释CSS，同时在头脑中跟踪哪些属性在不同情况下发生了变化。我厌倦了这一点，我不想再这样做了！自定义属性现在提供了逻辑与其实现之间的链接，所以我们不需要跟踪它，这非常有用！
###  逻辑折叠
在文档或函数顶部声明变量的想法并不是一个新想法。这是我们在大多数语言中所做的事情，现在我们也可以在CSS中完成。以这种方式编写CSS会在文档顶部和下面的CSS之间创建清晰的视觉区别。当我谈论他们时，我需要一种区分这些部分的方式，而“逻辑折叠”这个概念是我开始使用的一种隐喻。
在fold上方包含所有预处理器变量和自定义属性。这包括自定义属性可能具有的所有不同值。跟踪定制属性如何更改应该很容易。

低于CSS的CSS非常简单明了，易于阅读。在media查询和现代CSS的其他必要复杂性之前，它就像CSS一样。

看一看六列Flexbox网格系统的一个非常简单的例子：
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

在折叠下面可能看起来像这样：
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
我们立即知道这--row-display是一个值得改变的价值。最初，它会是block，所以flex值将被忽略。

这个例子是非常简单的，但如果我们扩大到包括灵活的宽度，柱填充剩余空间，很可能flex-grow，flex-shrink和flex-basis值将需要转换为自定义属性。你可以试试这个或者<a href="https://codepen.io/MadeByMike/pen/f42ce1a954af9796ae76c62a9ea801f4/">在这里看一个更详细的例子</a>。

### 用于主题的自定义属性
我主要反对使用全局动态变量的自定义属性，并希望暗示将自定义属性附加到:root选择器在很多情况下被认为是有害的。但是每个规则都有一个例外，对于自定义属性，它是主题。

全局定制属性的有限使用可以使整体变得更容易。

主题化通常是指让用户以某种方式定制UI。这可能是像更改个人资料页面上的颜色。或者它可能更加局部化。例如，你可以在Google Keep应用程序中选择笔记的颜色。

![Alt text](./34740899-55EF-4061-AB9D-9DFE6B8A8623.png)

主题化通常涉及编译单独的样式表以覆盖用户首选项的默认值，或为每个用户编译不同的样式表。这两者都可能是困难的并且对性能有影响。

使用自定义属性，我们不需要编译不同的样式表; 我们只需要根据用户的喜好更新属性的值。由于它们是继承的值，因此如果我们在根元素上执行此操作，则可以在我们的应用程序的任何位置使用它们。

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

上面的例子将值设置为它存在--THEME-COLOR的值--user-theme-color。如果--user-theme-color未设置，#d33a2c则将使用该值。这样，我们无需在每次使用时提供回退--THEME-COLOR

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
像这样间接设置全局动态属性可以防止它们在布局被覆盖，并确保用户设置始终从根元素继承。这是一个有用的约定来保护你的主题值并避免意外的继承。

如果我们想要将特定的属性公开，我们可以用:root选择器替换选择*器：
``` css
* {
  --THEME-COLOR: var(--user-theme-color, #d33a2c);            
}
body {
  --user-theme-color: green;
  background: var(--THEME-COLOR);
}
```
现在--THEME-COLOR重新计算每个元素的值，--user-theme-color因此可以使用局部值。换句话说，这个例子中的背景颜色是green。

你可以在 <a href="https://www.smashingmagazine.com/2018/05/css-custom-properties-strategy-guide/?utm_source=CSS-Weekly&utm_campaign=Issue-315&utm_medium=web#manipulating-color-with-custom-properties>使用自定义属性操纵颜色</a>一节中看到此模式的更多详细示例。

### 使用JAVASCRIPT更新自定义属性
如果你想使用JavaScript设置自定义属性，那么就有一个相当简单的API，它看起来像这样：
``` css
const elm = document.documentElement;
elm.style.setProperty('--USER-THEME-COLOR', 'tomato');
```
在这里，我设置--USER-THEME-COLOR文档元素的值，或换言之，:root它将由所有元素继承的元素。

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
除了十六进制值和已命名的颜色之外，CSS还具有诸如rgb()和的 颜色功能hsl()。这些允许我们指定颜色的单个组件，例如色调或亮度。自定义属性可以与颜色函数结合使用。
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
我们甚至可以抽象出更多的计算结果，并<a href="https://codepen.io/MadeByMike/pen/YLQWeb">使用自定义属性在CSS中</a>创建<a href="https://codepen.io/MadeByMike/pen/YLQWeb">颜色修改功能</a>。这个例子对于大多数主题的实际情况来说可能太复杂，但它展示了动态自定义属性的全部功能。

### 简化主题
使用自定义属性的优点之一是能够简化主题。应用程序不需要知道如何使用自定义属性。相反，我们使用JavaScript或服务器端代码来设置自定义属性的值。这些值的使用方式由样式表决定。

这意味着我们可以再次将逻辑与设计分开。如果你有技术设计团队，作者可以更新样式表并决定如何应用自定义属性，而无需更改一行JavaScript或后端代码。

自定义属性还允许将一些复杂的主题移动到CSS中，并且这种复杂性可能会对CSS的可维护性产生负面影响，所以请记住尽可能保持简单。

### 今天使用自定义属性
即使你支持IE10和11，也可以立即开始使用自定义属性。本文中的大部分示例都与我们如何编写和构建CSS有关。就可维护性而言，好处是显着的，然而，大多数例子只会减少更复杂的代码所能做的。

我使用一个名为<a href="https://github.com/MadLittleMods/postcss-css-variables">postcss-css-variables</a>的工具将自定义属性的大部分功能转换为相同代码的静态表示。其他类似的工具会忽略media查询或复杂选择器中的自定义属性，像预处理器变量一样处理自定义属性。

这些工具不能做的是模拟自定义属性的运行时功能。这意味着没有动态功能，如主题化或使用JavaScript更改属性。在许多情况下这可能是确定的。根据具体情况，UI自定义可能被认为是一种渐进式增强功能，对于旧版浏览器，默认主题可能完全可以接受。

### 加载正确的样式表
有很多方法可以使用postCSS。我使用一个gulp过程为较新和较旧的浏览器编译单独的样式表。我的gulp任务的简化版本如下所示：
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
这会产生两个CSS文件：一个是定制属性（styles.css），另一个是旧版浏览器（styles.no-vars.css）。我想要IE10和11 styles.no-vars.css以及其他浏览器来获取常规CSS文件。

通常，我主张使用功能查询，但<a href="https://caniuse.com/#search=%40supports">IE11不支持功能查询</a>，并且我们已经使用了自定义属性，以便在这种情况下提供不同的样式表。

智能地提供不同的样式表并避免无格式内容的闪烁并不是一项简单的任务。如果你不需要自定义属性的动态功能，则可以考虑提供所有浏览器styles.no-vars.css并将自定义属性简单地用作开发工具。

如果你想充分利用自定义属性的所有动态特性，我建议使用<a href="https://www.smashingmagazine.com/2015/08/understanding-critical-css/">关键的CSS技术</a>。遵循这些技术，主要样式表是异步加载的，而关键CSS是内联呈现的。你的页面标题可能如下所示：
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
如果你一直在努力有效地组织CSS，使用响应式组件很困难，想要实现客户端主题，或者只想用自定义属性从右脚开始，那么本指南应该告诉你所需要知道的一切。

它归结为了解CSS中动态变量和静态变量之间的区别以及一些简单的规则：

01  与设计分开的逻辑;<br/>
02  如果CSS属性发生更改，请考虑使用自定义属性;<br/>
03  更改自定义属性的值，而不是使用哪个自定义属性;<br/>
04  全局变量通常是静态的。<br/>
如果遵循这些约定，你会发现使用自定义属性比你想象的要容易得多。这甚至可能会改变你对CSS的处理方式。

#### 进一步阅读
“ <a href="https://www.smashingmagazine.com/2017/04/start-using-css-custom-properties/">It’s Time To Start Using Custom Properties</a> ”，Serg Hospodarets自定义属性的语法和特性的一般介绍。<br/>
“ <a href="https://csswizardry.com/2016/10/pragmatic-practical-progressive-theming-with-custom-properties/">Pragmatic, Practical, And Progressive Theming With Custom Properties</a> ”，Harry Roberts 关于主题的更多有用信息。<br/>
<a href="https://codepen.io/collection/naJLrB/">自定义属性集合</a>，CodePen上的Mike Riethmuller 
你可以尝试多个不同的示例。
