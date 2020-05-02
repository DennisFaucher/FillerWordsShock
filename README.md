# FillerWordsShock

Shock Collar for Presentation "Filler Words"

(No, not an actual shock collar🙂)

## Why

I've been speaking in public for about 40 years. I am always looking to improve my pace, clarity, informational value, and educational value. My new job, which I love, does not require as much public speaking as my old job and I am a little out of practice. I have noticed a few filler words creeping back into my presentations. Words such as "Um" and "Uh". 

I thought "Wouldn't it be great if, while I am presenting, an app was converting my spoken words to text real time and flagging filler words?" These flagged words would give me a little reminder to slow down and take more pauses, if needed.

## Version 1 - Google Speech to Text API (Semi-Fail)

![GCP Output](https://github.com/DennisFaucher/FillerWordsShock/blob/master/Google%20Mic%20to%20Text.png)

I am a student of machine learning and familiar with the Google Cloud APIs for all sorts of tasks. When a Google search for "Real Time Speech to Text" turned up a result for using Google Cloud Speech to Text for this, I was all in. [This page](https://cloud.google.com/speech-to-text/docs/streaming-recognize) provides code samples in multiple programming languages to send either recorded audio input (16 KHz, Mono) or microphone input to Google Cloud and transcribed text is sent back in real time.

Even though I have never used the Go programming language, I used the Go samples as Python is typically a nightmare of failed pip installs for me. Python really is very unreliable. 

I used Linux for this test as I started with macOS, but the real time microphone input requires the use of the gst-launch-1.0 program which does not exist on macOS. gst-launch-1.0 passes the microphone input to the Go program in the correct format for the Google speech to text API (gst-launch-1.0 -v pulsesrc ! audioconvert ! audioresample ! audio/x-raw,channels=1,rate=16000 ! filesink location=/dev/stdout | go run gcp-mic-text.go).

### Installing Go

To install the Go language, version 1.14.2, on Linux:

* cd /tmp
* wget https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
* sudo tar -xvf go1.11.linux-amd64.tar.gz
* sudo cp -r go /usr/local

Once done, running the "go version" command should return this output: "go version go1.14.2 linux/amd64"

### Creating the Real Time Microphone to Text Go Progran

I made a few changes to the sample real time program provided by Google [here](https://github.com/GoogleCloudPlatform/golang-samples/blob/master/speech/livecaption/livecaption.go). You can find my changed file in the Google folder of this repo.

Basically, I replaced the print of the entire array of alternative text results to index 0. I also printed just the text (.Transcript) and not all the other fields such as "Result: alternatives:<transcript:" and "confidence:0.9377223 > is_final:true result_end_time:<seconds:12 nanos:70000000 >". This made the output much easier to read.

My end goal was to flag my filler words such as "um" and "uh", so I Googled a bit of Go syntax and found the strings.ReplaceAll function. I added a bit of logic to replace all occurrences of "Um" with "\*\*Um\*\*" in the translated text.


````Go
for _, result := range resp.Results {
	// fmt.Printf("Result: %+v\n", result)
	before_text := result.Alternatives[0].Transcript
	after_text := strings.ReplaceAll(before_text, "Um", "**UM**")
	// fmt.Printf(" | %+v\n", result.Alternatives[0].Transcript)
	fmt.Printf(" | %+v\n", after_text)
}
````

Imagine my surprise after all this work when I could not get the word "Um" to show up in my speech to text no matter how many times I said um. It turns out that the Google speech to text model was built to supress filler words. Filler words will never show up in the transcripts. Oh, well. Let's see if Azure or IBM Watson leave filler words in the transcript. 

## Version 2 - IBM Watson Speech to Text API (Success!)

![Watson Keyword Detection](https://github.com/DennisFaucher/FillerWordsShock/blob/master/Watson%20Keywords.png)

So, fresh from my failure with Google speech to text, I searched for other speech to text APIs. I found this very helpful article which recommended Google, Microsoft, Dialogflow, IBM Watson, and Speechmatics: https://nordicapis.com/5-best-speech-to-text-apis/

### Short Stop at Microsoft Azure Cognitive Services

I have done some development in Azure, so I thought I would try [Microsoft Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/)  next. I started a project in Microsoft Cognitive Services but could not figure out how to upload audio to the Speaker Recognition API Quick Start page. I'm sure there is a way, but I Googled for a while and coudl not figure this out. On to IBM Watson Speech to Text

![Azure Cognitive](https://github.com/DennisFaucher/FillerWordsShock/blob/master/Azure%20Cognitive.png)

### On to IBM Watson Speech to Text

Next step, IBM Watson. I have never used any IBM Watson services before but have heard good things. Also, I do not have a billable account with IBM Watson as I do with AWS, Azure, and GCP. Thankfully, IBM Watson Speech to Text does not require a billing account for < 500 minutes of audio trnascribed per month. 

#### Create an IBM Account

If you don't already have one, create an account on ibm.com and head to this speech to text tutorial page: https://cloud.ibm.com/apidocs/speech-to-text

#### Get the Plumbing Tested with Curl

Curl is the easiest way to test that you have a working personalized IBM speech to text API key and service URL. To create your API and service URL, create a peech to text service on this page: https://cloud.ibm.com/catalog. 

![Speech Service](https://github.com/DennisFaucher/FillerWordsShock/blob/master/Speech%20to%20Text%20Service.png)

Once the service is created, you can find your API key and your service URL on this page: https://cloud.ibm.com/resources


