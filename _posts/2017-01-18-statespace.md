---
title: "State Space"
date: 2017-01-18
permalink: /posts/2017/01/statespace/
categories: 수학
tags: Linear-algebra
excerpt: "상태공간?"
---

제어이론을 크게 2가지로 나누자면 고전제어, 현대제어로 나눌 수 있다.  

1. 주파수도메인을 통해 시스템의 안정성을 해석하고, 제어기를 설계하는 이론
2. 시간도메인을 통해 시스템의 안정성을 해석하고, 제어기를 설계하는 이론

전자의 경우 주로 고전제어이론이라 이야기하며, 후자를 현대제어라 이야기들 한다.

상태공간이라는 개념은 현대제어이론에서 사용되는 개념으로, 고전제어에서 다루지 않았던(고전제어에서는 시스템의 input output 관계만을 이용: Impulse response) 시스템의 현재 상태를 이용할 수 있다. 이를 통해 현대제어이론에서는 고전제어 이론과 비교할 수 없을 정도의 안정성 해석과 제어기 디자인 최적화를 이루어낼 수 있게 되었다.

상태공간을 이해하기 위해서는 먼저 상태(state)가 무엇인지 정의되어야 할 것이다.

예를들어 자동차를 제어해 원하는 위치에 서도록 제어한다고 해보자.  
이때 input 은 자동차의 엑셀과 브레이크이며 output 은 자동차의 위치이다.  
고전제어의 경우 output 및 output 의 미분(여기서는 속도) 등의 정보를 이용하여 input을 결정하지만, 현대제어에서는 output만으로는 알 수 없는 자동차의 상태(e.g., 기어의 상태, 차량의 무게 등) 또한 이용하여 input을 결정할 수 있다.

위의 예에서 볼 수있듯이, 상태라는 것은 말 그대로 시스템의 *상태* 를 의미한다.
(위치, 속도, 가속도 등이 상태가 될 수 있다.) 

상태공간(State-space)라는 것은 이러한 상태들이 가질 수 있는 모든 값의 집합(set)으로 정의된다. 

상태공간에서 상태는 위의 예제처럼 어떠한 물리적 의미를 가질수도 있지만, 특별한 의미를 가지지 않을 수도 있다. 제어기 설계의 편의를 위해 상태를 변환하는 경우(similarity transformation)가 있기 때문이다. 예를들어 \\(\frac{1}{3}\times\\)속도\\(+\frac{2}{3}\times\\)위치 를 일종의 상태로 사용하는 경우, 이 상태는 속도, 위치 중 어떠한 의미도 가지지 않는다. 

자세한 내용은 [State-space representation](https://en.wikipedia.org/wiki/State-space_representation) 및 [Similarity transformation](http://mathworld.wolfram.com/SimilarityTransformation.html) 참조
