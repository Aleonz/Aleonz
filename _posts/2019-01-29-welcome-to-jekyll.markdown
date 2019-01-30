---
layout: post
title:  "使用CSS切换不同背景的字体颜色"
date:   2019-01-29 10:36:11 +0800
categories: jekyll update
paginate_path: "blog/page:num"
---
现在很多前端框架都已经添加了改变主题这个功能，但大多有个缺陷就是：`当选择背景色过暗、过亮时，文本字体颜色并不会跟着改变颜色，从而导致字体模糊不清晰。`这个体验不是完美的，为了解决这个问题，我查阅了一些资料，最终有了一个比较好的解决方案。效果如图：


![效果图]({{ "/images/20190129/color.gif" | absolute_url }})


其中，最简单实现的方法便是：`使用 HSL颜色 和 CSS变量 。`
这个技巧是基于：`CSS属性值范围溢出边界渲染特性，`就是CSS属性值超过正常的范围的时候，只要格式正确，也会渲染，而渲染的值就是合法边界值。例如：

{% highlight ruby %}
  /* opacity 透明度属性值合法范围是 0-1  */
  .view_opacity {
    opacity: -2;    #=>  解析为 0, 完全透明
    opacity: -1;    #=>  解析为 0, 完全透明
    opacity: 2;     #=>  解析为 1, 完全不透明
    opacity: 100;   #=>  解析为 1, 完全不透明
  }

  /* 色值，如HSL，S和L的范围都是 0%-100%  */
  .hsl_color {
    color: hsl(0, 0%, -100%);    #=>  解析为 hsl(0, 0%, 0%), 黑色
    color: hsl(0, 0%, 200%);     #=>  解析为 hsl(0, 0%, 100%), 白色
  }
{% endhighlight %}

将技巧应用于我们的字体颜色声明：
{% highlight ruby %}
  #CSS
    :root {
      --light: 80;
      /* 颜色被视为“光”的阈值。范围：从0到100的整数,建议 50 - 70 */
      --threshold: 60;
    }

    .btn {
      /* 任何低于阈值的亮度值将导致白色，任何高于阈值的值将导致黑色。 */
      --switch: calc((var(--light) - var(--threshold)) * -100%);
      color: hsl(0, 0%, var(--switch));
    }
{% endhighlight %}
让我们回顾一下代码：从亮度`--light`80开始并考虑60阈值，减法结果为20，乘以-100％，导致-2000％上限为0％。我们的背景比阈值轻，所以我们认为它很轻，并应用黑色文本。

如果我们将`--light`变量设置为20，则减法将导致-40，乘以-100％将变为4000％，上限为100％。我们的光变量低于阈值，因此我们将其视为“暗”背景并应用白色文本以保持高对比度。

当元素的背景变得太亮时，它很容易在白色背景下丢失。为了在真正的浅色上提供更好的UI，我们可以根据相同的背景颜色设置边框。
为此，我们可以使用相同的技术，但将其应用于HSLA声明的alpha通道。这样，我们可以根据需要调整颜色，然后完全透明或完全不透明。
{% highlight ruby %}
  #CSS
    :root {
      --light: 85;
      --border-threshold: 80;
    }

    .btn {
      /* 将边框颜色设置为相同颜色的30%较暗颜色。 */
      --border-light: calc(var(--light) * 0.7%);
      --border-alpha: calc((var(--light) - var(--border-threshold)) * 10);
      border: .1em solid hsla(var(--hue), calc(var(--sat) * 1%), var(--border-light), var(--border-alpha));
    }
{% endhighlight %}
如果背景亮度高于80，则边框颜色为原始亮度的70％，否则边框完全透明。

到这里可能你在考虑，我们认为的亮度与HSL亮度不同，这样计算出来的结果并不完美。幸运的是，我们有一些方法来衡量色调的亮度并调整我们的代码，以便它也能响应色调。目前比较受欢迎的两种亮度算法：
{% highlight ruby %}
  /* 使用sRGB Luma方法 */
  L =（红色* 0.2126 +绿色* 0.7152 +蓝色* 0.0722）/ 255

  /* 使用W3C亮度方法 */
  L =（红色* 0.299 +绿色* 0.587 +蓝色* 0.114）/ 255
{% endhighlight %}

最终实现代码为：
{% highlight ruby %}
# CSS
:root {
  /* 要在RGB声明中使用的主题颜色变量 */
  --red: 200;
  --green: 60;
  --blue: 255;
  /*颜色被视为“亮”的阈值。 范围：从0到1的小数，推荐 0.5 - 0.6*/
  --threshold: 0.5;
  /*将应用较暗边框的阈值。范围：从0到1的小数，推荐 0.8+*/
  --border-threshold: 0.8;
}

