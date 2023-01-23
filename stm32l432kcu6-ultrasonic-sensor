#include "stm32l4xx.h" /* Microcontroller information */

int sensor_value = 0;

/*---------------------------------------------------*/
/* Initialize GPIO pins used in the program */
/* PA[0] = UP DOWN Wave */
/* PA[1] = Sensor Comm Input */
/* PB[0] =  */
/*---------------------------------------------------*/
void PinSetup(){

    RCC->AHB2ENR |= 0x01;             // Enable GPIOA clock (bit 0) 
    GPIOA->MODER &= ~(0x00000003);   // Pin 0 drives the motor 
    GPIOA->MODER |= 0x00000002;   // Pin 0 is AF, alternate function
    GPIOA->AFR[0] &= ~(0x0000000F); //
    GPIOA->AFR[0] |= (0x01);       // AF1, TM2_CH1
	
	  GPIOA->MODER &= ~( 0x0000000C) ; // Clear Bits for Pin 1
    GPIOA->MODER |= 0x0000000C ;    // Pin1 is set to Analog Mode 11 , PA1 is connected to channel 6 ADC1_IN6
    
    RCC->AHB2ENR |= 0x02;    
    GPIOB->MODER &= ~(0x00000003); // Reset PB[0];
    GPIOB->ODR = 0;

}

void ADCconfiguration(){
	
    RCC->AHB2ENR |= 0x00002000 ; // Bit 13 is ADC Enable bit
    RCC->CCIPR &= ~(0x30000000) ; // Clear bit 28 29 for peripheral clock selection
    RCC->CCIPR |= 0x30000000 ;   // Set bit 28 29 to 11 system clock
  
    ADC1->CR &= ~(0x20000000) ;  // Clear bit 29 th for Deep Power Down Mode 
    ADC1->CR |= 0x10000000  ;    // Set 28th bit for AD voltage regulator
    //#############wait for regulor to star slide 29 delay(20us)############

     ADC1->CFGR |= 0x00002000 ;  // Set 13 bit for continuous mode , in ADC configuration register
     
     ADC1->SQR1 &= ~(0x0000000F) ; // First four bit 0000 means 1 channel
     ADC1->SQR1 &= ~(0x000007C0) ; // Reset Bit 10:6 Channel depend on GPIO pin, for PA1, it is chnnel 6
     ADC1->SQR1 |=   0x00000180 ; // Put SQR1 (bit 10:6 to 6 means scan channel 6?

     ADC1->CR |= 0x00000001 ;    // 0th bit AD Enable bit
     
     while (!(ADC1->ISR & 0x00000001)); //Test bit for ADC to be ready

     ADC1->CR |= 0x00000004 ;     // 3rd bit is ADSTART 

}

void ADCread(){

     while(!(ADC1->ISR & 0x00000004)); // Bit 2 is EOC bit, End of conversion, 
                                   // 1 means conversion has finished
     sensor_value = ADC1->DR;               // read data register
                                   // Reading this clears the EOC bit  in ISR register
                                   // 12-bit result may be left or right aligned ;
}

void TimerSetup() {
	
		RCC->APB1ENR1 |=  0x01 ;         //TIM2E enable clock */
		
		TIM2->CR1 |= 0x01 ;              // Output Enable for TIM2
		TIM2->CCMR1 &= ~(0x00000003) ;   // Capture Compare Mode set it output mode
		TIM2->CCMR1 &= ~(0x00000070) ;   //Clear bit 4:6
		TIM2->CCMR1 |= 0x00000060 ;      // Set bit 4:6 110 PWM mode 1

		TIM2->CCER &= ~(0x00000003) ;    //Clear bit 1:0 for enabling Ch 1 
		TIM2->CCER |= 0x01          ;    //Set bit 0 enable ch 1
		
		TIM2->DIER &= 0x0001;

		TIM2->PSC  = 1;                  //4MHz/(PSC+1)/(ARR+1)
		TIM2->ARR = 1999;              //set auto reload so that PWM period is 1ms
		TIM2->CCR1 = 1999;              //Initially set Duty Cycle to be 100%
        
}

void smallDelay();
void delay();

void smallDelay() {
  int i, j;
  for (i=0; i<4; ++i) { //outer loop
    j = i; //dummy operation for single-step test
  }
}

/*----------------------------------------------------------*/
/* Delay function - do nothing for about 1 second */
/*----------------------------------------------------------*/
void delay(){
    int volatile i,j,n;
    for (i=0; i<20; i++) { //outer loop
        for (j=0; j<11655; j++) { //inner loop
            n = j; //dummy operation for single-step test
        } //do nothing
    }
}

/*------------------------------------------------*/
/* Main program */
/*------------------------------------------------*/
int main(void) {

	int test_ctr;
	PinSetup(); //Configure GPIO pins
	ADCconfiguration();
	TimerSetup();
	
	TIM2->CCR1 = 10 * (TIM2->ARR + 1)/10;
	
	test_ctr = 0;
	
	/* Endless loop */
	while(1){
		test_ctr += 1;
		smallDelay();
		ADCread();
	}
}
