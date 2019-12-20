---
title: Flux
date: 2018-11-28 15:44:55
tags:
---

View 中的用户和应用的交互事件被actions捕获后，发送给dispatcher，dispatcher会将其发送到已注册的store，store的数据发生变化，触发change事件，view获取新数据并重新呈现

Dispatcher: 接受actions将其发送到已注册的store，每个store都会收到；每个应用中应该只有一个单独的dispatcher

Store: 保存应用数据。向应用的dispatcher注册，以便可以接受actions。数据只能通过响应actions进行变异。决定想要回应的行动。每次数据变化，都必须发出change事件。每个应用中应该有许多store

Actions: 定义应用的内部API。捕获了任何可能与应用交互的方式。是具有type字段和一些数据的简单对象。应该是对所发生行动的语义和描述，不应该描述实施细节。所有store都会收到actions

View: store中的数据显示在view中，可以使用任何框架。当view使用来自store的数据时，还必须订阅来自store的change事件。当store发出change时，view可以获取新数据并重新呈现。如果某个组件曾经使用过store并且没有订阅，那么可能会有一个微妙的错误等待找到。当用户与应用界面的各个部分交互时，通常会从view中dispatch actions

 