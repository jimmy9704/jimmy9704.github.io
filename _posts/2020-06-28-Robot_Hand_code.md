---
layout: post
title:  "[c][code vision AVR] Robot Hand Schematic"
date:   2020-06-28
desc: "Quick test on writing code snippets in a blog post"
keywords: "Robot Hand Schematic"
categories: [Mcu]
tags: [Mcu]
icon: icon-html
---


# Robot Hand by using code vision


<iframe width="949" height="535" src="https://www.youtube.com/embed/RAhKiy8a3mk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Mode 1


     - 로봇손과 가위바위보 게임을 진행한다
     - Robot은 랜덤하게 가위바위보중 하나를 택한다


Mode 2


     - Mode 1 ,4 에서의 전적을 eeprom에 기록한다
     - 최근 5경기 동안의 승률을 계산하여 출력한다


Mode 3


     - Player의 손동작을 따라하는 Mode이다


Mode 4


     - Bluetooth 모듈을 이용하여 다른 Player와 대결한다
     - Player1은 ADC를 이용하여 자신의 손동작을 인식시킨다
     - Player2는 Bluetooth 모듈을 이용하여 로봇손을 제어한다


```c
#include <mega128.h>
#include <io.h>
#include <delay.h>
#include <stdio.h>
#include <stdlib.h>

#define ADC_VREF_TYPE ((0<<REFS1) | (0<<REFS0) | (0<<ADLAR))

int my_data = 0;
int robot_data = 0;
int pwm1,pwm2;
int blt_data=3;
eeprom char score[5];

void save(int data){      //save data
    int i;
    for(i = 3; i>=0; i--){
        score[i+1] = score[i];
    }
    if(data == 0)    score[0] = 'L';    
    else if(data == 1) score[0] = 'D';
    else score[0] = 'W';
}
```
`void save(int data)`


게임 전적을 eeprom에 기록한다


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
`unsigned int read_adc(unsigned char adc_input)`


Flex센서의 값을 0~1023사이의 값으로 return한다


```c
//set robot data
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
`void set_my_data()`


read_adc로 읽은 데이터를 가위,바위,보로 분류한다


```c
void play(int robot_data){
    set_my_data();

    printf("$C\r");;        
    printf("$G,; 2, 1\r");
    delay_us(10);
    if (robot_data == 0){
        printf("$T, Robot: Rock\r");
        pwm1 = 600;
        OCR1AH =  pwm1>>8;
        OCR1AL =  pwm1 & 0x00ff ;    

        pwm2 = 170;
        OCR3AH =  pwm2>>8;
        OCR3AL =  pwm2 & 0x00ff ;   
    }
    else if (robot_data == 1){
        printf("$T, Robot: Scissors\r");
        pwm1 = 170;
        OCR1AH =  pwm1>>8;
        OCR1AL =  pwm1 & 0x00ff ;    

        pwm2 = 170;
        OCR3AH =  pwm2>>8;
        OCR3AL =  pwm2 & 0x00ff ;
    }
    else if (robot_data == 2){
        printf("$T, Robot: Paper\r");
        pwm1 = 170;
        OCR1AH =  pwm1>>8;
        OCR1AL =  pwm1 & 0x00ff ;    

        pwm2 = 600;
        OCR3AH =  pwm2>>8;
        OCR3AL =  pwm2 & 0x00ff ;
    }

    delay_ms(100);
    printf("$G,; 1, 1\r");
    if (my_data == 0)
        printf("$T, You: Rock\r");
    else if (my_data == 1)
        printf("$T, You: Scissors\r");
    else if (my_data == 2)
        printf("$T, You: Paper\r");

    delay_ms(2000);

    printf("$C\r");
    if (my_data == robot_data){
        printf("$G, 1, 1\r");
        printf("$T, draw\r");
        save(1);
    }
    else if( my_data == 0){
        if(robot_data == 1){
            printf("$G, 1, 1\r");
            printf("$T, you win\r");
            save(2);
        }
        else{
            printf("$G, 1, 1\r");
            printf("$T, you lose\r");
            save(0);    
        }    
    }
    else if( my_data == 1){
        if(robot_data == 2){
            printf("$G, 1, 1\r");
            printf("$T, you win\r");
            save(2);
        }
        else{
            printf("$G, 1, 1\r");
            printf("$T, you lose\r");  
            save(0);  
        }    
    }
    else if( my_data == 2){
        if(robot_data == 0){
            printf("$G, 1, 1\r");
            printf("$T, you win\r");  
            save(2);
        }
        else{
            printf("$G, 1, 1\r");
            printf("$T, you lose\r");     
            save(0);
        }    
    }      
}

