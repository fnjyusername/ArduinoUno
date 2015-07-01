# ArduinoUno

### Timer Interrupt CTC (Clear Timer on Compare )

Arduino timer interrupts allow you to momentarily pause events taking place in the loop() function at precisely timed intervals, while you execute a separate set of commands.  Once these commands are done the Arduino picks up again where it was in the loop(). It is very important the timing the loop() function time and the time interrupts will take place.

 Interrupts are useful for:
 1. Measuring an incoming signal at equally spaced intervals (constant sampling frequency)
 2. Calculating the time between two events
 
Arduino Uno Clock Speed:
The Arduino clock runs at 16MHz, this is the fastest speed that the timers can increment their counters.
So at  16MHz each tick of the counter represents 1/16,000,000 of a second (~63ns) so a counter will take 10/16,000,000 seconds to reach a value of 9 (counters are 0 indexed), and 100/16,000,000 seconds to reach a value of 99.

The Uno has three timers called timer0 (8bit and 255 Counts), timer1(16bit and 65535 counts), and timer2(8bit and 255 Counts). 

###### Maximun time of Interrupt  = (Count+1)/ClockSpeed

Timer  | Bit |  Count |  Maximum Interrupt |    
-------|-----|--------|--------------------|    
 0     |  8  |  255   |   256/16000000=16us|    
 1     | 16  | 65535  | 65536/16000000= 4ms|
 2     |  8  |  255   |   256/16000000=16us| 
 
From this table it is clearly that we can interrupt 16us for 8bit timer and 4ms for 16bit timer. Meaning you may not catch signal occuring say at 1sec interval.


### PRESCALER
We can control the speed of the timer counter incrementation using a prescaler.  A prescaler adjust the speed of timer with following equation: speed (Hz) = (Arduino clock speed (16MHz)) / prescaler

CLOCK SELECT BIT

TIMER0    

Prescale  | CS02 |CS01 |CS00 |REMARKS                  |         
----------|------|-----|-----|-------------------------|
 0        |   0  |  0  |  0  |  Timer Counter Stop     |     
 1        |   0  |  0  |  1  |  No Prescale (Fastest)  | 
 8        |   0  |  1  |  0  |                         |
 34       |   0  |  1  |  1  |                         | 
 256      |   1  |  0  |  0  |                         |  
 1024     |   1  |  0  |  1  |                         | 
 
 TIMER1    

Prescale  | CS12 |CS11 |CS10 |REMARKS                  |         
----------|------|-----|-----|-------------------------|
 0        |   0  |  0  |  0  |  Timer Counter Stop     |     
 1        |   0  |  0  |  1  |  No Prescale (Fastest)  | 
 8        |   0  |  1  |  0  |                         |
 64       |   0  |  1  |  1  |                         | 
 256      |   1  |  0  |  0  |                         |  
 1024     |   1  |  0  |  1  |                         | 

 TIMER2    

Prescale  | CS22 |CS21 |CS20 |REMARKS                  |         
----------|------|-----|-----|-------------------------|
 0        |   0  |  0  |  0  |  Timer Counter Stop     |     
 1        |   0  |  0  |  1  |  No Prescale (Fastest)  | 
 8        |   0  |  1  |  0  |                         |
 64       |   0  |  1  |  1  |                         | 
 256      |   1  |  0  |  0  |                         |  
 1024     |   1  |  0  |  1  |                         |
 

```
CLOCK SELECT LEGEND: CSxy
CS is Clock Source
x  is Timer number i.e. 2 for Timer2, 1 for Timer1, 0 for Timer0 
y  is Bit
```


##### FORMULA 1: Now we can calculate the interrupt frequency with the following equation:
`Interrupt Freq (Hz) = (Arduino clock speed 16,000,000Hz) / (prescaler * (compare match register + 1))`

##### FORMULA 2: We can also solve for the compare match register value that will give your desired interrupt frequency:
`compare match register = [ 16,000,000Hz/ (prescaler * desired interrupt frequency) ] - 1`


###### Example 1: Capture an Interrupt @ 2500us/400Hz using 1024 prescale

`compare match register = [ 16,000,000Hz/ (prescaler(1024) * desired interrupt frequency(400)) ] - 1   = 38`

38 < 255   Ok for 8bit Timer0/Timer2  
38 < 65536 Ok for 16bit Timer1

Because compare match register = 38 is less than maximum count 255 (Timer0/8bit) and less than 65536 counts (Timer1/16bit),
the frequencies can be handled by 8bit Timer0/Timer2 or 16bit Timer1. Now lets calculate the frequency for a check.

`Interrupt Frequency (Hz) = (clockspeed 16,000,000Hz) / (presclale(1024) * (compare match register + 1)) = 400Hz`

###### Example 2: Capture an Interrupt @ 2500us/400Hz using 8 prescale

`compare match register = [ 16,000,000Hz/ (prescaler(8) * desired interrupt frequency(400)) ] - 1   = 4999`

4999 > 255   Not Ok for 8bit Timer0/Timer2  
4999 < 65536 Ok for 16bit Timer1

Because compare match register = 4999 is greater than 255 of 8bit, Timer0 or Timer2 cannot handle the frequency on 8 prescale, but we can use 16bit Timer1 at prescale of 8.

