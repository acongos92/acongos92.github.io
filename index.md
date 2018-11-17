
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
            optional Genre genre = 3 [default = SCIFI];
        }

        repeated Book books = 2;
    }

The syntax likely seems familiar, and will be covered in a little more detail later. For now, you can think of a message as an "object" (though it 
cant be used as one yet) for our bookshelf message we first provide a string field called color (every book shelf should have a color after all), there
are many other data types, including int32, bool, and you can get the full list from our provided sources. we define a nested message called book, again providing 
a few string fields, and an optional genre. The difference between optional and required is that a value for a required field must be provided or a message will not 
be considered initialized (and it could not be parsed or converted to a string). Optional, as the name suggests, is not required. optional fields can have a default 
value set, or will be given a default value by the compiler. repeated simply indicates that we could have 0 or more of a particular element.
finally, the " =1" or "=2" markers provide a unique tag that the field uses in binary encoding.   

so, now that we have our .proto file and understand it a little, it's time to compile and use it with this command 

    protoc -I=%DIR% --python_out=%DIR% %DIR%\bookshelf.proto

where %DIR% is a previously set variable in the terminal (so as an example it could be C:\\Users\\you\\protoBuffTutorial). In any case, the output of this command will be 
a file called addressbook_pb2.py, this file will contain class definitions (aka usable python objects) that you can now use in your programs. At this point, you're probably 
asking whats the advantage here, why not just use a python class and not bother with this step. Well, within python programs that would work fine, but if you need to serialize
this data (and say share it with a java or c++ program) it will be challenging, where as with this protocol buffer, any supported language could access and use your serialized 
data with no additional steps. With this advantage in mind, lets put together a simple python program, that lets you create a bookshelf, then add books to it.


    import bookshelf_pb2
    import sys
    # builds and returns new empty bookshelf object 
    # with color attribute as color 
    def getNewBookshelf (color):
        return bookshelf_pb2.Bookshelf(color = color)

    def buildNewBook(book, title, author):
        book.name = title
        book.author = author

    # dont need to do this normally, 
    # this is to illustrate serialization/deserialization
    def printBookShelf(bookShelfString):
        tempShelf = bookshelf_pb2.Bookshelf()
        tempShelf.ParseFromString(bookShelfString)
        print(tempShelf)

    #
    # Main Procedure gets user input, builds a bookshelf object, adds a 
    # book to it, serializes the bookshelf and prints the serialized string
    #
    color = input("enter a color for your bookshelf: ")
    shelf = getNewBookshelf(color)
    while True:
        newTitle = input("Input a book title: ")
        newAuthor = input("Input the author: ")
        newGenre = input("Input a genre (hint try scifi): ")
        newBook = shelf.books.add()
        if newGenre == "scifi":
            newBook.genre = shelf.SCIFI
        else:
            newBook.genre = shelf.NOT_SCIFI
        buildNewBook(newBook, newTitle, newAuthor)
        print("bookshelf {")
        printBookShelf(shelf.SerializeToString())

As you can see, most of the code looks as if we're interacting with any normal python object. You can copy paste this code to see exactly 
how it works, but simply it builds a new bookshelf, and lets you add books with types to it. What we would like to draw attention to is 
the serialization of the Bookshelf object. this creates a serialized string, which is then printed in printBookShelf. For demonstration purposes 
a new object is constructed from the string passed to PrintBookShelf, and then the new bookshelf object is printed. This is meant to highlight 
the power of protocol buffers, any data you serialize could be read by any program with access to this protocol buffer regardless of language 
or implementation details. The usefullness of this should become even more clear when we talk about gRPC. 

There are of course many more useful methods and features of protocol buffers, but this covers the essential setup and basic ideas.

## Hello world using gRPC

Trying to do gRPC in Java is way more complicated than doing it python. So as a beginner, this should be a good place to start.

## Set up gRPC in python
Ensure you have pip version 9.0.1 or higher:

    $ python -m pip install --upgrade pip
