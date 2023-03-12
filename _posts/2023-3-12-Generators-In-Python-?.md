---
layout: post
title: Making sense of generators, coroutines, and “yield from” in Python?
subtitle: To Do before learning asyncio!
categories: Python
tags: [Python, System, Language]
---

## Consider the following (ridiculous) Python function:

```py
def myfunc():
    return 1
    return 2
    return 3
```

I can define the function, and then run it. What do I get back?

```
>>> myfunc()
1
```

Not surprisingly, I get 1 back. That’s because Python reaches that first “return” statement and returns 1. There’s no need to continue onto the second and third “return” statements. (Actually, from Python’s perspective, those latter two statements don’t even exist; they are removed from the bytecode altogether at compilation time.)

What happens if I write my function a bit differently, using “yield” instead of “return“?

```py
def myfunc():
    yield 1
    yield 2
    yield 3
```

If I run my function now, I get the following:

```
>>> myfunc()
<generator object myfunc at 0x10a92d450>
```

That’s right: Because I used “yield” instead of “return”, running the function doesn’t execute the function body. Rather, I get back a generator object, meaning something that implements the iterator protocol. For this reason, the second kind of function (using “yield”) is called a “generator function,” although you’ll often hear people describe them as “generators.”

Because generators (i.e., the objects returned by generator functions) implement the iterator protocol, they can be put into “for” loops:

```py
for one_item in myfunc():
    print(one_item)
```

What do we get back?

```
1
2
3
```

How does this work? With each iteration, the body of the generator function is executed. If there’s a “yield” statement, then that value is returned to the “for” loop. And then, most significantly, the generator goes to sleep, pausing immediately after that “yield” statement executes. When the next iteration occurs, the function wakes up at the point where it was paused, and continues running, as if nothing at all had happened.

In other words:

A generator function, when executed, returns a generator object.
The generator object implements the iterator protocol, meaning that it knows what to do in a “for” loop.
Each time the generator reaches a “yield” statement, it returns the yielded value to the “for” loop, and goes to sleep.
With each successive iteration, the generator starts running from where it paused (i.e., just after the most recent “yield” statement)
When the generator reaches the end of the function, or encounters a “return” statement, it raises a StopIteration exception, which is how Python iterators indicate that they’ve reached the end of the line.
We can simulate this all ourselves, as follows:

```py
>>> g = myfunc()
>>> next(g)
1
>>> next(g)
2
>>> next(g)
3
>>> next(g)
StopIteration
```

The “next” built-in function is how Python asks an iterator for … well, for the next object that it wants to produce. The response to “next” can either be an object or the StopIteration exception.

This kind of generator function can be quite useful: You can use it for caching, filtering, and treating infinite (or very large) data sets in smaller chunks.

But used in this way, generator functions are for one-way communication. We can retrieve information from a generator using “next”, but we cannot interact with it, modify its trajectory, or otherwise affect its execution while it’s running. (That’s not entirely true: The “throw” method allows you to force an exception to be raised within the generator, which you can use to affect what the generator should do.)

A number of years ago, Python introduced the “send” method for generators, and a modification of how “yield” can be used. Consider this code:

```py
def myfunc():
    x = ''
    while True:
        print(f'Yielding x ({x}) and waiting…')
        x = yield x
        if x is None:
            break
        print(f'Got x {x}. Doubling.')
        x = x * 2
```

The above code looks a bit weird, in that “yield’ is on the right side of an assignment statement. This means that “yield” must be providing a value to the generator. Where is it getting that value from?

Answer: From the “send” method, which can be invoked in place of the “next” function. The “send” method works just like “next”, except that you can pass any Python data structure you want into the generator. And whatever you send to the generator is then assigned to “x”.

Now, you need to “prime” it the first time with “next”, rather than “send”. But other than that, it works just like any other generator — except that whatever you “send” will then be a part of the coroutine. As before, each invocation of next/send will execute all of the code until and including the “yield” statement.

It might seem weird, but because the “yield” is on the right side of an assignment operator, and because the right side of assignment always executes before the left side, the generator goes to sleep after returning the right side, but before assigning any value to the left side. When it wakes up, the first thing that happens is that the sent value is assigned to the left side.

Here’s how that can look:

```py
g = myfunc()
next(g)
Yielding x () and waiting...
''
g.send(10)
Got x 10 Doubling.
Yielding x (20) and waiting...
20
g.send(123)
Got x 123 Doubling.
Yielding x (246) and waiting...
g.send(None)
StopIteration
```

