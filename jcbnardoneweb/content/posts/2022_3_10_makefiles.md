---
title: "C++/Java programmers guide to building and running bigger projects w/ GNU Make"
date: 2022-03-10T15:01:15-08:00
ShowReadingTime: true
hideSummary: true
ShowCodeCopyButtons: true

---


First learning C++ in university, I thought compiling and running C++ programs was just about as easy Python. All I needed was a compiler (GNU g++ would do just fine) and then I could just run the produced executable. Granted that <i>was</i> one more step than just having a simple Python module interpreted, but it was easy enough running this command in a terminal:

<!-- Shortcode for adding an image: -- {{< figure src="/images/moose.jpeg#center" height=600 >}} -->

```console
> g++ main.cpp -o out && ./out
```

However, naive younger me didn't realize how much more convoluted this command would need to get once you started working on projects what weren't just a simple 'main.cpp' file. You needed a much broader understanding of the g++ program before I wanted to start factoring in things like multiple source files, header files, external libraries, and ranging directory structures.

Quickly this command can get out of hand, and can become super tedious to write as you project gets larger and larger. There had to be a way to not only avoid writing this command out every time, but to also automate the changes that needed to be made to it as new files were added to the project.

Wanting to learn more, this sent me into a spiral of Google searches, almost all of which led me to the same undesirable solution: 

<p style="color:rgb(236, 119, 98); text-align:center; font-size: 25px">
	<i>"Download [IDE name here] and let it handle all of that for you!"</i>
</p>


And yes, that would have been easier. My CS professor always told us to never be afraid to stand on the shoulders of giants, and I always saw value to that metaphor, but something about being able to build and run my own code without having to download some monstrous software like Visual Studio or Eclipse felt very important to me. It felt empowering to know that the only tools I really needed were the compiler and some hopefully segfault-free source code.

At this point I had messed around with GNU Make in college on a few occasions, but had never really delved into its more obscure yet powerful functionality. Having stood the test of time as lightweight build tool, I thought it was time to really dig into into Make.

After months of practice writing Makefiles and reading documentation (and StackOverflow of course), I managed to create comfortably flexible Makefiles for both for C++ and Java that so far have served me quite well in getting new projects up and running quickly (all from the comfort of my terminal).

Before getting started, you'll need:
- GNU Make
- a C++ compiler (or a Java JDK)
- a terminal

First, let's take a look at the C++ makefile.

<h2 style="color:rgb(155, 228, 99); text-align:center;">
	<i>C++ Makefile</i>
</h2>

```make
#----------------------------------------------------------------#
#--------------------- Project Specifics ------------------------#
# name of dir that stores source files
SRCDIR	:=
# name of dir that stores header files
INCDIR	:=
# name of dir that stores object files
OBJDIR 	:=
# name of your final executable
EXEC 		:=
# your C compiler
CC			:=
# compilation flags
LIBPATH	:=
CFLAGS	:= -I$(INCDIR) -I$(LIBPATH) -std=c++17
# linker flags
LFLAGS	:= -L$(LIBPATH) -lmystd

.PHONY: run runonly clean
#----------------------------------------------------------------#
#----------------------------------------------------------------#

# creates string list of object files we need to create
OBJS 		:= $(patsubst $(SRCDIR)/%.cpp, $(OBJDIR)/%.o, $(wildcard $(SRCDIR)/*.cpp))
# creates string list of all our header files
HEADERS 	:= $(wildcard $(INCDIR)/*.hpp)

# Links object files and generates executable
$(EXEC): $(OBJS)
	@ echo Linking executable ...
	@ $(CC) $^ $(LFLAGS) -o $@
	@ rm -rf $(OBJS)
	@ echo Done.
	@ printf "\n"

# Creates every object file we need in OBJS using corresponding source file + headers
$(OBJDIR)/%.o: $(SRCDIR)/%.cpp $(HEADERS) | $(OBJDIR)
	@ echo Compiling $< ...
	@ $(CC) $< -c $(CFLAGS) -o $@

# Builds the object file directory if not already there
$(OBJDIR):
	@ mkdir -p $@

# Checks if needs to be rebuilt and runs the executable
run: $(EXEC)
	@ make runonly

# Just runs the executable
runonly:
	@ ./$(EXEC)

# Cleans all generated files
clean:
	@ rm -rf $(EXEC) $(OBJS)
```


And as promised, below is the Java makefile.

<h2 style="color:rgb(155, 228, 99); text-align:center;">
	<i>Java Makefile</i>
</h2>

```make
#----------------------------------------------------------------#
#--------------------- Project Specifics ------------------------#
# Path to your program start class relative to source directory
mainclass :=
# Full path to you java source code / packages
srcdir :=
# Name of the directory storing your class files
bindir :=
# Path(s) to all class files needed to compile and run
clspth :=
#----------------------------------------------------------------#
#----------------------------------------------------------------#

modlist := $(bindir)/modlist
# Recursive wildcard function - leave as is.
rwildcard = $(foreach d,$(wildcard $(1:=/*)), $(call rwildcard,$d,$2) $(filter $(subst *,%,$2),$d))
sources = $(call rwildcard, $(srcdir), *.java)
.PHONY: run runonly clean

# compares timestamps of the dummy file (modlist) with each source file
$(modlist): $(sources) | $(bindir)
	@ echo Compiling $(notdir $?) ...
	@ javac -d $(bindir) -classpath $(clspth) $?
	@ printf "Done!\n\n"
	@ touch $@

run: $(modlist)
	@ make runonly

runonly:
	@ java -classpath $(clspth) $(mainclass)

$(bindir):
	@ mkdir $@

clean:
	@ rm -rf $(bindir)
```
