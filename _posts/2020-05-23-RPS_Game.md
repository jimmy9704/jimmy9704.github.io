---
layout: post
title:  "[C][code vision AVR] RPS GAME by using ATmega128"
date:   2020-05-23
desc: "Quick test on writing code snippets in a blog post"
keywords: "Code Vision AVR"
categories: [C]
tags: [C]
icon: icon-html
---


# RPS GAME by using ATmega128


<iframe width="949" height="585" src="https://www.youtube.com/embed/7wcCp_b7-EE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


ATmega128과 장갑을 끼고 가위바위보 게임을 하는 프로그램이다


외부 인터럽트를 사용하였으며


ADC입력을 사용하였다



```c
#include <mega128.h>
#include <io.h>
#include <delay.h>
#include <stdio.h>
#include <stdlib.h>

#define ADC_VREF_TYPE ((0<<REFS1) | (0<<REFS0) | (0<<ADLAR))

int my_data = 0;
int robot_data = 0;
```
헤더들을 추가하고 장갑으로 인식한 값을 저장하기 위한 변수와 로봇의 값을 저정하기위한 변수를 전역변수로 설정한다



```c
// Read the AD conversion result
unsigned int read_adc(unsigned char adc_input)
{
    ADMUX=adc_input | ADC_VREF_TYPE;
    // Delay needed for the stabilization of the ADC input voltage
    delay_us(10);
    // Start the AD conversion
    ADCSRA|=(1<<ADSC);
    // Wait for the AD conversion to complete
    while ((ADCSRA & (1<<ADIF))==0);
    ADCSRA|=(1<<ADIF);
    return ADCW;
}
```
read_adc함수는 PORTF에 연결된 플렉스 센서의 값을 받아온다


adc_input은 ADMUX레지스터의 MUX비트들을 설정해준다



`ADCSRA|=(1<<ADSC);`


Single Conversion 모드 에서는 ADCSRA의 ADSC비트가 1로 설정되면 ADC변환이 시작한다.


`while ((ADCSRA & (1<<ADIF))==0);`

변환이 완료되어 데이터 레지스터가 업데이트 되면 ADIF비트는 자동으로 1로 설정된다


```c
void set_my_data(){
    int adc_data0 = read_adc(0);
    int adc_data1 = read_adc(1);

    if (adc_data0<300 && adc_data1<300)
        my_data = 0;
    if (adc_data0<300 && adc_data1>=300)
        my_data = 1;
    if (adc_data0>=300 && adc_data1>=300)
        my_data = 2;
}
```
장갑에 부착되어 있는 플렉스센서의 값에따라 가위 바위 보를 분류해준다


```c
interrupt [EXT_INT0] void ext_int0_isr(void)
{
    srand(read_adc(0));
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");

    printf("$G, 1, 1\r");
    printf("$T, 3\r");
    delay_ms(1000);
    printf("$G, 1, 1\r");
    printf("$T, 2\r");
    delay_ms(1000);
    printf("$G, 1, 1\r");
    printf("$T, 1\r");
    delay_ms(1000);

    robot_data = rand()%4;
    set_my_data();

    printf("$C\r");;    
    printf("$G,; 1, 1\r");
    if (my_data == 0)
        printf("$T, You: Rock\r");
    else if (my_data == 1)
        printf("$T, You: Scissors\r");
    else if (my_data == 2)
        printf("$T, You: Paper\r");

    printf("$G,; 2, 1\r");
    delay_us(10);
    if (robot_data == 0)
        printf("$T, Robot: Rock\r");
    else if (robot_data == 1)
        printf("$T, Robot: Scissors\r");
    else if (robot_data == 2)
        printf("$T, Robot: Paper\r");
    delay_ms(2000);

    printf("$C\r");
    if (my_data == robot_data){
        printf("$G, 1, 1\r");
        printf("$T, draw\r");
    }
    else if( my_data == 0){
        if(robot_data == 1){
            printf("$G, 1, 1\r");
            printf("$T, you win\r");
        }
        else{
            printf("$G, 1, 1\r");
            printf("$T, you lose\r");    
        }    
    }
    else if( my_data == 1){
        if(robot_data == 2){
            printf("$G, 1, 1\r");
            printf("$T, you win\r");
        }
        else{
            printf("$G, 1, 1\r");
            printf("$T, you lose\r");    
        }    
    }
    else if( my_data == 2){
        if(robot_data == 0){
            printf("$G, 1, 1\r");
            printf("$T, you win\r");
        }
        else{
            printf("$G, 1, 1\r");
            printf("$T, you lose\r");    
        }    
    }      



    delay_ms(3000);
    printf("$C\r");
}
```


