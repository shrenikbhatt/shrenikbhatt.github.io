---
title: Week 3 - Content I Consumed Last Week
date: 2025-01-19 12:00:00 -0400
categories: [Content Consumption and Reflection]
tags: []
---

Week 3 overview:

- Left-Right: Classical Algorithm

## Left-Right: Classical Algorithm

Today we are going to go over the Left-Right algorithm described in this [blog
post](https://concurrencyfreaks.blogspot.com/2013/12/left-right-classical-algorithm.html)
from Concurrency Freaks.

### Motivation

Imagine you and your friends are ordering food at a new restaurant. You read the
menu on the tv above the counter and choose something to order. You go to tell
the cashier your order just to get the reply that the order does not exist on
their menu. Confused you look back up and realize that one of the other workers
changed the menu and you and the cashier ended up reading different versions of
the menu at the same time.

This describes a problem faced in computer science called the Readers-Writer
Problem. You and the cashier are both readers here while the worker changing the
menu is the writer. The problem arises when both the writer and the reader are
working with the same data at the same time, causing confusion and errors in
many cases.

One existing solution for this problem is using locks. Going back to our
example, this would be the tv showing a message saying something like "An update
is in progress" and hiding the menu while it is being updated. This would ensure
that when the worker (writer) is changing data, you and the cashier (readers)
are unable to read it (writer lock). Similarly, if the worker notices someone
reading the menu, they avoid changing it (reader lock).

This solution works well but has one major problem. While the menu is being
written to, nobody can order anything! This can cause delays and frustrations
for customers who have to wait for it to come back up. You begin to wonder, is
there a way for this restaurant to update their menu without causing confusion
and also avoiding downtime? Turns out, there is!

### Wait Free Algorithms

The solution here would be to use something called a wait-free algorithm. This
sounds exactly like what it would do, allow for the update to happen without any
waiting from the readers. There are many algorithms that enable this, but today
we'll focus on a specific one called the Left-Right algorithm. This will also
focus on a single variant of the algorithm, although many exist.

In our restaurant example, we can achieve this by adding another tv beside the
original one. Only one of the tvs would be on at any given time. Also, we'll
need to assume that we can switch the state of a tv (on/off) instantly. The way
this would roughly work is that the tv that is on would show a menu and anytime
the an update needs to be made, it would be sent to the tv that is off. Now,
when we are sure nobody is reading the menu, we can swap both of the tv states
so that the tv that was on turns off and the tv that was off turns on. We also
then update the tv we just turned off to also contain the new menu. The next
time an update comes by, we would repeat the same procedure. This guarantees
that there is always a menu available for any customer (reader) and they never
have to wait for an update to finish. Since everyone reading the menu also
always reads the same version, there would be no confusion that arises either.

We can do something similar for any computer science problem dealing with the
Readers-Writer problem. The solution would involve using two variants of the
data and some logic around determining which variant is active, as well as some
logic around determining when nobody is reading the data and it is safe to
update.

### How It Works

The general idea of the Left-Right algorithm is to keep 2 instances of the data
you are working with, the left instance and the right instance. We also need the
following atomic variables:

- An indicator for determining which of the left and right instances is
currently the active (instance indicator)
- Two counters for the number of readers reading the current instance (lets call
these counter one and counter two)
- An indicator for determing which of the counters above we are currently
working with (counter indicator)

An atomic variable can be thought of as a variable that guarantees an operation
will complete in it's entirety, or not complete at all. In other words, it is
indivisible and acts as a single operation. This ensures data consistency for
the variable itself.

Let's assume that instance indicator is set to left and counter indicator is set
to one initially. Then, the algorithm would work as follows:

#### Readers

1. Increment the count of the counter specified by the counter indicator
   (counter one to begin with) by 1
2. Read the data in the instance specified by the instance indicator (left to
   begin with)
3. Once the data is read, decrement the count of the counter specified by the
   counter indicator

#### Writer

1. Write data to the opposite instance to the one specified by the instance
   indicator (right to begin with)
2. Swap the value in the instance indicator. This is done so that any new
   readers will start reading the correct value immediately
3. Wait until the current counter specified by counter indicator reaches zero
   (i.e. no readers are actively reading the unmodified instance)
4. Swap the value in counter indicator to point to the other counter and wait
   until it is zero again after the swap (in case some reader ended up getting
   through before the swap completed)
5. Write to the unmodified instance (left to begin with). Now both instances
   have the same data

Using the algorithm above, we can perform the writes without having any readers
wait for the writes to complete. We also ensure there is no ambiguity for what
data the readers are reading.

One small assumption we made above is that we assumed there is only a single
writer. To take it a step further, would this same algorithm work with multiple
writers? The answer is yes... mostly. For multiple writers, we have to ensure
that only a single writer is writing data at any given moment of time for this
algorithm. As a result, we can use a lock for only the writers. This would mean
that only the writer holding the lock can write data and perform the steps
listed above. Once they complete all 5 steps, they release the lock.

But you might be asking, why do we want to use a lock here when we just wrote an
algorithm to avoid locks? The reason has to do with the distribution of work
that is commonly seen for these types of problems. Typically, we read data way
more often than we write. As a result, we want to optimize things so that
readers have to wait as little as possible, while being okay with writers taking
longer to get their changes across.

### Conclusion

Concurrency Freaks laid out an excellent wait free algorithm that can be used to
remedy the Readers-Writer problem. The goal was to minimize the amount of time
readers spend waiting for data when there are changes actively being made to the
data. The Left-Right algorithm solves this problem very cleverly using a copy of
the data and some atomic variables (which are much cheaper to use than using
locks or other methods of concurrency control).
