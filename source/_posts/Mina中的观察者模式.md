---
title: Mina中的观察者模式
categories :
- 技术
tags :
- Java
- mina
---

Java的事件机制一般都是通过观察者模式实现的(?).
观察者模式就必须有观察者和被观察者.
Mina里里面,再一开始
`  acceptor = new NioSocketAcceptor(NetConfig.MINA_PROCESSOR);`
就在上面的父类`AbstractIoService`中注册了观察者:
```
protected AbstractIoService(IoSessionConfig sessionConfig, Executor executor) {
        if (sessionConfig == null) {
            throw new IllegalArgumentException("sessionConfig");
        }

        if (getTransportMetadata() == null) {
            throw new IllegalArgumentException("TransportMetadata");
        }

        if (!getTransportMetadata().getSessionConfigType().isAssignableFrom(sessionConfig.getClass())) {
            throw new IllegalArgumentException("sessionConfig type: " + sessionConfig.getClass() + " (expected: "
                    + getTransportMetadata().getSessionConfigType() + ")");
        }



//---------------这里!!!--------------
        // Create the listeners, and add a first listener : a activation listener
        // for this service, which will give information on the service state.
        listeners = new IoServiceListenerSupport(this);
        listeners.add(serviceActivationListener);



        // Stores the given session configuration
        this.sessionConfig = sessionConfig;

        // Make JVM load the exception monitor before some transports
        // change the thread context class loader.
        ExceptionMonitor.getInstance();

        if (executor == null) {
            this.executor = Executors.newCachedThreadPool();
            createdExecutor = true;
        } else {
            this.executor = executor;
            createdExecutor = false;
        }

        threadName = getClass().getSimpleName() + '-' + id.incrementAndGet();
    }

```
默认的listener再同一个类中类中定义为:
```
 private final IoServiceListener serviceActivationListener = new IoServiceListener() {
        public void serviceActivated(IoService service) {
            // Update lastIoTime.
            AbstractIoService s = (AbstractIoService) service;
            IoServiceStatistics _stats = s.getStatistics();
            _stats.setLastReadTime(s.getActivationTime());
            _stats.setLastWriteTime(s.getActivationTime());
            _stats.setLastThroughputCalculationTime(s.getActivationTime());

        }

        public void serviceDeactivated(IoService service) throws Exception {
            // Empty handler
        }

        public void serviceIdle(IoService service, IdleStatus idleStatus) throws Exception {
            // Empty handler
        }

        public void sessionCreated(IoSession session) throws Exception {
            // Empty handler
        }

        public void sessionClosed(IoSession session) throws Exception {
            // Empty handler
        }

        public void sessionDestroyed(IoSession session) throws Exception {
            // Empty handler
        }
    };
```
有了观察者,现在就是考虑如何再事件发生时通知观察者了.

