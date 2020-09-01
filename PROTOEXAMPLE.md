# Working with protocol buffers in C++

Out of all the APIs, the C++ proto API has the best performance. The biggest drawback is that it's in C++. 

In this document. I will step through the [C++ Proto API tutorial](https://developers.google.com/protocol-buffers/docs/cpptutorial) with some snarky comments to help you chug along.


## Our Goal

We want to create an address book application. Let's break down the chunks of information we need to represent:

- Person
- Phone number
- Address book itself

Now let's create a message for each of them!

## Step 1: Define your protocol format
