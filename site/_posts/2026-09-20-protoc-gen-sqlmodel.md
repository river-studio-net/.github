---
layout: post
series_tag: parley
title:  "Parley Sidequest #1 - A `protoc` Protobuf Plugin"
date:   2026-09-20 10:00:00 +0000
tags: parley postgresql sql python rust protobuf grpc
---

Part of the fun part of working on a large project is the side projects that stem from it with time. One such project with #parley is the Protobuf plugin I wrote for converting Protobuf schemas to SQLModel classes written in Python.
It is available [here](https://github.com/river-studio-net/protoc-gen-sqlmodel).

## SQLModel and Alembic
SQLModel is an ORM Python library that helps abstract SQL models into Python classes that are easier to work with, much like SQLAlchemy, but with built-in relationships to Pydantic that help it be much more ergonomic and developer-friendly. It's a great tool to work with if you're working in Python and SQL, and combined with alembic, it is also a great tool to manage your database with. Alembic is another Python library which is aimed towards managing databases and schemas as imperative migration scripts written in Python based on SQL models such as SQLAlchemy and SQLModel classes.

## Where I Come In
The problem of course with SQLModel and Alembic are that they are Python based. To those of you who read my #parley series, you already know that Parley is based on the Rust programming language. This of course creates a need to bridge the gap and this is where I come in. Imagine a world where we have a way to manage our SQL schemas in Protobuf, then use Alembic to manage our database, *and* generate classes for our Rust code to use when interacting with that database, wouldn't that be awesome?

## Cue `protoc-gen-sqlmodel`
Writing a Protobuff plugin was not an easy task, but it was much easier due to the [`protobuf-py`](https://protobufpy.com/) library recently rewritten from scratch by Buf Technologies. `protobuf-py` is a protobuf supporting system written in Python for Python code that uses Protobuf and wants to abstract things away. It is very ergonomic and developer-friendly and has rich support for all needs of a plugin developer as well. 
Using `protobuf-py` was easy with its awesome documentation and the plugin development went very smoothly (ahem ahem testing in production). 

### So How Does It Work?
Protoc outputs a file descriptor object managed as a std-out stream that is then passed as an input stream to the plugin, who is expected in turn to output a stream of the resulting files to std-out. Using `protobuf-py` abstracts all of that into convenient Python classes and when constructing the input it provides a very nice Schema object that we can work with and exposes the following structure:

```
Schema
└── DescFile
    ├── extensions 	Sequence[DescExtension]
    ├── dependencies 	Sequence[DescFile]
    ├── enums 	Sequence[DescEnum]
    │   └── values 	Sequence[DescEnumValue]
    ├── messages 	Sequence[DescMessage]
    │   ├── fields 	Sequence[DescField]
    │   ├── nested_enums 	Sequence[DescEnum]
    │   ├── nested_messages 	Sequence[DescMessage]
    │   └── oneofs 	Sequence[DescOneof]
    └── Sequence[DescService]
        └── ...
```

This structure can then be traversed and analysed into anything we like:

```python
def generate(schema: Schema) -> None:
    for desc in schema.files_to_generate:
        f = schema.generate_file(desc, "_sqlmodel.py")
        f.preamble(desc)
        for ext in desc.extensions:
            handle_extension(ext, f)
        for e in desc.enums:
            handle_leaf_enum(e, f)
        for m in desc.messages:
            handle_message(m, f)
```

From there the journey was based on online-debugging of running the plugin and then running to see the results and fix it letter by letter, which by the end of a hard days night resulted in a rather polished result if you ask me. 
I will not deep dive into the code and it's functions here for the sake of staying on the point, but there is [a quick intro provided by the library authors](https://protobufpy.com/writing-plugins/) and you know how to reach me if you need a helping hand :)

## Plugin Features
### Table Models
To mark a model as a table, include the following extension in your proto files:
```proto
syntax = "proto3";

package example.sqlmodel;

import "google/protobuf/descriptor.proto";

// Marks a protobuf message as a SQLModel table.
extend google.protobuf.MessageOptions {
  bool table = 50001;
}
```

And mark the message schema as:
```proto
syntax = "proto3";

package example.models;

import "sqlmodel_extensions.proto";

message Entity {
  option (example.sqlmodel.table) = true;
}
```

The generated SQLModel will be marked as a table and you can use it directly with alembic for DB management. 

### Fields
To express field options add the following proto extensions to you fields:
```proto
syntax = "proto3";

package example.sqlmodel;

import "google/protobuf/descriptor.proto";

extend google.protobuf.FieldOptions {
  string sa_type = 50002;
  bool primary_key = 50003;
  string default = 50004;
  string default_factory = 50005;
  string py_default = 50006;
  bool index = 50007;
  string foreign_key = 50008;
  bool relationship = 50009;
  string back_populates = 50010;
  bool cascade_delete = 50011;
  string on_delete = 50012;
  string passive_deletes = 50013;
  string link_model = 50014;
  string server_default = 50015;
  bool nullable = 50016;
  bool pguuid7 = 50017;
}
```

Then express fields like:

```proto
message Category {
  option (example.sqlmodel.table) = true;

  string id = 1 [
    (example.sqlmodel.default) = "None",
    (example.sqlmodel.pguuid7) = true
  ];
  
  map<string, string> category_details = 3 [
    (example.sqlmodel.sa_type) = "sqlmodel.JSON"
  ];

  repeated Channel channels = 4 [
    (example.sqlmodel.relationship) = true,
    (example.sqlmodel.back_populates) = "category"
  ];
  // commune is many to one relationship
  optional string commune_id = 5 [
    (example.sqlmodel.default) = "None",
    (example.sqlmodel.foreign_key) = "commune.id"
  ];
  optional Commune commune = 6 [
    (example.sqlmodel.relationship) = true,
    (example.sqlmodel.back_populates) = "categories"
  ];
}
```

### pguuid7
You may have noticed that we have an option called `pguuid7`. This option will result in the following field declared in the generated model:

```python
id: UUID = Field(sa_column=Column(UUID_(as_uuid=True), primary_key=True, nullable=False, server_default=text("uuidv7()")))
```

## Final Thoughts
If you found that useful, please reach out and let me know. I worked on this hard and would appreciate any feedback. Developing a plugin was a great experience thanks to `protobuf-py` and their amazing ecosystem, and I'm grateful to any other project that has brought us to this point in time (shoutout to betterproto2). 
Thank you for reading and see you in the next one!