还是在`AbstractPollingIoProcessor`中的`Processor`内部类的run方法中,(mina2.0.17)
会调用
```
     // Manage newly created session first
                    if(handleNewSessions() == 0) {
```
当有新的session连接时,新的session会加到newSessions里面,
```
 private int handleNewSessions() {
            int addedSessions = 0;

            for (S session = newSessions.poll(); session != null; session = newSessions.poll()) {
                if (addNow(session)) {
                    // A new session has been created
                    addedSessions++;
                }
            }

            return addedSessions;
        }
```
上面方法调用addNow(S)
```
 private boolean addNow(S session) {
            boolean registered = false;

            try {
                init(session);
                registered = true;

                // Build the filter chain of this session.
                IoFilterChainBuilder chainBuilder = session.getService().getFilterChainBuilder();
                chainBuilder.buildFilterChain(session.getFilterChain());

                // DefaultIoFilterChain.CONNECT_FUTURE is cleared inside here
                // in AbstractIoFilterChain.fireSessionOpened().
                // Propagate the SESSION_CREATED event up to the chain
                IoServiceListenerSupport listeners = ((AbstractIoService) session.getService()).getListeners();
                listeners.fireSessionCreated(session);
            } catch (Exception e) {
                ExceptionMonitor.getInstance().exceptionCaught(e);

                try {
                    destroy(session);
                } catch (Exception e1) {
                    ExceptionMonitor.getInstance().exceptionCaught(e1);
                } finally {
                    registered = false;
                }
            }

            return registered;
        }
```
里面
```
  IoServiceListenerSupport listeners = ((AbstractIoService) session.getService()).getListeners();
                listeners.fireSessionCreated(session);
```
listeners就是在开头代码里赋值的,而listener也在开头代码里注册了进去.
在它的`fireSessionCreated`方法里面
```
public void fireSessionCreated(IoSession session) {
        boolean firstSession = false;

        if (session.getService() instanceof IoConnector) {
            synchronized (managedSessions) {
                firstSession = managedSessions.isEmpty();
            }
        }

        // If already registered, ignore.
        if (managedSessions.putIfAbsent(session.getId(), session) != null) {
            return;
        }

        // If the first connector session, fire a virtual service activation event.
        if (firstSession) {
            fireServiceActivated();
        }

        // Fire session events.
        IoFilterChain filterChain = session.getFilterChain();
        filterChain.fireSessionCreated();
        filterChain.fireSessionOpened();

        int managedSessionCount = managedSessions.size();

        if (managedSessionCount > largestManagedSessionCount) {
            largestManagedSessionCount = managedSessionCount;
        }

        cumulativeManagedSessionCount.incrementAndGet();

        // Fire listener events.
        for (IoServiceListener l : listeners) {
            try {
                l.sessionCreated(session);
            } catch (Exception e) {
                ExceptionMonitor.getInstance().exceptionCaught(e);
            }
        }
    }
```
可以看到
```
   // Fire session events.
        IoFilterChain filterChain = session.getFilterChain();
        filterChain.fireSessionCreated();
        filterChain.fireSessionOpened();

```
sessionCreated、sessionOpened事件就是这么触发的.
sessionIdle事件在`mina的IoEvent`里面写了.
剩下的messageReceive合messageSent还是回到`AbstractPollingIoProcessor`里面,在`run`方法里面
```

                    // Now, if we have had some incoming or outgoing events,
                    // deal with them
                    if (selected > 0) {
                        // LOG.debug("Processing ..."); // This log hurts one of
                        // the MDCFilter test...
                        process();
                    }
```
`process()`方法:
```
private void process() throws Exception {
            for (Iterator<S> i = selectedSessions(); i.hasNext();) {
                S session = i.next();
                process(session);
                i.remove();
            }
        }
```
继续看`process(S session)`方法
```
 /**
         * Deal with session ready for the read or write operations, or both.
         */
        private void process(S session) {
            // Process Reads
            if (isReadable(session) && !session.isReadSuspended()) {
                read(session);
            }

            // Process writes
            if (isWritable(session) && !session.isWriteSuspended() && session.setScheduledForFlush(true)) {
                // add the session to the queue, if it's not already there
                flushingSessions.add(session);
            }
        }
```
#####1.messageReceived
就是在上面代码的`read`方法里面
```
private void read(S session) {
        IoSessionConfig config = session.getConfig();
        int bufferSize = config.getReadBufferSize();
        IoBuffer buf = IoBuffer.allocate(bufferSize);

        final boolean hasFragmentation = session.getTransportMetadata().hasFragmentation();

        try {
            int readBytes = 0;
            int ret;

            try {
                if (hasFragmentation) {

                    while ((ret = read(session, buf)) > 0) {
                        readBytes += ret;

                        if (!buf.hasRemaining()) {
                            break;
                        }
                    }
                } else {
                    ret = read(session, buf);

                    if (ret > 0) {
                        readBytes = ret;
                    }
                }
            } finally {
                buf.flip();
            }

            if (readBytes > 0) {
                IoFilterChain filterChain = session.getFilterChain();
                filterChain.fireMessageReceived(buf);
                buf = null;

                if (hasFragmentation) {
                    if (readBytes << 1 < config.getReadBufferSize()) {
                        session.decreaseReadBufferSize();
                    } else if (readBytes == config.getReadBufferSize()) {
                        session.increaseReadBufferSize();
                    }
                }
            } else {
                // release temporary buffer when read nothing
                buf.free(); 
            }

            if (ret < 0) {
                IoFilterChain filterChain = session.getFilterChain();
                filterChain.fireInputClosed();
            }
        } catch (Exception e) {
            if ((e instanceof IOException) &&
                (!(e instanceof PortUnreachableException)
                        || !AbstractDatagramSessionConfig.class.isAssignableFrom(config.getClass())
                        || ((AbstractDatagramSessionConfig) config).isCloseOnPortUnreachable())) {
                scheduleRemove(session);
            }

            IoFilterChain filterChain = session.getFilterChain();
            filterChain.fireExceptionCaught(e);
        }
    }

```
#####2.messageSent
在上面
```flushingSessions.add(session);```
加到`flushingSessions`里面.
在`run()`方法里面
```
   // Write the pending requests
                    long currentTime = System.currentTimeMillis();
                    flush(currentTime);
```
flush
```
private void flush(long currentTime) {
            if (flushingSessions.isEmpty()) {
                return;
            }

            do {
                S session = flushingSessions.poll(); // the same one with
                                                     // firstSession

                if (session == null) {
                    // Just in case ... It should not happen.
                    break;
                }

                // Reset the Schedule for flush flag for this session,
                // as we are flushing it now
                session.unscheduledForFlush();

                SessionState state = getState(session);

                switch (state) {
                case OPENED:
                    try {
                        boolean flushedAll = flushNow(session, currentTime);

                        if (flushedAll && !session.getWriteRequestQueue().isEmpty(session)
                                && !session.isScheduledForFlush()) {
                            scheduleFlush(session);
                        }
                    } catch (Exception e) {
                        scheduleRemove(session);
                        session.closeNow();
                        IoFilterChain filterChain = session.getFilterChain();
                        filterChain.fireExceptionCaught(e);
                    }

                    break;

                case CLOSING:
                    // Skip if the channel is already closed.
                    break;

                case OPENING:
                    // Retry later if session is not yet fully initialized.
                    // (In case that Session.write() is called before addSession()
                    // is processed)
                    scheduleFlush(session);
                    return;

                default:
                    throw new IllegalStateException(String.valueOf(state));
                }

            } while (!flushingSessions.isEmpty());
        }

```
flushNow
```
 private boolean flushNow(S session, long currentTime) {
            if (!session.isConnected()) {
                scheduleRemove(session);
                return false;
            }

            final boolean hasFragmentation = session.getTransportMetadata().hasFragmentation();

            final WriteRequestQueue writeRequestQueue = session.getWriteRequestQueue();

            // Set limitation for the number of written bytes for read-write
            // fairness. I used maxReadBufferSize * 3 / 2, which yields best
            // performance in my experience while not breaking fairness much.
            final int maxWrittenBytes = session.getConfig().getMaxReadBufferSize()
                    + (session.getConfig().getMaxReadBufferSize() >>> 1);
            int writtenBytes = 0;
            WriteRequest req = null;

            try {
                // Clear OP_WRITE
                setInterestedInWrite(session, false);

                do {
                    // Check for pending writes.
                    req = session.getCurrentWriteRequest();

                    if (req == null) {
                        req = writeRequestQueue.poll(session);

                        if (req == null) {
                            break;
                        }

                        session.setCurrentWriteRequest(req);
                    }

                    int localWrittenBytes;
                    Object message = req.getMessage();

                    if (message instanceof IoBuffer) {
                        localWrittenBytes = writeBuffer(session, req, hasFragmentation, maxWrittenBytes - writtenBytes,
                                currentTime);

                        if ((localWrittenBytes > 0) && ((IoBuffer) message).hasRemaining()) {
                            // the buffer isn't empty, we re-interest it in writing
                            setInterestedInWrite(session, true);

                            return false;
                        }
                    } else if (message instanceof FileRegion) {
                        localWrittenBytes = writeFile(session, req, hasFragmentation, maxWrittenBytes - writtenBytes,
                                currentTime);

                        // Fix for Java bug on Linux
                        // http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=5103988
                        // If there's still data to be written in the FileRegion,
                        // return 0 indicating that we need
                        // to pause until writing may resume.
                        if ((localWrittenBytes > 0) && (((FileRegion) message).getRemainingBytes() > 0)) {
                            setInterestedInWrite(session, true);

                            return false;
                        }
                    } else {
                        throw new IllegalStateException("Don't know how to handle message of type '"
                                + message.getClass().getName() + "'.  Are you missing a protocol encoder?");
                    }

                    if (localWrittenBytes == 0) {

                        // Kernel buffer is full.
                        if (!req.equals(AbstractIoSession.MESSAGE_SENT_REQUEST)) {
                            setInterestedInWrite(session, true);
                            return false;
                        }
                    } else {
                        writtenBytes += localWrittenBytes;

                        if (writtenBytes >= maxWrittenBytes) {
                            // Wrote too much
                            scheduleFlush(session);
                            return false;
                        }
                    }

                    if (message instanceof IoBuffer) {
                        ((IoBuffer) message).free();
                    }
                } while (writtenBytes < maxWrittenBytes);
            } catch (Exception e) {
                if (req != null) {
                    req.getFuture().setException(e);
                }

                IoFilterChain filterChain = session.getFilterChain();
                filterChain.fireExceptionCaught(e);
                return false;
            }

            return true;
        }
```
其中
```
else if (message instanceof FileRegion) {
                        localWrittenBytes = writeFile(session, req, hasFragmentation, maxWrittenBytes - writtenBytes,
                                currentTime);
```
writeFile
```
private int writeFile(S session, WriteRequest req, boolean hasFragmentation, int maxLength, long currentTime)
                throws Exception {
            int localWrittenBytes;
            FileRegion region = (FileRegion) req.getMessage();

            if (region.getRemainingBytes() > 0) {
                int length;

                if (hasFragmentation) {
                    length = (int) Math.min(region.getRemainingBytes(), maxLength);
                } else {
                    length = (int) Math.min(Integer.MAX_VALUE, region.getRemainingBytes());
                }

                localWrittenBytes = transferFile(session, region, length);
                region.update(localWrittenBytes);
            } else {
                localWrittenBytes = 0;
            }

            session.increaseWrittenBytes(localWrittenBytes, currentTime);

            if ((region.getRemainingBytes() <= 0) || (!hasFragmentation && (localWrittenBytes != 0))) {
                fireMessageSent(session, req);
            }

            return localWrittenBytes;
        }
```
里面
```
if ((region.getRemainingBytes() <= 0) || (!hasFragmentation && (localWrittenBytes != 0))) {
                fireMessageSent(session, req);
            }
```
就会出发messageSent了



