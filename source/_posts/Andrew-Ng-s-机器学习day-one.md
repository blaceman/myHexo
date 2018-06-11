---
title: Andrew Ng's 机器学习day one
date: 2017-03-05 17:11:56
tags:
---

##### 前言: 什么是机器学习? NG老师举了几个例子.谷歌网页搜索,脸书或苹果的照片程序认出你的朋友,电子邮件中过滤垃圾邮件的反垃圾邮件...有一种科学能让计算机学习却不需要好高深的程序

##### 机器学习定义: A computer program is said to learn from experience E with respect to some class of tasks T and performance measure P, if its performance at tasks in T, as measured by P, improves with experience E.”() 

Example: playing checkers.

E = the experience of playing many games of checkers

T = the task of playing checkers.

P = the probability that the program will win the next game.


#### 机器学习的分类: Supervised learning and Unsupervised learning.(监督学习 无监督学习)


##### 监督学习:we are given a data set and already know what our correct output should look like, having the idea that there is a relationship between the input and the output.(我们已经有了一个数据集,并且已经知道正确的输出应该是怎样的,得到一个输出与输入的关系)

##### 监督学习分为:Regression(回归问题)和Classification(分类问题)

##### 回归问题:我们试图预测连续输出的结果，这意味着我们试图将输入变量映射到一些连续函数.

##### 分类问题:我们试图预测离散输出的结果。换句话说，我们试图将输入变量映射到离散类别。

Example 1:

Given data about the size of houses on the real estate market, try to predict their price. Price as a function of size is a continuous output, so this is a regression problem.
<br> 

We could turn this example into a classification problem by instead making our output about whether the house "sells for more or less than the asking price." Here we are classifying the houses based on price into two discrete categories.
<br><br>                   

Example 2:


(a) Regression - Given a picture of a person, we have to predict their age on the basis of the given picture(给定一个人的图片，我们必须根据给定的图片预测他们的年龄)
<br> 

(b) Classification - Given a patient with a tumor, we have to predict whether the tumor is malignant or benign(给定患有肿瘤的患者，我们必须预测肿瘤是恶性的还是良性的)

##### 无监督学习: Unsupervised learning allows us to approach problems with little or no idea what our results should look like. We can derive structure from data where we don't necessarily know the effect of the variables.(无监督学习允许我们在不知道看起来的结果是什么的前提下,可以从不知道变量的影响的数据中导出结构) 

##### 我们可以通过基于数据中的变量之间的关系对数据进行聚类(clustering)来导出此结构。

##### 对于无监督学习，没有基于预测结果的反馈。


Example:
<br>

Clustering: Take a collection of 1,000,000 different genes, and find a way to automatically group these genes into groups that are somehow similar or related by different variables, such as lifespan, location, roles, and so on.(获取100万个不同基因的集合，并找到一种方法来自动将这些基因分组到不同变量（如寿命，位置，角色等）中相似或相关的组中) <br>

Non-clustering: The "Cocktail Party Algorithm", allows you to find structure in a chaotic environment. (i.e. identifying individual voices and music from a mesh of sounds at a cocktail party).(“鸡尾酒会算法”，允许你在混乱的环境中找到结构（即，在鸡尾酒会上从声音网中识别单个声音和音乐）)
