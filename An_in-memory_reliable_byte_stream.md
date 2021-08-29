# An in-memory reliable byte stream

## Function requirement

- Bytes are written on the “input” side and can be read, in the same sequence, from the “output” side.
- The byte stream is finite: 
  - the writer can end the input, and then no more bytes can be written. 
  - When the reader has read to the end of the stream, it will reach “EOF” (end of file) and no more bytes can be read.
- The byte stream will also be flow-controlled to limit its memory consumption at any given time.
  - The object is initialized with a particular “capacity”: the maximum number of bytes it’s willing to store in its own memory at any given point. 
  - The byte stream will limit the writer in how much it can write at any given moment, to make sure that the stream doesn’t exceed its storage capacity. 
  - As the reader reads bytes and drains them from the stream, the writer is allowed to write more.
  - The byte stream is finite, but it can be almost arbitrarily long (maybe much longer than the capacity) before the writer ends the input and finishes the stream.
  - The capacity limits the number of bytes that are held in memory (written but not yet read) at a given point, but does not limit the length of the stream.

- Used in single thread.

## Interface

### Writer

```c++
// Write a string of bytes into the stream. Write as many
// as will fit, and return the number of bytes written.
size_t write(const std::string &data);
// Returns the number of additional bytes that the stream has space for
size_t remaining_capacity() const;
// Signal that the byte stream has reached its ending
void end_input();
// Indicate that the stream suffered an error
void set_error();
```

### Reader

```c++
// Peek at next "len" bytes of the stream
std::string peek_output(const size_t len) const;
// Remove ``len'' bytes from the buffer
void pop_output(const size_t len);
// Read (i.e., copy and then pop) the next "len" bytes of the stream
std::string read(const size_t len);
bool input_ended() const; // `true` if the stream input has ended
bool eof() const; // `true` if the output has reached the ending
bool error() const; // `true` if the stream has suffered an error
size_t buffer_size() const; // the maximum amount that can currently be peeked/read
bool buffer_empty() const; // `true` if the buffer is empty
size_t bytes_written() const; // Total number of bytes written
size_t bytes_read() const; // Total number of bytes popped
```

## Solution

### Data structure

Life is short, choose std::deque. Here's the reason:

- write()  push back, read() pop front. (FIFO)
- peek_output() needs search by index.

Why not std::vector?

- pop front (by erase the front element) will move the rest elements, cost will be huge.

Why not std::queue or std::stack?

- FIFO is yes for queue but no for stack.
- Both do not support search by index, peek_output will be painful.

What can std::deque do ?

- push_back and push_front
- pop_back and pop_front
- operator[]

Choose std::deque, the implement codes will be trivial.