###小结
可以看到,这些事件都是由`AbstractPollingIoProcessor`的内部类`Processor`的`run`方法驱动的.而`run`方法就是在
```
private void startupProcessor() {
        Processor processor = processorRef.get();

        if (processor == null) {
            processor = new Processor();

            if (processorRef.compareAndSet(null, processor)) {
                executor.execute(new NamePreservingRunnable(processor, threadName));
            }
        }

        // Just stop the select() and start it again, so that the processor
        // can be activated immediately.
        wakeup();
    }
```
的时候开始运行的,有三个方法调用`startupProcessor`:
```
1.AbstractPollingIoProcessor.add()
2.AbstractPollingIoProcessor.remove()
3.AbstractPollingIoProcessor.dispose()
```
所以,在第一个连接过来的时候,`executor`里面的线程就在跑`run`方法了.


--------
接一开始的`newSessions`,为什么有新的连接,就会有·session·加进`newSessions`里面呢?

在mina的服务器代码里面,启动mina时需要调用`acceptor.bind();`
追踪`bind`方法
```
 public final void bind(Iterable<? extends SocketAddress> localAddresses) throws IOException {
        if (isDisposing()) {
            throw new IllegalStateException("The Accpetor disposed is being disposed.");
        }

        if (localAddresses == null) {
            throw new IllegalArgumentException("localAddresses");
        }

        List<SocketAddress> localAddressesCopy = new ArrayList<SocketAddress>();

        for (SocketAddress a : localAddresses) {
            checkAddressType(a);
            localAddressesCopy.add(a);
        }

        if (localAddressesCopy.isEmpty()) {
            throw new IllegalArgumentException("localAddresses is empty.");
        }

        boolean activate = false;
        synchronized (bindLock) {
            synchronized (boundAddresses) {
                if (boundAddresses.isEmpty()) {
                    activate = true;
                }
            }

            if (getHandler() == null) {
                throw new IllegalStateException("handler is not set.");
            }

            try {
                Set<SocketAddress> addresses = bindInternal(localAddressesCopy);

                synchronized (boundAddresses) {
                    boundAddresses.addAll(addresses);
                }
            } catch (IOException e) {
                throw e;
            } catch (RuntimeException e) {
                throw e;
            } catch (Exception e) {
                throw new RuntimeIoException("Failed to bind to: " + getLocalAddresses(), e);
            }
        }

        if (activate) {
            getListeners().fireServiceActivated();
        }
    }
```
有一行
``` Set<SocketAddress> addresses = bindInternal(localAddressesCopy);````
继续追踪
```
 protected final Set<SocketAddress> bindInternal(List<? extends SocketAddress> localAddresses) throws Exception {
        // Create a bind request as a Future operation. When the selector
        // have handled the registration, it will signal this future.
        AcceptorOperationFuture request = new AcceptorOperationFuture(localAddresses);

        // adds the Registration request to the queue for the Workers
        // to handle
        registerQueue.add(request);

        // creates the Acceptor instance and has the local
        // executor kick it off.
        startupAcceptor();

        // As we just started the acceptor, we have to unblock the select()
        // in order to process the bind request we just have added to the
        // registerQueue.
        try {
            lock.acquire();

            // Wait a bit to give a chance to the Acceptor thread to do the select()
            Thread.sleep(10);
            wakeup();
        } finally {
            lock.release();
        }

        // Now, we wait until this request is completed.
        request.awaitUninterruptibly();

        if (request.getException() != null) {
            throw request.getException();
        }

        // Update the local addresses.
        // setLocalAddresses() shouldn't be called from the worker thread
        // because of deadlock.
        Set<SocketAddress> newLocalAddresses = new HashSet<SocketAddress>();

        for (H handle : boundHandles.values()) {
            newLocalAddresses.add(localAddress(handle));
        }

        return newLocalAddresses;
    }
