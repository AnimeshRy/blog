---
layout: post
title: Sending a Secret Message 
subtitle: Three Pass Protocol Simplified
categories: Security
tags: [explained, computer-science, algorithms, web]
---

Let's say you want to send a secret message from yourself to a neighbor down the street

What would you normally do? You right the message down on a sheet of paper and put the paper into a box and hand the box to a messenger who delivers the box to your neighbor. The neighbor opens the box and reads your message.


![undraw_online_connection_6778.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1600684587727/6a53PG2Vb.png)

This was pretty straightforward but you might worry that this protocol is not very secure. 

In particular, when you hand the box to the messenger you worry that the messenger would just open the box and look at the message. Now you could lock the box before handing it to the messenger so that the messenger wouldn't be able to open the box and see what's inside. But that doesn't quite work since when your neighbor receives the box, they have no way to open it.

Asking the messenger to deliver the key and the box defeats the purpose, the messenger would just open the box with the key and eavesdrop on your communication.

So you decide that you need some protocol to send your secret message to your neighbor that doesn't involve giving keys to the messenger to unlock that message. 

This is where the **3 Pass Protocol** comes in handy. A protocol that lets you send a locked message to your neighbor without the need to hand the key to the messenger.
As the name might imply, this protocol involves passing the message back and forth through the messenger 3 times.

So let's imagine how it will work:-

![undraw_takeout_boxes_ap54.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1600685033395/PU5VCvZzf.png)

- In the first pass, you place the secret message in the box and lock it with your lock. You give the box to the messenger and hold onto your key and the messenger delivers the box to your neighbor. Your neighbor can't open the box right now but that's OK, the protocol is not over yet!

- In the second pass, your neighbor adds their lock in addition to your lock to the box and the messenger brings the box back to you with the two locks attached. Now you will remove your lock knowing the fact that the second lock will keep the box secure and send the messenger to his final pass.

- In the third and final pass, the messenger delivers the box to your neighbor with only one lock on the box, their own. The neighbor opens the lock with their key and reads the message. This was the **Three Pass Protocol**. Keep in mind that this was all done with the messenger having **NO** ability to peek into the box.

This protocol works for a few reasons, to highlight one it works as it is possible to remove locks in any order. You can remove your lock before your neighbor removes them or vice-versa.

In the digital world, there are encryption techniques that work like this too. Where you encrypt the secret message and your neighbor encrypts the result, then you can remove your encryption first before they remove theirs. In other words, the encryption is commutative.


![vpn-4046047_640.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1600685537771/aNVI272Pl.jpeg)

The **Eavesdropping Messenger** is now getting frustrated, despite their best efforts to intercept your message the three pass protocol seems to make it impossible.

Now the messenger gets an idea and realizes a vulnerability in this protocol. A vulnerability that is also present in many protocols for sending messages through a third party.

Next time when you initiate the protocol, the messenger puts his plan into action - 

In the first pass, you give the box with your lock to the messenger and tell them to deliver it to the neighbor. As per the protocol, the messenger should take the box to your neighbor to get the second lock on it but this time the messenger has his own plan. 
The messenger now adds their own lock rather than going to the neighbor and comes back to you. Now you see the second lock and think of it as the usual neighbor lock and remove your lock following the protocol.
Now the messenger has only one lock on the box their own. The messenger then opens the box and reads the message.

This type of vulnerability is called a `Man in the Middle Attack` and it shows up all over the world in computer security. Here is the [definition](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) if you need it. 

Now I have a question for you, What would you change in this protocol that would make it secure?
  
*Hint: There is no one right answer*

<!-- ## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box. -->
