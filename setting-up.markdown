---
title: Set It Up
layout: default
num: 1
---

##Setup##

Installation of the [SFML library](http://www.sfml-dev.org/download/sfml/2.1/)

Create a project [linking with SFML](http://www.sfml-dev.org/tutorials/2.1/)

Download [RtAudio files](https://www.music.mcgill.ca/~gary/rtaudio/) (one .h and one .cpp file)

[Link with RtAudio](https://www.music.mcgill.ca/~gary/rtaudio/compiling.html)

##Base Code##

Here's the base code you can copy and paste in main.cpp, compile and run to make sure both RtAudio and SFML run fine on your system.
This code first makes sure RtAudio is functionnal, then creates a window and runs an event loop using SFML.

```java
#include <iostream>
#include <cstdlib>
#include <math.h>

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