```
里面调用了` startupAcceptor();`
```
private void startupAcceptor() throws InterruptedException {
        // If the acceptor is not ready, clear the queues
        // TODO : they should already be clean : do we have to do that ?
        if (!selectable) {
            registerQueue.clear();
            cancelQueue.clear();
        }

        // start the acceptor if not already started
        Acceptor acceptor = acceptorRef.get();

        if (acceptor == null) {
            lock.acquire();
            acceptor = new Acceptor();

            if (acceptorRef.compareAndSet(null, acceptor)) {
                executeWorker(acceptor);
            } else {
                lock.release();
            }
        }
    }
```
开启了一个线程执行`Acceptor`的`run()`方法:
```
private void startupAcceptor() throws InterruptedException {
        // If the acceptor is not ready, clear the queues
        // TODO : they should already be clean : do we have to do that ?
        if (!selectable) {
            registerQueue.clear();
            cancelQueue.clear();
        }

        // start the acceptor if not already started
        Acceptor acceptor = acceptorRef.get();

        if (acceptor == null) {
            lock.acquire();
            acceptor = new Acceptor();

            if (acceptorRef.compareAndSet(null, acceptor)) {
                executeWorker(acceptor);
            } else {
                lock.release();
            }
        }
    }
```
可以看到
```
if (selected > 0) {
                        // We have some connection request, let's process
                        // them here.
                        processHandles(selectedHandles());
                    }

