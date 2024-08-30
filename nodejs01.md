# Installations

## Terminal mac

uname -a
to know your cpu architecture

## VS Code

cmd + shift + p > install code in path

## xcode-select

when installing xcode-select (xcode-select install), we'll get access to python3, gcc (c++ compiler)

## homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

then add to your path according the instructions

## nodejs

Recommended way is to use nvm to install nodejs for mac an linux
`brew update`
`brew install nvm`

then add to path as suggested

To change your shell: chsh /bin/zsh

`nvm install 22` install a LTS version

## wireshark and extension

the extension is in the dmg

## Calculator

Use it in programmer mode (hex, dec, binary...) too convert numbers

## Command line basics

cd pwd cat vim echo open ls clear which touch mv rm

The Prompt: what's before what you can type
usually, the machine name and current folder

Command line: the line we type to give commands to the machine

this is bash in cmd line:
`for i in {1..50}; do touch "index$i.html"; done`

`ls *.css`

`mkdir html css`
`mv *.html html`

mv index1.html test.html -> rename file
echo $PS1

vim text.txt
cat app.js -> display content of the file in ther terminal

man ascii | head -43 | tail -16 > ascii.txt

## nvm

to use the right node version in a project
cd project
touch .nvmrc
echo "v20.11.0" >> .nvmrc
or
node -v > .nvmrc
then use
nvm use (without specifying a version)

## nodejs under the hood

Processors can only undertand machine code.
processors cpu's architecture (mips, arm64, x86) have their own language
mips, arm64, x86 ar just 0 and 1 -> 011001010 010101000

machine code
🔼
assembly language
⏫
C/C++
⏫
Javascript/python

Javasrcipt -> Javascript engine -> 001 1100 01010
Spidermonkey was the first js engine (Brendan Eich) (converting into bytecodes)

V8 is a js engine dev for chrome (js engine and webassembly engine written in c++) implementing Ecmascript and WebAssembly

Nodejs is depending on v8 and libuv. You have others too (llhttp, zlib, openssl, c-ares...) but less important. It provides tools like npm, gyp and gtest.

// brief history
1936 first computer
1949 assembly language
1970 C
1977 first personal computer: Apple 1
1983 internet
1991 first web site
1995 Javascript
1995 internet explorer
1997 Ecmascript
2002 Firefox
2008 sept: chrome v8
2009 may: nodejs

Libuv
nodejs is a system language and a server side technology to create network applications
It needs to deal with databases, files, network requests, scaling...
Libuv is handling all these actions.
A C library to abstract non blocking operations across all platforms. Provides file system handling, DNS, network, child processes, pipes, signals, polling and streaming, offloading and async threads

Event loop -- specific to nodejs
how v8 and libuv interact with it?

js is single-thread: one thing at a time starting at the begining of the file.
when using a for loop the whole process is unresponsive until the for loop is finished

check image

event driving system

when calling timers or readFile you are pushing events in the event loop.
process.nextTick() is the process called in node, when you want to execute code after the main thread is done

so libuv is what allows the non-blocking thread
libuv has a ThreadPool of 4.
Network requests are not done with ThreadPool
