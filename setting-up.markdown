---
title: Setting Up
layout: default
num: 1
---

[Pierre Guilluy](http://proteinderklang.com/Credits/PierreGuilluy.html)

##Base Code##

Here's the base code you can copy and paste in main.cpp, compile and run to make sure both RtAudio and SFML run fine on your system.
This code first makes sure RtAudio is functionnal, then creates a window and runs an event loop using SFML.

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
		std::cout << "\nNo audio devices found!\n" << std::endl;
		return EXIT_FAILURE;
	}

    // create a window
    sf::RenderWindow window(sf::VideoMode(640, 480), "Blast it!", sf::Style::Default);
    window.setActive();
    window.setKeyRepeatEnabled(false);

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
                    
                // Key pressed
                case sf::Event::KeyPressed:
                    switch (event.key.code)
                    {
                        case sf::Keyboard::Escape:
                            window.close();
                            break;
                    }
                    break;
            }
        }
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
short* buffer = new short[512 * 2 * 2];
```

###Init Audio Stream###

For this workshop we’ll configure the audio system like this:
frequency: 44100 Hz
2 channels (stereo) interleaved
512 samples audio buffer

Let’s create a global "Data" structure that will hold with those information:

```java
struct Data
{
    // audio
    unsigned char channels = 2;
    unsigned int sampleRate = 44100;
    unsigned int bufferFrames = 512;
};
```

Create an object of type Data at the very beginning of the main function:

```java
    Data data;
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

We pass our Data object as the last argument "void* userData" so it will passed to the callback function.

Remark:
In case of an error occuring, RtAudio functions throw exceptions instead of returning an error code.

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
    Data* data = (Data*)userData;
    std::cout << streamTime << std::endl;
}
```

We got our Data object back from the callback userData.

Remark: 
In Duplex mode, RtAudio calls a single callback, but using other APIs you can get an input callback + an output callback.

Remark: 
The audio callback doesn’t occur in the main thread, it’s an important information if you plan to share data with the audio engine.

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

##Monitoring##

Ok! So, it's nice, we have our audio system running but we don't hear anything yet do we?
With an extra line in the callback function we can actually hear us singing through our headphones.

```java
    memcpy(outputBuffer, inputBuffer, data->bufferFrames * data->channels * sizeof(int));
```

Every audio frame, we basically copy the input buffer to the output one, creating kind of an audio monitoring system.

##Visualization##

Wouldn't it be nice to actually visualize the buffer that comes out? 

The we will create a buffer that will hold a frame of audio so we can display it as lines.
To do so, let's add an display buffer in our Data struct.

```java
    short* displayBuffer = NULL;
```

... then initialize it with in the main function:

```java
    data.displayBuffer = new short[data.bufferFrames * data.channels];
    memset(data.displayBuffer, 0, data.bufferFrames * data.channels * sizeof(short));
```

We need to update the buffer every audio frame, we'll add the following line at the end of the audio callback:

```java
    if (data->displayBuffer)
        memcpy(data->displayBuffer, (short*)outputBuffer, data->bufferFrames * data->channels * sizeof(short));
```

Finally we'll put the actual drawing code at the end of the main loop.

```java
        sf::VertexArray lines(sf::LinesStrip, data.bufferFrames);
        int yOffset = window.getSize().y / 2;
        int yScale = 65535 / window.getSize().y * 1.1;
        float xStep = (float)window.getSize().x / data.bufferFrames;
        for (int i=0;
             i < data.bufferFrames;
             i++)
        {
            lines[i].position = sf::Vector2f(xStep * i, yOffset + data.displayBuffer[i*2] / yScale);
        }
        window.clear();
        window.draw(lines);
        window.display();
```

We basically draw a series of lines joining the samples of the audio buffer continuously, the values being centered along the horizontal axis of the window.

##Sampling##

We now can hear and watch audio, but let's do something a bit more fun.
We will create a loop station, similar to the looping pedal beatboxers use.

Let's add a few members to the Data struct:

```java
    // sampling
    bool playing = false;
    bool recording = false;
    short* samplingBuffer = NULL;
    unsigned int samplingBufferMaxFrames = 0;
    unsigned int samplingBufferCurrentFrames = 0;
    unsigned int playbackCursor = 0;
