# Protocol Buffers for Dummies

Explaining protocol buffers in plain (and slightly snarky) language.

## Why we even need Protocol Buffers

Short answer: efficient data serialization. Take Google's word for it.

Long answer: You technically have other options for data serialization. Here's why using protos is better ([at least it is according to Google](https://developers.google.com/protocol-buffers/docs/cpptutorial), who is admittedly be a little biased since they are the ones who made protos):

- Save the language's data structures in binary form and send that. All data is ultimately a bunch of 0s and 1s, after all. However, this can break in so many ways, because you need to make sure the code is compiled with exactly the same memory layout, endianness, and so on. Programmers can't even agree on the same code editor, so this is a lost cause.
- Write your own algorithm to encode data items into a string ([traumatic flashbacks to Leetcode hard questions about tree serialization](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)). This is a solid option for simple data, but once your data structures become more and more complicated, you're just wasting time reinventing the wheel.
- Extensible Markup Language (XML). This is a markup language made for data serialization. Unfortunately, it's notoriously space-greedy, not to mention that navigating XML structures is super complicated.

Thus, the final conclusion is that protos are easier to use and more space efficient than other data serialization options out there! All internal Google applications use protos for their data serialization, so that should count for something. Trust me, lots of Googlers don't even use Chromebooks (I'm a Google engineer typing this sentence out on my Macbook), so we don't blindly adopt products just because our employer made it.

## How to generate actual code from `.proto` files

TLDR: The protocol buffer compiler (also referred to as protoc) does its magic and generates the code file in whatever programming language you specify. After [downloading everything you need](https://developers.google.com/protocol-buffers/docs/downloads), and have all your `.proto` files ready to go, invoke the compiler in your terminal with the command below:

`protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR path/to/file.proto`

Let's break down what's going on in that command.

- `protoc`: Our handy-dandy protocol compiler!
- `--proto_path=IMPORT_PATH`: Specifies the directory to look for `.proto` files when resovling the import directives within the `.proto` files (yes, `.proto` files can import each other).
  - *Help, I have multiple directories to search for `.proto` files in.*  No problem, pass the `--proto_path` option as many times as you need.
  - *Huh, I saw somebody run the protocol buffer compiler without using `--proto_path`. Is it optional?* That person probably used `-I` instead as a short form to save their finger from the extra typing labor. Just be aware that it has the same functionality.
- `--cpp_out=DST_DIR`: This is the **output directive**. It generates C++ code to `DST_DIR` (a.k.a. whatever directory you desire), with a class for each message type defined in your `.proto` file/s.
  - *Ew, I hate C++. How do I generate code from my `.proto` files in another language?* Switch up the output directive: `--java_out` for Java, `--python_out` for python, etc. Google protocal buffers currently support C++. C#, Dart, Go, Java, and Python.
  - *I don't see the language I'm using in the [official Google documentation](https://developers.google.com/protocol-buffers/docs/tutorials).* Beg Google to add support for your language. Personally I think the languages they support cover most usecases. Perhaps it's time to figure out how to add a C++ or python server to your project.
- `/path/to/file.proto`: The `/proto` file/s you are providing as input. You can specify (and probably will, for more complex projects) multiple `.proto` files as well! I usually put all of my `.proto` files into one directory and use regex so I'm not typing out every single file name: `/path/to/allprotos/*.proto`.

## Tags

This is arguably the most important part of a Protobuf.

### Tag Values

You can assign different numbers to tags. These aren't literal values of the fields, but indications of how much space each field takes up.

- `1...15`: Allocates 1 byte of space. Use these for frequently populated fields.
- `16...2047`: Allocates 2 bytes of space.

### Reserved tags

## Fields

## Default Values

All fields, if not specified or unknown, will take a default value.

- `bool`: false
- `number` (`int32`, etc...): 0
- `string`: empty string
- `bytes`: empty bytes
- `enum`: first value (has a tag value of 0)
- `repeated`: empty list

## Repeated Fields

- To express a "list" or an "array", create a repeated field by prepending the field name with the `repeated` keyword.
- The list can take any number (0 or more) of elements that you want.
- The opposite of a repeated field is a *singular field*, but we don't explicitly specify this. It's just the default type of a field.

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

## Imports

When importing, make sure to include the absolute file path from the root of your project.

## Protoc

We can use `protoc` to generate code in any language. Type `protoc` in the Terminal for all of the flags and commands.

Let's dissect the command below:

`protoc -I=my_protos --python_out=path/to/folder my_protos/*.proto`

- `-I=my_protos`: Specify where your `.proto` files are
- `--python_out=path/to/folder`: Generates a python source file to `path/to/folder`. You can pass in Java, C++, etc
- `my_protos/*.proto`: Specifies that we will use all the `.proto` files in the `my_protos` folder to generate our source file.

IMPORTANT NOTE: Make sure to pass an absolute file path to all of the flags. Using relative file paths has been buggy.
