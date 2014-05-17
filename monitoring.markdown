---
title: Monitoring
layout: default
num: 2
---

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
