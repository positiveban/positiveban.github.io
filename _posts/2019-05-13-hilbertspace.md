---
title: "Vector spaces"
categories: 수학
tags: Linear-algebra, Control-Theory
date: 2019-05-13 10:00:00
permalink: /posts/2019/05/hilbertspace/
excerpt: "Hilbert space, normed vector space."
---

# Ring
아래와 같은 2개의 연산을 만족하는 집합

(R,+) is a commutative group, i.e.,  
associative addition  
commutative addition  
additive zero and inverse  

(R,\*) is a semi-group, i.e., a\*(b\*c) = (a\*b)\*c

# Vector space
__Linear vector space X__ consist of  
* a set of elements, called vectors    
* a set of scalars in a field F  
* two operators, called vector addition and scalar multiplication


# Normed Vector space
__Normed vector spaces__ with a real-valued function, || ||, such as  
Nonnegativity  
Strict positivity  
Homogeneity  
Triangle inequality  


# Banach space
A complete normed vector space  
어떤 space 내부 혹은 경계에 빠진 점이 없는 거리 공간  

A normed vector space X is complete if every Cauchy sequence from X has a limit in X

A sequence {xn} in a normed space is said to be a Cauchy sequence if ||xn - nm||-> 0 as n,m -> infty; i.e., given epsilon>0, there is an integer N such that ||xn - xm||<epsilon for n,m >N


# Hilbert space
A Banach space with an inner product < , >  
Symmetry  
Additivity  
Homogeneity  
Strict positivity


각 space 의 중요성 및 쓰임에 대해...