Now lets calculate the frequency for a check.

`Interrupt Freq (Hz) = (clockspeed 16,000,000Hz) / (presclale (8) * (compare match register(4999) + 1)) = 400Hz`
 
### Using the ATmega PWM registers directly 
The ATmega328P has three timers known as Timer 0, Timer 1, and Timer 2. Each timer has two output compare registers e.g. OC0A and OC0B for Timer0 (see TABLE 1.0) that control the PWM width for the timer's two outputs: when the timer reaches the compare register value, the corresponding output is toggled. 

The two outputs for each timer will normally have the same frequency, but can have different duty cycles (depending on the respective output compare register). 

##### TABLE 1.0 TIMER OUTPUT PIN ASSIGNMENT OCRnx, Timer(n), Output channel (x)

Timer  | output	|Arduino| Chip | Port Pin 
-------|--------|-------|------|---------
Timer0 |  OCR0A	 |  6	   | 12	  |  PD6
Timer0 |  OCR0B	 |  5	   | 11	  |  PD5
Timer1 |  OCR1A	 |  9	   | 15	  |  PB1
Timer1 |  OCR1B	 |  10	  | 16	  |  PB2
Timer2 |  OCR2A	 |  11	  | 17	  |  PB3
Timer2 |  OCR2B	 |  3	   |  5  	|  PD3

### Default to Arduino
The Arduino performs some initialization of the timers. 
The Arduino initializes the prescaler on all three timers to divide the clock by 64. 
Timer 0 is initialized to Fast PWM, while Timer 1 and Timer 2 is initialized to Phase Correct PWM. 
See the Arduino source file wiring.c for details. 

######TABLE 2.0 DEFAULT SETTINGS

Timer  | Prescale	|     PWM Mode   |  Default Utilization |
-------|--------  |----------------|----------------------|
Timer0 |    64  	 |  FAST	PWM      | millis() and delay()	|
Timer1 |    64  	 |  PHASE CORRECT | Tone lib and  micros |  
Timer2 |    64  	 |  PHASE CORRECT |                   	  |

#####Note: Using the PWM outputs is safe if you don't change the frequency

The Arduino uses Timer 0 internally for the millis() and delay() functions, so be warned that changing the frequency of this timer will cause those functions to be erroneous. 


## BRIEF START ON PWM
Below is our first illustration of creating PWM output at Pin3 (OCR2A) and Pin 11 (OCR2B), we recall these pin is were the timer2 output channel A and B is routed. We need to first set this pin as OUTPUT with pinMode.
```c
Example using Default setting of Timer2 i.e. prescaled to 64

void Setup():
  pinMode(3, OUTPUT);
  pinMode(11, OUTPUT);
  TCCR2A = _BV(COM2A1) | _BV(COM2B1) | _BV(WGM20);
  TCCR2B = _BV(CS22);

void loop():
  OCR2A = 125;   //49.0% Duty Cycle, OCR2A duty cycle range 1 to 255 Output to Pin 3
  OCR2B =  20;   // 7.8% Duty Cycle, OCR2B duty cycle range 1 to 255 Output to Pin 11 
```


Timers are usually used in one of the following modes: Next is how to  set Timer Register in order to configure the mode i.e. NORMAL, Phase correct PWM, FAST PWM, CTC, (Recall Arduino is initialized at default mode on TABLE 2.0 above)

## Methodology – PWM Mode

#### TIMER REGISTER

REGISTER  |   7 bit  |   6 bit  |   5 bit  |   4 bit  |   3 bit  |  2 bit   |  1 bit   |  0 bit   |
----------|----------|----------|----------|----------|----------|----------|----------|----------|
 TCCR0A   |  COM0A1  |  COM0A0  |  COM0B1  |  COM0B0  |     /    |     /    |   WGM01  |   WGM00  |
 TCCR1A   |  COM1A1  |  COM1A0  |  COM1B1  |  COM1B0  |     /    |     /    |   WGM11  |   WGM10  |
 TCCR2A   |  COM2A1  |  COM2A0  |  COM2B1  |  COM2B0  |     /    |     /    |   WGM21  |   WGM20  |

#### Waveform Generation Mode –  
 
MODE |   WGM01  |   WGM00  | Mode of Opreation   |    TOP    |Update of OCR|  TOVx Flag | 
-----|----------|----------|---------------------|-----------|-------------|------------|
 0   |     0    |    0     |        NORMAL       |   0XFF    |  Immidiate  |     MAX    |
 1   |     0    |    1     |  PWM, PHASE CORRECT |   0XFF    |     TOP     |     BOT    |
 2   |     1    |    0     |         CTC         |   OCRX    |  Immidiate  |     MAX    |
 3   |     1    |    1     |       FAST PWM      |   0XFF    |     TOP     |     MAX    |
 
 
* http://www.fiz-ix.com/2012/01/how-to-configure-arduino-timer-2-registers-to-drive-an-ultrasonic-transducer-with-a-square-wave/
* http://maxembedded.com/2011/08/avr-timers-pwm-mode-part-i/
* http://www.righto.com/2009/07/secrets-of-arduino-pwm.html
* https://www.youtube.com/watch?v=9JXGIeM3BSI