```
`void play(int robot_data)`


robot_data를 bluetooth또는 random하게 전달받아 	게임을 진행한다


```c
// External Interrupt 0 service routine
//mode 1
interrupt [EXT_INT0] void ext_int0_isr(void)
{
    srand(read_adc(0));
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");
    PORTA.0 = 1;
    printf("$G, 1, 1\r");
    printf("$T, 3\r");
    delay_ms(1000);

    printf("$G, 1, 1\r");
    printf("$T, 2\r");
    delay_ms(1000);

    printf("$G, 1, 1\r");
    printf("$T, 1\r");
    delay_ms(1000);
    PORTA.0 = 0;
    robot_data = rand()%3;

    play(robot_data);

    delay_ms(2000);
    printf("$C\r");
}
```
`interrupt [EXT_INT0] void ext_int0_isr(void)`


step1 ) 3,2,1카운팅을 하며 스피커로 음성을 출력한다


step2 ) 카운팅이 종료되면 play함수를 호출한다


step 3) srand(read_adc(0));


atmega128의 경우 seed값을 time에따라 받을 수 없다 따라서 adc값을 seed로 받는다


```c
// External Interrupt 1 service routine
//mode 2
interrupt [EXT_INT1] void ext_int1_isr(void)
{
    int win = 0;   
    int lose = 0;
    int i;
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");

    printf("$G, 1, 1\r");
    printf("$T, 1 2 3 4 5  win\r");

    printf("$G, 2, 1\r");

    for(i = 0; i<5; i++){  
        printf("$T, %01c\r",score[i]);
        if(score[i] == 87) {
            win++;
        }
    }  

        for(i = 0; i<5; i++){  
        printf("$T, %01c\r",score[i]);
        if(score[i] == 76) {
            lose++;
        }
    }

    printf("$G, 2, 12\r");
    if(win == 5)
        printf("$T, %03d\r",win*20);
    else
        printf("$T, %02d\r",win*100/(win+lose));
    printf("$G, 2, 15\r");
    printf("$T, %%\r");

    while(PIND.6 == 0);

    delay_ms(30);

    printf("$I\r");
    printf("$C\r");     
}
```
`interrupt [EXT_INT1] void ext_int1_isr(void)`


전적을 보여준다


```c
// External Interrupt 6 service routine
//mode 3
interrupt [EXT_INT6] void ext_int6_isr(void)
{
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");
    printf("$G, 1, 1\r");
    printf("$T,follow mode\r");  

    delay_ms(30);

    while(PIND.6 == 0){          

        pwm1 = -2.125*read_adc(1)+919;
        OCR1AH =  pwm1>>8;
        OCR1AL =  pwm1 & 0x00ff;  

        pwm2 = 2.125*read_adc(0) - 143.75;
        OCR3AH =  pwm2>>8;
        OCR3AL =  pwm2 & 0x00ff ;        
    }   

    delay_ms(20);
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");
    delay_ms(20);
}
```
`interrupt [EXT_INT6] void ext_int6_isr(void)`


pwm1 = -2.125*read_adc(1)+919;


위 코드를 사용하여 ADC값을 pwm으로 바꿔 	로봇손의 	서보모터에 전달한다


```c
//USART1 RX Interrupt
//get bluetooth data
interrupt [USART1_RXC] void usart1_rx_isr(void)
{
    blt_data = UDR1-48;
}

