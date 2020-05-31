---
layout: post
title:  "[C][code vision AVR] DigitalClock"
date:   2020-05-23
desc: "Quick test on writing code snippets in a blog post"
keywords: "DigitalClock"
categories: [C]
tags: [C]
icon: icon-html
---


# DigitalClock by using ATmega128


<iframe width="949" height="535" src="https://www.youtube.com/embed/wJ6wRbBmAS8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


ATmega128을 이용하여 만든 디지털 시계이다



```c
#include <mega128.h>
#include <stdio.h>
#include <string.h>

int sec = 0, min = 0, hour =  1, cnt = 0;
bit APM = 0 , flag = 0;

// External Interrupt 0 service routine
interrupt [EXT_INT0] void ext_int0_isr(void){
    sec++;
    flag = 1;
}

// External Interrupt 1 service routine
interrupt [EXT_INT1] void ext_int1_isr(void){
    min++;
    flag = 1;
}                      

// External Interrupt 2 servic-e routine
interrupt [EXT_INT2] void ext_int2_isr(void){
    hour++;
    flag = 1;
}
```


외부 인터럽트 스위치를 이용하여


0번스위치는 시간 1번 스위치는 분 2번스위치는 시간을 조절할 수 있게하였다


```c
// Timer 0 overflow interrupt service routine
interrupt [TIM0_OVF] void timer0_ovf_isr(void)
{
    // Reinitialize Timer 0 value
    TCNT0=0x06;
    cnt++;   

    if (cnt == 1000){
        sec++;
        cnt = 0;
        flag = 1;
    }
    if (sec == 60){
        min++;
        sec = 0;
        flag = 1;
    }
    if (min == 60){
        hour++;
        min = 0;
        flag = 1;                              
    }
    if (hour == 13){
        APM = ~APM;
        hour = 1;
        flag = 1;                              
    }
}
```
인터럽트 타이머를 이용하여 1ms단위로 인터럽트를 걸어주며 cnt값을 증가시킨다


cnt값이 1000이 되면 1s이므로 sec값을 1증가시킨다


hour 값이 13이 되면 1로 hour를 초기화 시키며 bit APM을 토글 시킨다.


APM값에 따라 am과 pm이 표시된다


flag가 1일때 display에 새롭게 작성된다


```c
void main(void)
{
char* am = "am";
char* pm = "pm";
char result[2];

// Timer/Counter 0 initialization                                   
// Clock source: System Clock
// Clock value: 250.000 kHz
// Mode: Normal top=0xFF
// OC0 output: Disconnected
// Timer Period: 1 ms
ASSR=0<<AS0;
TCCR0=(0<<WGM00) | (0<<COM01) | (0<<COM00) | (0<<WGM01) | (1<<CS02) | (0<<CS01) | (0<<CS00);
TCNT0=0x06;
OCR0=0x00;

// Timer(s)/Counter(s) Interrupt(s) initialization
TIMSK=(0<<OCIE2) | (0<<TOIE2) | (0<<TICIE1) | (0<<OCIE1A) | (0<<OCIE1B) | (0<<TOIE1) | (0<<OCIE0) | (1<<TOIE0);
ETIMSK=(0<<TICIE3) | (0<<OCIE3A) | (0<<OCIE3B) | (0<<TOIE3) | (0<<OCIE3C) | (0<<OCIE1C);

```


am과 pm을 표시하기위한 문자열을 변수로 지정해준다


timer0을 250khz로 분주해주며 `TCNT0=0x06`으로 설정하여 카운트의 시작이 6부터 시작하도록 설정한다


```c
// External Interrupt(s) initialization
// INT0: On
// INT0 Mode: Rising Edge
// INT1: On
// INT1 Mode: Rising Edge
// INT2: On
// INT2 Mode: Rising Edge
// INT3: On
// INT3 Mode: Rising Edge
// INT4: On
// INT4 Mode: Rising Edge
// INT5: On
// INT5 Mode: Rising Edge
// INT6: Off
// INT7: Off
EICRA=(1<<ISC31) | (1<<ISC30) | (1<<ISC21) | (1<<ISC20) | (1<<ISC11) | (1<<ISC10) | (1<<ISC01) | (1<<ISC00);
EICRB=(0<<ISC71) | (0<<ISC70) | (0<<ISC61) | (0<<ISC60) | (1<<ISC51) | (1<<ISC50) | (1<<ISC41) | (1<<ISC40);
EIMSK=(0<<INT7) | (0<<INT6) | (1<<INT5) | (1<<INT4) | (1<<INT3) | (1<<INT2) | (1<<INT1) | (1<<INT0);
EIFR=(0<<INTF7) | (0<<INTF6) | (1<<INTF5) | (1<<INTF4) | (1<<INTF3) | (1<<INTF2) | (1<<INTF1) | (1<<INTF0);


// USART0 initialization
// Communication Parameters: 8 Data, 1 Stop, No Parity
// USART0 Receiver: Off
// USART0 Transmitter: On
// USART0 Mode: Asynchronous
// USART0 Baud Rate: 9600
UCSR0A=(0<<RXC0) | (0<<TXC0) | (0<<UDRE0) | (0<<FE0) | (0<<DOR0) | (0<<UPE0) | (0<<U2X0) | (0<<MPCM0);
UCSR0B=(0<<RXCIE0) | (0<<TXCIE0) | (0<<UDRIE0) | (0<<RXEN0) | (1<<TXEN0) | (0<<UCSZ02) | (0<<RXB80) | (0<<TXB80);
UCSR0C=(0<<UMSEL0) | (0<<UPM01) | (0<<UPM00) | (0<<USBS0) | (1<<UCSZ01) | (1<<UCSZ00) | (0<<UCPOL0);
UBRR0H=0x00;
UBRR0L=0x67;

// Globally enable interrupts
#asm("sei")
```


외부 인터럽트를 사용하기위해 메모리를 설정해준다


rising edge에서 동작하도록 설정했다


또한 USART통신을 하기위해 메모리를 설정해주었다


USART통신은 LCD와 통신하기 위해 사용한다


```c
printf("$I\r"); //화면 초기화
printf("$C\r");  
printf("$L,1\r");



    while (1){
        if (flag == 1){
            if (APM)
                strcpy(result,pm);
            else
                strcpy(result,am);

            printf("$G, 1, 1\r");
            printf("$T,%02d : %02d : %02d %01c%01c\r",hour,min,sec,result[0],result[1]);

            flag = 0;
        }
    }
}
```

while문에 들어가기전 LCD를 초기화 해주며 while문을 통하여 display에 시간을 표시한다
