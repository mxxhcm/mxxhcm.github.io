---
title: network and protocol
date: 2020-01-03 20:03:53
tags:
 - 网络
 - 协议
categories: 网络
---


## 主机（端系统）
各种终端设备，笔记本，手机，等等。

## 通信链路和分组交换机(packet switch)
端系统通过通信链路和分组交换机连接在一起。

## 公共因特网
公共因特网是指一个特定的网络，通常被称为因特网。

## 因特网服务提供商(ISP)
主机通过Internet Service Provider(ISP)接入因特网，ISP提供的是链路接入，不提供内容。

## 协议(Protocol)
定义了在两个或者多个通信实体之间交换报文的格式和次序，以及在报文传输/接收或其他方面上所采取的动作。

## 传输控制协议(TCP)
Transmission Control Protocol，

## 网际协议(IP)
Internet Prococol，定义了在路由器和端系统中发送和接收的packet的格式。

## RFC(Request For Comment)
RFC是一个标准文档，其中包含很多协议的制定。

## 分组(packet)交换
应用程序之间叫交换报文(message)，通常源主机会将message划分为更小的分组(packet)。在源主机和目的主机之间，每个packet都通过通信链路和packet交换机传送。

## 存储转发传输和输出缓存
多数分组交换机在链路的输入端使用存储转发传输(store-and-forward-tranmission)，在交换机开始向输出链路传送该packet的第一个bit之前，它必须接收到整个packet。这个过程存在一个存储转发时延。
每个交换机都有多条链路和它相连，对于每条相连的链路，这个分组交换机有一个输出缓存(output buffer)，也叫输出队列(output deque)，用于存放路由器准备发往的那条链路的packet。如果这个输出链路被其他packet占用了，这个packet需要等待。这个过程存在一个排队时延。

这些时延的变换和网络用塞水平相关，如果输出缓存满了，就会出现丢包(packet loss)。

## 时延
除了存储转发时延（传输时延），排队时延，还有节点处理时延，和传播时延，这些时延加起来是节点总时延。

##