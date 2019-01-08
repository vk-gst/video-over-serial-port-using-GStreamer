# video-over-serial-port-using-GStreamer
This repository contains information over the challenges I faced, to get video streaming over a serial port interface, and the methods involved in debugging it. The post is more focussed towards debugging the serial interface to understand the behavior, rather than explaining the solution of video streaming over serial port. 


Recently I was working on developing an embedded video server for streaming live video using the GStreamer framework. During the course of my work, I encountered a particular situation, where I wanted to try out if GStreamer could be used to stream video over the serial port interface, with the limitations of bandwidth and baud rate. I had 2 radio modules from Silicon labs that operated at 433Mhz and were exposed via the `/dev/tty` interfaces in the Linux file system. Since Linux file system exposes serial ports as device files, I came up with the idea of reading and writing the video frames just as reading and writing to a file. GStreamer provides means to read and write to files, in a simpler and easier way.
Even though the radio modules, were close to each other, I could observe a lot of packet losses, which resulted in garbled video output. 
After a lot of reading through and a generous help from some GStreamer experts, I could narrow down the issue and proceed. Here are a few tips I took to debug this issue: 

 

1. make a test file of binary random numbers; pick a suitable size, 
my choice was 1 MB for a 115200 baud test, 

    `dd if=/dev/urandom of=test bs=1024k count=1` 

2. test the file 

    `sha256sum test` 

3. set serial ports to raw, no echo, 

    `stty 115200 raw -echo < /dev/ttyUSB0` 
    `stty 115200 raw -echo < /dev/ttyUSB1` 

4. in one terminal, start receiving, 

    `pv /dev/ttyUSB1 > test.received `

5. in another terminal, start transmitting, 

    `pv test > /dev/ttyUSB0`

6. when the transmission is finished, stop the receiver with ctrl-c, 

7. compare the file sizes, they should be equal, 

     `ls -l tes* `
    `-rw-r--r-- 1 root root 1048576 Jun 27 06:29 test `
    `-rw-r--r-- 1 root root 1048576 Jun 27 06:32 test.received` 

8. compare the file sha256sum, 

9. use a binary comparison program, or  hexdump each file and use a textual differences program to analyze the differences. 


With the above steps, I found out that using the radio module, I was receiving only 70% of the bytes that I was transmitting from source. The rest of the bytes were lost due to packet loss and other interferences. 

The next step, I performed was to use a NULL modem cable, and test the same thing. Voila! I could receive the complete data as it is and the `sha256sum` for the Tx and Rx files matched. 

Now this same approach, can be used to debug some wireless modules that are exposed in Linux as serial port. This would give you an idea on how reliable the wireless module is, and an overview over the packet losses. 

The next thing was to use the same concept on a custom in house built modem - which was also exposed as a `tty` interface in Linux framework. However the baud was much higher as compared to a primitive UART device. At about 2MBaud, I could get a decent video streaming working - albeit the latency was noticeable. But considering the limitations of the modem, I feel it still can be considered as an acceptable behaviour. I would experiment again with a new codec and low bitrate to see, if it improves the behaviour with regards to video latency.
