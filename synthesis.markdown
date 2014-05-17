---
title: Synthesis It
layout: default
num: 4 
---


Now it's getting serious, ex nihilo in your face. It's not anymore about smashing back some pre made sick tunes, it's about creating them	
	
##Generating a simple tone##

Generating (as opposed to sampling) means that you create something out of nothing, you don't copy past. One of the most basic signals is the sine wave. Lucky you, that's where we'll start.

The sine wave is indeed based on the sinus so it'll be `sin(...)`. The signal evolve over time, so it should be `sin(t)`. Time here is discretise, and even more, it's linked with the number of value in the buffer. We'll have to loop over it, and the value will evolve depending on it `sin(i)`. But we'll need to precise the frequency of the wave. Some maths are involved here. First, we need to divide by the sample rate, and then multiple by the right requence we're aiming for. Last, since we're dealing in fact with angles, we need to multiple that value that would be in the `[0,1]` range by `2*Pi` so that it'll be i the `[0, 2*Pi]` range. So all in all:

```java
    float freq = 440;
    float val = 1.0*sin(2.0* M_PI / (data->sampleRate / freq)  * i);
```

Now that we have the right value, we need to put that in the buffer. The buffer is twice the number of buffer frames because it's stereo. Since we want the same value on both channels, and it's formating as L R L R ..., we'll just have to repeat twice the value.

```java
for (int i=0; i < nBufferFrames; i++)
{
    float freq = 440;
    float val = 1.0*sin(2.0* M_PI / (data->sampleRate / freq)  * i);
    ((short*)outputBuffer)[2*i+0] = CLIP( val * 32767, -32767, 32767);
    ((short*)outputBuffer)[2*i+1] = ((short*)outputBuffer)[2*i+0]; 
}
```

Remark:  
Don't forget to put that loop in your call back, not in your main thread.

Perfect! Or is it? Try it out and ... for those that are used to sound, something should be off. You can't hear the perfection of the sin wave that defines it. That's because we're always initialing back time at zero at the start of the buffer. We should have a global time variable! Let's put one in our data structure and initialise it at 0: `int iS = 0;`.

Now let's use it in our loop.

```java
for (int i=0; i < nBufferFrames; i++)
{
    data->iS++;
    float freq = 440;
    float val = 1.0*sin(2.0* M_PI / (data->sampleRate / freq)  * _iS);
    ((short*)outputBuffer)[2*i+0] = CLIP( val * 32767, -32767, 32767);
    ((short*)outputBuffer)[2*i+1] = ((short*)outputBuffer)[2*i+0]; 
}
```

Once we better structure what a synth and a note is, we'll trigger that by keyboard

We add iS to data, as the global time reference. We need global time reference because we need continuity over the buffers.
Sound is periodic signal, based on a frequency that defines the pitch.

##Form of the tone##

Infinities of various formes are possible for synthesis, it's fun to play with different generations. The purest is the sinusoide, then among the big classics you have the saw, the triangle, the white noise and the favorite of cheap tune user: the square.

```java
//Triangle
    if(_iS%ratio < 0.5 * ratio)
        valNote += -1 + 4.0/ratio*(_iS%ratio);
    else
		valNote +=  1 - 4.0/ratio*(_iS%ratio -ratio/2);
	break;

case SIN:
	valNote += 1.0*sin(2.0* M_PI / ratio * _iS);
	break;
case SAW:
	valNote +=  1 - 2.0/ratio*(_iS%ratio);
	break;

case SQUARE:
	if(_iS%ratio > 0.5 * ratio)
		valNote +=  1;
	else
		valNote += -1;
	break;

case WHITE_NOISE:
		valNote += (float)rand() / (RAND_MAX) * 2 -1;
			break;
```

Not only can you create varying signals by default, but you can always combine them. You can for instance add them:

```java
valNote += 1.0*sin(2.0* M_PI / ratio * _iS) +  1 - 2.0/ratio*(_iS%ratio);
```

##Multiple tones##
We need to have possibily multiple tones by synthetiser. For that we need a dynamic recording the notes we have, and a structure for those notes.

```java
...
```

##Envelope of sounde##

