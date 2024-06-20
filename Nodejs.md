# Nodejs

JS / Python (high languages) -> C/C++ -> Assembly Language -> machine code 01101111




## Event emitter

const EventEmitter = require("events")

class Emitter extends EventEmitter {
}

const myE = new Emitter();
myE.on("foo", () => {
  console.log("An event occurred")
});
myE.emit("foo")

event.once(...)

this is just javascript
emit / once / on / off


## Buffer

A buffer is a location in memory that holds a certain amoutn of data

1 byte = 8 bits

base 16 hexadecimal
  0x456 0x prefix is for hexadecimal numbers
  -> 4 * 16 power of 2 + 5 * 16 power of 1 + 6 = 1024 + 80 + 6 = 1110

  0 1  2 3 4 5 6 7 8 9 A B C D E F
->0        ...        10        15

hex have only 6 chars (0xffffff) while dec. have 8 and binary have 24

32bits -> 8 groups of 4 bits
0101 0101 0111 1101 0101 1111 0000 0001
 5     5   7    D    5    F    0    1   in hex

0 -> 0000
1 -> 0001
2 -> 0010
3 -> 0011
4 -> 0100
5 -> 0101
6 -> 0110
7 -> 0111
8 -> 1000
9 -> 1001
A -> 1010
B -> 1011
C -> 1100
D -> 1101
E -> 1110
F -> 1111

Characters sets
how do we go from binary numbers to text?
we need a system of mapping them: it assigns each character a number

we have unicode standard and ascii standard
unicode defines 149 813 chars (version 15 in 2023) worlwide writing system
's' is number 115 in unicode

ascii defines 128 chars a-z A-Z 0-9 punctuation and Del char and a couple more
ascii only for english language

> man ascii
's' is 163 in octal or 115 in decimal or 63 in hexadecimal
char 9 is number 39 in hexadecimal ascii
only uses 8bits for each character 
ascii is a subset of unicode

Encoders / Decoders
Encoder turns something meaningful (image) into binary 0011 0010 ...
Decoder does the opposite

Character encodings
a system of assigning a sequence of bytes to a character

utf-8 is the most common defined by the unicode standard, so chars have the same numbers as the unicode
    s            t   r     i     n        g
0111 0011 |                         | 0110 0111
  115         116   114   105   110  103
each char is stored on 8 bits and the MSB is always 0 (the first of 8 bits)

in utf-8 0111 0100 is t
another system would give something different
so very important to specify what system you uses

Buffers
A buffer is a container in memory
4bytes (32bits)
Buffer acts like an array, they have elements which are numbered-index
4 bytes means 4index, 0,1,2,3
all elements would be filled with zeros
so byte[0] = 0000 0000
and all others too
Buffer size is fixed
They are data structure designed to work with binary data
They have functions to help you manipulate binary data
read, encode, decode, move information and more

Buffers in action
const { Buffer } = require("buffer");

// create a buffer of 4 bytes (32bits)
const memoryContainer = Buffer.alloc(4);
console.log(memoryContainer); // <Buffer 00 00 00 00>
console.log(memoryContainer[0]); // 0

memoryContainer[0] = 0xf4; // 1111 0100
console.log(memoryContainer[0]); // 244

memoryContainer[1] = 0x34;
memoryContainer[2] = 0xb6;
memoryContainer[3] = 0xff;

console.log(memoryContainer); // <Buffer f4 34 b6 ff> // 244 52 182 255
// 8bits min value is 0 (00)
// 8bits max value is 255 (ff)

if we store negative value in the second element, it can shift the 3rd element, so you need to store the minus into the 1st bit, inverse all other bits
but whatever
if you want to use negative or float numbers use variables not Buffers
use a method starting with .write like writeInt8(-34, 2)
// memoryContainer.writeInt8(-34, 2);
// console.log(memoryContainer.readInt8(2));

console.log(memoryContainer.toString("hex"));

Challenge:
// write in memory: 0100 1000 0110 1001 0010 0001

const { Buffer } = require("buffer");

// create a buffer
const buff = Buffer.alloc(3); // 24 bits / 8 -> 3 bytes

buff[0] = 0x48;
buff[1] = 0x69;
buff[2] = 0x21;

console.log(buff); // <Buffer 48 69 21>
console.log(buff.toString("utf-8")); // Hi!

// or 
// const buff = Buffer.from([0x48, 0x69, 0x21]);
// console.log(buff.toString('utf-8')); // Hi!

