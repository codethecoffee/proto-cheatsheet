# Protocol Buffers (Google Protobuf) for Dummies

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
