---
title: Week 2 - Content I Consumed Last Week
date: 2025-01-12 12:00:00 -0400
categories: [Content Consumption and Reflection]
tags: []
---

Week 2 overview:

- Why Inferable.ai Chose Long Polling Over Websockets

## Why Inferable.ai Chose Long Polling Over Websockets

Inferable.ai wrote a [blog
post](https://www.inferable.ai/blog/posts/postgres-nodejs-longpolling.mdx)
describing how their exploration of websockets led them towards a simpler
solution of using long-polling for managing real-time updates

### TLDR

- Inferable.ai wanted better real-time updates at scale for their workers and
agents
- They found that using long polling helped simplify various aspects that would
otherwise require changes, including:
  - Keeping existing observability and logging patterns
  - Avoiding added complexity to the authentication flow
  - No additional complexities related to their existing infrastructure setup
  - Limited client side changes

### The Problem

Inferable.ai had two main problems they wanted to solve in real-time:

  1. Worker nodes needed to figure out when a new job was available
  2. Agents required updates about execution state and chat state which was
     needed to stream efficiently

They wanted to solve these problems without bringing their DB instance down with
excessive querying patterns. They also preferred a solution that would not
require significant changes to their existing code structure.

### Background

Before diving into the solution that Inferable.ai used, it's important to
understand what polling is and what websockets are.

To build some imagery, imagine you (the server) are driving your friend Bob (the
client) to their favourite theme park. Unfortunately, Bob decided to sit at the
very back of your car which happens to have no windows for him to see what is
going on outside, or if you have arrived.

There are two methods for Bob to figure out whether you both have reached your
destination.

#### Method 1: "Are we there yet" AKA Polling

One way for Bob to figure out if you have arrived at your destination is by
repeatedly asking "Are we there yet" at some constant interval (say every 3
seconds). You would respond with "No" until eventually you do reach your
destination and finally respond with "Yes". This is akin to how polling would
work between a client and a server, where the client would repeatedly ask the
server for updates until the task at hand is completed.

#### Method 2: "We have arrived" AKA Websockets

The second method for Bob to figure out if you have arrived at your destination
is by you telling Bob when you have arrived, without him needing to ask. This is
how websockets would work where the server would hold a connection with the
client and notify them of updates through that connection. However, this also
means that the server would have to stay connected to the client the entire time
so that it is able to send updates to it. This adds some more complexity and
overhead as compared to Bob just asking you repeatedly about whether you would
have arrived or not (you would have to not only remember that Bob is in fact in
your car but also what kind of information he is looking for).

#### Long Polling vs. Short Polling

Inferable.ai specifically mentions using long polling. How is that different
from short polling? The main difference is that:

- Short polling would be when you respond to Bob immediately every time he asks
you "Are we there yet?" regardless if you have any updates to share or not
- Long polling would mean that once Bob asks you, you would temporarily remember
him and provide updates until you forget about him again. Then he would go on to
ask again and this would repeat until you reach your destination.

### The Solution

Inferable.ai found that they could implement long polling would provide the
benefits of real-time updates without introducing significant changes to their
existing code structure. In order to ensure that the long polling pattern worked
efficiently, they created the appropriate indexes on their DB. This ensured that
their frequent polling queries would be fast and would not add significant load
to their DB.

They also discovered some hidden benefits to their long polling solution:

- Re-using existing observability: All the existing HTTP metrics and logging
patterns worked out of the box with this solution as it did not introduce any
new protocols and state
- No additional authentication: They were able to keep their existing HTTP
authentication and existing security patterns
- Infrastructure compatibility: They did not have to worry about firewalls
blocking new websocket connections or introducing new proxy rules as long
polling used their existing HTTP solution
- No state management: Websockets would require keeping connection state.
However, long polling would be stateless since it would use HTTP under the hood
- Minimal client changes: Clients can just use their existing HTTP calls without
having to introduce any special libraries.

### Best Practices

Inferable.ai mentions various best practices for reliable operation including:

- Enforcing TTL for HTTP connections and ensuring that the polling logic always
returns within this this TTL
- Allowing clients to configure their TTL while enforcing a hard limit on the
server side
- Ensuring the TTL stays under the minimum TTL across the entire infrastructure
stack
- Adding a reasonable interval between polling to avoid piling up requests to
the DB
- Implementing exponential backoff for more efficient resource usage
