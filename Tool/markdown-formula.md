# 一文学会在Markdown中编辑数学符号与公式

在用Markdown写博客时会涉及到数学符号与公式的编辑，下面进行汇总。随手记录，方便你我他。



- 行内公式：将公式插入到本行内

```bash
$0.98^{365} \approx 0.0006$
```

我的365天：$0.98^{365} \approx 0.0006$



- 单独的公式块：将公式插入到新的一行内，并且居中

```bash
$$
1.02^{365} \approx 1377.4
$$
```
在座各位大佬的365天：
$$
1.02^{365} \approx 1377.4
$$

注意：

1. 在博客园用Markdown写博客需要启用数学公式支持，如下：

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210819163846076-1128461557.png)

2. 在博客园可以在公式上右键查看详情：

   ![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210822134624501-390959711.png)

3. 如果使用Typora编写Markdown，解析行内公式需要手动设置一下， 文件 -> 偏好设置 -> Markdown -> Markdown扩展语法 -> 勾选 “内联公式”，重启软件，Typora才会解析行内公式。

![](http://img2020.cnblogs.com/blog/1546632/202108/1546632-20210819164501658-996062202.png)

## 符号

### 上下标、运算符

|            |                        显示效果                        |                    markdown公式语法                    |
| :--------: | :----------------------------------------------------: | :----------------------------------------------------: |
|    上标    |                 $x^2、 x^y 、e^{365}$                  |                 `x^2、 x^y 、e^{365}`                  |
|    下标    |                    $x_0、a_1、Y_a$                     |                    `x_0、a_1、Y_a`                     |
|    分式    |              $\frac{x}{y}、\frac{1}{x+1}$              |              `\frac{x}{y}、\frac{1}{x+1}`              |
|     乘     |                        $\times$                        |                        `\times`                        |
|     除     |                         $\div$                         |                         `\div`                         |
|    加减    |                         $\pm$                          |                         `\pm`                          |
|    减加    |                         $\mp$                          |                         `\mp`                          |
|    求和    |                         $\sum$                         |                         `\sum`                         |
| 求和上下标 | $\sum_0^3 、\sum_0^{\infty} 、\sum_{-\infty}^{\infty}$ | `\sum_0^3 、\sum_0^{\infty} 、\sum_{-\infty}^{\infty}` |
|    求积    |                        $\prod$                         |                        `\prod`                         |
|    微分    |                       $\partial$                       |                       `\partial`                       |
|    积分    |               $\int 、\displaystyle\int$               |               `\int 、\displaystyle\int`               |
|   不等于   |                         $\neq$                         |                         `\neq`                         |
|  大于等于  |                         $\geq$                         |                         `\geq`                         |
|  小于等于  |                         $\leq$                         |                         `\leq`                         |
|   约等于   |                       $\approx$                        |                       `\approx`                        |
| 不大于等于 |                     $x+y \ngeq z$                      |                     `x+y \ngeq z`                      |
|    点乘    |                      $a \cdot b$                       |                      `a \cdot b`                       |
|    星乘    |                       $a \ast b$                       |                       `a \ast b`                       |
|  取整函数  |       $\left \lfloor \frac{a}{b} \right \rfloor$       |       `\left \lfloor \frac{a}{b} \right \rfloor`       |
|  取顶函数  |        $\left \lceil \frac{c}{d} \right \rceil$        |        `\left \lceil \frac{c}{d} \right \rceil`        |

### 括号

|                  |                     显示效果                      |                 markdown公式语法                  |
| :--------------: | :-----------------------------------------------: | :-----------------------------------------------: |
| 圆括号（小括号） |           $\left( \frac{a}{b} \right)$            |           `\left( \frac{a}{b} \right)`            |
| 方括号（中括号） | $\left[ \frac{a}{b} \right]$或者$[ \frac{x}{y} ]$ | `\left[ \frac{a}{b} \right]`或者`[ \frac{x}{y} ]` |
| 花括号（大括号） |           $\lbrace \frac{a}{b} \rbrace$           |           `\lbrace \frac{a}{b} \rbrace`           |
|      角括号      |    $\left \langle \frac{a}{b} \right \rangle$     |    `\left \langle \frac{a}{b} \right \rangle`     |
|     混合括号     |              $\left [ a,b \right )$               |              `\left [ a,b \right )`               |



### 三角函数、指数、对数

|      |  显示效果   | markdown公式语法 |
| :--: | :---------: | :--------------: |
| sin  |  $\sin(x)$  |    `\sin(x)`     |
| cos  |  $\cos(x)$  |    `\cos(x)`     |
| tan  |  $\tan(x)$  |    `\tan(x)`     |
| cot  |  $\cot(x)$  |    `\cot(x)`     |
| log  | $\log_2 10$ |   `\log_2 10`    |
|  lg  |  $\lg 100$  |    `\lg 100`     |
|  ln  |   $\ln2$    |      `\ln2`      |

### 数学符号

|                   |                    显示效果                    |                markdown公式语法                |
| :---------------: | :--------------------------------------------: | :--------------------------------------------: |
|       无穷        |                    $\infty$                    |                    `\infty`                    |
|       矢量        |                   $\vec{a}$                    |                   `\vec{a}`                    |
|     一阶导数      |                   $\dot{x}$                    |                   `\dot{x}`                    |
|     二阶导数      |                   $\ddot{x}$                   |                   `\ddot{x}`                   |
|    算数平均值     |                   $\bar{a}$                    |                   `\bar{a}`                    |
|     概率分布      |                   $\hat{a}$                    |                   `\hat{a}`                    |
|     虚数i、j      |                $\imath、\jmath$                |                `\imath、\jmath`                |
|    省略号(一)     |                $1,2,3,\ldots,n$                |                `1,2,3,\ldots,n`                |
|    省略号(二)     |           $x_1 + x_2 + \cdots + x_n$           |           `x_1 + x_2 + \cdots + x_n`           |
|    省略号(三)     |                    $\vdots$                    |                    `\vdots`                    |
|    省略号(四)     |                    $\ddots$                    |                    `\ddots`                    |
|   斜线与反斜线    |    $\left / \frac{a}{b} \right \backslash$     |    `\left / \frac{a}{b} \right \backslash`     |
|     上下箭头      | $\left \uparrow \frac{a}{b} \right \downarrow$ | `\left \uparrow \frac{a}{b} \right \downarrow` |
|     $\angle$      |                    $\angle$                    |                    `\angle`                    |
|     $\prime$      |                    $\prime$                    |                    `\prime`                    |
|   $\rightarrow$   |                 $\rightarrow$                  |                 `\rightarrow`                  |
|   $\leftarrow$    |                  $\leftarrow$                  |                  `\leftarrow`                  |
|   $\Rightarrow$   |                 $\Rightarrow$                  |                 `\Rightarrow`                  |
|   $\Leftarrow$    |                  $\Leftarrow$                  |                  `\Leftarrow`                  |
|    $\Uparrow$     |                   $\Uparrow$                   |                   `\Uparrow`                   |
|   $\Downarrow$    |                  $\Downarrow$                  |                  `\Downarrow`                  |
| $\longrightarrow$ |               $\longrightarrow$                |               `\longrightarrow`                |
| $\longleftarrow$  |                $\longleftarrow$                |                `\longleftarrow`                |
| $\Longrightarrow$ |               $\Longrightarrow$                |               `\Longrightarrow`                |
| $\Longleftarrow$  |                $\Longleftarrow$                |                `\Longleftarrow`                |
|     $\nabla$      |                    $\nabla$                    |                    `\nabla`                    |
|    $\because$     |                   $\because$                   |                   `\because`                   |
|   $\therefore$    |                  $\therefore$                  |                  `\therefore`                  |
|      $\mid$       |                     $\mid$                     |                     `\mid`                     |
|   $\backslash$    |                  $\backslash$                  |                  `\backslash`                  |
|     $\forall$     |                   $\forall$                    |                   `\forall`                    |
|     $\exists$     |                   $\exists$                    |                   `\exists`                    |
|    $\backsim$     |                   $\backsim$                   |                   `\backsim`                   |
|      $\cong$      |                    $\cong$                     |                    `\cong`                     |
|      $\oint$      |                    $\oint$                     |                    `\oint`                     |
|    $\implies$     |                   $\implies$                   |                   `\implies`                   |
|      $\iff$       |                     $\iff$                     |                     `\iff`                     |
|   $\impliedby$    |                  $\impliedby$                  |                  `\impliedby`                  |

### 连线符号

|                     显示效果                     |                 markdown公式语法                 |
| :----------------------------------------------: | :----------------------------------------------: |
|             $\overleftarrow{a+b+c}$              |             `\overleftarrow{a+b+c}`              |
|             $\overrightarrow{a+b+c}$             |             `\overrightarrow{a+b+c}`             |
|           $\overleftrightarrow{a+b+c}$           |           `\overleftrightarrow{a+b+c}`           |
|             $\underleftarrow{a+b+c}$             |             `\underleftarrow{a+b+c}`             |
|            $\underrightarrow{a+b+c}$             |            `\underrightarrow{a+b+c}`             |
|          $\underleftrightarrow{a+b+c}$           |          `\underleftrightarrow{a+b+c}`           |
|                $\overline{a+b+c}$                |                `\overline{a+b+c}`                |
|               $\underline{a+b+c}$                |               `\underline{a+b+c}`                |
|           $\overbrace{a+b+c}^{Sample}$           |           `\overbrace{a+b+c}^{Sample}`           |
|          $\underbrace{a+b+c}_{Sample}$           |          `\underbrace{a+b+c}_{Sample}`           |
|   $\overbrace{a+\underbrace{b+c}_{1.0}}^{2.0}$   |   `\overbrace{a+\underbrace{b+c}_{1.0}}^{2.0}`   |
| $\underbrace{a\cdot a\cdots a}_{b\text{ times}}$ | `\underbrace{a\cdot a\cdots a}_{b\text{ times}}` |



### 高级运算符

|              |                           显示效果                           |                       markdown公式语法                       |
| :----------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  平均数运算  |                       $\overline{xyz}$                       |                       `\overline{xyz}`                       |
| 开二次方运算 |                         $\sqrt {xy}$                         |                         `\sqrt {xy}`                         |
|   开方运算   |                        $\sqrt[n]{x}$                         |                        `\sqrt[n]{x}`                         |
| 极限运算(一) |         $\lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$         |         `\lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}`         |
| 极限运算(二) |  $\displaystyle \lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$  |  `\displaystyle \lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}`  |
| 求和运算(一) |         $\sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$         |         `\sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}`         |
| 求和运算(二) |  $\displaystyle \sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$  |  `\displaystyle \sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}`  |
| 积分运算(一) |                   $\int^{\infty}_{0}{xdx}$                   |                   `\int^{\infty}_{0}{xdx}`                   |
| 积分运算(二) |            $\displaystyle \int^{\infty}_{0}{xdx}$            |            `\displaystyle \int^{\infty}_{0}{xdx}`            |
|   微分运算   | $\frac{\partial x}{\partial y}、\frac{\partial^2x}{\partial y^2}$ | `\frac{\partial x}{\partial y}、\frac{\partial^2x}{\partial y^2}` |

### 集合运算

|            |            显示效果            |        markdown公式语法        |
| :--------: | :----------------------------: | :----------------------------: |
|    属于    |           $A \in B$            |           `A \in B`            |
|   不属于   |          $A \notin B$          |          `A \notin B`          |
|    子集    |   $x \subset y、y \supset x$   |   `x \subset y、y \supset x`   |
|   真子集   | $x \subseteq y、y \supseteq x$ | `x \subseteq y、y \supseteq x` |
|    并集    |           $A \cup B$           |           `A \cup B`           |
|    交集    |           $A \cap B$           |           `A \cap B`           |
|    差集    |        $A \setminus B$         |        `A \setminus B`         |
|    同或    |         $A \bigodot B$         |         `A \bigodot B`         |
|    同与    |        $A \bigotimes B$        |        `A \bigotimes B`        |
|    异或    |        $A \bigoplus B$         |        `A \bigoplus B`         |
|  实数集合  |          $\mathbb{R}$          |          `\mathbb{R}`          |
| 自然数集合 |          $\mathbb{Z}$          |          `\mathbb{Z}`          |

### 希腊字母

|  大写字母  | markdown语法 |  小写字母  | markdown语法 | 中文注音 |
| :--------: | :----------: | :--------: | :----------: | :------: |
|    $A$     |     `A`      |  $\alpha$  |   `\alpha`   |  阿尔法  |
|    $B$     |     `B`      |  $\beta$   |   `\beta`    |   贝塔   |
|  $\Gamma$  |   `\Gamma`   |  $\gamma$  |   `\gamma`   |   伽马   |
|  $\Delta$  |   `\Delta`   |  $\delta$  |   `\delta`   |  德尔塔  |
|    $E$     |     `E`      | $\epsilon$ |  `\epsilon`  | 伊普西龙 |
|    $Z$     |     `Z`      |  $\zeta$   |   `\zeta`    |   截塔   |
|    $H$     |     `H`      |   $\eta$   |    `\eta`    |   艾塔   |
|  $\Theta$  |   `\Theta`   |  $\theta$  |   `\theta`   |   西塔   |
|    $I$     |     `I`      |  $\iota$   |   `\iota`    |   约塔   |
|    $K$     |     `K`      |  $\kappa$  |   `\kappa`   |   卡帕   |
| $\Lambda$  |  `\Lambda`   | $\lambda$  |  `\lambda`   |  兰布达  |
|    $M$     |     `M`      |   $\mu$    |    `\mu`     |    缪    |
|    $N$     |     `N`      |   $\nu$    |    `\nu`     |    纽    |
|   $\Xi$    |    `\Xi`     |   $\xi$    |    `\xi`     |   克西   |
|    $O$     |     `O`      | $\omicron$ |  `\omicron`  | 奥密克戎 |
|   $\Pi$    |    `\Pi`     |   $\pi$    |    `\pi`     |    派    |
|    $P$     |     `P`      |   $\rho$   |    `\rho`    |    肉    |
|  $\Sigma$  |   `\Sigma`   |  $\sigma$  |   `\sigma`   |  西格马  |
|    $T$     |     `T`      |   $\tau$   |    `\tau`    |    套    |
| $\Upsilon$ |  `\Upsilon`  | $\upsilon$ |  `\upsilon`  | 宇普西龙 |
|   $\Phi$   |    `\Phi`    |   $\phi$   |    `\phi`    |   佛爱   |
|    $X$     |     `X`      |   $\chi$   |    `\chi`    |    西    |
|   $\Psi$   |    `\Psi`    |   $\psi$   |    `\psi`    |   普西   |
|  $\Omega$  |   `\Omega`   |  $\omega$  |   `\omega`   |  欧米伽  |

### 字体转换

若要对公式的某一部分字符进行字体转换，可以用 `{\font {需转换的部分字符}}` 命令，其中`\font`部分可以参照下表选择合适的字体。一般情况下，公式默认为意大利体。

|    字体    |    显示效果     |  markdown语法   |
| :--------: | :-------------: | :-------------: |
|   罗马体   |     $\rm D$     |     `\rm D`     |
|    花体    |    $\cal D$     |    `\cal D`     |
|  意大利体  |     $\it D$     |     `\it D`     |
|  黑板粗体  |    $\Bbb D$     |    `\Bbb D`     |
|    粗体    |     $\bf D$     |     `\bf D`     |
|  数学斜体  |    $\mit D$     |    `\mit D`     |
|   等线体   |     $\sf D$     |     `\sf D`     |
|   手写体   |    $\scr D$     |    `\scr D`     |
|  打字机体  |     $\tt D$     |     `\tt D`     |
| 旧德式字体 |    $\frak D$    |    `\frak D`    |
|    黑体    | $\boldsymbol D$ | `\boldsymbol D` |



## 公式



### 基本函数公式



- 行内公式：$\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt$

```bash
$\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt$
```




- 行间公式：

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt
$$

```bash
$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt
$$
```




- $y_k=\varphi(u_k+v_k)$

```bash
$y_k=\varphi(u_k+v_k)$
```




- $y(x)=x^3+2x^2+x+1$

```bash
$y(x)=x^3+2x^2+x+1$
```




- $x^{y}=(1+{\rm e}^x)^{-2xy}$

```bash
$x^{y}=(1+{\rm e}^x)^{-2xy}$
```




- $\displaystyle f(n)=\sum_{i=1}^{n}{n*(n+1)}$

```bash
$\displaystyle f(n)=\sum_{i=1}^{n}{n*(n+1)}$
```

### 分段函数



- 分段函数：

$$
y=\begin{cases}
2x+1, & x \leq0\\
x, & x>0
\end{cases}
$$

```bash
$$
y=\begin{cases}
2x+1, & x \leq0\\
x, & x>0
\end{cases}
$$
```



- 方程组：

$$
\left \{
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\
a_2x+b_2y+c_2z=d_2 \\
a_3x+b_3y+c_3z=d_3
\end{array}
\right.
$$

```bash
$$
\left \{ 
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\ 
a_2x+b_2y+c_2z=d_2 \\ 
a_3x+b_3y+c_3z=d_3
\end{array}
\right.
$$
```



### 积分

- 积分书写：

$$
\int_{\theta_1(x)}^{\theta_2(x)}=l
$$

```bash
$$
\int_{\theta_1(x)}^{\theta_2(x)}=l
$$
```



- 二重积分：

$$
\iint dx dy=\sigma
$$

```bash
$$
\iint dx dy=\sigma
$$
```



- 三重积分：

$$
\iiint dx dydz=\nu
$$

```bash
$$
\iiint dx dydz=\nu
$$
```



### 微分和偏微分

- 一阶微分方程：

$$
\frac{dy}{dx}+P(x)y=Q(x)
$$

```bash
$$
\frac{dy}{dx}+P(x)y=Q(x)
$$
```


$$
\left. \frac{{\rm d}y}{{\rm d}x} \right|_{x=0}=3x+1=1
$$

```bash
$$
\left. \frac{{\rm d}y}{{\rm d}x} \right|_{x=0}=3x+1=1
$$
```



- 二阶微分方程：

$$
y''+py'+qy=f(x)
$$

```bash
$$
y''+py'+qy=f(x)
$$
```



$$
\frac{d^2y}{dx^2}+p\frac{dy}{dx}+qy=f(x)
$$

```bash
$$
\frac{d^2y}{dx^2}+p\frac{dy}{dx}+qy=f(x)
$$
```



- 偏微分方程：

$$
\frac{\partial u}{\partial t}= h^2 \left( \frac{\partial^2 u}{\partial x^2} +\frac{\partial^2 u}{\partial y^2}+ \frac{\partial^2 u}{\partial z^2}\right)
$$

```bash
$$
\frac{\partial u}{\partial t}= h^2 \left( \frac{\partial^2 u}{\partial x^2} +\frac{\partial^2 u}{\partial y^2}+ \frac{\partial^2 u}{\partial z^2}\right)
$$
```



### 矩阵和行列式

起始标记 `\begin{matrix}` ,结束标记`\end{matrix}`,每一行末尾标记\\，行间元素之间以&分隔。在起始、结束标记处用下列词替换`matrix`。

- `pmatrix` ：小括号边框

$$
\begin{pmatrix}
1&2\\
3&4\\
\end{pmatrix}
$$

```bash
$$
\begin{pmatrix}
1&2\\
3&4\\
\end{pmatrix}
$$
```



- `bmatrix` ：中括号边框

$$
\begin{bmatrix}
1&2\\
3&4\\
\end{bmatrix}
$$

```bash
$$
\begin{bmatrix}
1&2\\
3&4\\
\end{bmatrix}
$$
```



- `Bmatrix` ：大括号边框

$$
\begin{Bmatrix}
1&2\\
3&4\\
\end{Bmatrix}
$$

```bash
$$
\begin{Bmatrix}
1&2\\
3&4\\
\end{Bmatrix}
$$
```



- `vmatrix` ：单竖线边框

$$
\begin{vmatrix}
1&2\\
3&4\\
\end{vmatrix}
$$

```bash
$$
\begin{vmatrix}
1&2\\
3&4\\
\end{vmatrix}
$$
```



- `Vmatrix` ：双竖线边框

$$
\begin{Vmatrix}
1&2\\
3&4\\
\end{Vmatrix}
$$

```bash
$$
\begin{Vmatrix}
1&2\\
3&4\\
\end{Vmatrix}
$$
```



- 无框矩阵：

$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}
$$

```bash
$$
\begin{matrix}
    1 & x & x^2 \\
    1 & y & y^2 \\
    1 & z & z^2 \\
\end{matrix}
$$
```



- 单位矩阵：

$$
\begin{bmatrix}
1&0&0\\
0&1&0\\
0&0&1\\
\end{bmatrix}
$$

```bash
$$
\begin{bmatrix}
1&0&0\\
0&1&0\\
0&0&1\\
\end{bmatrix}
$$
```



- $m \times n$矩阵：

$$
A=\begin{bmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{bmatrix}
$$

```bash
$$
A=\begin{bmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{bmatrix}
$$
```



- 行列式：

$$
D=\begin{vmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{vmatrix}
$$

```bash
$$
D=\begin{vmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{vmatrix}
$$
```



- 表格：

$$
\begin{array}{c|lll}
{}&{a}&{b}&{c}\\
\hline
{R_1}&{c}&{b}&{a}\\
{R_2}&{b}&{c}&{c}\\
\end{array}
$$

```bash
$$
\begin{array}{c|lll}
{}&{a}&{b}&{c}\\
\hline
{R_1}&{c}&{b}&{a}\\
{R_2}&{b}&{c}&{c}\\
\end{array}
$$
```



- 增广矩阵：

$$
\left[  \begin{array}  {c c | c}
1 & 2 & 3 \\
4 & 5 & 6 \\
\end{array}  \right]
$$

```bash
$$
\left[  \begin{array}  {c c | c} 
1 & 2 & 3 \\
4 & 5 & 6 \\
\end{array}  \right]
$$
```



## 案例

- `^`表示上标,` _` 表示下标。如果上下标的内容多于一个字符，需要用`{}`将这些内容括成一个整体。上下标可以嵌套，也可以同时使用。

$$
x^{y^z}=(1+{\rm e}^x)^{-2xy^w}
$$

```bash
$$
x^{y^z}=(1+{\rm e}^x)^{-2xy^w}
$$
```

其中`\rm`表示字体转换，上面有过具体说明。



- `()`、`[]`和`|`表示符号本身，使用 `\{` `\}` 来表示 {}。当要显示大号的括号或分隔符时，要用` \left` 和` \right` 命令。

$$
f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right)
$$

```bash
$$
f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right)
$$
```



- 行标的使用：在公式末尾前使用`\tag{行标}`来实现行标。

$$
f\left(
\left[
\frac{
1+\left\{x,y\right\}
}{
\left(
\frac{x}{y}+\frac{y}{x}
\right)
\left(u+1\right)
}+a
\right]^{3/2}
\right)
\tag{公式1}
$$

```bash
$$
f\left(
   \left[ 
     \frac{
       1+\left\{x,y\right\}
     }{
       \left(
          \frac{x}{y}+\frac{y}{x}
       \right)
       \left(u+1\right)
     }+a
   \right]^{3/2}
\right)
\tag{公式1}
$$
```



- 有时要用 `\left.` 或` \right.` 进行匹配而不显示本身。

$$
\left. \frac{{\rm d}u}{{\rm d}x} \right| _{x=0}
$$

```bash
$$
\left. \frac{{\rm d}u}{{\rm d}x} \right| _{x=0}
$$
```



- 添加注释文字` \text`

$$
f(n)= \begin{cases}
n/2, & \text {if $n$ is even} \\
3n+1, & \text{if $n$ is odd} \\
\end{cases}
$$

```bash
$$
f(n)= \begin{cases}
n/2, & \text {if $n$ is even} \\
3n+1, & \text{if $n$ is odd} \\
\end{cases}
$$
```



- 整齐且居中的方程式序列

$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
& = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\
& = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
& = \frac{73}{12}\sqrt{1-\frac{1}{73^2}} \\
& \approx \frac{73}{12}\left(1-\frac{1}{2\cdot73^2}\right) \\
\end{align}
$$

```bash
$$
\begin{align}
    \sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
              & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\ 
              & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
              & = \frac{73}{12}\sqrt{1-\frac{1}{73^2}} \\ 
              & \approx \frac{73}{12}\left(1-\frac{1}{2\cdot73^2}\right) \\
\end{align}
$$
```



- 在一个方程式序列的每一行中注明原因

$$
\begin{align}
v + w & = 0  & \text{Given} \tag 1 \\
-w & = -w + 0 & \text{additive identity} \tag 2 \\
-w + 0 & = -w + (v + w) & \text{equations $(1)$ and $(2)$} \\
\end{align}
$$

```bash
$$
\begin{align}
    v + w & = 0  & \text{Given} \tag 1 \\
       -w & = -w + 0 & \text{additive identity} \tag 2 \\
   -w + 0 & = -w + (v + w) & \text{equations $(1)$ and $(2)$} \\
\end{align}
$$
```



- 文字在左对齐显示

$$
\left.
\begin{array}{l}
\text{if $n$ is even:} & n/2 \\
\text{if $n$ is odd:} & 3n+1 \\
\end{array}
\right\}
=f(n)
$$

```bash
$$
    \left.
        \begin{array}{l}
            \text{if $n$ is even:} & n/2 \\
            \text{if $n$ is odd:} & 3n+1 \\
        \end{array}
    \right\}
    =f(n)
$$
```



- 连分式

$$
x = a_0 + \cfrac{1^2}{a_1 +
\cfrac{2^2}{a_2 +
\cfrac{3^2}{a_3 +
\cfrac{4^4}{a_4 +
\cdots
}
}
}
}
$$

```bash
$$
x = a_0 + \cfrac{1^2}{a_1 +
            \cfrac{2^2}{a_2 +
              \cfrac{3^2}{a_3 +
                \cfrac{4^4}{a_4 + 
                  \cdots
                }
              }
            }
          }
$$
```



- 表格

通常，一个格式化后的表格比单纯的文字或排版后的文字更具有可读性。
数组和表格均以 `\begin{array}` 开头，并在其后定义列数及每一列的文本对齐属性，`c l r `分别代表居中、左对齐及右对齐。若需要插入垂直分割线，在定义式中插入 `|` ，若要插入水平分割线，在下一行输入前插入` \hline` 。
与矩阵相似，每行元素间均须要插入` &` ，每行元素以 \\ 结尾，最后以 `\ end{array}` 结束数组。
$$
\begin{array}{c|lcr}
n & \text{左对齐} & \text{居中对齐} & \text{右对齐} \\
\hline
1 & 0.24 & 1 & 125 \\
2 & -1 & 189 & -8 \\
3 & -20 & 2000 & 1+10i \\
\end{array}
$$

```bash
$$
\begin{array}{c|lcr}
    n & \text{左对齐} & \text{居中对齐} & \text{右对齐} \\
    \hline
    1 & 0.24 & 1 & 125 \\
    2 & -1 & 189 & -8 \\
    3 & -20 & 2000 & 1+10i \\
\end{array}
$$
```
