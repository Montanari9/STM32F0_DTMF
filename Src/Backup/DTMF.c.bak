#include "stm32f0xx_hal.h"
#include "tim.h"
#include "dtmf.h"

// 62,5KHz @ 16MHz PWM Center Aligned
#define PWM_DC50    (PWM_FREQ/2)		

extern unsigned char t_tone;

//************************** SINE TABLE *************************************
// One period sampled on 128 samples and
// quantized on 7 bit
//**************************************************************************
const unsigned char auc_SinParam [128] = {
64,67,
70,73,
76,79,
82,85,
88,91,
94,96,
99,102,
104,106,
109,111,
113,115,
117,118,
120,121,
123,124,
125,126,
126,127,
127,127,
127,127,
127,127,
126,126,
125,124,
123,121,
120,118,
117,115,
113,111,
109,106,
104,102,
99,96,
94,91,
88,85,
82,79,
76,73,
70,67,
64,60,
57,54,
51,48,
45,42,
39,36,
33,31,
28,25,
23,21,
18,16,
14,12,
10,9,
7,6,
4,3,
2,1,
1,0,
0,0,
0,0,
0,0,
1,1,
2,3,
4,6,
7,9,
10,12,
14,16,
18,21,
23,25,
28,31,
33,36,
39,42,
45,48,
51,54,
57,60};

//***************************  x_SW  ***************************************
//				Table of x_SW : x_SW = ROUND(8*N_samples*f*510/Fck)
//**************************************************************************

//high frequency (column)
//1209hz  ---> x_SW = 39
//1336hz  ---> x_SW = 44
//1477hz  ---> x_SW = 48
//1633hz  ---> x_SW = 53

const unsigned char auc_frequencyH [4] = { 39, 44, 48, 53}; //TIM1@16MHz 

//low frequency (row)
//697hz  ---> x_SW = 23
//770hz  ---> x_SW = 25
//852hz  ---> x_SW = 28
//941hz  ---> x_SW = 31

const unsigned char auc_frequencyL [4] = { 23, 25, 28, 31};  //TIM1@16MHz

void Send_Digit_Tone (unsigned char digi)
{

	switch (digi)
	{
		case Digstar: //ok
			x_SWb = auc_frequencyL[3];
			x_SWa = auc_frequencyH[0];	
			break;
			
		case Dighash: //ok
			x_SWb = auc_frequencyL[3];
			x_SWa = auc_frequencyH[2];	
			break;
				
		case Dig9: //ok
			x_SWb = auc_frequencyL[2];
			x_SWa = auc_frequencyH[2];	
			break;
				
		case Dig8: //ok
			x_SWb = auc_frequencyL[2];
			x_SWa = auc_frequencyH[1];	
			break;
				
		case Dig7: //ok
			x_SWb = auc_frequencyL[2];
			x_SWa = auc_frequencyH[0];	
			break;
				
		case Dig6: //ok
			x_SWb = auc_frequencyL[1];
			x_SWa = auc_frequencyH[2];	
			break;
				
		case Dig5: //ok
			x_SWb = auc_frequencyL[1];
			x_SWa = auc_frequencyH[1];	
			break;
				
		case Dig4: //ok
			x_SWb = auc_frequencyL[1];
			x_SWa = auc_frequencyH[0];	
			break;
				
		case Dig3: //ok
			x_SWb = auc_frequencyL[0];
			x_SWa = auc_frequencyH[2];	
			break;
				
		case Dig2: //ok
			x_SWb = auc_frequencyL[0];
			x_SWa = auc_frequencyH[1];	
			break;
				
		case Dig1: //ok
		  x_SWa = auc_frequencyH[0];
			x_SWb = auc_frequencyL[0];
			break;
				
		case Dig0: //ok
			x_SWb = auc_frequencyL[3];
			x_SWa = auc_frequencyH[1];	
			break;
		}
		/* TIM1 Main Output Enable */
		HAL_TIM_PWM_Start(&htim1,TIM_CHANNEL_1);
	  HAL_TIM_Base_Start_IT(&htim1);
}


//**************************  global variables  ****************************
unsigned char x_SWa = 0x00;               // step width of high frequency
unsigned char x_SWb = 0x00;               // step width of low frequency
unsigned int  i_CurSinValA = 0;           // position freq. A in LUT (extended format)
unsigned int  i_CurSinValB = 0;           // position freq. B in LUT (extended format)
unsigned int  i_TmpSinValA;               // position freq. A in LUT (actual position)
unsigned int  i_TmpSinValB;               // position freq. B in LUT (actual position)

//**************************************************************************
// Initialization
//**************************************************************************
void init_dtmf (void)
{
//	HAL_TIM_PWM_Start(&htim1,TIM_CHANNEL_1);
	HAL_TIM_Base_Start_IT(&htim1);
}


//**************************************************************************
// Timer overflow interrupt service routine
//**************************************************************************
void interrupt_service_tim_dtmf (void)
{ 
  static uint32_t tmp;
	
	//PD3=0; debug only
	//#asm
	//	BRES	500Fh,#3
	//#endasm
  // move Pointer about step width aheaed
  i_CurSinValA += x_SWa;       
  i_CurSinValB += x_SWb;
  
	// normalize Temp-Pointer
  i_TmpSinValA  =  (char)((i_CurSinValA/8)&(0x007F)); 
  i_TmpSinValB  =  (char)((i_CurSinValB/8)&(0x007F));
	
  // calculate PWM value: high frequency value + 3/4 low frequency value
  tmp = (auc_SinParam[i_TmpSinValA]) + ((auc_SinParam[i_TmpSinValB]-(auc_SinParam[i_TmpSinValB]>>2)));
  /* Set the Pulse value */
	/* Set the Capture Compare Register value */
  TIM1->CCR1 = tmp;
	//PD3=1; debug only
	//#asm
	//	BSET	500Fh,#3
	//#endasm	
}
