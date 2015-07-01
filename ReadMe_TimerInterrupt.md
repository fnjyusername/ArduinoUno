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
 34       |   0  |  1  |  1  |                         | 
 256      |   1  |  0  |  0  |                         |  
 1024     |   1  |  0  |  1  |                         | 

 TIMER2    

Prescale  | CS22 |CS21 |CS20 |REMARKS                  |         
----------|------|-----|-----|-------------------------|
 0        |   0  |  0  |  0  |  Timer Counter Stop     |     
 1        |   0  |  0  |  1  |  No Prescale (Fastest)  | 
 8        |   0  |  1  |  0  |                         |
 34       |   0  |  1  |  1  |                         | 
 256      |   1  |  0  |  0  |                         |  
 1024     |   1  |  0  |  1  |                         |
 

```
CLOCK SELECT LEGEND: CSxy
CS is Clock Source
x  is Timer number i.e. 2 for Timer2, 1 for Timer1, 0 for Timer0 
y  is Bit
```
  
 Example:
 


 