Buffer.from() will figure out how many bytes it needs

You can also do it like this:
Buffer.from("486921", "hex")

you can specify your char encodings
buff = Buffer.from("Hi!", "utf-8") // if you log it: <Buffer 48 69 21>

symbl.cc


Allocating huge Buffers
create a buffer with 1gigabyte (1e9 = 1,000,000,000 bytes = 1GB)
const b = Buffer.alloc(0.5e9) // 500,000,000 500MB

// b.length is the size of the buffer in bytes.
setInterval(() => {
  for (let i = 0; i < b.length; i++) {
    b[i] = 0x22;
  }
})
1 megabyte (MB) = 1000 * 1000 bytes
1 mebibyte (MiB) = 1024 * 1024 bytes
same for GB and GiB

faster way to fill a buffer:
b.fill(0x22) instead of the for loop (2 to 3 times faster)

you can also use constants
const { constants } = require("buffer")
console.log(constants.MAX_LENGTH)

here you allocate the memory usage from your operating system, crash it will if
 too much allocation for your node program

Fastest way to allocate buffer
allocUnsafe by it's insecure, you have to take security measures by filling it

Buffer.from() and Buffer.concat() will allocate the right size and only the data that you need

nodejs by default will allocate 8KiB into ram for buffer (Buffer.poolSize)
you can use it only if you use Buffer.allocUnsafe not with Buffer.alloc
and if allocUnsafe asks for less than half the poolSize and floor of that (poolSize >>> |)
1010 0101 >>> 1 : remove the LSB and shift all bits to the right
=> _101 0010 => 0101 0010

30 >>> 2 -> will log 7

Buffer.allocUnsafeSlow(8)
it will not try to use the allocated buffer


How to store negative numbers and floats


Streams
Using writable Streams
Implementing our own writable streams
Using readable Streams
Implementing our own readable streams 
Implementing Duplex streams
Using Duplex streams
Then Transform streams (decryption/encryption)


CJS vs ESM 
commonjs (module.exports + require) vs ecmascript modules (imports + export in .mjs)

## Streams

1. Writable streams


2. implement them
3. Readable streams
4. implement Readable
5. Duplex streams
6. Transfrom streams (encrypt / decrypt)

## Networking

Net Dgram Dnc readline process tty Path

### How networks work?

How to share data between two computers years ago without connection?
Use something physical, disk or anything, and insert it into the 2nd computer and pull it out on the 1st computer

Now you will use a switch and cables to connect to the computer and then login to the switch
your computer needs a networking card to do that - that means a unique mac address for the computer
48 bits like f1:31:87:f1:5b:3c
When you send info, you send a packet (source (mac-address1), destination (mac2), data (binary)) to the switch
And the mac2 receives the packet through its network card

How to make all the network switches talk together?
We have a layer above it: Router with ip address (unique address to a specific device)
The Routers can be tied by cables and route data between them
132.10.0.1
City 132.10 the first two units
The last two units are for the local network or subnets
The port, if added, will allow to access to some parts of the computer

Mac <-> Router (with switch and wifi) <-> Internet
Your router assigns a new ip address to any device connecting to it
for ex. Router 10.0.100.1 mac 10.0.101.218 ipad: 10.0.101.220

127.0.0.1 -> A loop back address: traffic gets loop back to itself (ipv4 version)
localhost is an alias

ipad can connect to mac if the mac is opening a port for that and what to do in that case
127.0.0.1:4080 should be accessible on the ipad if you use the shared connection of your phone
ipconfig or ifconfig will give your local ip address 172.20.10.2 for ex
and update your server with this hostname

### Networking Layers

Networking: Way of communication between computers

Physical Layer: cables -> Bits 
Data Link layer: switches -> Frames ()
Network Layer: Packets 
Transport Layer: Segments, transmission of the packets
Application Layer: Data

Physical Layer: signal, binary transmission
Data Link layer: mac addresses
Network Layer: IP Addresses, path determination
Transport Layer: end to end connections - TCP/UDP (move info from one point to another)
Application Layer: nodejs (our part as software engineer)

### Simple TCP

Move info from one point to another
use the net module from node (http module is built upon it)

const server = net.createServer((socket) => {
  // the socket is my connection, a duplex stream
  socket.on('data', (dataBuffer) => {
    console.log(dataBuffer.toString("utf-8"))
  })
});