//mode 4
void mode4(){
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");
    PORTA.0 = 1;
    blt_data = 3;
    printf("$G, 1, 1\r");
    printf("$T, 3\r");
    delay_ms(1000);

    printf("$G, 1, 1\r");
    printf("$T, 2\r");
    delay_ms(1000);

    printf("$G, 1, 1\r");
    printf("$T, 1\r");
    delay_ms(1000);
    PORTA.0 = 0;

    if (blt_data >= 3){
        printf("$G, 1, 1\r");
        printf("$T, time out\r");
    }
    else play(blt_data);

    delay_ms(2000);
    printf("$C\r");      
}
```
`interrupt [USART1_RXC] void usart1_rx_isr(void)`


blt_data = UDR1-48;


위 코드를 이용하여 블루투스의 값을 읽어온다


3초안에 블루투스값을 받기 위해 인터럽트로 설정하였다


```c
void main(void)  
{
    unsigned int adc_data = 0;     

    DDRB = 0xFF;
    DDRA = 0xFF;   
    DDRE = (0<<DDE7) | (1<<DDE3) ;
    // Port D initialization
    DDRD=(0<<DDD6);
    PORTD=(0<<PORTD6);


    // Timer/Counter 1 initialization
    TCCR1A=(1<<COM1A1) | (0<<COM1A0) | (0<<COM1B1) | (0<<COM1B0) | (0<<COM1C1) | (0<<COM1C0) | (1<<WGM11) | (0<<WGM10);
    TCCR1B=(0<<ICNC1) | (0<<ICES1) | (1<<WGM13) | (1<<WGM12) | (0<<CS12) | (1<<CS11) | (1<<CS10);
    TCNT1H=0x00;
    TCNT1L=0x00;
    ICR1H=0x13;
    ICR1L=0x87;
    OCR1AH=0x00;
    OCR1AL=0x00;
    OCR1BH=0x00;
    OCR1BL=0x00;
    OCR1CH=0x00;
    OCR1CL=0x00;    


    // Timer/Counter 3 initialization
    TCCR3A=(1<<COM3A1) | (0<<COM3A0) | (0<<COM3B1) | (0<<COM3B0) | (0<<COM3C1) | (0<<COM3C0) | (1<<WGM31) | (0<<WGM30);
    TCCR3B=(0<<ICNC3) | (0<<ICES3) | (1<<WGM33) | (1<<WGM32) | (0<<CS32) | (1<<CS31) | (1<<CS30);
    TCNT3H=0x00;
    TCNT3L=0x00;
    ICR3H=0x13;
    ICR3L=0x87;
    OCR3AH=0x00;
    OCR3AL=0x00;
    OCR3BH=0x00;
    OCR3BL=0x00;
    OCR3CH=0x00;
    OCR3CL=0x00;

    // Timer(s)/Counter(s) Interrupt(s) initialization
    TIMSK=(0<<OCIE2) | (0<<TOIE2) | (0<<TICIE1) | (0<<OCIE1A) | (0<<OCIE1B) | (0<<TOIE1) | (0<<OCIE0) | (0<<TOIE0);
    ETIMSK=(0<<TICIE3) | (0<<OCIE3A) | (0<<OCIE3B) | (0<<TOIE3) | (0<<OCIE3C) | (0<<OCIE1C);

    // External Interrupt(s) initialization
    // INT0: On
    // INT0 Mode: Rising Edge
    // INT1: On
    // INT1 Mode: Rising Edge
    // INT2: Off
    // INT3: Off
    // INT4: Off
    // INT5: Off
    // INT6: On
    // INT6 Mode: Rising Edge
    // INT7: Off
    EICRA=(0<<ISC31) | (0<<ISC30) | (0<<ISC21) | (0<<ISC20) | (1<<ISC11) | (1<<ISC10) | (1<<ISC01) | (1<<ISC00);
    EICRB=(0<<ISC71) | (0<<ISC70) | (1<<ISC61) | (1<<ISC60) | (0<<ISC51) | (0<<ISC50) | (0<<ISC41) | (0<<ISC40);
    EIMSK=(0<<INT7) | (1<<INT6) | (0<<INT5) | (0<<INT4) | (0<<INT3) | (0<<INT2) | (1<<INT1) | (1<<INT0);
    EIFR=(0<<INTF7) | (1<<INTF6) | (0<<INTF5) | (0<<INTF4) | (0<<INTF3) | (0<<INTF2) | (1<<INTF1) | (1<<INTF0);      


    // Function: Bit7=Out Bit6=Out Bit5=Out Bit4=Out Bit3=Out Bit2=Out Bit1=Out Bit0=In
    DDRF=(0<<DDF7) | (0<<DDF6) | (0<<DDF5) | (0<<DDF4) | (0<<DDF3) | (0<<DDF2) | (0<<DDF1) | (0<<DDF0);
    // State: Bit7=0 Bit6=0 Bit5=0 Bit4=0 Bit3=0 Bit2=0 Bit1=0 Bit0=T
    PORTF=(0<<PORTF7) | (0<<PORTF6) | (0<<PORTF5) | (0<<PORTF4) | (0<<PORTF3) | (0<<PORTF2) | (0<<PORTF1) | (0<<PORTF0);      


    ADMUX=ADC_VREF_TYPE;
    ADCSRA=(1<<ADEN) | (0<<ADSC) | (0<<ADFR) | (0<<ADIF) | (0<<ADIE) | (0<<ADPS2) | (1<<ADPS1) | (1<<ADPS0);
    SFIOR=(0<<ACME);


    UCSR0A=(0<<RXC0) | (0<<TXC0) | (0<<UDRE0) | (0<<FE0) | (0<<DOR0) | (0<<UPE0) | (0<<U2X0) | (0<<MPCM0);
    UCSR0B=(0<<RXCIE0) | (0<<TXCIE0) | (0<<UDRIE0) | (0<<RXEN0) | (1<<TXEN0) | (0<<UCSZ02) | (0<<RXB80) | (0<<TXB80);
    UCSR0C=(0<<UMSEL0) | (0<<UPM01) | (0<<UPM00) | (0<<USBS0) | (1<<UCSZ01) | (1<<UCSZ00) | (0<<UCPOL0);
    UBRR0H=0x00;
    UBRR0L=0x67;  

    UBRR1L = 103;  UCSR1B = 0x90; SREG = 0x80;

    // Globally enable interrupts
    #asm("sei")    

    delay_ms(300);
    printf("$I\r");
    printf("$C\r");
    printf("$B,0\r");

    while (1)
        {   
            pwm1 = 170;
            OCR1AH =  pwm1>>8;
            OCR1AL =  pwm1 & 0x00ff ;    

            pwm2 = 600;
            OCR3AH =  pwm2>>8;
            OCR3AL =  pwm2 & 0x00ff ;
            printf("$G, 1, 1\r");
            printf("$T,Push Button to  start\r");

            if(PINE.7 == 1 ) mode4();         
        }
}
```
`void mode4()`


인터럽트 내부에서 인터럽트가 중복 실행 되지않아 폴링방식으로 모드를 실행한다


`while (1)`


초기 화면을 출력하며 외부 인터럽트가 실행되면서 모드로 넘어간다


폴링 방식으로 mode4 실행조건을 체크한다