Now, this is admittedly pretty neat: Our coroutine hangs around, waiting for us to give it a number to dial.

For a long time, it seemed like such coroutines were solutions looking for problems. After all, what can you do with such a thing? From what I can tell, people in the Python world were excited about this sort of idea, but aside from a handful who really understood the potential, coroutines were ignored and seen as somewhere between weird and esoteric.

(I should add that the term “coroutine” has changed ts meaning somewhat in the Python world over the last few years, as the “asyncio” library has gained in popularity. I have nothing against asyncio, and have been increasingly impressed with what it does, and how it does it. But that’s not the sort of coroutine I’m talking about here. Note that asyncio’s coroutines started off as generators, and there are still many things to understand in asyncio via generators. But that’s not my topic here.)

So, where do you use a generator-based coroutine? How can you think about it?

My suggestion: Think of it as an in-program microservice. A nanoservice, if you will, available to your Python program.

Why do I say this? Because the moment that you think of it this way, what you do and don’t want to do with coroutines becomes much clearer.

Want to communicate with a database? Use a coroutine, whose local variables will stick around across queries, and can thus remain connected without using lots of ugly global variables. Send your SQL queries to the coroutine, and get back the query results.
Want to communicate with an external network service, such as a stock-market quote system? Use a coroutine, to which you can send a tuple of symbol and date, and from which you’ll receive the latest information in a dictionary.
Want to automatically translate files from one format to another? Use a coroutine, which can take input in one encoding/format and produce output in another encoding/format.
Let’s create two simple coroutines that demonstrate how this can work. First, a Pig Latin translator, which will receive strings in English and will return them translated into Pig Latin:

```py
def pl_sentence(sentence):
    output = []
for one_word in sentence.split():
    if one_word[0] in 'aeiou':
        output.append(one_word + 'way')
    else:
        output.append(one_word[1:] + one_word[0] + 'ay')
    return ' '.join(output)

def pig_latin_translator():
    s = ''
    while True:
        s = yield pl_sentence(s)
        if s is None:
            break
```

Our service coroutine is “pig_latin_translator”, which uses the “pl_sentence” function to do its translation work. Let’s fire it up:

```
> > > g = pig_latin_translator()
> > > next(g)
> > > ''
> > > g.send('this is a test')
> > > 'histay isway away esttay'
> > > g.send('hello')
> > > 'ellohay'
```

Amazing! Whenever we want to translate some English into Pig Latin, we can do so with our translator, sitting in memory and waiting to serve us. Perhaps this isn’t the most elegant or sophisticated use of coroutines, but it certainly works.

Let’s look at another example: A corporate support chatbot. You know, the sort of thing that appears on a company’s Web site, allows you to enter your complaints, and then actually helps you. No, wait — that’s science fiction; in reality, such chat bots are always unable to help, while telling you how important you are. Let’s create such an unhelpful chatbot:

```py
import random

def bad_service_chatbot():
    answers = ["We don't do that",
               "We will get back to you right away",
               "Your call is very important to us",
               "Sorry, my manager is unavailable"]
    yield "Can I help you?"
    s = ''
    while True:
        if s is None:
            break
        s = yield random.choice(answers)
```

This chatbot, as its name implies, waits for your input, and then ignores it entirely, returning a canned message meant to make you feel good about yourself and the service you’re getting. Of course, you don’t really feel good after such a conversation, but at least the company has saved on salaries, right?

But I digress.

Let’s see what happens when we run our chatbot:

```
> > > g2 = bad_service_chatbot()
> > > next(g2)
> > > 'Can I help you?'
> > > g2.send('I want to complain')
> > > "We don't do that"
> > > g2.send("No, really. I want to complain.")
> > > "Sorry, my manager is unavailable"
```

A number of years ago, Python introduced a new form of “yield”, known as “yield from“. And I have to say that the documentation and examples are … well, they make the simple case very obvious and easy to understand, but make the hard case quite difficult to understand. I hope that I can clear that up.

The basic idea is that if you have a function, it’s normal to call other functions from within it. That’s a standard technique in programming, one which allows us to write shorter, more specific functions, as well as to take advantage of abstraction.

But what if you have a generator that wants to return data from another generator, or any other iterable? You could do something like this:

```py
def wrapper(data):
    for one_item in data:
yield one_item
```

