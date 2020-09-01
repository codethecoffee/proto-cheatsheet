# Working with protocol buffers in C++

Out of all the APIs, the C++ proto API has the best performance. The biggest drawback is that it's in C++. 

In this document. I will step through the [C++ Proto API tutorial](https://developers.google.com/protocol-buffers/docs/cpptutorial) with some snarky comments to help you chug along.


## Our Goal

We want to create an address book application. Let's break down the chunks of information we need to represent:

- Person
- Phone number
- Address book itself

Now let's create a message for each of them!

## Step 1: Create your protocol message

```proto2
// Specify whether you're using proto2 or proto3 at the top of the file
syntax = "proto2";

/***********/
// PACKAGE //
/***********/
// Your message will be packaged under "tutorial" now
// It's good practice to use packages to prevent naming conflicts
// between different projects. Fittingly, in C++ our generated
// classes will be placed in a namespace of the same name ("tutorial")
package tutorial;

message Person {
    // REQUIRED V.S. OPTIONAL FIELDS
    // Be careful about setting fields as required; it can be a headache
    // if the fields end up becoming optional later on. Some developers
    // just stick to making all fields optional no matter what.
    required string name = 1;
    required int32 id = 2;
    optional string email = 3;

    // ENUMS
    // It's good practice to define enums if you have fields with a
    // fixed set of values they could possibly take. Much more readable
    enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
    }

    // NESTED MESSAGE DEFINITIONS
    // You can define messages within other message definitions
    message PhoneNumber {
        required string number = 1;
        optional PhoneType type = 2 [default = HOME];
    }

    // REPEATED FIELDS
    // The repeated keyword allows us to have multiple numbers
    repeated PhoneNumber phones = 4;
}
```

For more details on the structure of a proto message, check out the general overview page!

## Step 2: Compile your proto file

Time to convert your `.proto` file to code! Type out the following command in your terminal:

`protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR path/to/file.proto`

You should see a header and implementation file in the specified destination folder.