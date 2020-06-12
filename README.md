# reflections-on-crdt

When I started studying distributed databases and crdt - it was promising for many reasons:

### [ha]
1. it Seems that this is something new.
2. It seems that it's free.
3. it Seems that this will solve a lot of problems in the project and allow you to do amazing things.

I started by reading articles from Wikipedia [[10](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)] and scientific articles [[20](https://hal.inria.fr/hal-00932836/file/CRDTs_SSS-2011.pdf)] since 2011, probably everyone starts with them.

### [ha-ha]
1. It seemed very simple and clear without A PhD in CS.
2. I found that I have been using this for a long time, without knowing about it, I even wanted to replace vuex (this is a disgusting redux), with actors.
3. It seemed that you can use it without going into detail.
4. It seemed that there are many solutions that simply work out of the box.

Already knowing that you can make a composition from CRDT. And knowing what patch theory is [[30](https://www.researchgate.net/publication/258521116_A_Categorical_Theory_of_Patches)], Difference synchronization [[40](https://static.googleusercontent.com/media/research.google.com/ru//pubs/archive/35605.pdf)] and operational transformation [[50](https://www.researchgate.net/publication/301282694_Operational_Transformation_In_Co-Operative_Editing)]
I read about JSON-CRDT [[60](https://www.cl.cam.ac.uk/~arb33/papers/KleppmannBeresford-CRDT-JSON-TPDS2017.pdf)].

It was obvious that it was not possible to Express intent using state casts, which meant that rollback/apply/replicate operations could not be performed on a state.

### [ha-ha-ha]
1. CRDT composition and unified CRDT Seemed to be the right way to go.
2. it Seemed that expressing operations as data in the form of messages in queues, rather than events that record changes, could Express intentions that could be replicated.

After a while, I decided to check out the video content on YouTube, recordings of speeches at conferences are a good way to get to know the world around me.
Naturally, I looked at Viktor Grishchenko's Russian-language report [[70](https://www.youtube.com/watch?v=1ddm7WCMclA&t=4s)], there I learned about MVCC and read several articles about different databases and approaches to implementing transactions.
There were many references to very ancient research in the DB articles. It turns out that to understand CRDT properly, you need to understand such things as temporal logic, Allen algebra, parallelism, and a huge number of algorithms and data structures on graphs. Yes, this is not what I expected in the beginning. I even had to look at the lock-free hash tables and CAS, because they are built on similar principles as CRDT.

### [ha]
1. It turned out that CRDT is a new take on something old.
2. it Turned out that not only do you need to choose the implementation of the CRDT itself, but depending on the choice, you may need a reliable transport without loss, duplication, reordering, and a bunch of restrictions.
3. it Turned out that CRDT is a problem, and if you want to do amazing things, you will have to suffer.

### [ha-ha]
1. It is better to first obtain a PhD.
2. My implementation was simple(it was an observable Set CRDT), and this is what I wanted to get rid of or improve when I found out about CRDT. I still combine it with patches and OT, and I'll tell you why.
3. it Turned out that this must be well understood to maintain.
4. it Turned out that to make a good CRDT, you need to do operations from the subject area, which means that ready-made solutions will not work out of the box.

### [ha-ha-ha]
1. the Composition leads to a strong complication, imagine that you have 2 arrays next to each other, and they have different methods. I don't think you'll like this abstraction.
2. Yesterday I wanted an iPhone and put it in the trash from my laptop. Today I want a Samsung and put it in the basket with the tablet. My wife went to the laptop-there are 2 phones in the trash.
  1. Intentions may change over time. Operations determine intent only at the time of execution.
  2. The lack of surgery also means.
  3. Almost all operations have context (and when not? - With a fixed present, the future is independent of the past.). you can of course make assumptions for things that do not have serious consequences in case of misinterpretation of intentions when merging, such as likes in social networks.

## What practical conclusions have I drawn so far?

  If errors in the interpretation of the transaction intent on the site cost money(haha, I know as many as two shopping baskets where you can buy more because of crdt).
  Then you can't make an optimistic update between replicas. You should always do as in git, the master branch, a branch for each client and the ability, or need, to make changes by the client.
  
  Conditionally, if you have a shopping cart, it is more logical to add a button like " I Think you added products to the cart from another device, add?" and "Reject".
  
  Naturally, in complex systems, especially financial systems, you should provide a merge interface, only offering the default merge implementation. Thanks to crdt and additional knowledge from the subject area, you can almost always tell the user exactly where you don't have enough knowledge to automatically merge.

  The time on replicas converges when storing the log, so you can better organize the log of operations.
  
  CRDT allows you to do a lot of useful things, but let the user decide that 10+4 is +4 or =14, not the developer.
  
  Sometimes you need to change the crdt value at the object level(i.e., make value object, not entity), not properties, because conditionally in the list of tasks, it may turn out that we passed the task as done, but changed the name on another replica, and in the end we marked the wrong task.
