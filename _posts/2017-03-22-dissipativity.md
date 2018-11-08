---
title: "Dissipativity and Passivity"
date: 2017-03-22
permalink: /posts/2017/03/dissipativity/
categories: 수학
tags: Control_Theory
excerpt: 소모성? 수동성?
---
Robust control, Nonlinear system 을 공부할 때 반드시 나오는 개념 

Dissipativity: 소모성  
Passivity: 수동성

어떤 시스템이 소모성이다, 수동성이다? 생각해보면 간단하다.  
시스템이 가지고 있는 (혹은 공급되는) 에너지를 소모하는 시스템이라 하면 *소모성*.  
시스템이 스스로 뭔가를 하지 않고 (active 하지 않고, 에너지를 만들어내지 않고) 수동적으로만 움직인다면 *수동성*이라 한다.

간단한 Spring-Mass-Damper 시스템을 살펴보자.

$$
\begin{eqnarray}
\dot{x}_1&=&x_2\\
\dot{x}_2&=&-\frac{k}{m}x_1-\frac{b}{m}x_2+\frac{f}{m}\\
y&=&x_2
\end{eqnarray}
$$

위와 같은 spring-mass-damper 시스템에서의 에너지는 아래와 같이 운동에너지와 위치에너지의 합으로 나타내진다.

$$
V=\frac{1}{2}kx^2_1+\frac{1}{2}mx^2_2
$$

이와 같이 위의 시스템이 가지고 있는 에너지 함수를 Storage function 이라고 부르기도 한다. (에너지를 저장하고 있다하여 저장함수)  
이러한 저장함수의 시간에 따른 변화량 (시간에 대한 미분)을 살펴보면

$$
\begin{eqnarray}
\dot{V}&=&\begin{bmatrix}kx_1&mx_2\end{bmatrix}\begin{bmatrix}x_2\\-\frac{k}{m}x_1-\frac{b}{m}x_2+\frac{f}{m}\end{bmatrix}\\
&=&-by^2+yf
\end{eqnarray}
$$

위의 관계식에 대해 *supply rate* 를 $$w(t)=-\tilde{b}y(t)+y(t)f(t), 0<\tilde{b}\le b$$ 이라고 정의하면, 아래와 같이 storage function과 supply rate 간의 관계를 얻어낼 수 있다. [참고](http://control.ee.ethz.ch/~apnoco/Lectures2014/dissipativity.pdf)

$$
V(t_1)-V(t_0)\le\int^{t_1}_{t_0}{w(t)}dt
$$

위 수식은 $$t_0$$ 시점부터 $$t_1$$ 시점까지의 에너지 변화량과 시스템에 공급된 에너지와의 관계를 나타냄을 알 수 있다.  
시스템의 에너지 변화량이 공급된 에너지량보다 작거나 같을 때, 우리는 해당 시스템을 소모적(dissipative) 이라고 이야기한다. 또한 위의 부등식을 dissipation inequality (소모 부등식) 이라 정의한다.

위 spring-mass-damper 시스템에서 damping coefficient 가 0일 때 $$(b=0)$$ supply rate $$w(t)=yf$$ 가 된다. 이렇게 supply rate 가 input 과 output 의 곱의 형태이며, 해당 supply rate가 위의 소모부등식을 만족할 때, 시스템이 passive 하다. 라고 이야기한다.

즉, passivity 는 dissipativity 의 특별케이스이다.

위의 예제에서는 시스템에 에너지를 소모하는 항(e.g., damping) 이 없음에도 불구하고 input 을 통해 에너지가 공급(혹은 y를 통해 소모)되지 않을 때 (u=0) Storage function 이 증가하지 않는다. 이는 시스템의 input이 bound 되어 있을 때, output도 bound 되어 있음을 이야기 하며, 이는 시스템이 안정함을 의미한다.  
전기회로로 이해하자면, 회로에 active 소자 (energy source: current, voltage)가 없고, 수동소자(passive elements)인 R,L,C 만으로 이루어진 시스템으로 이해할 수 있다. 