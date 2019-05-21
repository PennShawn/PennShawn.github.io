---
title: Mina 为每个session构建filterChain
categories :
- 技术
tags :
- Java
- mina
---

AbstractPollingIoProcessor里面有个内部类Processer,只要有第一个session过来,
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
Processer就会执行循环,循环里面有个方法`handleNewSessions() `,一直从newSessions这个Queue<S>中取poll()出session,然后addNow(session)
```
 /**
     * Loops over the new sessions blocking queue and returns the number of
     * sessions which are effectively created
     * 
     * @return The number of new sessions
     */
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
在addNow(S)中,会调用session的`chainBuilder`的`chainBuilder.buildFilterChain(session.getFilterChain());`,最后发送`listeners.fireSessionCreated(session);`事件.
```
/**
     * Process a new session : - initialize it - create its chain - fire the
     * CREATED listeners if any
     * 
     * @param session The session to create
     * @return <tt>true</tt> if the session has been registered
     */
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
`buildFilterChain`就会遍历最开始
`acceptor.getFilterChain().addLast("protocol",
                new ProtocolCodecFilter(new NetCodecFactory()));
        acceptor.getFilterChain().addLast("session", new SessionFilter());
        acceptor.getFilterChain().addLast("security", new SecurityFilter());`
这些方法构建的的`erties`,对session的filterChain进行`addLast`:
```
public void buildFilterChain(IoFilterChain chain) throws Exception {
        for (Entry e : entries) {
            chain.addLast(e.getName(), e.getFilter());
        }
    }
```
`addLast`中,会对每一个entry进行注册,
```
public synchronized void addLast(String name, IoFilter filter) {
        checkAddable(name);
        register(tail.prevEntry, name, filter);
    }
```
这里的entry就是在之前的addLast里创建的
```
/**
     * @see IoFilterChain#addLast(String, IoFilter)
     * 
     * @param name The filter's name
     * @param filter The filter to add
     */
    public synchronized void addLast(String name, IoFilter filter) {
        register(entries.size(), new EntryImpl(name, filter));
    }
```
此处的EntryImpl只需要用到name和filter
```
private final class EntryImpl implements Entry {
        private final String name;

        private volatile IoFilter filter;

        private EntryImpl(String name, IoFilter filter) {
            if (name == null) {
                throw new IllegalArgumentException("name");
            }
            
            if (filter == null) {
                throw new IllegalArgumentException("filter");
            }

            this.name = name;
            this.filter = filter;
        }
```
接上面的buildFilterChain的addLast,这里才是构建真正的filterChain,里面的entry有完整的preEntry、nextEntry、filter、nextFilter等属性.
```
 /**
     * Register the newly added filter, inserting it between the previous and
     * the next filter in the filter's chain. We also call the preAdd and
     * postAdd methods.
     */
    private void register(EntryImpl prevEntry, String name, IoFilter filter) {
        EntryImpl newEntry = new EntryImpl(prevEntry, prevEntry.nextEntry, name, filter);

        try {
            filter.onPreAdd(this, name, newEntry.getNextFilter());
        } catch (Exception e) {
            throw new IoFilterLifeCycleException("onPreAdd(): " + name + ':' + filter + " in " + getSession(), e);
        }

        prevEntry.nextEntry.prevEntry = newEntry;
        prevEntry.nextEntry = newEntry;
        name2entry.put(name, newEntry);

        try {
            filter.onPostAdd(this, name, newEntry.getNextFilter());
        } catch (Exception e) {
            deregister0(newEntry);
            throw new IoFilterLifeCycleException("onPostAdd(): " + name + ':' + filter + " in " + getSession(), e);
        }
    }
```
这里先跟更具filter和name,先new一个新的`EntryImpl`,插入到`tail`的前面.EntryImpl源码如下:
```
private final class EntryImpl implements Entry {
        private EntryImpl prevEntry;

        private EntryImpl nextEntry;

        private final String name;

        private IoFilter filter;

        private final NextFilter nextFilter;

        private EntryImpl(EntryImpl prevEntry, EntryImpl nextEntry, String name, IoFilter filter) {
            if (filter == null) {
                throw new IllegalArgumentException("filter");
            }

            if (name == null) {
                throw new IllegalArgumentException("name");
            }

            this.prevEntry = prevEntry;
            this.nextEntry = nextEntry;
            this.name = name;
            this.filter = filter;
            this.nextFilter = new NextFilter() {
                public void sessionCreated(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionCreated(nextEntry, session);
                }

                public void sessionOpened(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionOpened(nextEntry, session);
                }

                public void sessionClosed(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionClosed(nextEntry, session);
                }

                public void sessionIdle(IoSession session, IdleStatus status) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionIdle(nextEntry, session, status);
                }

                public void exceptionCaught(IoSession session, Throwable cause) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextExceptionCaught(nextEntry, session, cause);
                }

                public void inputClosed(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextInputClosed(nextEntry, session);
                }

                public void messageReceived(IoSession session, Object message) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextMessageReceived(nextEntry, session, message);
                }

                public void messageSent(IoSession session, WriteRequest writeRequest) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextMessageSent(nextEntry, session, writeRequest);
                }

                public void filterWrite(IoSession session, WriteRequest writeRequest) {
                    Entry nextEntry = EntryImpl.this.prevEntry;
                    callPreviousFilterWrite(nextEntry, session, writeRequest);
                }

                public void filterClose(IoSession session) {
                    Entry nextEntry = EntryImpl.this.prevEntry;
                    callPreviousFilterClose(nextEntry, session);
                }

                public String toString() {
                    return EntryImpl.this.nextEntry.name;
                }
            };
        }
```
最终,一个双向链表构建完成了,一般事件从head到tail,`filterClose`和`filterWrite`是从tail到head.
