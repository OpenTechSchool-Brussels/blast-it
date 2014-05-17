---
title: Set It Up
layout: default
num: 1
---

Let's get this party started.

##So many tools in the wilderness##
Yes, so many, and so little time... Computers are particularly happy with music because both speaks the same language: signal. As such, computers are good at generating, recording and manipulating sound and hence music. This is partly why they are wildly used in music creation nowadays, would that be as a studio (recording, mastering…), to perform and launch samples, or straight as a music instrument. If many softwares exist to abstract these tasks, through this workshop we want you to get a low-level and deep understanding of the element at play: sound, signal, waves, frequencies, timbre... and all other crazy stuff.

Through this workshop, you’ll be coding in C++ with SFML for interaction, and RTaudio for the sound part. For those that are interested, you will have the possibility to create a VST plugin (Yeah, right... not gonna happen in this itteration of the worksop...), a module that you can plug in music application such as Ableton Live, GarageBand & others.

Let's first setup our battle station.

##Setup##

Lucky Molly, Rtaudio is a breeze to install and use. SFML, depending on your system, not so much. Setting up instructions are a tad of a mess or now, but we're here to help you around if you need help.

####RtAudio####
Let's start with the audio part: 
* first download it: [RtAudio files](https://www.music.mcgill.ca/~gary/rtaudio/) (one .h and one .cpp file) 
* then here is how to use it: [Link with RtAudio](https://www.music.mcgill.ca/~gary/rtaudio/compiling.html)

If you're using windows Visual Studio, you just need to add the header files and don't forget to add a link to dsound.lib.

####SFML####
And now for the visual part: 
* Installation of SFML : [SFML library](http://www.sfml-dev.org/tutorials/2.1/#getting-started) 
* If needed, [Download Page](http://www.sfml-dev.org/download/sfml/2.1/)

##Base Code##

Ok, you're super prowd of yourself because you have your full stack ready to rock but ... does it work? Let's have a base code to be sure that everything is in order. We said no copy pasting but this time and just this time (and actually another one too) we'll look elsewhere.

What we need to check everything is first to instantiate an RtAudio object, see if it recognise audio devices. Second, we will create a graphic windows in SFML and a loop so that our program doesn't stop right ahead. Inside the loop we will test for user input (this is where you'll log key pressed event for instance).

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

Remark:  
In linux, the line you need to compile is:
```bash
g++ -Wall -D__LINUX_ALSA__ -o main main.cpp ./rtaudio/RtAudio.cpp -lasound -lpthread -lsfml-graphics -lsfml-window -lsfml-system
```

It's not super important to get the in & out of this code, just have a global grasp over it. If you're more curious about this aspect of the workshop, be sure to let us know.

##Digital Audio##

Now we're on the right track :D

###Sound###

Sound is basically physical vibrations, waves travelling in the air. It can be modelised as an analogue signal and hence be computer understandable. An analogue signal is just a value that change over time, you'll have many occasions to see them during the workshop and you'll learn to recognise and love them (but not in public please). This value changing over time defines two important parameters. First you have the amplitude (how high is that value) and second you have its evolution over time.

Alas, computers understand signals, but not analogue ones. They are cold hearted machines (as much as your next human) that only process discreet values. For that they need to convert this analogue signal to a digital one. Once done, it can read, record & manipulate it. When it needs to output it (headphones, speakers...) it then converts the digital back to analogue. Instead of having a smooth curve for the signal, we'll have some very dense histograms (discrete values). But how discrete do we want them?

You can imagine that the denser the histogram, the closer it is to the curve. The frequency of sampling this curve is called the sample rate (a classic is 44100Hz) and is one of the two parameters that defines the quality of a sample, how true it is to its analogue older brother. This sampling deals with time and frequency. If you remember well, our variable defining the analogue signal was also defined by another parameter: the amplitude. In this case too, we can have a more or less precise definition of the amplitude given by the sample format (ex: 8, 16 or 24 bit). You will get to use and play with both parameters along the line.

Last, we have two ears. So we don't listen in a monophonic way, we actually can locate where sound is coming from. This is why computers often don't output to one speaker, but to multiple ones (from mono to stereo, or even 5.1). This number of outputs is called the number of channel (in stereo, 2);

Software-wise, the behaviour is the same whatever the platform or hardware (from a micro-controller to a computer or a smartphone): a callback is called whenever the hardware needs data in or out. The frequency of the callback depends on the way you configure the audio stack for your program, especially the callback buffer size. The smaller the audio buffer the lower the latency. In that regard, all platforms aren’t equal, so this workshop we’ll work with 512 samples sized buffers, but don’t hesitate to experiment with different values and experiment the limits of what your system can do.

Remark: on Windows, ASIO drivers usually offer way better latency than standard DirectSound ones in case your audio hardware is compatible. RtAudio, the audio framework we’ll use in this workshop both support DirectSound and ASIO.

###Audio Buffer###

An audio sample is a set of values that correspond to the different audio channels.
In our configuration an audio channel length is 16bit x 2 channels for a total of 32bit / sample.

We chose Interleaved format so Left and Right channels are arranged successively in the audio buffer:
[ interleaved illustration: L R L R L R L R etc ]
In order to easily access the individual channels we’ll represent audio buffers as `short* buffer;` in the code.


