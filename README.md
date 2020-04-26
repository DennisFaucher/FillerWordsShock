# FillerWordsShock

Shock Collar for Presentation "Filler Words"

(No, not an actual schock collar🙂)

## Why

I've been speaking in public for about 40 years. I am always looking to improve my pace, clarity, informational value, and educational value. My new job, which I love, does not require as much public speaking as my old job and I am a little out of practice. I have noticed a few filler words creeping back into my presentations. Words such as "Um" and "Uh". 

I thought "Wouldn't it be great if, while I am presenting, an app was converting my spoken words to text real time and flagging filler words?" These flagged words would give me a little reminder to slow down and take more pauses, if needed.

## Version 1 - Google Speech to Text API (Semi-Fail)

![GCP Output](https://github.com/DennisFaucher/FillerWordsShock/blob/master/Google%20Mic%20to%20Text.png)

I am a student of machine learning and familiar with the Google Cloud APIs for all sorts of tasks. When a Google search for "Real Time Speech to Text" turned up a result for using Google Cloud Speech to Text for this, I was all in. [This page](https://cloud.google.com/speech-to-text/docs/streaming-recognize) provides code samples in multiple programming languages to send either recorded audio input (16 KHz, Mono) or microphone input to Google Cloud and transcribed text is sent back in real time.

Even though I have never used the Go programming language, I used the Go samples as Python is typically a nightmare of failed pip installs for me. Python really is very unreliable. 

I used Linux for this test as I started with macOS, but the real time microphone input requires the use of the gst-launch-1.0 program which does not exist on macOS. gst-launch-1.0 passes the microphone input to the Go program in the correct format for the Google speech to text API (gst-launch-1.0 -v pulsesrc ! audioconvert ! audioresample ! audio/x-raw,channels=1,rate=16000 ! filesink location=/dev/stdout | go run gcp-mic-text.go).

To install the Go language, version 1.14.2, on Linux:

* cd /tmp
* wget https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
* sudo tar -xvf go1.11.linux-amd64.tar.gz
* sudo cp -r go /usr/local

Once done, running the "go version" command should return this output: "go version go1.14.2 linux/amd64"