```
> > > g = wrapper('abcd')
> > > list(g)
> > > ['a', 'b', 'c', 'd']
```

In other words, we turn to “g”, our generator. And with each iteration, we ask it for its next element. What does the generator do? It invokes a “for” loop on “data”. So with each iteration, we’re asking “g”, and “g” is asking “data”. We can shorten this code with “yield from”:

```py
def wrapper(data):
    yield from data
```

```
> > > g = wrapper('abcd')
> > > list(g)
> > > ['a', 'b', 'c', 'd']
```

We got the same result, even though the body of “wrapper” is now dramatically shorter. “yield from” basically lets us outsource the “yield” to another iterable, namely “data”. Our generator is basically saying, “I don’t want to deal with this any more, so I’ll just ask data to take over from here.”

This is the simple use case for “yield from”, and it’s not really very compelling. After all, did they need to add new syntax to the language in order to reduce our “for” loops? The answer is “no.”

So what is “yield from” used for? Consider the two coroutines that we wrote above, for Pig Latin and customer service. Imagine that the companies providing these services have now merged, and that we would like to have a single in-memory service that handles both of them. In other words, we would like to have a coroutine to which we can send “1” to translate Pig Latin and “2” to get customer service.

This all sounds fine, until we realize that we’re somehow going to need to get a value from the caller’s “send” method, and then pass it along to one of our coroutines. That’s going to look rather messy, no?

And so, the real reason to use “yield from” is when you have a coroutine that acts as an agent between its caller and other coroutines. By using “yield from”, you not only outsource the yielded values to a sub-generator, but you also allow that sub-generator to get inputs from the user. For example:

```py
def switchboard():
    choice = yield "Send 1 for Pig Latin, 2 for support"
    if choice == 1:
        yield from pig_latin_translator()
    elif choice == 2:
        yield from bad_service_chatbot()
    else:
        return
```

Now, what happens if we invoke this?

```
> > > s = switchboard()
> > > next(s)
> > > 'Send 1 for Pig Latin, 2 for support'
> > > s.send(1)
> > > ''
> > > s.send('hello')
> > > 'ellohay'
> > > s.send('are you awake')
> > > 'areway ouyay awakeway'
```

Fantastic, right? We’re calling “s.send” — meaning, our messages are being sent to the switchboard coroutine. But because it has used “yield from”, our message is passed along to “pig_latin_translator”. And when the translation is done, that coroutine yields its value, which bubbles up directly to the original caller.

Of course, I can also get customer support:

```
> > > s = switchboard()
> > > next(s)
> > > 'Send 1 for Pig Latin, 2 for support'
> > > s.send(2)
> > > 'Can I help you?'
> > > s.send('hello')
> > > 'Your call is very important to us'
```

Pretty nifty, eh? But we can do even better, allowing people to go back from our sub-generator to our main one, and then choose a different one:

```py
def switchboard():
    while True:
        choice = yield "1 for PL, 2 for support, 3 to exit"
        if choice == 1:
            yield from pig_latin_translator()
        elif choice == 2:
            yield from bad_service_chatbot()
        elif choice == 3:
            return
        else:
            print('Bad choice; try again')
```

Here’s an example of how that would work:

```
> > > s = switchboard()
> > > 'Send 1 for PL, 2 for support, 3 to exit'
> > > next(s)
> > > s.send(2)
> > > 'Can I help you?'
> > > s.send('hi there')
> > > 'Sorry, my manager is unavailable'
> > > s.send('la la la')
> > > "We don't do that"
> > > s.send(None)
> > > 'Send 1 for Pig Latin, 2 for support, 3 to exit'
> > > s.send(1)
> > > ''
> > > s.send('hello')
> > > 'ellohay'
> > > s.send(None)
> > > 'Send 1 for PL, 2 for support, 3 to exit'
> > > s.send(3)
> > > StopIteration
```

### So, what have we seen here?

- Coroutines are like in-memory microservices, with state that remains across calls.
  We use “next” to prime a coroutine the first time, and then use “send” to deliver additional messages.

- If we want to provide a meta-microservice, or a coroutine that invokes other coroutines, then we can use “yield from”.

- “yield from” connects the initial “send” method with the sub-coroutine, effectively passing through the coroutine that’s using “yield from”.

- I hope that this helps you to consider when and how to use coroutines — and also how you can use “yield from” in your code in more sophisticated ways than just avoiding “for” loops.