server.listen(3099, "127.0.0.1", () => {
  console.log("Server running on", server.address())
  //Server running on { address: '127.0.0.1', family: 'IPv4', port: 3099 }
});

Create a socket (another file)
const socket = net.createConnection({
  host: "127.0.0.1",
  port: 3099
}, () => {
  socket.write("A simple message from a simple sender");
})

Wireshark or Activity Monitor on Network to see what is sent on the network
Wireshark will monitor everything that is sent to the network
If we allow a 8 bytes buffer, why do we see a length of 68 or 56 in wireshark for our tcp protocol?
check the source port and the hostname to be sure it's your request
WS is capturing every bit, Activity Monitor only shows the bytes received or sent and not the headers, WS captures also the headers
TCP transmission Control protocol
16 bits to represent the source or the destination for ex.

Why do we have multiple packets?

### TCP/UDP

Transport Layer
adds some port number to the packets

2 main protocols TCP and UDP
TCP will make sure that every bit you send is sent
UDP doesn't care about loss but about speed, it's used on streaming stuff (video, voice, ...)

How TCP works?
3-way handshake
<TCP Headers>: 
  source port (16 bits)
  destination port (16 bits)
  sequence number (32 bits)
  acknowledgement number (32 bits)
  Other like window header and more like checksum, length, flags...
  Data
This whole thing is a segment

<UDP Headers>
  Source Port
  Destination Port
  Segment Length (16 bits)
  Checksum (16 bits) (check if file corrupted)
  Data
64 bits is fixed

### Port numbers

iana port numbers
ssh 22
http 80
0-1023 system ports (to avoid)
1024-49151 user ports
49152-65535 dynamic / private ports

ex: on port 2, you can use different protocols, like a tcp and a udp on it, but not 2 tcp, the protocols should be different
default port is 80 for http application

SCTP protocol exists too

IPC server from the docs -> InterProcessCommunication
many google processes for ex. that communicate with each other (using a common path)

### Chat app

client.js and server.js
net module: createServer (server serves something to the client) and createConnection (client) are the most important

socket is two end points talking together

on client: better to check for "end" rather than "close"

net.Server (from createServer) and net.Socket (from createConnection)

the readline module reads one line at a time from a stream.
const readline = require('node:readline');
const { stdin: input, stdout: output } = require('node:process');

const rl = readline.createInterface({ input, output });

rl.question('What do you think of Node.js? ', (answer) => {
  // TODO: Log the answer in a database
  console.log(`Thank you for your valuable feedback: ${answer}`);
  //SOCKET.write(message);
  rl.close();
});


socket is an end point and it has a connection A --- B
we have to store on the server the list of all sockets in order to send message to all sockets

### Improving UI of chat app

interact with the console

TTY module (terminal)
clearLine, moveCursor, and more
can be used with stdout or stderr

clearLine(1 | -1 | 0) and moveCursor(dx,  dy)

### How to identify our users?

check code

### Notify in chat app

we register each socket in clients +
    clients.map((client) => {
      client.socket.write("user" + id + " > " + message);
    })

But all our data can be read from wireshark, we should encrypt the data tp be secure (https, vpn) and check who is sending the data!
And our client could be a telnet from elsewhere, very dangerous, use tokens and passwords.

use this pattern:
Application Presentation Session

### Deploy to AWS
### IPv4
### DNS
### IPv6
### Uploader app
### UDP and Dgram


## Streams

Writable stream 
A stream is an object with internal buffer, and events, properties and methods
internal buffer 16384 bytes

stream.on("drain") -> when the buffer is emptied and written, you can push again into the internal buffer
stream.write(data) -> buffer -> write the data once buffer is filled

readable stream
stream.push(data) -> buffer -> stream.on("data", (chunk) => { ... })

how to write?
const fs = require("node:fs/promises");

(async () => {
  console.time("writeMany")
  const fileHandle = await fs.open("test.txt", "w")

  const stream = fileHandle.createWriteStream()
  console.log(stream.writableHighWaterMark); // 16384
  console.log(stream.writableLength); // 0

  let i = 0;
  const writeMany = () => {
    while (i < 1000000) {
      const buff = Buffer.from(` ${i}`, "utf-8");
      
      if (i === 999999) {
        return stream.end(buff);
      }

      // buffer is full, wait for the drain
      if (!stream.write(buff)) {
        break;
      }

      i++
    }
  }
  writeMany();

  stream.on("drain", () => {
    writeMany();
  });

  stream.on("finish", () => {
    fileHandle.close()
    console.timeEnd("writeMany")
  })
})();

