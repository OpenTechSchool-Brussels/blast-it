---
title: Setting Up
layout: default
num: 1
---
##test##
blablabla

```java
   var i = 3;
```

[Bonjour](google.com)
[Image](http://media.melty.fr/google-journee-de-la-femme-image-438484-article-ajust_930.jpg)

##Base Code##

```java

#include <iostream>

#include <SFML/Window.hpp>
#include <SFML/Graphics.hpp>

#include "RtAudio.h"

int main (int argc, char** argv)
{
    // create the RtAudio object
	RtAudio adac;
	if (adac.getDeviceCount() < 1)
    {
		std::cout << "\nNo audio devices found!\n";
		return EXIT_FAILURE;
	}

    // create a window
    sf::RenderWindow window(sf::VideoMode(640, 480), "Blast it!", sf::Style::Default);
    window.setActive();

    // main loop
    while (window.isOpen())
    {
        // handle events
        sf::Event event;
        while (window.pollEvent(event))
        {
            switch (event.type)
            {
                // Close window: exit
                case sf::Event::Closed:
                    window.close();
                    break;
            
                // Resize event: adjust the viewport
                case sf::Event::Resized:
                    window.setView(sf::View(sf::FloatRect(0.f, 0.f, static_cast<float>(event.size.width), static_cast<float>(event.size.height))));
                    break;
            }
        }

        // display
        window.clear();
        window.display();
    }

    return EXIT_SUCCESS;
}

```

##Digital Audio##

###Sound###
Sound is basically waves travelling in the air, an analog signal. 
Computer-based audio hardware converts this signal to digital in order to be stored or treated by software, and converts the digital back to analog to be able to output it to a pair of headphones or speakers.

A digital audio signal is defined by a sample rate (ex: 44100Hz or samples / second) and a sample size (ex: 8, 16, 24bit) and a number of channels per sample (ex: mono, stereo, 5.1).

Software-wise, the behavior is the same whatever the platform or hardware (from a microcontroller to a computer or a smartphone): a callback is called whenever the hardware needs data in or out. The frequency of the callback depends on the way you configure the audio stack for your program, especially the callback buffer size. The smaller the audio buffer the lower the latency. In that regard, all platforms aren’t equal, so this workshop we’ll work with 512 samples sized buffers, but don’t hesitate to experiment with different values and experiment the limits of what your system can do.

Remark: on Windows, ASIO drivers usually offer way better latency than standard DirectSound ones in case your audio hardware is compatible. RtAudio, the audio framework we’ll use in this workshop both support DirectSound and ASIO. 

###Audio Buffer###

An audio sample is a set of values that correspond to the different audio channels.
In our configuration an audio channel length is 16bit x 2 channels for a total of 32bit / sample.

We chose Interleaved format so Left and Right channels are arranged successively in the audio buffer:
[ interleaved illustration: L R L R L R L R etc ]
In order to easily access the individual channels we’ll represent audio buffers as short* buffer; in the code.
Therefor, to allocate a 512-sample-long stereo buffer we’ll do:

```java
short* buffer = new short[512 * 2];
```

###Init Audio Stream###

For this workshop we’ll configure the audio system like this:
frequency: 44100 Hz
16bit / channel / sample
2 channels (stereo) interleaved
512 samples audio buffer

Let’s create a Data structure that will hold with those information:

```java
struct Data
{
    // audio
    unsigned char channels = 2;
    unsigned int sampleRate = 44100;
    unsigned int bufferFrames = 512;
    unsigned char channelBytes = 2;
    unsigned int bufferBytes = 0;
};
```

Create an object of type Data and set the bufferBytes attribute with the size of the audio buffers we will deal with (in bytes):

```java
    Data data;
    data.bufferBytes = data.bufferFrames * data.channels * data.channelBytes;

```

Using both input and output, we will use RtAudio in duplex mode.
RtAudio therefor requires a RtAudio::StreamParameters object for both input and output devices:

```java
	RtAudio::StreamParameters iParams, oParams;
	iParams.deviceId = adac.getDefaultInputDevice();
	iParams.nChannels = data.channels;
	iParams.firstChannel = 0;
	oParams.deviceId = adac.getDefaultOutputDevice();
	oParams.nChannels = data.channels;
	oParams.firstChannel = 0;
```

Now we can open an audio stream by using the different values we've just setup:

```java
	try
	{
		adac.openStream( &oParams, &iParams, RTAUDIO_SINT16, data.sampleRate, &data.bufferFrames, &ioCallback, (void*)&data);
	}
	catch (RtAudioError& e)
	{
		std::cout << '\n' << e.getMessage() << '\n' << std::endl;
		exit( 1 );
	}
```

Remark:
Note that in case of an error occuring, RtAudio functions throw exceptions instead or return an error code.

Before starting the stream, we need to create an 'ioCallback' function that will be called every 512 samples. 
Since we've setup a 44100Hz stream, the function will be called every 1000 / (44100 / 512) = 0.0116 seconds (11.6 milliseconds).
We will check that by displaying the streamTime that's given by the callback function:


```
int ioCallback (void *outputBuffer,
                void *inputBuffer,
                unsigned int nBufferFrames,
                double streamTime,
                RtAudioStreamStatus status,
                void *userData)
{
    std::cout << streamTime << std::endl;
}
```

Remark: in Duplex mode, RtAudio calls a single callback, but using other APIs you can get an input callback + an output callback. 

Remark: the audio callback doesn’t occur in the main thread, it’s an important information if you plan to share data with the audio engine.

Now that everything's in place, it's time to test that out by starting our audio stream and get our callback function called:

```java
	try
	{
		adac.startStream();
	}
	catch (RtAudioError& e)
	{
		std::cout << '\n' << e.getMessage() << '\n' << std::endl;
	}
```

To prevent our program from quitting directly, let's add a sleep call of 100 in the while loop:

```java
    sf::sleep(sf::milliseconds(10));
```

##Monitoring##

Ok! So, it's nice, we have our audio system running but we don't hear anything do we? 
Let's add a single line to the callback function to actually hear us speaking through our headphones!

##Visualization##

##Sampler##

##Effect##