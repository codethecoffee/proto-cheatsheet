# Protocol Buffers for (Coding) Dummies

A (hopefully) slightly more entertaining introduction to Google protos. For people who got bored of reading [Google's official documentation](https://developers.google.com/protocol-buffers/docs/overview), and want a slightly less formal tone of writing.

## Why do we even need Protocol Buffers

Short answer: Efficient data serialization. Take my word for it.

Long answer: You technically have other options for data serialization. Let's look at each of them and explain why they kind of suck:

- Option 1: Save the language's data structures in binary form and send that. After all, data is ultimately just a bunch of 0s and 1s. However, this approach can break in so many ways; you need to make sure the code is compiled with exactly the same memory layout, endianness, and so on. Programmers can't even agree on a text editor (read this wiki article about the infamous [editor war](https://en.wikipedia.org/wiki/Editor_war) if you want to procrastinate), so this is a lost cause.
- Option 2: Write your own algorithm to encode data items into a string ([cue the  traumatic flashbacks to Leetcode hard questions about tree serialization](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)). This is a solid option for simple data, but once your data structures become more and more complicated, you're wasting time reinventing the wheel.
- Option 3: Extensible Markup Language (XML). This is a markup language made just for this task, plus it's (somewhat) human readable! Unfortunately, XML is notoriously space-greedy, not to mention that navigating XML structures can be a headache.

The (self-declared) winner: protos!

All internal Google applications use protos for their data serialization, so that should count for something. Lots of Googlers don't even use Chromebooks (I'm a Google engineer typing this out on my Macbook), so we don't blindly adopt products just because our employer made it.

## Example of a (heavily commented) Proto Message

Here is a `.proto` message representing the data needed in an address book. Scroll down more to see my detailed notes on different aspects of a proto message.

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

## Fields

A proto message is effectively a glorified set of fields. So if you understand fields, you basically know what a proto message is.

### Field Types

All the standard simple data types work (`bool`, `int32`, `int64`, `float`, `double`, `string`...are there even any more?).

If you're handling some complicated data and those primitive types don't suffice, **you can use other proto messages as field types as well!** Notice the lines `repeated PhoneNumber phones = 4;` and `repeated Person people = 1;`; both of them are fields in the `Person` message, and use the messages `PhoneNumber` and `Person` respectively as field types. This "nesting" is what makes it so easy to represent complex data structures as proto messages.

### Required v.s. Optional Fields

You'll notice that some fields are prepended with the `optional` modifier while others have the `required` modifier. As suggested by the name, optional fields are optional and required fields are required. That's probably one of the most redundant sentences I've formed in my entire life.

If you're interested, here are some important nuances:

- `required`: If a value for this field is not provided, the message will be considered "uninitialized". Be careful; in optimized builds, they skip the `required` field check, so the message will be written anyway. You'll find out when you parse the uninitialized message (and fail at it) that the required field wasn't populated properly.
- `optional`: If an optional field is not set, a default value is used (check out the "Default Values of Fields" section in this doc). You can set your own default value for simple field types, but for more complicated ones (a.k.a. anything where you use another proto message as a data type, so the `repeated PhoneNumber phones` field in my addressbook example).

### Tag Numbers

You can assign different numbers to fields. Notice the `=1` and `=2` in my code snippet. These aren't literal values of the fields, but unique identifiers (a.k.a. "tags") for each respective field. The compiler will freak out if you try ot use the same tag for several fields. Identity theft is a serious crime.

- `1...15`: Use these tag values for frequently populated fields - they require one less byte to encode than the higher numbers. I know one byte doesn't sound like much, but it can go a long way.
- `16...2047`: Basically use these when you use up 1 through 15.

### Default Values of Fields

All fields, if not specified or unknown, will take a default value.

- `bool`: false
- `number` (`int32`, etc...): 0
- `string`: empty string
- `bytes`: empty bytes
- `enum`: first value (has a tag value of 0)
- `repeated`: empty list

### Repeated Fields

To express a "list" or an "array", create a repeated field by prepending the field name with the `repeated` keyword. This allows a field to be repeated any number of times. The order of the values if preserved in the protocol buffer.

The opposite of a repeated field is a *singular field*, but we don't explicitly specify this. It's just the default type.

```proto3
// A list of phone numbers that is optional to provide at signup
repeated string phone_numbers = 7;
```

### Enums

If you know all the values a field that take in advance, use an `Enum` type. Note that the enum value.

```proto2
// we currently consider only 3 eye colors
enum EyeColor {
    // The default value is UNKNOWN_EYE_COLOR, unless specified otherwise
    UNKNOWN_EYE_COLOR = 0; 
    EYE_GREEN = 1;
    EYE_BROWN = 2;
    EYE_BLUE = 3;
}

// it's an enum as defined above
EyeColor eye_color = 8;
```

## Packages

Add a line `package my.package.name` at the top of the `.proto` file to place a protocal buffer message into a particular package.

Packages help to prevent name conflicts, just like C++ namespaces (`my.package.Person`). Be careful when importing protos into other protos; you need to specify the correct package name when accessing packaged Protos, or else your compiler will freak out at you. Yes, I've done this many times. The bane of my existence is incorrect import statements.

## Time to compile - how to make protoc magically create code files from proto files

`.proto` files are cool and all, but what we really care about is generating code files from it that we can actually work with. Your C++ program doesn't know what to do with a `.proto` file.

The protocol buffer compiler (also referred to as protoc) does its magic and generates the code file in whatever programming language you specify. After [you download everything you need](https://developers.google.com/protocol-buffers/docs/downloads) and have all your `.proto` files ready to go, invoke the compiler in your terminal with the command below:

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

Once you run that command, you'll get these two files in your specified destination directory:

- `addressbook.ph.h`: The header file that declares the classes protoc generated for us.
- `addressbook.pb.cc`: The implementation of those classes

## Protocol Buffer C++ API

Our wonderful protocol buffer compiler generated a custom protocol buffer API from our `addressbook.proto` file! Look at all the code `protoc` generated from that Terminal command:

```C++
  // name field
  inline bool has_name() const;
  inline void clear_name();
  // Normal getter
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value); //setter
  inline void set_name(const char* value); // setter
  // Mutable getter, which gives you a direct pointer to the string
  // so that you can mutate it (thus the name)
  inline ::std::string* mutable_name();

  // id field
  inline bool has_id() const;
  inline void clear_id();
  inline int32_t id() const;
  inline void set_id(int32_t value);

  // email field
  inline bool has_email() const;
  inline void clear_email();
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline ::std::string* mutable_email();

  /**********************/
  // REPEATED FIELD API //
  /*********************/
  // Get the number of phone numbers
  inline int phones_size() const; 
  inline void clear_phones();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
  inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  inline ::tutorial::Person_PhoneNumber* add_phones();
```

The compiler will generate getters and setters for each of your fields. The `mutable` getters return a direct pointer to the value. Note that for a mutable getter, even if the field isn't initialized, it will simply initialize a empty instance.

### Repeated Fields API

`repeated` fields get more than just setters and getters.

- `inline int phones_size() const;`: Check the `_size` of your mutable field. In this case, this woul dbe number of phone numbers (kind of like the size of an array)
- `inline const ::tutorial::Person_PhoneNumber& phones(int index) const;`: Getter. Get a specific phone number using its index. 
- `inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);`: Setter. You can mutate a phone number at a specific index.
- `inline ::tutorial::Person_PhoneNumber* add_phones();`: Lets you add a new `phones` field to the message. Instead of passing in the new value as an argument, you call this method and then proceed to mutate the newly created entry using our setter in the bullet point above.

### Enums and Nested Classes API

Where are the nested messages and enums that I defined in my `.proto` file? For us, that would be `PhoneNumber` and `PhoneType` respectively. 

- *Where's my `PhoneType` enum?* The enum values have a type of `Person::PhoneType` and can be accessed as `Person::MOBILE`, `Person::HOME`, and `Person::WORK`. 
- *Where's my `PhoneNumber` nested message?* It is a nested class that can be accessed at `Person::PhoneNumber`. Technicaly, under the hood, this class is `Person_PhoneNumber`, but our compiler defines a typedef that lets you access it as if it were a nested class. Good compiler.

### Standard Message Methods

There are even MORE methods in each message class that lets you do stuff to the message itself, not just individual fields within it. This protocol buffers compiler is giving you lots of power.

- `bool IsInitialized() const;`: Checks if all the required fields have been set. If you use any required fields, good practice to call this or else your program might break. Honestly, I'm lazy so it seems easier to just never used `required` fields in the first place.
- `string DebugString() const;`: Returns a human-readable representation of the message. Unfortunately we can't read 0s and 1s, so this is a very useful method. Use it when debugging.
- `void CopyFrom(const Person& from);`: Overwrites the message with the given message's values. If you're going to change a lot of values in a message, you might as well call this instead of calling the setter of each individual field.
- `void Clear();`: Clears all the elements back to an empty state. Blank slate.

### Parsing and Serialization

Finally (I swear, I'll stop introducing new methods after this section), our lovely compiler generated methods for writing and reading messages.

- `bool SerializeToString(string* output) const;`: Serializes the message ands tores the bytes in the provided string. No need for you to write your own string serialization algorithm, so screw you Leetcode encoding questions!
- `bool ParseFromString(const string& data);`: Parses a message from a given string (a.k.a. deserializing).
- `bool SerializeToOstream(ostream* output);`: Writing the message to some C++ ostream.
- `bool ParseFromIstream(istream* input);`: Parses a message from some C++ `istream`.

There are way more parsing and serialization methods, but those are the ones we'll use in this tutorial!

## Let's write a message

Instead of staring at the documentation, let's actually use our protocol buffers API! Let's say I want my address book app to write personal details to an address book file. In order to do this, I need to do the following:

1. Create and populate instances of my protocol buffer classes (generated by the protocol compiler).
2. Write the info to an output stream.

```C++
// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person *person)
{
    cout << "Enter person ID number: ";
    int id;
    cin >> id;
    person->set_id(id);
    cin.ignore(256, '\n');

    cout << "Enter name: ";
    getline(cin, *person->mutable_name());

    cout << "Enter email address (blank for none): ";
    string email;
    getline(cin, email);
    if (!email.empty())
    {
        person->set_email(email);
    }

    while (true)
    {
        cout << "Enter a phone number (or leave blank to finish): ";
        string number;
        getline(cin, number);
        if (number.empty())
        {
            break;
        }

        tutorial::Person::PhoneNumber *phone_number = person->add_phones();
        phone_number->set_number(number);

        cout << "Is this a mobile, home, or work phone? ";
        string type;
        getline(cin, type);
        if (type == "mobile")
        {
            phone_number->set_type(tutorial::Person::MOBILE);
        }
        else if (type == "home")
        {
            phone_number->set_type(tutorial::Person::HOME);
        }
        else if (type == "work")
        {
            phone_number->set_type(tutorial::Person::WORK);
        }
        else
        {
            cout << "Unknown phone type.  Using default." << endl;
        }
    }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char *argv[])
{
    // Verify that the version of the library that we linked against is
    // compatible with the version of the headers we compiled against.
    GOOGLE_PROTOBUF_VERIFY_VERSION;

    if (argc != 2)
    {
        cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
        return -1;
    }

    tutorial::AddressBook address_book;

    {
        // Read the existing address book.
        fstream input(argv[1], ios::in | ios::binary);
        if (!input)
        {
            cout << argv[1] << ": File not found.  Creating a new file." << endl;
        }
        else if (!address_book.ParseFromIstream(&input))
        {
            cerr << "Failed to parse address book." << endl;
            return -1;
        }
    }

    // Add an address.
    PromptForAddress(address_book.add_people());

    {
        // Write the new address book back to disk.
        fstream output(argv[1], ios::out | ios::trunc | ios::binary);
        if (!address_book.SerializeToOstream(&output))
        {
            cerr << "Failed to write address book." << endl;
            return -1;
        }
    }

    // Optional:  Delete all global objects allocated by libprotobuf.
    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

## We wrote a message, so now let's read one.