### Readable streams

const fs = require("node:fs/promises");

(async () => {
  // 
  const fileRead = await fs.open("test.txt", "r");
  const fileWrite = await fs.open("dest.txt", "w");

  const streamRead = fileRead.createReadStream({
    highWaterMark: 64 * 1024 // 65536
  });
  const streamWrite = fileWrite.createWriteStream()

  streamRead.on("data", (chunk) => {
    // 65486 kilobytes each chunk 64kb
    // console.log("-------------");
    // console.log(chunk.length);
    
    if (!streamWrite.write(chunk)) {
      streamRead.pause()
    }
  });

  streamWrite.on("drain", () => {
    streamRead.resume();
  })
})();

### Selectively write chunk of data using streams

  let split = '';
  streamRead.on("data", (chunk) => {

    const numbers = chunk.toString("utf-8").split('  ');
    // at the end of the buffer number can be truncated
    // a 7.9 MB file = 7,892,693 bytes * 8 = 63,141,544 bits
    // 1hex = 0000 (4 bits)
    // utf-8: one char = 8 bits: 0000 0000
    // each element of the buffer have 8 bits
    // each array element have 2 bytes
    // <Buf 00, 12>
    // so how to split right?
    // find where we have a split and mark it for the next chunk
    // and check the last element of the previous chunk


    // first element of next chunk
    if (Number(numbers[0]) !== Number(numbers[1]) -1) {
      if (split) {
        numbers[0] = split.trim() + numbers[0].trim();
      }
    }

    // last element
    if (Number(numbers[numbers.length - 2]) + 1 !== Number(numbers[numbers.length -1])) {
      split = numbers.pop();
    }

    numbers.forEach((num) => {
      let n = Number(num);
      if (n % 2 === 0) {
        if (!streamWrite.write(" " + n + " ")) {
          streamRead.pause()
        }
      }
    })

    // if (!streamWrite.write(chunk)) {
    //  streamRead.pause()
    // }
    
    streamWrite.on("drain", () => {
      streamRead.resume();
    })

    streamRead.on("end", () => {
      console.log("Done reading")
    })
  });

Flowing and paused mode: all readable streams begin in paused mode
readable.readableFlowing can be null false or true

dont use a combination of on("data") and pipe

Pipe is to  limit the buffering of data to acceptable levels such that sources and destinations of differing speeds will not overwhelm the available memory.

No pipe in production (you have to handle errors and destroy manually, you can call 'pump' module for that), anyway instead use pipeline():

const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');
const zlib = require('node:zlib');

async function run() {
  const ac = new AbortController();
  const signal = ac.signal;

  setImmediate(() => ac.abort());
  await pipeline(
    fs.createReadStream('archive.tar'),
    zlib.createGzip(),
    fs.createWriteStream('archive.tar.gz'),
    { signal },
  );
}

run().catch(console.error); // AbortError

stream.finished: when you need to do cleanup on streams

### Duplex 

Duplex contains 2 separate Buffer: a Readable and a Writable

### Transforms

Transform like a Duplex has 2 buffer, but the change is that the readable can pass data to the Writable
Should implement the _transform function

const { Transform } = require('node:stream');
const fs = require("node:fs/promises")

// encryption.decryption -> crypto
// hashing/salting -> crypto
// compression -> zlib module
// decoding/encoding -> buffer text

class Encrypt extends Transform {
  _transform(chunk, encoding, callback) {
    for (let index = 0; index < chunk.length; index++) {
      // <Buffer 54 6f 20 62 65 20 72 65 61 64 20 6c 61 74 65 72 21>
      // 54 is 84 in decimal
      // we can add one to each byte or any number unti  65535 (16 bits)
      /*
      const b = Buffer.from('string');
      b[0] = b[0] + 1;
      */
      if (chunk[index] !== 255) {
        chunk[index] = chunk[index] + 1
      }
      
    }
   callback(null, chunk)
  }
}

(async () => {
  const readFileHandle = await fs.open('read.txt', 'r');
  const writeFileHandle = await fs.open('write.txt', 'w');

  const readStream = readFileHandle.createReadStream();
  const writeStream = writeFileHandle.createWriteStream();

  const encrypt = new Encrypt();

  readStream.pipe(encrypt).pipe(writeStream);
})();

## Outro

You can use stream for video streaming, reading writing copying files
also reading from db with node-pg (postgres) or pg-query-stream from npm


