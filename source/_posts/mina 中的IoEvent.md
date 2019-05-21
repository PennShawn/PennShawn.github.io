---
title:  mina 中的IoEvent
categories :
- 技术
tags :
- Java
- mina
---

IoEvent中
```
    public void run() {
        fire();
    }
```
他里面的run()方法回一直调用fire方法,IoFilterEvent继承了IoEvent,
每个IoFilterEvent和session绑定:
```
public IoFilterEvent(NextFilter nextFilter, IoEventType type, IoSession session, Object parameter) {
        super(type, session, parameter);

        if (nextFilter == null) {
            throw new IllegalArgumentException("nextFilter must not be null");
        }

        this.nextFilter = nextFilter;
    }
```
fire()方法如下:
```
    @Override
    public void fire() {
        IoSession session = getSession();
        NextFilter nextFilter = getNextFilter();
        IoEventType type = getType();

        if (DEBUG) {
            LOGGER.debug("Firing a {} event for session {}", type, session.getId());
        }

        switch (type) {
        case MESSAGE_RECEIVED:
            Object parameter = getParameter();
            nextFilter.messageReceived(session, parameter);
            break;

        case MESSAGE_SENT:
            WriteRequest writeRequest = (WriteRequest) getParameter();
            nextFilter.messageSent(session, writeRequest);
            break;

        case WRITE:
            writeRequest = (WriteRequest) getParameter();
            nextFilter.filterWrite(session, writeRequest);
            break;

        case CLOSE:
            nextFilter.filterClose(session);
            break;

        case EXCEPTION_CAUGHT:
            Throwable throwable = (Throwable) getParameter();
            nextFilter.exceptionCaught(session, throwable);
            break;

        case SESSION_IDLE:
            nextFilter.sessionIdle(session, (IdleStatus) getParameter());
            break;

        case SESSION_OPENED:
            nextFilter.sessionOpened(session);
            break;

        case SESSION_CREATED:
            nextFilter.sessionCreated(session);
            break;

        case SESSION_CLOSED:
            nextFilter.sessionClosed(session);
            break;

        default:
            throw new IllegalArgumentException("Unknown event type: " + type);
        }

        if (DEBUG) {
            LOGGER.debug("Event {} has been fired for session {}", type, session.getId());
        }
    }
```
所以,每当一个session产生事件时,都会触发相应IoEventType类型的处理事件.





------

同样在AbstractPollingIoProcessor的Processor.run()中
循环里面有 notifyIdleSessions(currentTime);方法
调用:
```
 public static void notifyIdleSession(IoSession session, long currentTime) {
        notifyIdleSession0(session, currentTime, session.getConfig().getIdleTimeInMillis(IdleStatus.BOTH_IDLE),
                IdleStatus.BOTH_IDLE, Math.max(session.getLastIoTime(), session.getLastIdleTime(IdleStatus.BOTH_IDLE)));

        notifyIdleSession0(session, currentTime, session.getConfig().getIdleTimeInMillis(IdleStatus.READER_IDLE),
                IdleStatus.READER_IDLE,
                Math.max(session.getLastReadTime(), session.getLastIdleTime(IdleStatus.READER_IDLE)));

        notifyIdleSession0(session, currentTime, session.getConfig().getIdleTimeInMillis(IdleStatus.WRITER_IDLE),
                IdleStatus.WRITER_IDLE,
                Math.max(session.getLastWriteTime(), session.getLastIdleTime(IdleStatus.WRITER_IDLE)));

        notifyWriteTimeout(session, currentTime);
    }
```
