
# Overview

This tutorial will discuss what remote procedure calls (gRPC) and protocol buffers (protof) are.
we will start with two python based quick start guides that will have you up and running, after
those guides, there will be some additional details. Anything we miss can be found in the full 
documentation for [Protocol Buffers](https://developers.google.com/protocol-buffers/docs/pythontutorial) and [gRPC](https://grpc.io/)

## Protocol Buffer Quick Start 

First things first, this quick start assumes a windows 10 installation, with python 3.7.1. If you need any help setting up
python you can head [here](https://www.python.org/), and if you would rather work in a different language protocol buffers 
support many, you can check those out on the previously mentioned source

The first thing you will need to utilize protocol buffers is the protocol buffer compiler, you can get it [here](https://github.com/protocolbuffers/protobuf/releases/tag/v3.6.1)
for simplest use, grab the precompiled bianry (either protoc-3.6.1-win32.zip, or the osx or linux binary depending on your OS)
after downloading and extracting the file in a directory of your choice, make sure it gets added to your path (on windows this means setting
an environment variable) so you can reference the compiler, protoc, from a terminal.
