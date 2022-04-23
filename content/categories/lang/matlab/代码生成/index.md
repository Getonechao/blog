+++
title= "matlab--符号表达式转LaTeX、C、mathml代码"
description= "matlab--符号表达式转LaTeX、C、mathml代码"
date= 2022-04-18T21:58:01+08:00
author= "somebody"
draft= true
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    " matlab"
]

+++

# matlab--符号表达式转LaTeX、C、mathml代码

网址--[Code Generation - MATLAB & Simulink - MathWorks 中国](https://ww2.mathworks.cn/help/symbolic/code-generation.html)

# latex

~~~matlab
 syms f(t);
 y=f==5*exp(-0.8*t)*sin(pi*t);
 latex(y)
~~~

ans= 'f\left(t\right)=5\,{\mathrm{e}}^{-\frac{4\,t}{5}}\,\sin\left(\pi \,t\right)'

## Mathml

~~~matlab
syms f(t);
 y=f==5*exp(-0.8*t)*sin(pi*t);
 mathml(y)
~~~

<math xmlns='http://www.w3.org/1998/Math/MathML' display='block'>
       <mrow>
         <mrow>
           <mi>f</mi>
           <mrow>
             <mo form='prefix'>(</mo>
             <mrow>
               <mi>t</mi>
             </mrow>
             <mo form='postfix'>)</mo>
           </mrow>
         </mrow>
         <mo form='infix'>&equals;</mo>
         <mrow>
           <mn>5</mn>
           <mo form='infix'>&InvisibleTimes;</mo>
           <msup>
             <mo>&ee;</mo>
             <mrow>
               <mo>-</mo>
               <mfrac>
                 <mrow>
                   <mn>4</mn>
                   <mo form='infix'>&InvisibleTimes;</mo>
                   <mi>t</mi>
                 </mrow>
                 <mn>5</mn>
               </mfrac>
             </mrow>
           </msup>
           <mo form='infix'>&InvisibleTimes;</mo>
           <mrow>
             <mi>sin</mi>
             <mrow>
               <mo form='prefix'>(</mo>
               <mrow>
                 <mrow>
                   <mi>&pi;</mi>
                   <mo form='infix'>&InvisibleTimes;</mo>
                   <mi>t</mi>
                 </mrow>
               </mrow>
               <mo form='postfix'>)</mo>
             </mrow>
           </mrow>
         </mrow>
       </mrow>
     </math>

# C

~~~matlab
 1. 得到一个函数表达式
 syms f(t);
 f=5*exp(-0.8*t)*sin(pi*t);
 ccode(f)
 
 2. 矩阵
 I3 = sym(eye(3));
 I3code = ccode(I3)
 
~~~

结果

    ans ='  t0 = exp(t*(-4.0/5.0))*sin(t*3.141592653589793)*5.0;'

```
I3code =
    '  I3[0][0] = 1.0;
       I3[1][1] = 1.0;
       I3[2][2] = 1.0;'
```