.btn {
  /*设置基类的背景*/
    background: rgb(var(--red), var(--green), var(--blue));

/* 使用sRGB Luma方法计算灰度（可以看成亮度）*/
/* 由Microsoft影像巨擘共同开发的一种彩色语言协议*/
# lightness = (red * 0.2126 + green * 0.7152 + blue * 0.0722) / 255  

  --r: calc(var(--red) * 0.2126);
  --g: calc(var(--green) * 0.7152);
  --b: calc(var(--blue) * 0.0722);
  --sum: calc(var(--r) + var(--g) + var(--b));
  
  --perceived-lightness: calc(var(--sum) / 255);
  
/*高于阈值的任何亮度值将被视为“亮”，因此应用黑色文本颜色。 其它都会被认为是黑色，并使用白色字体。*/
  color: hsl(0, 0%, calc((var(--perceived-lightness) - var(--threshold)) * -10000000%));
  

/* 仅当背景颜色亮度高于边界阈值时，才将边框设置为基色的50％深色阴影。*/
/*为了实现这一点，我使用相同的零下或高于最大值的技术，这次只使用RGBA声明中的Alpha值. */
/*这会导致边框完全透明或完全不透明 */
  --border-alpha: calc((var(--perceived-lightness) - var(--border-threshold)) * 100);
  
  border-width: .2em;
  border-style: solid;
  border-color: rgba(calc(var(--red) - 50), calc(var(--green) - 50), calc(var(--blue) - 50), var(--border-alpha));
}

.btn--w3c{
/*    使用W3C亮度方法的替代计算*/
#   lightness = (red * 0.299 + green * 0.587 + blue * 0.114) / 255

  --r: calc(var(--red) * 0.299);
  --g: calc(var(--green) * 0.587);
  --b: calc(var(--blue) * 0.114);
}

.btn--secondary{
  /* 将背景颜色设置为60º旋转色调,进行对比 */
  filter:hue-rotate(60deg);
}

/* 基础样式*/
body{
  background: honeydew;
  padding:1rem;
  max-width: 600px;
  margin: auto;
}

.sliders{
  display:grid;
  grid-template-columns: repeat(auto-fit, minmax(150px,1fr));
  grid-gap:1em;
}

.buttons{
  display:grid;
  grid-template-columns: repeat(auto-fit, minmax(200px,1fr));
  grid-gap: 1em;
}

.buttons h2{
  grid-column: 1 / -1;
  margin-bottom:0;
}

.btn{
  padding: 1em;
  font-size: 1.5rem; 
  border-radius: 0.2em;
  box-sizing: border-box;
  flex:1;
}

input[type=range]{
  display:flex;
  flex-direction:column;
  color: black;
  text-align:center;
  margin: 10px;
  font-weight:600;
}

input::before{
  content: attr(id);
  text-transform: capitalize;
}

input[id=red]::after{
  counter-reset: red var(--red);
  content: counter(red);
}

input[id=green]::after{
  counter-reset: green var(--green);
  content: counter(green);
}

input[id=blue]::after{
  counter-reset: blue var(--blue);
  content: counter(blue);
}

# HTML
<aside class="sliders">
  <input type="range" id="red" min="0" max="255" value="200" step="1"> 
  <input type="range" id="green" min="0" max="255" value="60" step="1"> 
  <input type="range" id="blue" min="0" max="255" value="255" step="1">
</aside>

<main class="buttons">  
  <h2>sRBG 亮度 方法</h2>
  <button class="btn">原色</button> 
  <button class="btn btn--secondary">二次色</button> 
  <h2>W3C 亮度 方法</h2>
  <button class="btn btn--w3c">原色</button> 
  <button class="btn btn--w3c btn--secondary">二次色</button>
</main>

# JS
/*JS仅用于设置基本颜色中使用的CSS自定义属性,颜色更改以CSS计算*/
const root = document.documentElement;
const inputs = [].slice.call(document.querySelectorAll('input'));

inputs.forEach(input => input.addEventListener('change', handleUpdate));
inputs.forEach(input => input.addEventListener('mousemove', handleUpdate));

/*JS中 	获取	CSS变量可以使用getPropertyValue()方法*/
/*JS中	改变	CSS变量可以使用setProperty()方法*/
/*console.log(getComputedStyle(root).getPropertyValue('--red'))*/
function handleUpdate(e) {
  if (this.id === 'red') root.style.setProperty('--red', this.value);
  if (this.id === 'green') root.style.setProperty('--green', this.value);
  if (this.id === 'blue') root.style.setProperty('--blue', this.value);
}

{% endhighlight %}

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
