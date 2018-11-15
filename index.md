
# Overview

This tutorial will discuss what remote procedure calls (gRPC) and protocol buffers (protof) are.
we will start with some very brief background, followed by two python based quick start guides that will have you up and running, 
after those guides, there will be some additional details. Anything we miss can be found in the full 
documentation for [Protocol Buffers](https://developers.google.com/protocol-buffers/docs/pythontutorial) and [gRPC](https://grpc.io/)

## Protocol Buffer Quick Start 

First things first, this quick start assumes a windows 10 installation, with python 3.7.1. If you need any help setting up
python you can head [here](https://www.python.org/), and if you would rather work in a different language protocol buffers 
support many, you can check those out on the previously mentioned source

The first thing you will need to utilize protocol buffers is the protocol buffer compiler, you can get it [here](https://github.com/protocolbuffers/protobuf/releases/tag/v3.6.1)
for simplest use, grab the precompiled binary (either protoc-3.6.1-win32.zip, or the osx or linux binary depending on your OS)
after downloading and extracting the file in a directory of your choice, make sure it gets added to your path (on windows this means setting
an environment variable) so you can reference the compiler, protoc, from a terminal.

so we have the protocol buffer compiler set up, next thing we need is a .proto file for it to compile. To keep things simple, create a directory
that will house all of the files you plan to use in this tutorial, lets call protoBuffTutorial. In this directory lets create our .proto file, 
call it bookshelf.proto. The source for the file looks like this

    syntax = "proto2";

    package protobufftutorial;

    message Bookshelf {
        required string color = 1; 

        enum Genre {
            SCIFI = 0;
            NOT_SCIFI = 1;
        }
        message Book {
            required string name = 1; 
            required string author = 2;
            optional Genre = 3 [default = SCIFI];
        }

        repeated Book = 2;
    }