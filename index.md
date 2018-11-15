
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