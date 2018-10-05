---
layout: article
title: LaTeX and Markdown
mathjax: true
mermaid: true
chart: true
mathjax_autoNumber: true
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 139, 87 , .4), rgba(139, 34, 139, .4))'
    src: assets/cover3.jpg
---
曾经给队友发的文件的归档 测试flowchart
<!--more-->
#### $\LaTeX$

因为之前有提到LaTeX 所以就简单地推荐一本书还有软件
电子书《一份不太简短的LATEX 2ε 介绍》（或《103 分钟了解LATEX2ε 》）
https://github.com/louisstuart96/lshort-new-zh-cn
这本书的特点就是薄 而且尽可能的做到了全面 如果打算学的话 建议花一天时间通览然后再挑重点看一看
国内还有一本书就是刘海洋的《LaTeX入门》，会厚很多也详尽很多

剩下就是论文的模版和搜索引擎以及熟能生巧了
模版网站 http://www.latexstudio.net/
自己试过主流的几种编辑器方案之后还是觉得TeXstudio最简单好用

#### Markdown

个人觉得算是那种轻松入门又可以受益颇多的东西，这个PDF就是从Markdown导出的，平时用来排版文字，代码，数学公式是极为方便的，比如说：

```matlab
M=[1 0 0;0 1 0;0 0 1];K=[4 -2 0;-2 3 -1;0 -2 3];p=1;omega=3;
[t,y] = ode45(@(t,y) vdp(t,y,M,K,p,omega),[0 50],[0;0;0;0;0;0]);
plot(t,y(:,1),t,y(:,2),t,y(:,3));
xlabel('Time t');
ylabel('Solution x_i');
legend('x_1','x_2','x_3');

function dxdt = vdp(t,y,M,K,p,omega)
MK=inv(M)*(-K*[y(1);y(2);y(3)]+[p*sin(omega*t);0;0]);
dxdt = [y(4);y(5);y(6);MK];
end
```

数学公式的写法和LaTeX基本相同：

```latex
试从\int^{+\infty}_0e^{-y^2}{\rm{d}}y=\frac{\sqrt{\pi}}{2}推出L(x)=\int^{+\infty}_0e^{-y^2-\frac{c^2}{y^2}}\rm{d}x=\frac{\sqrt{\pi}}{2}e^{-2c}
```

渲染出来就是
$$
试从\int^{+\infty}_0e^{-y^2}{\rm{d}}y=\frac{\sqrt{\pi}}{2}推出L(x)=\int^{+\infty}_0e^{-y^2-\frac{c^2}{y^2}}\rm{d}x=\frac{\sqrt{\pi}}{2}e^{-2c}
$$
甚至绘制简单的框图：

```
st=>start: Start
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
```

```mermaid
st=>start: Start
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end

st->op->cond
cond(yes)->e
cond(no)->op
```

因为排版简洁美观，读起来也更加舒服，用来简单的写个作业或者论文初稿也是可以的。
入门的教程的话 推荐两个
Github的Guid
https://guides.github.com/features/mastering-markdown/
西电开源社区的帖子
https://linux.xidian.edu.cn/forum/d/27-markdown
