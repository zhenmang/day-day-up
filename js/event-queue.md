### Event Queue

**js单线程，遇到异步阻塞，怎么办呢？**

异步会进入Event Table，等待回调条件满足时，推入Event Queue，这时，主线程若执行完毕，会将回调拉入主线程执行

![event-queue](/imgs/event-queue.jpg)

注：如果主线程中的同步代码非常多，执行需要很长时间，会导致异步队列中，达到执行条件的回调无法及时执行，只有主线程任务结束才会执行Event Queue中存在的回调。