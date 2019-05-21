---
title: Mina nextFilter
categories :
- 技术
tags :
- Java
- mina
---

mina中nextFilter是不一定代表的是在filterChain中的下一个filter,在 DefaultIoFilterChain中,EntryImpl是一个内部类,源码如下:
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
                /**
                 * {@inheritDoc}
                 */
                @Override
                public void sessionCreated(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionCreated(nextEntry, session);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void sessionOpened(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionOpened(nextEntry, session);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void sessionClosed(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionClosed(nextEntry, session);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void sessionIdle(IoSession session, IdleStatus status) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextSessionIdle(nextEntry, session, status);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void exceptionCaught(IoSession session, Throwable cause) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextExceptionCaught(nextEntry, session, cause);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void inputClosed(IoSession session) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextInputClosed(nextEntry, session);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void messageReceived(IoSession session, Object message) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextMessageReceived(nextEntry, session, message);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void messageSent(IoSession session, WriteRequest writeRequest) {
                    Entry nextEntry = EntryImpl.this.nextEntry;
                    callNextMessageSent(nextEntry, session, writeRequest);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void filterWrite(IoSession session, WriteRequest writeRequest) {
                    Entry nextEntry = EntryImpl.this.prevEntry;
                    callPreviousFilterWrite(nextEntry, session, writeRequest);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public void filterClose(IoSession session) {
                    Entry nextEntry = EntryImpl.this.prevEntry;
                    callPreviousFilterClose(nextEntry, session);
                }

                /**
                 * {@inheritDoc}
                 */
                @Override
                public String toString() {
                    return EntryImpl.this.nextEntry.name;
                }
            };
        }
```
可以看到,nextFilter实现了?NextFilter接口,实现的方法中,filterWrite 和 filterClose调用的entry代码 ` Entry nextEntry = EntryImpl.this.prevEntry;`,这两个调用的都是之前的entry.



