---
layout: post
title:  "Memory efficient way of reading and downloading file in Ruby"
image: /assets/files_image.jpg
tags: rails
---

### To read a large file from the disk

[`File.foreach`](https://ruby-doc.org/core-2.7.1/IO.html#method-c-foreach){:target='\_blank'} method reads the file line by line that is why it is safe to use for large files.  
It can accept the block to execute for the each line of a file.

Example:  

```ruby
File.foreach('example.jsonl') { |line| JSON.parse(line) }
```

However, in the Ruby documentation you will not find this method defined on the [`File`](https://ruby-doc.org/core-2.7.1/File.html){:target='\_blank'} class. It is defined on [`IO`](https://ruby-doc.org/core-2.7.1/IO.html){:target='\_blank'} which is a superclass of [`File`](https://ruby-doc.org/core-2.7.1/File.html){:target='\_blank'}.


### To download the large files from Internet

We can use [`IO.copy_stream`](https://ruby-doc.org/core-2.7.1/IO.html#method-c-copy_stream){:target='\_blank'} to download the large files from the Internet. It will create a stream instead of loading the whole file into memory before writing.

Example:  

```ruby
IO.copy_stream(open('http://example.com/file.jsonl'), 'file.jsonl')
```
<br />

To sum it up...

Memory friendly methods:  
[`File.foreach`](https://ruby-doc.org/core-2.7.1/IO.html#method-c-foreach){:target='\_blank'}, [`IO.copy_stream`](https://ruby-doc.org/core-2.7.1/IO.html#method-c-copy_stream){:target='\_blank'}

Other methods to read a file:  
[`File.read`](https://ruby-doc.org/core-2.7.1/IO.html#method-c-read){:target='\_blank'}, [`File.readlines`](https://ruby-doc.org/core-2.7.1/IO.html#method-c-readlines){:target='\_blank'}
