---
layout: default
num: 5
title: "Attack Decay Sustain Release It!"
---

##Envelope of sound##
You're already there? Holly molly you were fast! We're still ironing out the part of this workshop (read: it was too sunny outside to stay inside...). Some code will come up soon, and explanations will shortly follow.

```java
struct note {
	
	enum { ATTACK, DECAY, SUSTAIN, RELEASE, SLEEP} state = ATTACK; 
	float freq;
	int i_start;
	// Attack Decay Sustain Release
	float t_a = 1.2;
	float t_d = 0.3;
	float a_s = 0.7;
	float t_r = 0.1;
	
	note(float _freq, int _i) { freq = _freq; i_start =_i; }

	//Attack Decay Sustain Release
	float ADSR( int _i) {

	    float tps = 1.0/4410;
	    switch(state) {
		case ATTACK:
//		    std::cout << "A " << (_i - i_start)*tps << " " << (_i - i_start)*tps/t_a<< std::endl;
		    if( (_i - i_start)*tps > t_a) {
			i_start = _i;
			state=DECAY;
		    }
		    return (_i - i_start)*tps/t_a;
		    break;

		case DECAY:
//		    std::cout << "D " << (_i - i_start)*tps << std::endl;
		    if( (_i - i_start)*tps > t_d) {
			i_start = _i;
			state=SUSTAIN;
		    }
		    return 1-(1-a_s)*(_i - i_start)*tps/t_d;
		    break;

		case SUSTAIN:
//		    std::cout << "S " << std::endl;
		    return a_s;
		    break;

		case RELEASE:
//		    std::cout << "R " << (_i - i_start)*tps << std::endl;
		    if( (_i - i_start)*tps > t_r)
			state=SLEEP;
		    return a_s*(1-(_i - i_start)*tps/t_r);
		    break;

	    }

	    return 1;
	}

};
```

And add in the callback:

