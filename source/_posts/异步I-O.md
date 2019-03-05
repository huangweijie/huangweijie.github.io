title: 异步I/O
tags: 经验
date: 2017-05-07
---

**异步I/O与非阻塞I/O**

阻塞I/O：调用之后等到系统内核层面完成所有操作后调用结束，造成CPU等待I/O，浪费等待时间，CPU处理能力不能充分利用。
非阻塞I/O：调用之后立即返回，CPU时间片可以用来处理其他事务，提升性能。但其立即返回的不是完整数据，而是调用状态，应用程序需要利用轮询的办法重复调用I/O来获取完整数据和确认是否完成。（让CPU处理状态判断，浪费CPU资源）

**轮询技术**

read：最原始，性能最低。通过重复调用检查I/O状态完成。在得到最终数据前，CPU一直耗在等待上。
select：改进成通过文件描述符上事件状态来判断。有一个较弱限制，通过一个1024长度数组来存储状态，最多可同时检查             1024个文件描述符。
poll：改进成通过链表存储，性能限制有所改善，跟select类似。
epoll：Linux下效率最高的I/O事件通知机制，在进入轮询时没有检查到I/O事件时进行休眠，直到事件发生唤醒。它真实利用了事件通知、执行回调的方式，而不是遍历查询，不会浪费CPU，执行效率较高。
kqueue：该方案实现与epoll类似，不过它仅在FreeBSD系统下存在。

轮询技术满足了非阻塞I/O确保获取完整数据的需求，但对于应用程序仍然算是同步，应用程序仍然需要等待I/O完全返回。

**请求对象**

从javascript发起调用到内核执行I/O操作的过渡过程中，存在一种中间产物，叫请求对象。
javascript调用->node核心模块->c++内建模块->通过libuv进行系统调用
libuv作为封装层，有两个平台的实现（win和unix），实际上调用了uv_fs_open()方法，在其调用过程中，创建了FSReqWrap对象，javascript层传入的参数和方法封装在这个请求对象中，回调函数则设置在这个对象的oncomplete_sym属性上。
请求对象是异步I/O过程的重要中间产物，所有状态都保存在这个对象，包括送入线程池等待执行及I/O操作完毕后的回调处理。

小结
异步I/O：单线程、事件循环、观察者和I/O线程池。在Node中，javascript是单线程，node自身是多线程的，只是I/O线程使用的CPU较少。另一个需要重视的观点是，除了用户代码无法并行执行外，所有I/O（磁盘I/O和网络I/O等）是可以并行起来的。

**非I/O的异步API**

立即异步执行：
setTimeout(function(){
    //
},0);
这种方法较浪费性能
process.nextTick(function(){
    console.log('延迟执行')；
})；
console.log('正常执行')；
输出：
正常执行
延迟执行

setImmediate(function(){
    console.log('延迟执行')；
})；
console.log('正常执行')；
输出：
正常执行
延迟执行
process.nextTick()跟setImmediate()类似，但：
process.nextTick(function(){
    console.log('nextTick延迟执行')；
})；
setImmediate(function(){
    console.log('setImmediate延迟执行')；
})；
console.log('正常执行')；
输出：
正常执行
nextTick延迟执行
setImmediate延迟执行
process.nextTick()中回调函数优先级高于setImmediate()。因为事件循环对观察者的检查有先后顺序，process.nextTick()属于idle观察者，setImmediate()属于check观察者。
每一轮循环检查中，idle观察者>I/O观察者>check观察者

process.nextTick()的回调函数存在一个数组中，每轮循环全部执行完；
setImmediate()每轮只执行链表中的一个回调函数。

事件驱动和高性能服务器
经典服务器模型：
同步式：一次处理一个请求，其余请求处于等待状态；
每进程/每请求：每个请求创建一个进程，可处理多个请求，但不具备扩展性，系统资源有限；
每线程/每请求：每个请求创建一个线程，可处理多个请求，线程比进程轻量，但每个线程也占内存，对于大型站点仍然不够。





