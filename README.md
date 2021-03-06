# FillerWordsShock

Shock Collar for Presentation "Filler Words"

![Shock Collar](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Shock%20Collar.png)

(No, not an actual shock collar🙂)

## Why

I've been speaking in public for about 40 years. I am always looking to improve my pace, clarity, informational value, and educational value. My new job, which I love, does not require as much public speaking as my old job and I am a little out of practice. I have noticed a few filler words creeping back into my presentations. Words such as "Um" and "Uh". 

I thought "Wouldn't it be great if, while I am presenting, an app was converting my spoken words to text real time and flagging filler words?" These flagged words would give me a little reminder to slow down and take more pauses if needed.

## Version 1 - Google Speech to Text API (Semi-Fail)

![GCP Output](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Google%20Mic%20to%20Text.png)

I am a student of machine learning and familiar with the Google Cloud APIs for all sorts of tasks. When a Google search for "Real Time Speech to Text" turned up a result for using Google Cloud Speech to Text for this, I was all in. [This page](https://cloud.google.com/speech-to-text/docs/streaming-recognize) provides code samples in multiple programming languages to send either recorded audio input (16 KHz, Mono) or microphone input to Google Cloud and then transcribed text is sent back in real time.

Even though I have never used the Go programming language, I used the Go samples as Python is typically a nightmare of failed pip installs for me. Python really is very unreliable. 

I used Linux for this test as I started with macOS, but the real time microphone input requires the use of the gst-launch-1.0 program which does not exist on macOS. gst-launch-1.0 passes the microphone input to the Go program in the correct format for the Google speech to text API (gst-launch-1.0 -v pulsesrc ! audioconvert ! audioresample ! audio/x-raw,channels=1,rate=16000 ! filesink location=/dev/stdout | go run gcp-mic-text.go).

### Installing Go

To install the Go language, version 1.14.2, on Linux:

* cd /tmp
* wget https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
* sudo tar -xvf go1.11.linux-amd64.tar.gz
* sudo cp -r go /usr/local

Once done, running the "go version" command should return this output: "go version go1.14.2 linux/amd64"

### Creating the Real-Time Microphone to Text Go Progran

I made a few changes to the sample real-time program provided by Google [here](https://github.com/GoogleCloudPlatform/golang-samples/blob/master/speech/livecaption/livecaption.go). You can find my changed file in the Google folder of this repo.

Basically, I replaced the print of the entire array of alternative text results to print only index 0 (the highest confidence translation). I also printed just the text (.Transcript) and not all the other fields such as "Result: alternatives:<transcript:" and "confidence:0.9377223 > is_final:true result_end_time:<seconds:12 nanos:70000000 >". This made the output much easier to read.

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

Imagine my surprise after all this work when I could not get the word "Um" to show up in my speech-to-text no matter how many times I said um. It turns out that the Google speech to text model was built to supress filler words. Filler words will never show up in the transcripts. Oh, well. Let's see if Azure or IBM Watson leave filler words in the transcript. 

## Version 2 - Azure (Confusing) then IBM Watson (Success!)

![Watson Keyword Detection](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Watson%20Keywords.png)

So, fresh from my semi-failure with Google speech to text, I searched for other speech-to-text APIs. I found this very helpful article which recommended Google, Microsoft, Dialogflow, IBM Watson, and Speechmatics: https://nordicapis.com/5-best-speech-to-text-apis/

### Short Stop at Microsoft Azure Cognitive Services

I have done some simple development in Azure, so I thought I would try [Microsoft Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/) next. I started a project in Microsoft Cognitive Services but could not figure out how to upload audio to the Speaker Recognition API Quick Start page. I'm sure there is a way, but I Googled for a while and could not figure this out. On to IBM Watson Speech-to-Text...

![Azure Cognitive](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Azure%20Cognitive.png)

### On to IBM Watson Speech-to-Text

Next step, IBM Watson. I have never used any IBM Watson services before but have heard good things. Also, I do not have a billable account with IBM Watson as I do with AWS, Azure, and GCP. Thankfully, IBM Watson Speech to Text does not require a billing account for < 500 minutes of audio trnascribed per month. Thank you very much.

#### Create an IBM Account

If you don't already have one, create an account on ibm.com and head to this speech to text tutorial: https://cloud.ibm.com/apidocs/speech-to-text#recognize-audio

#### Get the Plumbing Tested with Curl

Curl is the easiest way to test that you have a working personalized IBM speech to text API key and service URL. To create your API and service URL, create a speech-to-text service on this page: https://cloud.ibm.com/catalog. 

![Speech Service](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Speech%20to%20Text%20Service.png)

Once the service is created, you can find your API key and your service URL on this page: https://cloud.ibm.com/resources

Find your speech-to-text resource, click on it and copy your API key and service URL

![Resource List](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Resource%20List.png)

![Credentials](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Credentials.png)

Now, back to the tutorial page to test curl access to speech-to-text. https://cloud.ibm.com/apidocs/speech-to-text#recognize-audio

You will need to download the IBM sample audio file for testing. You can find that audio file here: https://watson-developer-cloud.github.io/doc-tutorial-downloads/speech-to-text/reference/audio-file.flac

You should now have all the pieces you need for your first test (assuming you have curl on your machine). For these instructions we will asume:
* API Key: heAbpAFxASAMAuIBmADrkLwAlWx5cAt2_e1aUJw348AU
* Service URL: https://api.us-south.speech-to-text.watson.cloud.ibm.com/instances/af3a7056-ef6d-4f39-896f-e164f930afef/v1/recognize (Need to append /v1/recognize to your service URL
* Audio File Location: /Users/foobar/Music/audio-file.flac

Try running this from your command line: 

````bash
curl -X POST -u "apikey:heAbpAFxASAMAuIBmADrkLwAlWx5cAt2_e1aUJw348AU" \
--header "Content-Type: audio/flac" \
--data-binary @/Users/foobar/Music/audio-file.flac \
https://api.us-south.speech-to-text.watson.cloud.ibm.com/instances/af3a7056-ef6d-4f39-896f-e164f930afef/v1/recognize
````

If everything went well, you should see this output:

````JSON
{
   "results": [
      {
         "alternatives": [
            {
               "confidence": 0.94, 
               "transcript": "several tornadoes touched down as a line of severe thunderstorms swept through Colorado on Sunday "
            }
         ], 
         "final": true
      }
   ], 
   "result_index": 0
}
````

If you want to create your own WAV files for testing, the files have to be 16KHz and mono. On my Mac, I used the "To WAV Converter" app found on this web site https://amvidia.com/wav-converter and used these settings:

![To WAV Converter Settings](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/ToWAVConverter.png)


#### Creating a Custom Microphone to Text App with Trigger Words

I was going to write a custom Go program as I did with GCP, until I stumbled across this beauty: https://speech-to-text-demo.ng.bluemix.net/. Everything I need. I cloned the GitHub repo to my Mac from this URL: https://github.com/watson-developer-cloud/speech-to-text-nodejs. To run this locally, just type "npm start" from the speech-to-text-nodejs directory (assuming you have npm installed). Point your web browser to localhost:3000 and you will be able to use the demo locally.

![Microphone Demo](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Working%20Demo.png)

This demo is written is JavaScript and I have just about diddly experience in JavaScript, so finding the files to change to my liking took my poor puny brain a few days. With the immense help of both the "grep -iRl" command and the Atom IDE project search, I was able to find the files I needed to change.

##### Change #1 - Replace default keywords to spot

Not really a big deal as this field is user-editable before clicking "Record Audio" in the app, but still nice to find. You can modify the default keywords that come up in the app in the file /src/data/samples.json. Line 41 in the section "en-US_BroadbandModel" is where the default keywords are defined. You can change/reduce these to suit your needs.

````JSON
"en-US_BroadbandModel": [
    {
      "filename": "en-US_Broadband_sample1.wav",
      "keywords": "IBM, admired, AI, transformations, cognitive",
      "speaker_labels": true
    },
    {
      "filename": "en-US_Broadband_sample2.wav",
      "keywords": "Artificial Intelligence, data, predict, learn",
      "speaker_labels": true
    }
  ],
````

##### Change #2 - Highlight Ums & Uhs

While Google suppresses all filler words. IBM Watson replaces filler words with the string "%HESITATION". I discovered this through testing and eventually found the documentation on this page: https://cloud.ibm.com/docs/speech-to-text?topic=speech-to-text-basic-response#hesitation. All I needed to do was make sure that %HESITATION was one of my default keywords and I could be reminded of my filler words when speaking

##### Change #3 - Simplify and Intesify Spoken Filler Word Alerts

The default display for discovered keywords is \[Word\]: Spotted - Start Time-End Time (Confidence%). For example:

**Dennis**: Spotted - 7.26-7.94s (90%)

That's lovely, but I wanted something simpler such as 💥\[Word\] (Confidence%). With the help of my friend Atom, I found the display syntax in the file ./views/keywords.jsx on lines 45-47:

````JavaScript
        <span className="base--p_light">
          {(spottings || []).map(s => `${s.start_time}-${s.end_time}s (${Math.round(s.confidence * 100)}%)`).join(', ')}
        </span>
````

I hacked away at this file until I had something that worked for me. Here is my change:

````JavaScript
        <span className="base--p_light">
           {(spottings || []).map(s => `💥 (${Math.round(s.confidence * 100)}%)`).join(', ')}
        </span>
````

Here is the new output for discovered keywords:

**Dennis**: Spotted - 💥 (97%)

##### Change #4 - Increase Confidence Floor

During my testing, I was receiving alerts even when I did not say filler words. It turns out that the demo program has a confidence floor of 1% before a translated word is flagged. I wanted to increase this confidence floor to 50% and test again. While opening random files in the repo in Atom, I found this section of ./views/demo.jsx on lines 117-119:

````Javascript
      keywords_threshold: keywords.length
        ? 0.01
        : undefined, // note: in normal usage, you'd probably set this a bit higher
````

I changed the 0.01 to 0.50 and the new section now looks like this:

````Javascript
      keywords_threshold: keywords.length
        ? 0.50
        : undefined, // note: in normal usage, you'd probably set this a bit higher
````

You can find this modified file in the Watson directory of this repo.

##### Listening and Displaying Alerts on Another Device

So, now everything is working. Yay. Now what microphone do I use to listen for and alert on filler words? That microphone needs to be different from the microphone I am using to present over WebEx at my desk. I had a few options:

1. Connect a second microphone to my Mac and tell the browser on my Mac to listen to this second microphone
2. Connect a second microphone to the Linux VM used for the Google test and listen in this VM
3. Listen from the microphone and browser on my phone propped in front of me while I present

I chose option 3 even though option 3 required a hack.

##### Google Chrome Hack to Allow Microphone Input for Insecure Web Site

Here's the issue I ran into. This demo code runs a simple http web server. Most browsers, reasonably, do not allow microphone nor camera input to insecure web servers. Opening the microphone on my Mac to a web server running on the same Mac (localhost) is OK. Opening the microphone on my phone to some random IP address that just happens to be my Mac is not OK. Thankfully, you can override this microphone block for trusted, insecure web servers in Chrome by going to the chrome://flags/#unsafely-treat-insecure-origin-as-secure setting in Chrome:

![Insecure Settings](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Insecure.png)

Change the setting from Disabled to Enabled and type the full path to your web server in the text field. For example: http://192.168.86.24:3000 Once you do this, you will be prompted with a button to relaunch your browser. Relaunch your browser and you should be able to use the microphone on your phone (or any other device) to provide input to your speech to text application. Here is a photo of the entire system in use the other day as I was giving a PowerPoint presentation:

![Phone Monitor](https://github.com/DennisFaucher/FillerWordsShock/blob/master/images/Phone%20Monitor.png)

## Thank You

Thank you for taking the time to read this tutorial. Please let me know if you have any issues making this work for yourself, if you find any errors, or have any suggestions.
