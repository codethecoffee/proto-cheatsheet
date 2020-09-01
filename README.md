# Protocol Buffers for Dummies (who code)

Explaining protocol buffers in plain (and slightly snarky) language.

## Why do we even need Protocol Buffers

Short answer: Efficient data serialization. Take my word for it.

Long answer: You technically have other options for data serialization. Let's look at each of them and explain why they kind of suck:

- Option 1: Save the language's data structures in binary form and send that. After all, data is ultimately just a bunch of 0s and 1s. However, this approach can break in so many ways; you need to make sure the code is compiled with exactly the same memory layout, endianness, and so on. Programmers can't even agree on a text editor (read this wiki article about the infamous [editor war](https://en.wikipedia.org/wiki/Editor_war) if you want to procrastinate), so this is a lost cause.
- Option 2: Write your own algorithm to encode data items into a string ([cue the  traumatic flashbacks to Leetcode hard questions about tree serialization](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)). This is a solid option for simple data, but once your data structures become more and more complicated, you're wasting time reinventing the wheel.
- Option 3: Extensible Markup Language (XML). This is a markup language made just for this task, plus it's (somewhat) human readable! Unfortunately, XML is notoriously space-greedy, not to mention that navigating XML structures can be a headache.

The (self-declared) winner: protos!

All internal Google applications use protos for their data serialization, so that should count for something. Lots of Googlers don't even use Chromebooks (I'm a Google engineer typing this out on my Macbook), so we don't blindly adopt products just because our employer made it.

## Protoc magic: how to get code files from .proto files

TLDR: The protocol buffer compiler (also referred to as protoc) does its magic and generates the code file in whatever programming language you specify. After [you download everything you need](https://developers.google.com/protocol-buffers/docs/downloads) and have all your `.proto` files ready to go, invoke the compiler in your terminal with the command below:

`protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR path/to/file.proto`

Let's break down what the heck is going on in that line.

- `protoc`: Our handy-dandy protocol compiler!
- `--proto_path=IMPORT_PATH`: Specifies the directory to look for `.proto` files when resolving the import directives (`.proto` files can import other `.proto` files).
  - *Help, I have multiple directories to search for `.proto` files in.*  No problem, pass the `--proto_path` option as many times as you need.
  - *Huh, I saw somebody run the protocol buffer compiler without using `--proto_path`. Is it optional?* That person probably used `-I`, a short version of `--proto_path`.
- `--cpp_out=DST_DIR`: This is the **output directive**. It generates C++ code to `DST_DIR` (a.k.a. whatever directory you desire), with a class for each message type defined in your `.proto` file/s.
  - *Ew, I hate C++. How do I generate code from my `.proto` files in another language?* Switch up the output directive: `--java_out` for Java, `--python_out` for python, etc. Google protocal buffers currently support C++. C#, Dart, Go, Java, and Python.
  - *I don't see the language I'm using in the [official Google documentation](https://developers.google.com/protocol-buffers/docs/tutorials).* Beg Google to add support for your language. Personally I think the languages they support cover most usecases; consider adding a C++, Java, or python server to your project.
- `/path/to/file.proto`: File path to all the `.proto` file/s you are providing as input. You can specify multiple `.proto` files. I usually put all of my `.proto` files into one directory and use regex so I'm not typing out every single file name: `/path/to/allprotos/*.proto`.

## Example of a (heavily commented) Proto Message

Here is a `.proto` message representing the data needed in an address book. Scroll down more to see my detailed notes on different aspects of a proto message.

```proto2
// Specify whether you're using proto2 or proto3
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

## Fields

A proto message is effectively a glorified set of fields. So if you understand fields, you basically know what a proto message is.

### Field Types

All the standard simple data types work (`bool`, `int32`, `int64`, `float`, `double`, `string`...are there even any more?).

If you're handling some complicated data and those primitive types don't suffice, **you can use other proto messages as field types as well!** Notice the lines `repeated PhoneNumber phones = 4;` and `repeated Person people = 1;`; both of them are fields in the `Person` message, and use the messages `PhoneNumber` and `Person` respectively as field types. This "nesting" is what makes it so easy to represent complex data structures as proto messages.

## Required v.s. Optional Fields

You'll notice that some fields are prepended with the `optional` modifier while others have the `required` modifier. As suggested by the name, optional fields are optional and required fields are required. That's probably one of the most redundant sentences I've formed in my entire life.

If you're interested, here are some important nuances:

- `required`: If a value for this field is not provided, the message will be considered "uninitialized". Be careful; in optimized builds, they skip the `required` field check, so the message will be written anyway. You'll find out when you parse the uninitialized message (and fail at it) that the required field wasn't populated properly.
- `optional`: If an optional field is not set, a default value is used (check out the "Default Values of Fields" section in this doc). You can set your own default value for simple field types, but for more complicated ones (a.k.a. anything where you use another proto message as a data type, so the `repeated PhoneNumber phones` field in my addressbook example).

### Tag Numbers

You can assign different numbers to fields. Notice the `=1` and `=2` in my code snippet. These aren't literal values of the fields, but unique identifiers (a.k.a. "tags") for each respective field. The compiler will freak out if you try ot use the same tag for several fields. Identity theft is a serious crime.

- `1...15`: Use these tag values for frequently populated fields - they require one less byte to encode than the higher numbers. I know one byte doesn't sound like much, but it can go a long way.
- `16...2047`: Basically use these when you use up 1 through 15.

## Default Values of Fields

All fields, if not specified or unknown, will take a default value.

- `bool`: false
- `number` (`int32`, etc...): 0
- `string`: empty string
- `bytes`: empty bytes
- `enum`: first value (has a tag value of 0)
- `repeated`: empty list

## Repeated Fields

To express a "list" or an "array", create a repeated field by prepending the field name with the `repeated` keyword. This allows a field to be repeated any number of times. The order of the values if preserved in the protocol buffer.

The opposite of a repeated field is a *singular field*, but we don't explicitly specify this. It's just the default type.

```proto3
// A list of phone numbers that is optional to provide at signup
repeated string phone_numbers = 7;
```

## Enums

If you know all the values a field that take in advance, use an `Enum` type. Note that the enum value.

```proto3
// we currently consider only 3 eye colors
enum EyeColor {
    UNKNOWN_EYE_COLOR = 0; // The default value
    EYE_GREEN = 1;
    EYE_BROWN = 2;
    EYE_BLUE = 3;
}

// it's an enum as defined above
EyeColor eye_color = 8;
```

## Packages

Add a line `package my.package.name` at the top of the `.proto` file to place a protocal buffer message into a particular package.

It is very important to define the packages in which your protocol buffer messages live. When your code gets compiled, it will be placed inside the package you specified.

Packages also help to prevent name conflicts, in the same way that C++ namespaces do (`my.package.Person`). Be careful when importing protos into other protos; you need to specify the correct package name when accessing packaged Protos.

Packages help all the different languages (C++, Java, Python, etc) compile correctly from `.proto` files.
