---
title: Listen & Visualize It
layout: default
num: 2
---

##Audio Stream##

So ... back to coding. As you saw, there are many configurations that defines the way we handle the sound. We'll put all this funky stuff in one structure for us to deal with it better. This structure is small at first, but will grow more complexe with sun, water and love.

You already know about the sample rate & sample format, the number of channels and the buffer frame. Let's start by adding and defining them in the data structure.

```java
struct Data
{
    // audio
    unsigned char channels = 2;
    unsigned int sampleRate = 44100;
    RtAudioFormat sampleFormat = RTAUDIO_SINT16;    
    unsigned int bufferFrames = 512;
};
```

Don't forget to create an instance of this structure in your main (`Data data;`).

Now we need to tell Rtaudio how we will be using its magic. Since we will not only play sound but record some sick tunes too, we'll be working in duplex mode (both input and output). We need to tell RtAudio in both case how we want them parametrised. First they need a deviceID (what mic, and what speakers/headphones... to use), then the number of channels and last the offset of the first channel.

```java
	RtAudio::StreamParameters iParams, oParams;
	iParams.deviceId = adac.getDefaultInputDevice();
	iParams.nChannels = data.channels;
	iParams.firstChannel = 0;
	oParams.deviceId = adac.getDefaultOutputDevice();
	oParams.nChannels = data.channels;
	oParams.firstChannel = 0;
```

Armed with all that, we can now open an audio stream. For that, we need to feed it with first the two parametres (output, input). Then the sample format, sample rate and number of frames in the buffer. Then it gets interesting. We share an ioCallback object and a data one. ioCallback will be the function called each time the sound card needs you (outputing new buffers and needing new in entry). Last, the data object is a way to share information, and yep, you guessed it, we'll share our own data object.  
In case of an error occuring, RtAudio functions throw exceptions instead of returning an error code.

```java
	try
	{
		adac.openStream( &oParams, &iParams, data.sampleFormat, data.sampleRate, &data.bufferFrames, &ioCallback, (void*)&data);
	}
	catch (RtAudioError& e)
	{
		std::cout << '\n' << e.getMessage() << '\n' << std::endl;
		return EXIT_FAILURE;
	}
```

In order for the callback to be called, we need to define it! The call back has already a structure for us. It takes in input references to the outputBuffer (what will be heard) and the inputBuffer (what we recorded), as well as the number of frames in those buffers, the stream time, a status variable and last but not least: our data object. This fonction is a global one, so you need to type it at the root of your code (before your main function).

The 'ioCallback' function will hence be called every 512 samples. Since we've setup a 44100Hz stream, the function will be called every 1000 / (44100 / 512) = 0.0116 seconds (11.6 milliseconds). Let's check that by displaying the streamTime that's given by the callback function:

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
    return 0;
}
```

We got our Data object back from the callback userData.

Remark:  
In Duplex mode, RtAudio calls a single callback, but using other APIs you can get an input callback + an output callback.

Remark:   
The audio callback doesn’t occur in the main thread, it’s an important information if you plan to share data with the audio engine.

Now that everything's in place, it's time to test that out by starting our audio stream and get our callback function called (in our main function):

```java
	try
	{
		adac.startStream();
	}
	catch (RtAudioError& e)
	{
		std::cout << '\n' << e.getMessage() << '\n' << std::endl;
		return EXIT_FAILURE;
	}
```

##Monitoring##

Ok! So, it's nice, we have our audio system running but we don't hear anything yet do we? Let's make a monitoring system in which we will hear back in our headphones what the mic is recording. For that, we just need to take all the data in the inputBuffer (the recording) and copy it in the outputBuffer (what we'll hear). There is an simple fonction that allows us to do so: memcpy. First you feed it with where you want to copy, then from where you want to copy, and last the size of what you want to copy.

Through the power of memcpy, and with just an extra line in the callback function we can actually hear us singing through our headphones (in our call back).

```java
    memcpy(outputBuffer, inputBuffer, data->bufferFrames * data->channels * sizeof(short));
```

Every audio frame, we basically copy the input buffer to the output one, creating kind of an audio monitoring system.

##Visualization##

Wouldn't it be nice to actually visualize the buffer that comes out? 

For that, let's create a buffer that will hold a frame of audio so we can display it as lines over the screen.
To do so, let's add a display buffer in our Data struct ` short* displayBuffer = NULL;` and then initialize it within the main function. We first need to initialize it with the little sister of the `memcpy` function: `memset`. We feed this fonction with the buffer we want to feed, the value we want to use and the number of time we want it in the buffer:

```java
    data.displayBuffer = new short[data.bufferFrames * data.channels];
    memset(data.displayBuffer, 0, data.bufferFrames * data.channels * sizeof(short));
```

We then need to update the buffer every audio frame (so in the allback). What we want is just to copy the output buffer to this new display buffer 
We'll add the following line at the end of the audio callback:

```java
    memcpy(data->displayBuffer, (short*)outputBuffer, data->bufferFrames * data->channels * sizeof(short));
```

Finally we'll put the actual drawing code at the end of the main loop. We don't care much much about that part. We just scale the values so that it takes our whole screen, then put them as a chain in a VertexArry object, and then we tell the window created by SFML to draw them. The points we add that creates the lines has the value of the signal over the y axis over time as the x axis. If you're curious, ask us, but this is not the main point of the workshop (and yes, this is the last time you can copy paste :p).

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

How cool is that?

