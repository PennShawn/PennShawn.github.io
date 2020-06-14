---
title: mina连接建立的过程
categories: 
 - 技术
tags:
 - Java
 - mina
---

从服务端的角度来看,首先是
```

 /**
     * MINA NIO线程数量
     */
public static int MINA_PROCESSOR = Runtime.getRuntime().availableProcessors() + 1;

NioSocketAcceptor  acceptor = new NioSocketAcceptor(NetConfig.MINA_PROCESSOR);
```
进入构造方法
```
 protected AbstractPollingIoAcceptor(IoSessionConfig sessionConfig, Class<? extends IoProcessor<S>> processorClass,
            int processorCount) {
        this(sessionConfig, null, new SimpleIoProcessorPool<S>(processorClass, processorCount), true, null);
    }
```
会根据传进来的线程数创建一个`SimpleIoProcessorPool`.
```
 /**
     * Creates a new instance of SimpleIoProcessorPool with an executor
     *
     * @param processorType The type of IoProcessor to use
     * @param executor The {@link Executor}
     * @param size The number of IoProcessor in the pool
     * @param selectorProvider The SelectorProvider to used
     */
    @SuppressWarnings("unchecked")
    public SimpleIoProcessorPool(Class<? extends IoProcessor<S>> processorType, Executor executor, int size, 
            SelectorProvider selectorProvider) {
        if (processorType == null) {
            throw new IllegalArgumentException("processorType");
        }

        if (size <= 0) {
            throw new IllegalArgumentException("size: " + size + " (expected: positive integer)");
        }

        // Create the executor if none is provided
        createdExecutor = executor == null;

        if (createdExecutor) {
            this.executor = Executors.newCachedThreadPool();
            // Set a default reject handler
            ((ThreadPoolExecutor) this.executor).setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        } else {
            this.executor = executor;
        }

        pool = new IoProcessor[size];

        boolean success = false;
        Constructor<? extends IoProcessor<S>> processorConstructor = null;
        boolean usesExecutorArg = true;

        try {
            // We create at least one processor
            try {
                try {
                    processorConstructor = processorType.getConstructor(ExecutorService.class);
                    pool[0] = processorConstructor.newInstance(this.executor);
                } catch (NoSuchMethodException e1) {
                    // To the next step...
                    try {
                        if(selectorProvider==null) {
                            processorConstructor = processorType.getConstructor(Executor.class);
                            pool[0] = processorConstructor.newInstance(this.executor);
                        } else {
                            processorConstructor = processorType.getConstructor(Executor.class, SelectorProvider.class);
                            pool[0] = processorConstructor.newInstance(this.executor,selectorProvider);
                        }
                    } catch (NoSuchMethodException e2) {
                        // To the next step...
                        try {
                            processorConstructor = processorType.getConstructor();
                            usesExecutorArg = false;
                            pool[0] = processorConstructor.newInstance();
                        } catch (NoSuchMethodException e3) {
                            // To the next step...
                        }
                    }
                }
            } catch (RuntimeException re) {
                LOGGER.error("Cannot create an IoProcessor :{}", re.getMessage());
                throw re;
            } catch (Exception e) {
                String msg = "Failed to create a new instance of " + processorType.getName() + ":" + e.getMessage();
                LOGGER.error(msg, e);
                throw new RuntimeIoException(msg, e);
            }

            if (processorConstructor == null) {
                // Raise an exception if no proper constructor is found.
                String msg = String.valueOf(processorType) + " must have a public constructor with one "
                        + ExecutorService.class.getSimpleName() + " parameter, a public constructor with one "
                        + Executor.class.getSimpleName() + " parameter or a public default constructor.";
                LOGGER.error(msg);
                throw new IllegalArgumentException(msg);
            }

            // Constructor found now use it for all subsequent instantiations
            for (int i = 1; i < pool.length; i++) {
                try {
                    if (usesExecutorArg) {
                        if(selectorProvider==null) {
                            pool[i] = processorConstructor.newInstance(this.executor);
                        } else {
                            pool[i] = processorConstructor.newInstance(this.executor, selectorProvider);
                        }
                    } else {
                        pool[i] = processorConstructor.newInstance();
                    }
                } catch (Exception e) {
                    // Won't happen because it has been done previously
                }
            }

            success = true;
        } finally {
            if (!success) {
                dispose();
            }
        }
    }
```
可以看到` pool = new IoProcessor[size];`创建了一个size大小的pool,让后赋值
```
if(selectorProvider==null) {
                            pool[i] = processorConstructor.newInstance(this.executor);
                        } else {
                            pool[i] = processorConstructor.newInstance(this.executor, selectorProvider);
                        }
```
之后就可已跟着<<Mina中的观察者模式>>中的bind方法深入下去,到启动一个Acceptor线程
```
  @Override
        public void run() {
            assert acceptorRef.get() == this;

            int nHandles = 0;

            // Release the lock
            lock.release();

            while (selectable) {
                try {
                    // Process the bound sockets to this acceptor.
                    // this actually sets the selector to OP_ACCEPT,
                    // and binds to the port on which this class will
                    // listen on. We do that before the select because 
                    // the registerQueue containing the new handler is
                    // already updated at this point.
                    nHandles += registerHandles();

                    // Detect if we have some keys ready to be processed
                    // The select() will be woke up if some new connection
                    // have occurred, or if the selector has been explicitly
                    // woke up
                    int selected = select();

                    // Now, if the number of registered handles is 0, we can
                    // quit the loop: we don't have any socket listening
                    // for incoming connection.
                    if (nHandles == 0) {
                        acceptorRef.set(null);

                        if (registerQueue.isEmpty() && cancelQueue.isEmpty()) {
                            assert acceptorRef.get() != this;
                            break;
                        }

                        if (!acceptorRef.compareAndSet(null, this)) {
                            assert acceptorRef.get() != this;
                            break;
                        }

                        assert acceptorRef.get() == this;
                    }

                    if (selected > 0) {
                        // We have some connection request, let's process
                        // them here.
                        processHandles(selectedHandles());
                    }

                    // check to see if any cancellation request has been made.
                    nHandles -= unregisterHandles();
                } catch (ClosedSelectorException cse) {
                    // If the selector has been closed, we can exit the loop
                    ExceptionMonitor.getInstance().exceptionCaught(cse);
                    break;
                } catch (Exception e) {
                    ExceptionMonitor.getInstance().exceptionCaught(e);

                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e1) {
                        ExceptionMonitor.getInstance().exceptionCaught(e1);
                    }
                }
            }

            // Cleanup all the processors, and shutdown the acceptor.
            if (selectable && isDisposing()) {
                selectable = false;
                try {
                    if (createdProcessor) {
                        processor.dispose();
                    }
                } finally {
                    try {
                        synchronized (disposalLock) {
                            if (isDisposing()) {
                                destroy();
                            }
                        }
                    } catch (Exception e) {
                        ExceptionMonitor.getInstance().exceptionCaught(e);
                    } finally {
                        disposalFuture.setDone();
                    }
                }
            }
        }
```

