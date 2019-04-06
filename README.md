## Gostreamer Pipeline Example

This is an example on how to use [Gostreamer](https://github.com/picadoh/gostreamer) to build an execution pipeline

### Getting Gostreamer

    $ go get github.com/picadoh/gostreamer

### Build

    streamer$ go build hashtag_counter_pipeline.go

### Components
**Collector** receives a function that collects information and publish it in a channel in the form of a message.

**Processor** picks an input channel with messages and executes some function (passed as parameter) over it

**Demux** demultiplexes the input stream into multiple output streams based on a given number of output channels and a given index function.

### Hashtag Counter Pipeline

    ------------         -----------         --------         ---------         -----------
    | Collector | ----> | Extractor | ----> | Filter | ----> | Counter | ----> | Publisher |
    ------------         -----------         --------         ---------         -----------
         |                                                                           |
         |                                                                           |
        \|/                                                                         \|/
      input.txt                                                                    sysout


**Input (input.txt):**

    hello #world
    #world #golang
    the #world is #awesome
    #welcome to my #world
    just exploring #golang

**Config:**

Configuration is made to key/value properties in a text file as below:

    # Parallelism
    parallelism.collector = 2
    parallelism.extractor = 2
    parallelism.filter = 2
    parallelism.counter = 5
    parallelism.publisher = 5

    # Source
    # acceptable values for mode: socket, file
    source.mode = file
    source.file = input.txt
    source.port = 9999

### Running

    $ ./hashtag_counter_pipeline pipeline.cfg

If running in socket mode, you may want to produce some test messages, a simple way of doing it is by using nc command

    $ nc localhost 9999
    hello #world

**Sample Output:**

    2016/09/25 16:59:18.846279 Loaded configuration: map[parallelism.counter:5 parallelism.publisher:5 source.mode:file source.file:input.txt source.port:9999 parallelism.collector:2 parallelism.extractor:2 parallelism.filter:2]
    2016/09/25 16:59:18.846540 Read message from file: &{2016-09-25 16:59:18.846464351 +0100 WEST map[text:hello #world]}
    2016/09/25 16:59:18.846553 Read message from file: &{2016-09-25 16:59:18.846548939 +0100 WEST map[text:#world #golang]}
    2016/09/25 16:59:18.846560 Read message from file: &{2016-09-25 16:59:18.846558126 +0100 WEST map[text:the #world is #awesome]}
    2016/09/25 16:59:18.846565 Extracted word: hello
    2016/09/25 16:59:18.846569 Extracted word: #world
    2016/09/25 16:59:18.846573 Extracted word: #world
    2016/09/25 16:59:18.846578 Read message from file: &{2016-09-25 16:59:18.846576087 +0100 WEST map[text:#welcome to my #world]}
    2016/09/25 16:59:18.846604 Extracted word: #golang
    2016/09/25 16:59:18.846614 Extracted word: the
    2016/09/25 16:59:18.846621 Filtered hashtag #world
    2016/09/25 16:59:18.846646 Filtered hashtag #golang
    2016/09/25 16:59:18.846622 Extracted word: #welcome
    2016/09/25 16:59:18.846624 Filtered hashtag #world
    2016/09/25 16:59:18.846673 Publishing #golang/1
    2016/09/25 16:59:18.846679 Filtered hashtag #welcome
    2016/09/25 16:59:18.846683 Publishing #world/1
    2016/09/25 16:59:18.846692 Publishing #world/2
    2016/09/25 16:59:18.846708 Publishing #welcome/1
    2016/09/25 16:59:18.846652 Extracted word: #world
    2016/09/25 16:59:18.846667 Read message from file: &{2016-09-25 16:59:18.846621924 +0100 WEST map[text:just exploring #golang]}
    2016/09/25 16:59:18.846762 Extracted word: to
    2016/09/25 16:59:18.846779 Extracted word: my
    2016/09/25 16:59:18.846758 Filtered hashtag #world
    2016/09/25 16:59:18.846794 Publishing #world/3
    2016/09/25 16:59:18.846753 Extracted word: is
    2016/09/25 16:59:18.846804 Extracted word: #awesome
    2016/09/25 16:59:18.846811 Extracted word: just
    2016/09/25 16:59:18.846818 Extracted word: #world
    2016/09/25 16:59:18.846823 Filtered hashtag #awesome
    2016/09/25 16:59:18.846831 Filtered hashtag #world
    2016/09/25 16:59:18.846828 Extracted word: exploring
    2016/09/25 16:59:18.846844 Publishing #world/4
    2016/09/25 16:59:18.846846 Extracted word: #golang
    2016/09/25 16:59:18.846839 Publishing #awesome/1
    2016/09/25 16:59:18.846856 Filtered hashtag #golang
    2016/09/25 16:59:18.846914 Publishing #golang/2
    2016/09/25 16:59:18.846968 final count report: map[#welcome:%!s(int=1) #awesome:%!s(int=1) #golang:%!s(int=2) #world:%!s(int=4)]