PORTA.0에 연결되어있는 Pull down 저항이 rising 될때 함수가 실행되며 가위바위보 게임이 시작된다


ATmega128의 경우 time을 srand의 seed로 받을 수 없어서 버튼을 누를때의 플렉스센서 값을 seed로 받아


매번 랜덤하게 robot_data를 설정해주었다.



```c
void main(void)  
{
    unsigned int adc_data = 0;

    EICRA=(0<<ISC31) | (0<<ISC30) | (0<<ISC21) | (0<<ISC20) | (0<<ISC11) | (0<<ISC10) | (1<<ISC01) | (1<<ISC00);
    EICRB=(0<<ISC71) | (0<<ISC70) | (0<<ISC61) | (0<<ISC60) | (0<<ISC51) | (0<<ISC50) | (0<<ISC41) | (0<<ISC40);
    EIMSK=(0<<INT7) | (0<<INT6) | (0<<INT5) | (0<<INT4) | (0<<INT3) | (0<<INT2) | (0<<INT1) | (1<<INT0);
    EIFR=(0<<INTF7) | (0<<INTF6) | (0<<INTF5) | (0<<INTF4) | (0<<INTF3) | (0<<INTF2) | (0<<INTF1) | (1<<INTF0);

    // Globally enable interrupts
    #asm("sei")
```


외부인터럽트를 사용하기 위해 메모리를 설정해주었다.


PORTA.0번핀을 제외한 A의 핀은 OFF로 설정하였으며 0번핀은 Rising Edge애서 동작하도록 하였다.


```c
// Function: Bit7=Out Bit6=Out Bit5=Out Bit4=Out Bit3=Out Bit2=Out Bit1=Out Bit0=In
DDRF=(0<<DDF7) | (0<<DDF6) | (0<<DDF5) | (0<<DDF4) | (0<<DDF3) | (0<<DDF2) | (0<<DDF1) | (0<<DDF0);
// State: Bit7=0 Bit6=0 Bit5=0 Bit4=0 Bit3=0 Bit2=0 Bit1=0 Bit0=T
PORTF=(0<<PORTF7) | (0<<PORTF6) | (0<<PORTF5) | (0<<PORTF4) | (0<<PORTF3) | (0<<PORTF2) | (0<<PORTF1) | (0<<PORTF0);      
```


플렉스 센서가 연결된 PORTF의 모드를 설정해준다


```c
ADMUX=ADC_VREF_TYPE;
ADCSRA=(1<<ADEN) | (0<<ADSC) | (0<<ADFR) | (0<<ADIF) | (0<<ADIE) | (0<<ADPS2) | (1<<ADPS1) | (1<<ADPS0);
SFIOR=(0<<ACME);


UCSR0A=(0<<RXC0) | (0<<TXC0) | (0<<UDRE0) | (0<<FE0) | (0<<DOR0) | (0<<UPE0) | (0<<U2X0) | (0<<MPCM0);
UCSR0B=(0<<RXCIE0) | (0<<TXCIE0) | (0<<UDRIE0) | (0<<RXEN0) | (1<<TXEN0) | (0<<UCSZ02) | (0<<RXB80) | (0<<TXB80);
UCSR0C=(0<<UMSEL0) | (0<<UPM01) | (0<<UPM00) | (0<<USBS0) | (1<<UCSZ01) | (1<<UCSZ00) | (0<<UCPOL0);
UBRR0H=0x00;
UBRR0L=0x67;      

delay_ms(300);
printf("$I\r");
printf("$C\r");
printf("$B,0\r");
```


ADC를 사용하기 위한 메모리들을 초기화 해주고 LCD를 키고 Clear해준다


```c
while (1)
    {   
        printf("$G, 1, 1\r");
        printf("$T, Push Button to start\r");
    }
}
```


while문을 지속적으로 실행하며 버튼에의한 인터럽트가 실행되면 게임이 시작된다.
