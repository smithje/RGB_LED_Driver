RGB_LED_Driver
==============

This software can be used to drive an analog RGB LED strip using a raspberry pi 
and adafruit's 16-channel 12-bit PWM/Servo Driver - PCA9685:
http://www.adafruit.com/products/815


The Circuit:
------------
Here's the basic idea:

  - Hook up the pi to the PCA9685 breakout board using the I2C connections.  
  - Connect the pi's 3.3V output to VCC on the PCA9685 breakout board.  Leave V+ floating.
  - Follow this tutorial for the RGB LED strips: http://learn.adafruit.com/rgb-led-strips/usage
      - I used the N-channel MOSFETs - three of them, one for each channel
      - Connect the +12V from the LED strip to an external power supply (do NOT use your pi for this!)
      - Connect the ground side of the power supply to the pi ground
      - Instead of using the PWM outputs from the arduino, we'll use the PWM outputs from the PCA9685.
      - Connect up the PWM output 0 to the MOSFET with the red wire from the LED strip.  
           Output 1 goes to green, output 2 goes to blue.

[Here's a picture](http://github.com/smithje/RGB_LED_Driver/blob/master/LED_Strip_bb.png)


Dependencies:
-------------

Adafruit's PWM Servo Driver software (Adafruit_PWM_Servo_Driver) must be in your PYTHONPATH.  That software imports Adafruit_I2C, which should be in the same directory

Usage:
------

This program is designed to be used from the command line.  You should be able to see most options by typing:

```
./RGB_Driver.py --help
usage: RGB_Driver.py [-h] [-r RED RED] [-g GREEN GREEN] [-b BLUE BLUE]
                     [-s STEPS] [-d DELAY] [-o]
                     [--red-pin {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}]
                     [--green-pin {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}]
                     [--blue-pin {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}]
                     [--repeat REPEAT] [--reverse] [--random] [--time TIME]
                     [--max-random-walk MAX_RANDOM_WALK]

drive a rgb led strip through a pwm module

optional arguments:
  -h, --help            show this help message and exit
  -r RED RED, --red RED RED
                        The beginning and end values for red in fade mode. Max
                        and min values for red in random mode.
  -g GREEN GREEN, --green GREEN GREEN
                        The beginning and end values for green in fade mode.
                        Max and min values for green in random mode..
  -b BLUE BLUE, --blue BLUE BLUE
                        The beginning and end values for blue in fade mode.
                        Max and min values for blue in random mode..
  -s STEPS, --steps STEPS
                        Number of steps in the fade. Not used with --random
  -d DELAY, --delay DELAY
                        Number of seconds between the steps or random changes,
                        can be a float
  -o, --turn-off        Turn off when the fade or random event is over.
  --red-pin {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}
                        The red pwm pin
  --green-pin {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}
                        The green pwm pin
  --blue-pin {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15}
                        The blue pwm pin
  --repeat REPEAT       Repeat the fade this many times. Unused with --random.
  --reverse             Reverse the fade, to return back to the initial state.
                        Unused with --random.
  --random              Move around randomly between the max and min values
                        specified with --red --green and --blue. Use with
                        --time and --max-random-walk
  --time TIME           Used with --random. Move randomly for this many
                        seconds
  --max-random-walk MAX_RANDOM_WALK
                        The max that each channel will be allowed to change
                        between steps in random mode.
```
                        

NOTE: You will likely need to run this as root or use sudo because of the I2C interface, unless you jump through a bunch more hoops.  Using sudo can be tricky
because of the PYTHONPATH.  You can do something like this, though:

        sudo PYTHONPATH=$PYTHONPATH:/path/to/Adafruit_PWM_Servo_Driver ./RGB_Driver.py -r 0 4095 --repeat 3 -o
        
Alternatively, you could just copy RGB_Driver.py into the Adafruit_PWM_Servo_Driver directory to save yourself some grief.        


There are two modes: 

1. A fade from one color to another (and, optionally back again with --reverse).  
This can be repeated any number of times using --repeat.  You can change the speed of the fade
by modifying --delay and --steps.

2. Random color changes within bounds and at a rate defined by --delay.  The color changes a maximum of 
--max-random-walk per change.  It will continue to change for approximately --time seconds.

Of course, you can also import the module and use random_walk and fade_rgb directly.



Examples:
---------
1. Fade from off to max red, then turn off:

        sudo ./RGB_Driver.py -r 0 4095 -o


2. Fade from off to max red and back and repeat this 3 times, turning off at the end:

        sudo ./RGB_Driver.py -r 0 4095 -o --repeat 3 --reverse

3. Fade slowly from red to blue:

        sudo ./RGB_Driver.py -r 4095 0 -b 0 4095 -o -d .1 -s 100

4. A random twilightish twinkle for 20s:

        sudo ./RGB_Driver.py -r 500 1000 -g 1024 2048 -b 2048 4095 -s 100 -d 0.1 -o --random --max-random-walk 100 --time 20

5. Fireplace!

        sudo ./RGB_Driver.py -r 2000 4095 -g 0 1024 -b 0 0 -s 100 -d 0.1 -o --random --max-random-walk 100 --time 20

6. Seizure!

        sudo ./RGB_Driver.py -r 0 4095 -g 0 4095 -b 4095 0  --delay 0.01  -o --random --max-random-walk 2000 --time 10
        
[Here's a video showing these examples in order](http://youtu.be/62QUJVkC3B4)


What's Next?
------------

The PWM breakout board has 16 outputs, so, with the right power supply, we could drive up to 5 strips at the same time.

I plan on updating the code to allow for driving multiple strips.
