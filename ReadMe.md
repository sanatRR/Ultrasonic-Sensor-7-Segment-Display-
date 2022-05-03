## **EMBEDDED SYSTEMS LAB PROJECT REPORT**




**Batch: 2**


**Team: 5**




|Sl#|Full Name|Reg#|Roll#|CCE section name|
| :-: | :-: | :-: | :-: | :-: |
|1|SHAH RUSHABH|190953214|51|A (Batch 2)|
|2|ARYAN ROY|190953220|52|A (Batch 2)|
|3|RAORANE SANAT|190953222|53|A (Batch 2)|


**Branch: Computer & Communication** 


**Section: A**


**Date of Submission: 25th April 2022**


Description automatically generated](Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.001.png)


## **Problem Statement**
Write a program to interface Doppler sensor (HC-SR04) to LPC 1768 and display the distance of an object on a 7-Segment Display

## **Required Hardware Components:**
- LPC 1768 MICROCONTROLLER
- HC-SR04 ultrasonic sensor
- ` `DC Power Supply
- ` `FRC cables
- ` `Bread board
- ` `Jumper Cables


## **Working Principle of HC-SR04 Sensor:**
The HC-SR04 Ultrasonic Distance Sensor is a sensor used for detecting the distance to an object using sonar.

It uses non-contact ultrasound sonar to measure the distance to an object and consists of two ultrasonic transmitters (basically speakers), a receiver, and a control circuit.

The transmitters emit a high frequency ultrasonic sound, which bounce off any nearby solid objects, and the receiver listens for any return echo, which is then processed by the control

circuit to calculate the time difference between the signal being transmitted and received.

This time can subsequently be used, along with some clever math, to calculate the distance between the sensor and the reflecting object.


## **HC-SR04 Interfacing Steps:**
- To start distance measurement a short pulse of 10us is applied to Trigger pin.
- After receiving trigger pulse, the HC-SR04 Module sends a burst of 8 ultrasonic pulses at 40Khz.
- It will then output a HIGH for the amount to time taken for the sound waves to reach back.
- The pulse HIGH time is measured and then used to calculate the distance.


**Pinout:** The module has got 4 pins. VCC(+5V), TRIG, ECHO, GND.


![Graphical user interface

Description automatically generated](Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.002.png)

## **Calculations:**
![](images/Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.003.png)Speed of sound in air,
### ***Vs = 343 m/s = 0.0343 cm/us***

Distance Travelled = Speed x Time taken
### ***D’ = 343 m/s x T s = 0.0343 cm/us x T us***

Since the ultrasonic waves travel as an echo back and forth, the total distance travelled is ascertained as

**D = D’/2 = (0.0343 x T)/2 = 0.01715 x T**


![Diagram

Description automatically generated](images/Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.004.jpeg)

# **Circuit diagram:**
![Diagram

Description automatically generated](images/Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.005.jpeg)

**Source Code (.c file):**


#include <lpc17xx.h>

#include <stdlib.h>

#define PRESCALE (25-1)

char sevenseg[] = {0x3f,0x6,0x5b,0x4f,0x66,0x6d,0x7d,0x7,0x7f,0x6f};

int round\_val(float num){

`        `float val = num - abs(num);

`        `if(val <= 0.5)

`            `return (int)num;

`        `else

`            `return (int)num + 1;

}



void initTimer0(void);

void startTimer0(void);

unsigned int stopTimer0(void);

void delayUS(unsigned int microseconds);

void delayMS(unsigned int milliseconds);


void initTimer0(void) //PCLK must be = 25Mhz!

{

`    `LPC\_TIM0->CTCR = 0x0;

`    `LPC\_TIM0->PR = PRESCALE; //Increment TC at every 24999+1 clock cycles

`    `LPC\_TIM0->TCR = 0x02; //Reset Timer

}

void startTimer0(void)

{

`    `LPC\_TIM0->TCR = 0x02; //Reset Timer

`    `LPC\_TIM0->TCR = 0x01; //Enable timer

}





unsigned int stopTimer0(void)

{

`    `LPC\_TIM0->TCR = 0x00; //Disable timer

`    `return LPC\_TIM0->TC;

}


void delayUS(unsigned int microseconds) //Using Timer0

{

`    `LPC\_TIM0->TCR = 0x02; //Reset Timer

`    `LPC\_TIM0->TCR = 0x01; //Enable timer

`    `while(LPC\_TIM0->TC < microseconds); //wait until timer counter reaches the desired delay

`    `LPC\_TIM0->TCR = 0x00; //Disable timer

}

void delayMS(unsigned int milliseconds) //Using Timer0

{

`    `delayUS(milliseconds \* 1000);

}

void delay\_trigger(){

`    `LPC\_GPIO0->FIOSET = (0x1<<15);

`    `delayUS(10);

`    `LPC\_GPIO0->FIOCLR = (0x1<<15);

}

int echo\_monitor(){

`    `float pulse\_time = 0,distance=0;

`    `while((LPC\_GPIO0->FIOPIN & (0x1<<16)) == 0x0);  //Wait till echo is low

`    `startTimer0();                                                          //Initialize the echo timer

`    `while((LPC\_GPIO0->FIOPIN & (0x1<<16)) == 0x1<<16);  //Wait till echo is high

`    `pulse\_time = stopTimer0();                                      //Get count of echo timer

`  `distance = (0.0343\*pulse\_time)/2;

`    `return round\_val(distance);

}




void display(int number,int displayTimeSeconds){

`    `int j,i,timeUS = 1000000\*displayTimeSeconds,temp;

`    `startTimer0();

`    `while(LPC\_TIM0->TC <timeUS){

`        `temp = number;

`        `for(j=0;j<4;j++){

`                `LPC\_GPIO1->FIOPIN = j<<23;

`                `LPC\_GPIO0->FIOPIN = sevenseg[(temp%10)]<<4;

`              `for(i=0;i<1000;i++);

`                `temp /= 10;

`            `}

`    `}

`    `stopTimer0();

`    `LPC\_GPIO1->FIOPIN = 00<<23;

`    `LPC\_GPIO0->FIOPIN = 0xF9<<4; //Display E.

}

int main(void){

`    `int distance;

`    `SystemInit(); //CLK = 12 MHz and PCLK = 25MHz

`    `SystemCoreClockUpdate();





`    `//Trigger, Echo, 7 segment

`    `LPC\_PINCON->PINSEL0 = 0x0; //SET TO GPIO

`    `LPC\_PINCON->PINSEL1 = 0x0;  //SET TO GPIO (For the echo pin) 

`    `LPC\_GPIO0->FIODIR = 0x0FFF0;



`    `//Decoder

`    `LPC\_PINCON->PINSEL3 = 0x0;      //Decoder   GPIO config 

`    `LPC\_GPIO1->FIODIR = 0xF<<23;    //Decoder output config



`    `//Timer setup

`    `initTimer0();



`    `while(1){

`        `delay\_trigger();

`        `distance = echo\_monitor();

`        `display(distance,1);

`    `}

}





`            `![](images/Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.006.png)




![](images/Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.007.png)




![](images/Aspose.Words.ea0014dc-e67a-4fdb-8e36-19a396a75626.008.png)


#
# **Result:**
We have successfully built and developed an efficient and accurate ultrasonic Distance Detector using the HC- SR04 ultrasonic sensor, interfaced with LPC 1768 Microcontroller, which is able to detect and calculate the exact distance between the device and the object in front of it up to a distance of approximately 400 cm, and have displayed the calculated distance on 4 multiplexed Seven-Segment Displays connected to the System.
##
