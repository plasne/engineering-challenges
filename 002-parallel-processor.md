# Parallel Processor

A few years ago, I developed a system that would convert a brick seismic file into a SEGY trace siesmic files. There are open source projects that do this, but they all worked by using a lot of memory, disk, and processing power on a single node. Instead, I wanted to write something that used dozens or hundreds of container replicas process the conversion as quickly as possible. The details of how this works is beyond the scope of this article, but consider this very simple example:

```
brick 1:      brick 2:      trace:
                            file-header
aaaaaaaaaa    dddddddddd    aaaaaaaaaadddddddddd   (xl 0-19, il 0)
bbbbbbbbbb    eeeeeeeeee    bbbbbbbbbbeeeeeeeeee   (xl 0-19, il 1)
cccccccccc    ffffffffff    ccccccccccffffffffff   (xl 0-19, il 2)

NOTE: Each letter is a series of samples of a specific size (ex. 100 samples, each 4 bytes)
```

Think of a brick as a cube where CROSSLINEs are X, INLINEs are Y, and SAMPLEs are Z.

```
\------------------\
|\                  \ samples (z)
| \                  \
|  \------------------\
|  |  crosslines (x)  |
|  |                  |
\  |                  | inlines (y)
 \ |                  |
  \|------------------|
```

Think of a trace format where each trace has a header followed by a series of samples. The header contains information about the trace, such as the crossline and inline number. The samples are the actual seismic data.

```
FILE HEADERS
TRACE HEADER (crossline: 0, inline: 0)
TRACE SAMPLES (ex. 100 samples)
TRACE HEADER (crossline: 1, inline: 0)
TRACE SAMPLES (ex. 100 samples)
...
TRACE HEADER (crossline: 1001, inline: 0)
TRACE SAMPLES (ex. 100 samples)
TRACE HEADER (crossline: 0, inline: 1)
TRACE SAMPLES (ex. 100 samples)
...
```

There is a lot to this problem, but here we are just going to concentrate on producing the output file.

## Challenge

Write the design for a system that meets the following requirements:

- Ultimately a single output file needs to be produced with a file header followed by a bunch of CROSSLINE, INLINE, and SAMPLE data.

- SEGY headers can contain arbitrary data. In this case, we need to know the minimum and maximum value of samples for each CROSSLINE. For example:

    - CROSSLINE 0 MIN: 0.0 MAX: 1.0
    - CROSSLINE 1 MIN: 0.1 MAX: 1.4

- Assume the input files are 10,000 bricks each consisting of 10 CROSSLINEs, 10 INLINEs, with 100 SAMPLEs each. Each sample is 4 bytes each. Think of the bricks in a 100x100 grid. Each brick would be 40k in size.

- Assume the output files have 1000 CROSSLINEs, 1000 INLINEs, and 100 SAMPLEs. The output file will be greater than 400 MB including the headers.

- You must demonstrate how at least 10 container replicas can process in parallel.

- Assume you want to complete the write as quickly as possible.

## Considerations

There are some things to consider when designing this system:

- If the processing is distributed, how are you getting the minimum and maximum values for each CROSSLINE?

- Once you have the minimum and maximum values for each CROSSLINE, how and when are you writing that to the header?

- How do you know when you are done processing all the data so that you can close the file?

    - Keep in mind that if you are using a queue, it could show empty but still have messages leased that are processing or failed.

- Make sure you understand what locks and performance constraints might be introduced when writing with many simultaneous replicas.

- Ensure you understand any limitations of the technology you are using. For instance, the number of blocks that can be written to a block blob.