```
`processHandles`方法:
```
private void processHandles(Iterator<H> handles) throws Exception {
            while (handles.hasNext()) {
                H handle = handles.next();
                handles.remove();

                // Associates a new created connection to a processor,
                // and get back a session
                S session = accept(processor, handle);

                if (session == null) {
                    continue;
                }

                initSession(session, null, null);

                // add the session to the SocketIoProcessor
                session.getProcessor().add(session);
            }
        }
```
这个方法里面调用了三个方法:
#####1.    S session = accept(processor, handle);
```
protected NioSession accept(IoProcessor<NioSession> processor, ServerSocketChannel handle) throws Exception {

        SelectionKey key = null;

        if (handle != null) {
            key = handle.keyFor(selector);
        }

        if ((key == null) || (!key.isValid()) || (!key.isAcceptable())) {
            return null;
        }

        // accept the connection from the client
        SocketChannel ch = handle.accept();

        if (ch == null) {
            return null;
        }

        return new NioSocketSession(this, processor, ch);
    }
```
new了一个session并且注册了SocketChannel
#####2. initSession(session, null, null);
```
    protected final void initSession(IoSession session, IoFuture future, IoSessionInitializer sessionInitializer) {
        // Update lastIoTime if needed.
        if (stats.getLastReadTime() == 0) {
            stats.setLastReadTime(getActivationTime());
        }

        if (stats.getLastWriteTime() == 0) {
            stats.setLastWriteTime(getActivationTime());
        }

        // Every property but attributeMap should be set now.
        // Now initialize the attributeMap.  The reason why we initialize
        // the attributeMap at last is to make sure all session properties
        // such as remoteAddress are provided to IoSessionDataStructureFactory.
        try {
            ((AbstractIoSession) session).setAttributeMap(session.getService().getSessionDataStructureFactory()
                    .getAttributeMap(session));
        } catch (IoSessionInitializationException e) {
            throw e;
        } catch (Exception e) {
            throw new IoSessionInitializationException("Failed to initialize an attributeMap.", e);
        }

        try {
            ((AbstractIoSession) session).setWriteRequestQueue(session.getService().getSessionDataStructureFactory()
                    .getWriteRequestQueue(session));
        } catch (IoSessionInitializationException e) {
            throw e;
        } catch (Exception e) {
            throw new IoSessionInitializationException("Failed to initialize a writeRequestQueue.", e);
        }

        if ((future != null) && (future instanceof ConnectFuture)) {
            // DefaultIoFilterChain will notify the future. (We support ConnectFuture only for now).
            session.setAttribute(DefaultIoFilterChain.SESSION_CREATED_FUTURE, future);
        }

        if (sessionInitializer != null) {
            sessionInitializer.initializeSession(session, future);
        }

        finishSessionInitialization0(session, future);
    }
```
对`session`各种初始化
#####3. session.getProcessor().add(session);
这里的`Processor`就是`AbstractPollingIoProcessor`
```
public final void add(S session) {
        if (disposed || disposing) {
            throw new IllegalStateException("Already disposed.");
        }

        // Adds the session to the newSession queue and starts the worker
        newSessions.add(session);
        startupProcessor();
    }
```
这里就解决了上面没有提到的
1.将`session`加到`newSessions`里面;
2.`startupProcessor();`运行了那个关键的`Processor`的`Run()`方法
```
 private void startupProcessor() {
        Processor processor = processorRef.get();

        if (processor == null) {
            processor = new Processor();

            if (processorRef.compareAndSet(null, processor)) {
                executor.execute(new NamePreservingRunnable(processor, threadName));
            }
        }

        // Just stop the select() and start it again, so that the processor
        // can be activated immediately.
        wakeup();
    }
```