Install gRPC:

    $ python -m pip install grpcio
 Install gRPC tools. Python’s gRPC tools include the protocol buffer compiler protoc and the special plugin for generating server and client code from .proto service definitions. 
 
    $ python -m pip install grpcio-tools googleapis-common-protos

Now you are all set.
You define gRPC services in ordinary proto files, with RPC method parameters and return types specified as protocol buffer messages.
Lets begin with a hello world example. The protobuf tutorial above should help you understand the below snippets. We begin by writing a proto file for the Request, Response and the service methods. As mentioned earlier, the proto files help to establish the contract between the client and the server. So this is all that is needed in the contract.
Lets say you want to define a Greeter service that has 2 methods - SayHello and SayHelloAgain where both of the methods accept HelloRequest and retrn HelloResponse. So you define the helloworld.proto file as : 

    // The greeter service definition.
    service Greeter {
    // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply) {}
    //sends another greeting
    rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
    }

    // The request message containing the user's name.
    message HelloRequest {
      string name = 1;
    }

    // The response message containing the greetings
    message HelloReply {
      string message = 1;
    }
Above you can see the simple service calls like 'SayHello' and 'SayHelloAgain'. These are called **unary RPCs**. Other kinds of RPCs are **server streaming**, **client streaming** and **bidirectional streaming**
    
    rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse)   {} //server streaming
    rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {} //client streaming
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){} //bidirectional streaming
    
Now you need to generate code for this proto definitions. For this we use protoc. This allows you to generate gRPC client and server code, as well as the regular protocol buffer code for populating, serializing, and retrieving your message types.
On the server side, the server implements the methods declared by the service and runs a gRPC server to handle client calls. The gRPC infrastructure decodes incoming requests, executes service methods, and encodes service responses.
On the client side, the client has a local object known as stub (for some languages, the preferred term is client) that implements the same methods as the service. The client can then just call those methods on the local object, wrapping the parameters for the call in the appropriate protocol buffer message type - gRPC looks after sending the request(s) to the server and returning the server’s protocol buffer response(s).
We will be covering just the unary gRPC. Now lets compile the proto file and generate code. 
    
    $ python -m grpc_tools.protoc -I<<path where your proto file is>> --python_out=. --grpc_python_out=. <<path where your proto file is>>\helloworld.proto

This should generate **helloworld_pb2** and **helloworld_pb2_grpc**.
Now lets write the greeter_server.py
    
    from concurrent import futures
    import time
    import grpc
    import helloworld_pb2
    import helloworld_pb2_grpc

    _ONE_DAY_IN_SECONDS = 60 * 60 * 24

    class Greeter(helloworld_pb2_grpc.GreeterServicer):

        def SayHello(self, request, context):
            return helloworld_pb2.HelloReply(message='Hello, %s!' % request.name)

        def SayHelloAgain(self, request, context):
            return helloworld_pb2.HelloReply(message='Hello again, %s!' % request.name)
    
    def serve():
        server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
        helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
        server.add_insecure_port('[::]:50051')
        server.start()
        try:
            while True:
                time.sleep(_ONE_DAY_IN_SECONDS)
        except KeyboardInterrupt:
        server.stop(0)

    if __name__ == '__main__':
        serve()
And write greeter_client.py
    
    import grpc
    import sys
    import helloworld_pb2
    import helloworld_pb2_grpc


    def run():
        channel = grpc.insecure_channel('localhost:50051')
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        response = stub.SayHello(helloworld_pb2.HelloRequest(name=sys.argv[1]))
        print("Greeter client received: " + response.message)
        response = stub.SayHelloAgain(helloworld_pb2.HelloRequest(name=sys.argv[1]))
        print("Greeter client received: " + response.message)
        channel.close()

    if __name__ == '__main__':
        run()

And you are all set!! on a new command window, run your greeting_server
    
    $ python greeter_server.py
In another terminal, run the client
    
    $ python greeter_client.py sampleText