```

We initialize the buffer so it can hold about 5 seconds of audio.
For conveniency we make sure the buffer has a number of samples that's a multiple of data.bufferFrames (= 512).

```java
    // init the sampling data
    char maxSeconds = 5;
    data.samplingBufferMaxFrames = data.sampleRate * maxSeconds;
    data.samplingBufferMaxFrames -= data.samplingBufferMaxFrames % data.bufferFrames;
    data.samplingBuffer = new short[data.samplingBufferMaxFrames * data.channels];
```

###Recording###

In the audio callback we will copy the successive audio-input buffers to the sampling buffer, until the user stops it or when its full.
We keep track of the number of recorded frames in data->samplingBufferCurrentFrames.

```java
    // recording
    if (data->recording)
    {
        // record samples
        short* buffer = data->samplingBuffer + data->samplingBufferCurrentFrames * data->channels;
        memcpy((void*)buffer, inputBuffer, data->bufferFrames * data->channels * sizeof(short));
        data->samplingBufferCurrentFrames += nBufferFrames;
        if (data->samplingBufferCurrentFrames >= data->samplingBufferMaxFrames)
        {
            data->recording = false;
            data->playing = true;
        }
    }
```

###Playback###

When playing back we go the other way around, we increment the playback cursor and copy the right number of audio frames to the output buffer. 
In this case we use samplingBufferCurrentFrames as the maximum frame we can play until we have to loop the cursor back to the beginning.
We do so by using the modulo (%) operator which is perfect for looping.

```java
    if (data->playing && data->samplingBufferCurrentFrames > 0)
    {
        // play the sampling buffer in loop
        short* buffer = data->samplingBuffer + data->playbackCursor * data->channels;
        data->playbackCursor += nBufferFrames;
        data->playbackCursor %= data->samplingBufferCurrentFrames;
        memcpy(outputBuffer, buffer, data->bufferFrames * data->channels * sizeof(short));
    }
```

Remark: 
You may want to temporarily deactivate the monitoring to be able to listen to the audio playback.
Don't worry, we will re-activate it a bit later.

###Control###

We miss a way to control the recording and playback at will.
Let's go to the main function and handle two new key events.

```java
                        case sf::Keyboard::Space:
                            data.recording = !data.recording;
                            data.playing = !data.recording;
                            if (data.recording)
                                data.samplingBufferCurrentFrames = 0;
                            else
                                data.playbackCursor = 0;
                            break;
                            
                        case sf::Keyboard::Return:
                            data.playing = !data.playing;
                            if (data.playing)
                                data.playbackCursor = 0;
                            break;
```

You will record a new audio loop by pressing Space once to start and another time to stop.
The playback will start immediately after the recording's finished.
When you're bored of listening to the same loop over and over, you can either record a new one, or stop it by pressing the Return key - you can press it again to restart the loop.
Note that we make sure the different cursors are reset before recording or playing back. 

Have fun:
You can easily modify the playback algorithm to modify the speed of the playback, and control it using the mouse for example.

##Mixing##

At this point we can either monitor our audio input or use a loop station.
Wouldn't it be nice if we could mix both, so we can sing on top of a beatbox loop for example?

Like on a mixing table, we will combine several signals in a single one.
In addition to that, each signal will have its own audio gain so we can control its weight in the mix.

In digital audio, mixing signals is basically their values on top of each other.
As you might think the process will quickly generate values that are greater that what's possible in 16bit.

In order to prevent that from happening we will perform the mixing of the different signal values in an buffer of integers (32bit) that can hold much bigger values than a buffer of shorts (16bit).

We will first add such a buffer in the Data structure

```java
    // mixing
    int* mixingBuffer = NULL;
```

and instanciate it with the same size as the others, the difference being that one value has twice the size of the previous buffers:

```java
data.mixingBuffer = new int[data.bufferFrames * data.channels];
```

Remark:
Make sure to do so before starting the audio stream!

The idea now is to reset the mixing buffer before adding values from different signal into it.

First, make sure to fill the mixing buffer with zeros at the very beginning of the audio callback:

```java
    memset(data->mixingBuffer, 0, data->bufferFrames * data->channels * sizeof(int));
```

Note that we use sizeof(int) instead of sizeof(short);

Then,



##Effect##