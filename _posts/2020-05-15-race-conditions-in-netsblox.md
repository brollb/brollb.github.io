---
layout: post
title:  "Intro to Race Conditions in NetsBlox"
date:   2020-05-15 14:34:00
categories: general netsblox
---

## Background
Sharing can be tough for both young children and parallel applications. In both cases, a lack of coordination can result in undesirable results. Although they may be equally challenging, I will restrict this blog post to focus on the latter case. Perhaps one of the most significant challenges can occur around race conditions. A race condition (or data race) can occur when two or more agents/processes try to access the same data at the same time. This post will be focusing specifically on both high and low-level data races and will show examples in [NetsBlox](https://netsblox.org).

### Cloud Variables
The examples shown here will be using NetsBlox's cloud variables. Cloud variables can be accessed using the RPC blocks under the *Network* tab.

<center><img src="/images/cloud-variables.png" style="padding: 25px 25px 25px 25px; width: 500px"/></center>

(More information about the RPCs can be accessed by right clicking the block and selecting "help.")

Cloud variables, as the name suggests, can be used to set variables in the cloud that can be shared across users and projects. Variables can also be password-protected to ensure that no other users or projects accidentally read or edit variable values (recommended).

**Why do we need to use cloud variables? Can't we create a data race using concurrency on a local program?**

Although it is possible to create a data race without using any networking capabilities, it is a bit challenging and the resulting examples can be a bit contrived. This is due to the concurrency model used by Scratch, Snap!, and NetsBlox. In these environments, concurrent scripts actually share a single process and simply switch between the currently executing script. This context switching occurs according to some pretty natural rules (as demonstrated by the ability to write programs without direct instruction on these mechanisms!). This includes yielding control of the current process
- at the end of loops
- during any *wait*-ing block
- synchronous network requests
This ensures that any data race must cross one of these boundaries (not terribly natural).

Cloud variables, on the other hand, do not have any mechanism for ensuring that only one user is running at a single time (nor would we want that!). Therefore, it can be relatively easy to encounter a race condition when many users are using the same variables and not using any coordination mechanisms.

## Low-level Data Races
A *low-level data race* occurs when users try to read and write the same variable in parallel. For example, suppose we wanted to make an application to poll users for their height. We could first make a project which simply asks the users for their height and then adds it to a list stored in a cloud variable:

<center><img src="/images/race-conditions/naive-height-poll.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

As the variable does not exist, you will likely encounter the following error: `cdr isn't a list: Variable not found`. This is easily fixed by adding some error checking:

<center><img src="/images/race-conditions/naive-height-poll-check-errs.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

Great! Now the program seems to work and you can add lots of heights to the list! However, if we try to have multiple people do this at once, we will encounter some issues. If you are having a hard time seeing the issue, try the following:
- Open the project in two different browser windows
- Press the green flag in both
- Enter the height information in each
- Check the value of the `heights` cloud variable

When multiple people get and set the same variable, the second user is actually overwriting the first! This issue may feel familiar if you have every collaborated on a document via email or dropbox :) Each user has to make sure they have the latest version of the document before making edits. If two users download the document from dropbox, edit it, and then save them over the shared version, the second copy will overwrite the edits made by the first user - just like in our example. This is essentially a low-level data race.

In my experience, one way to get around this when collaborating on a document is to simply only have one person edit the document at a time. This approach also works in parallel programming and is called a *mutex*, *mutex lock*, or *mutual exclusion object*. A *mutex* ensures that only a single thing can access a given resource at a given time. In NetsBlox, this is possible using the *lockVariable* and *unlockVariable* RPCs. Variables can be locked only by users that have access to the variable (ie, know the password). Also, as locked variables can only be accessed by the person who acquired (created) the lock, it is best to minimize the amount of time a variable spends locked so that other programs are not kept waiting. To prevent indefinite waiting for a lock to be released, locks in NetsBlox are released after 5 seconds.

Using a *mutex*, we can now fix our programming by locking the variable that we are about to use:

<center><img src="/images/race-conditions/height-poll-with-locks.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

Since we would like to minimize the amount of time we have the variable locked, we should probably move the `ask` block to be called outside of the lock - in its current state, the lock could expire if the user is slow to respond! Our final code is now:

<center><img src="/images/race-conditions/height-poll.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

## High-level Data Races
A *high-level data race* occurs when the individual programs are coordinating properly for the individual getting and setting of variables but the sequence of these accesses can still result in undesirable results. An easy example of this occurs when some data can be accessed in batch or individually as interspersing individual and batch operations could result in a blend of the final result. Consider an example in a textual programming language such as Java. Suppose we would like to set the coordinates of a point with the following functions: `setX`, `setY`, and `setXAndY`. These functions are setting shared values and are automatically acquiring a lock (ie, calling something like `lockVariable` automatically). If one user calls `setXAndY` while another calls `setX` and `setY` in parallel, the resulting X and Y values could result in a point which consists of values from each individual user (but not setting it to either specified value). Concretely, we could imagine one user wants to set the point to (1, 1) using the `setX` and `setY` functions while another user wants to set the point to (2, 2) using the `setXAndY` function. These calls could be executed as follows:
- `setX(1)` (first user)
- `setXAndY(2, 2)` (second user)
- `setY(1)` (first user)

In this example, the final X and Y values for the point would be 2,1 - a completely different point from what was desired by either user!

### NetsBlox Example
In NetsBlox, we cannot demonstrate this particular example as data cannot be shared between different cloud variables. However, we can construct a somewhat similar example of a high-level data race by extending the previous example to also prompt the user for his or her weight.

We could start by extending the earlier application and simply duplicating the code from before to ask for the weight as well.

<center><img src="/images/race-conditions/height-weight-poll.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

Suppose we would also like to visualize the responses and plot them. We might do so using the "Chart" service as follows (using the "Matrix Operations" and "Structured data" libraries):

<center><img src="/images/race-conditions/visualize-results.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

This code results in the following figure where each point shows a given responder's height and weight:

<center><img src="/images/race-conditions/figure.png" style="padding: 25px 25px 25px 25px; width: 500px"/></center>

This example demonstrates an assumption that we are making about our data: a single responder's height and weight are in the same position in each list. That is, if the first value in `heights` is 65 and we would like to know the weight for this responder, we can check the first value in `weights`. This is similar to the earlier example using points as their is an underlying relationship between the values being set. In the earlier example, we were looking at the X and Y values of a single point whereas in this example, we are considering the weight and height of a single person (given by the position in the list).

However, is this assumption correct? Can we guarantee that we can get the specific person's weight by getting the value in the same position from the `weights` list?

Given that this example is designed to demonstrate a high-level race condition, it is probably obvious that this is not correct :)
This can be demonstrated similar to the exploit in the first example. Two responders - which I will refer to as "responder 1" and "responder 2", could perform the following actions:
- Enter height (responder 1)
- Enter height (responder 2)
- Enter weight (responder 2)
- Enter weight (responder 1)

#### How do I fix this?
If we recall, the core issue arose from this relationship between these two values which may not be correct. Namely, the heights and weights could be matched by position (similar to `zip` in a variety of other languages) and each pair would correspond to a single responder. However, this relationship is not enforced anywhere programmatically and each list could be edited independently.

The easiest fix is to simply store the values in a single variable. This will ensure that the height and weight lists will be edited together and will not get out of sync due to independent edits.

<center><img src="/images/race-conditions/height-weight-poll-final.png" style="padding: 25px 25px 25px 25px; width: 600px"/></center>

