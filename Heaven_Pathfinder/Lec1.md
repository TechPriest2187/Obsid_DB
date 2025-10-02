- Reading the presentation literature is highly encouraged

### Concurrent programming
- A program is concurrent if it has more than 1 flow control(Thread)
- Initially only 2 threads run: main and the system thread
- Each thread thinks that it is executed on a separate processor, even though a prcessor handles 2+ threads
- A thread can be implemented in two ways: implementing interface Runnable and extending class Thread. The difference is, using Runnable interface leaves us a slot to extend some class, if needed, and implement other interfaces as well. We should use extending Thread class only if we don't need inheritance from another class.