如果有session连接
```
 @SuppressWarnings("unchecked")
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
在` S session = accept(processor, handle);`中把之前创建的`SimpleIoProcessorPool`和session绑定,然后在` session.getProcessor().add(session);`中,`session.getProcessor()`
```
 @SuppressWarnings("unchecked")
    private IoProcessor<S> getProcessor(S session) {
        IoProcessor<S> processor = (IoProcessor<S>) session.getAttribute(PROCESSOR);

        if (processor == null) {
            if (disposed || disposing) {
                throw new IllegalStateException("A disposed processor cannot be accessed.");
            }

            processor = pool[Math.abs((int) session.getId()) % pool.length];

            if (processor == null) {
                throw new IllegalStateException("A disposed processor cannot be accessed.");
            }

            session.setAttributeIfAbsent(PROCESSOR, processor);
        }

        return processor;
    }
```
根据session的id获得一个之前在pool中放的processor,然后`add(session)`
```
 /**
     * {@inheritDoc}
     */
    @Override
    public final void add(S session) {
        if (disposed || disposing) {
            throw new IllegalStateException("Already disposed.");
        }

        // Adds the session to the newSession queue and starts the worker
        newSessions.add(session);
        startupProcessor();
    }
```
然后看`startupProcessor()`方法
```
  /**
     * Starts the inner Processor, asking the executor to pick a thread in its
     * pool. The Runnable will be renamed
     */
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
如果这个processorRef是null还没有被初始化,就`processor = new Processor();`这个processor我们就很熟悉了
```
/**
     * The main loop. This is the place in charge to poll the Selector, and to
     * process the active sessions. It's done in - handle the newly created
     * sessions -
     */
    private class Processor implements Runnable {
        /**
         * {@inheritDoc}
         */
        @Override
        public void run() {
            assert processorRef.get() == this;

            lastIdleCheckTime = System.currentTimeMillis();
            int nbTries = 10;

            for (;;) {
                try {
                    // This select has a timeout so that we can manage
                    // idle session when we get out of the select every
                    // second. (note : this is a hack to avoid creating
                    // a dedicated thread).
                    long t0 = System.currentTimeMillis();
                    int selected = select(SELECT_TIMEOUT);
                    long t1 = System.currentTimeMillis();
                    long delta = t1 - t0;

                    if (!wakeupCalled.getAndSet(false) && (selected == 0) && (delta < 100)) {
                        // Last chance : the select() may have been
                        // interrupted because we have had an closed channel.
                        if (isBrokenConnection()) {
                            LOG.warn("Broken connection");
                        } else {
                            // Ok, we are hit by the nasty epoll
                            // spinning.
                            // Basically, there is a race condition
                            // which causes a closing file descriptor not to be
                            // considered as available as a selected channel,
                            // but
                            // it stopped the select. The next time we will
                            // call select(), it will exit immediately for the
                            // same
                            // reason, and do so forever, consuming 100%
                            // CPU.
                            // We have to destroy the selector, and
                            // register all the socket on a new one.
                            if (nbTries == 0) {
                                LOG.warn("Create a new selector. Selected is 0, delta = " + delta);
                                registerNewSelector();
                                nbTries = 10;
                            } else {
                                nbTries--;
                            }
                        }
                    } else {
                        nbTries = 10;
                    }
                    
                    // Manage newly created session first
                    if(handleNewSessions() == 0) {
                        // Get a chance to exit the infinite loop if there are no
                        // more sessions on this Processor
                        if (allSessionsCount() == 0) {
                            processorRef.set(null);

                            if (newSessions.isEmpty() && isSelectorEmpty()) {
                                // newSessions.add() precedes startupProcessor
                                assert processorRef.get() != this;
                                break;
                            }

                            assert processorRef.get() != this;

                            if (!processorRef.compareAndSet(null, this)) {
                                // startupProcessor won race, so must exit processor
                                assert processorRef.get() != this;
                                break;
                            }

                            assert processorRef.get() == this;
                        }
                    }

                    updateTrafficMask();

                    // Now, if we have had some incoming or outgoing events,
                    // deal with them
                    if (selected > 0) {
                        // LOG.debug("Processing ..."); // This log hurts one of
                        // the MDCFilter test...
                        process();
                    }
                    
                    // Write the pending requests
                    long currentTime = System.currentTimeMillis();
                    flush(currentTime);
                    
                    // Last, not least, send Idle events to the idle sessions
                    notifyIdleSessions(currentTime);
                    
                    // And manage removed sessions
                    removeSessions();
                    
                    // Disconnect all sessions immediately if disposal has been
                    // requested so that we exit this loop eventually.
                    if (isDisposing()) {
                        boolean hasKeys = false;

                        for (Iterator<S> i = allSessions(); i.hasNext();) {
                            IoSession session = i.next();

                            scheduleRemove((S) session);

                            if (session.isActive()) {
                                hasKeys = true;
                            }
                        }

                        wakeup();
                    }
                } catch (ClosedSelectorException cse) {
                    // If the selector has been closed, we can exit the loop
                    // But first, dump a stack trace
                    ExceptionMonitor.getInstance().exceptionCaught(cse);
                    break;
                } catch (Exception e) {
                    ExceptionMonitor.getInstance().exceptionCaught(e);

                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e1) {
                        ExceptionMonitor.getInstance().exceptionCaught(e1);
                    }
                }
            }

            try {
                synchronized (disposalLock) {
                    if (disposing) {
                        doDispose();
                    }
                }
            } catch (Exception e) {
                ExceptionMonitor.getInstance().exceptionCaught(e);
            } finally {
                disposalFuture.setValue(true);
            }
        }
```
所以,实际上,整个过程就是有1个Acceptor,n个Processor线程控制着这些session.每个Processor之后都会为它对应的那些session的消息执行filterChain的流程.
![image](http://note.youdao.com/yws/res/3166/746FEDFCF7FE440CB3D95999DEF0E461)
