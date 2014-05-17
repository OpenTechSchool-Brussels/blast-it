---
title: Setting Up
layout: default
num: 1
---

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