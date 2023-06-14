---
layout: post
title: "Why Gaussian Distrbution?"
categories: project
---

Why is the Gaussian distrbution observed in chaotic situations? Why is it the most used in predictive modelling methods? We see like normal distrbutions in asset price movements, population statictics. In fact, the simplest (yet most widely used) method of linear regression is based on assumptions of normal distrbution of residuals and features. 

But why does this probability distribution so special compare to other functions that look similar? John Hershel came up with this function in 1850 while trying to design a probability distribution function. Let's take the example of a dart board and let's try to define a Probability Distribution Function (PDF) of a dart hitting a particular spot. 

If we try to give it a thought, we will need the function to be radially symmetric, i.e. depend only on the radial distance from the centre of the dart board and that the thrower is equally likely to throw a dart 2cm to the left or right of bull's eye, assuming no knowledge of the population. This is because if the thrower (or the board) is rotated by some angle, the PDF should not change. Additionally, the probability of hitting a point far from the bull's eye is less compared to hitting an area near the eye. Hence, we can write the joint PDF of x, y as 

$ f(x, y) = f(\sqrt{x^2 + y^2}) = f(r)$

The second property that we want the function to follow is that the random variables X and Y are indpenedent. This means that if we had knowledge of Y, the distrbution of X still dpeends solely on X. Now there was something here that slighly confused me (it may be obvious for you). If we look at the 2D Gaussian Distrubution, it seemed like if I cut the curve at a given Y and look at the distrbution, the shape of the distrbution seems to change with Y. That is because we are looking at a joint PDF, and not the marginal PDF of X. In fact, it follows the below property and the joing PDF, if Y is known, is a constant factor (P(Y=y)) times the PDF of X.

$ f(x, y) = h(x) . g(y)$

Since X and Y are independent and radially symmetric. we might as well assume that h and g are the same functions. This leaves us with a functional equation as below:

$ f(x, y) = f(\sqrt{x^2 + y^2}) = h(x) . h(y)$

Substituting y = 0 and x = r, we get

$ f(r, 0) = f(r) = h(r) . h(0)$

where f(0) is a constant. Setting this value as 1 for simplcity. This implies that f = h when h(0) = 1. Hence we need to solve a functional equation of the form:

$ f(\sqrt{x^2 + y^2}) = f(x) . f(y) $

Substituting $f(\sqrt{x}) = t(x)$ for x > 0, we get the below form. This implies that the function is such that it can transform a sum of variables into a product. 

$t(x^2 + y^2) = t(x^2) . t(y^2)$ 
$\implies$ $t(x^2 + y^2 + z^2) = t(x^2) . t(y^2) . t(z^2)$ 

$\implies t(y) = t(3.x) = t(x)^3$

In the above equation, if we assume x = 1, y becomes 3 and so we get $t(k) = t(1)^k$. Substituting $f(1) = e^c$ where $c$ is a constant, we get:

$t(x) = e^{cx}$

If we want the function to represent a PDF, we want the area under such a curve to be finite so we can normalize it with the area ans the area under the curve becomes 1. This means that $c < 0$. So we can rewrite the equation as below considering c as positive.

$t(x) = e^{-cx}$

$\implies f(x) = e^{-cx^2}$

In essence, this is how Hershel came up with what we call the Guassian Function. From our calculus class, we know that the integral of this function can be computed by using polar coordinates similar to the dartboard example. $c$ controls the height (or shape) of the curve here. Since the integral of this fucntion comes out to be $\sqrt{\pi}$, we see  $\sqrt{\pi}$ in the denominator of the guassian. The inherent radial symmetry of this function is an indication of why $\sqrt{\pi}$ has a role in the area under this distrbution.

Derivations of functions like these are not uncommon, wherein some inherent factors lead to the discovery of some useful functions. Another brilliant example of such a function is the Nash Welfare Function that has a lot of significance in Game Theory and market mechanisms, which is a topic for a different post.


This is inspired by the video from 3Blue1Brown about the signifciance of $\pi$ in a Guassian/Normal Distribution. I just wanted to check how well I understood it and how would I explain it to someone from my perspective, if I didnt have that video.


This sentence uses `$` delimiters to show math inline:  $\sqrt{3x-1}+(1+x)^2$

