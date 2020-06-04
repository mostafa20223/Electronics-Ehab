# Electronics-Ehab

***We seek to develop a <span style = "color: red;"> digital filteration </span> to analog signal using <span style = "color: blue;"> microcontroller 8051, adc0808 and dac0808 </span>***

## Simulation
![Simulation](https://github.com/mostafa20223/Electronics-Ehab/blob/master/Circuit%20Schematic%20and%20Design/Simulation.png)

## Report
[Report](https://github.com/mostafa20223/Electronics-Ehab/blob/master/Report/PDF/Research-Project-SBE303B-team4.pdf)

## Code
``` C
#include <reg51.h>

sbit ADD_A = P2^0;    
sbit ADD_B = P2^1;    
sbit ADD_C = P2^2; 
sbit clk = P2^3;
sbit ALE = P2^4;
sbit OE = P2^5;
sbit SC = P2^6;
sbit EOC = P2^7;
sfr input = 0x80;
sfr DAC = 0x90;
sbit lowpass = P3^0;
sbit highpass = P3^1;
sbit notchh = P3^2;
sbit chnl1 = P3^3;
sbit chnl2 = P3^4;

float first = 0;
float second = 0;
float third = 0;
float filter[5] = {0, 0, 0, 0, 0};
float low_filter[5] = {0.035654, 0.24104, 0.44661, 0.24104, 0.035654};
float high_filter[5] = {-0.000078131, -0.00052738, 0.9991, -0.00052738, -0.000078131};
float notch_filter[5] = {-0.00042197, -0.0028487, 0.99515, -0.0028487, -0.00042197};
// float notch_filter[5] = {0.035709, 0.24107, 0.44644, 0.24107, 0.035709};
float x[5] = {0, 0, 0, 0, 0};

void delay(int dly)
{
	int i;
	for(i = dly; i > 0; i--);
}	   

void timer0() interrupt 1
{
	clk =~ clk;
}

void main()
{
	EOC = 1;
	input = 0xff;
	ALE = 0;
	OE = 0;
	SC = 0;	
	TMOD = 0x22;
	TH0 = 0xFD;
	IE = 0x82;
	TR0 = 1;
	P3 = 0xff;
	DAC = 0xff;
	while(1)
	{
		x[1] = x[0];
		x[2] = x[1];
		x[3] = x[2];
		x[4] = x[3];
		ALE = 1;
		delay(2);
		SC = 1;
		delay(1);
		ALE = 0;
		delay(1);
		SC = 0;
		while(EOC == 1);
		while(EOC == 0);
		OE = 1;
		x[0] = input;		
		if (chnl1 == 0 && chnl2 == 1)
		{
			ADD_C = 0, ADD_B = 0, ADD_A = 0;
			delay(2);
			if (lowpass == 0 && highpass == 1 && notchh == 1)
			{
				DAC = low_filter[4] * input + low_filter[3] * x[3] + low_filter[2] * x[2] + low_filter[1] * x[1] + low_filter[0] * x[0];
				OE = 0;
			}
			else if (highpass == 0 && lowpass == 1 && notchh == 1)
			{
				DAC = high_filter[4] * input + high_filter[3] * x[3] + high_filter[2] * x[2] + high_filter[1] * x[1] + high_filter[0] * x[0];
				OE = 0;				
			}
			else if (notchh == 0 && highpass == 1 && lowpass == 1)
			{
				// DAC = notch_filter[6] * input + notch_filter[5] * x[5] + notch_filter[4] * x[4] + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				DAC = notch_filter[4] * input + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				// DAC = notch_filter[14] * input + notch_filter[13] * x[13] + notch_filter[12] * x[12] + notch_filter[11] * x[11] + notch_filter[10] * x[10] + notch_filter[9] * x[9] + notch_filter[8] * x[8] + notch_filter[7] * x[7] + notch_filter[6] * x[6] + notch_filter[5] * x[5] + notch_filter[4] * x[4] + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				OE = 0;				
			}			
			else if (lowpass == 1 && highpass == 1 && notchh == 1)
			{
				first = low_filter[4] * input + low_filter[3] * x[3] + low_filter[2] * x[2] + low_filter[1] * x[1] + low_filter[0] * x[0];
				second = high_filter[4] * input + high_filter[3] * x[3] + high_filter[2] * x[2] + high_filter[1] * x[1] + high_filter[0] * x[0];
				third = notch_filter[4] * input + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				DAC = first + second + third;
				OE = 0;
			}
			else
			{
				DAC = filter[4] * input + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
				// DAC = filter[4] * input + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
				// DAC = filter[14] * input + filter[13] * x[13] + filter[12] * x[12] + filter[11] * x[11] + filter[10] * x[10] + filter[9] * x[9] + filter[8] * x[8] + filter[7] * x[7] + filter[6] * x[6] + filter[5] * x[5] + filter[4] * x[4] + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
				OE = 0;
			}
		}
		else if (chnl2 == 0 && chnl1 == 1)
		{
			ADD_C = 0, ADD_B = 0, ADD_A = 1;
			delay(2);
			if (lowpass == 0 && highpass == 1 && notchh == 1)
			{
				// DAC = low_filter[6] * input + low_filter[5] * x[5] + low_filter[4] * x[4] + low_filter[3] * x[3] + low_filter[2] * x[2] + low_filter[1] * x[1] + low_filter[0] * x[0];
				DAC = low_filter[4] * input + low_filter[3] * x[3] + low_filter[2] * x[2] + low_filter[1] * x[1] + low_filter[0] * x[0];
				// DAC = low_filter[14] * input + low_filter[13] * x[13] + low_filter[12] * x[12] + low_filter[11] * x[11] + low_filter[10] * x[10] + low_filter[9] * x[9] + low_filter[8] * x[8] + low_filter[7] * x[7] + low_filter[6] * x[6] + low_filter[5] * x[5] + low_filter[4] * x[4] + low_filter[3] * x[3] + low_filter[2] * x[2] + low_filter[1] * x[1] + low_filter[0] * x[0];
				OE = 0;
			}
			else if (highpass == 0 && lowpass == 1 && notchh == 1)
			{
				// DAC = high_filter[6] * input + high_filter[5] * x[5] + high_filter[4] * x[4] + high_filter[3] * x[3] + high_filter[2] * x[2] + high_filter[1] * x[1] + high_filter[0] * x[0];
				DAC = high_filter[4] * input + high_filter[3] * x[3] + high_filter[2] * x[2] + high_filter[1] * x[1] + high_filter[0] * x[0];
				// DAC =  high_filter[14] * input + high_filter[13] * x[13] + high_filter[12] * x[12] + high_filter[11] * x[11] + high_filter[10] * x[10] + high_filter[9] * x[9] + high_filter[8] * x[8] + high_filter[7] * x[7] + high_filter[6] * x[6] + high_filter[5] * x[5] + high_filter[4] * x[4] + high_filter[3] * x[3] + high_filter[2] * x[2] + high_filter[1] * x[1] + high_filter[0] * x[0];
				OE = 0;				
			}
			else if (notchh == 0 && highpass == 1 && lowpass == 1)
			{
				// DAC = notch_filter[6] * input + notch_filter[5] * x[5] + notch_filter[4] * x[4] + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				DAC = notch_filter[4] * input + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				// DAC = notch_filter[14] * input + notch_filter[13] * x[13] + notch_filter[12] * x[12] + notch_filter[11] * x[11] + notch_filter[10] * x[10] + notch_filter[9] * x[9] + notch_filter[8] * x[8] + notch_filter[7] * x[7] + notch_filter[6] * x[6] + notch_filter[5] * x[5] + notch_filter[4] * x[4] + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				OE = 0;				
			}			
			else if (lowpass == 1 && highpass == 1 && notchh == 1)
			{
				first = low_filter[4] * input + low_filter[3] * x[3] + low_filter[2] * x[2] + low_filter[1] * x[1] + low_filter[0] * x[0];
				second = high_filter[4] * input + high_filter[3] * x[3] + high_filter[2] * x[2] + high_filter[1] * x[1] + high_filter[0] * x[0];
				third = notch_filter[4] * input + notch_filter[3] * x[3] + notch_filter[2] * x[2] + notch_filter[1] * x[1] + notch_filter[0] * x[0];
				DAC = first + second + third;
				OE = 0;
			}
			else
			{
				DAC = filter[4] * input + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
				// DAC = filter[4] * input + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
				// DAC = filter[14] * input + filter[13] * x[13] + filter[12] * x[12] + filter[11] * x[11] + filter[10] * x[10] + filter[9] * x[9] + filter[8] * x[8] + filter[7] * x[7] + filter[6] * x[6] + filter[5] * x[5] + filter[4] * x[4] + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
				OE = 0;
			}
		}
		else if ((chnl2 == 0 && chnl1 == 0) || (chnl2 == 1 && chnl1 == 1))
		{
			DAC = filter[8] * input + filter[7] * x[7] + filter[6] * x[6] + filter[5] * x[5] + filter[4] * x[4] + filter[3] * x[3] + filter[2] * x[2] + filter[1] * x[1] + filter[0] * x[0];
			OE = 0;
		}
	}
}
```

