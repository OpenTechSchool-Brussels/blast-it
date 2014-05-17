---
title: Synthesis
layout: default
num: 4 
---


Now it's getting serious, ex nihilo in your face. It's not anymore about smashing back some pre made sick tunes, it's about creating them	
	
##Generating a simple tone##
We add iS to data, as the global time reference. We need global time reference because we need continuity over the buffers.
Sound is periodic signal, based on a frequency that defines the pitch.


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

##Form of the tone##

Infinities of various formes for synthesis, based on varying timbers. Purest is the sinusoide, then among the big classics you have the saw, the triangle, the white noise and the favorite of cheap tune user: the square.

```java
case TRI:
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

