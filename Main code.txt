#include <blackfin.h>
#include <sys/exception.h>
#include "filter.h"
#include <cdefBF533.h>
#include "sysreg.h"
#include "ccblkfn.h"

//FILTERI
#define Nh 101
fir_state_fr16 my_filter_state;
fract16 delay[Nh] = {0};

fract16 np_1000hz[101] = {
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 3, 5, 9, 13, 18, 25, 33, 44, 57, 72, 91, 112, 136, 164, 196, 231, 269, 311, 357, 405, 456, 509, 563, 619, 675, 730, 785, 837, 887, 932, 974, 1011, 1041, 1066, 1084, 1095, 1098, 1095, 1084, 1066, 1041, 1011, 974, 932, 887, 837, 785, 730, 675, 619, 563, 509, 456, 405, 357, 311, 269, 231, 196, 164, 136, 112, 91, 72, 57, 44, 33, 25, 18, 13, 9, 5, 3, 2, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
}; //600bih

fract16 np_2000hz[101] = {
0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 2, 2, 3, 3, 2, 1, 0, -4, -9, -16, -25, -36, -48, -62, -76, -90, -102, -111, -115, -112, -101, -78, -42, 8, 74, 157, 257, 374, 505, 649, 802, 962, 1123, 1281, 1431, 1568, 1688, 1786, 1859, 1903, 1918, 1903, 1859, 1786, 1688, 1568, 1431, 1281, 1123, 962, 802, 649, 505, 374, 257, 157, 74, 8, -42, -78, -101, -112, -115, -111, -102, -90, -76, -62, -48, -36, -25, -16, -9, -4, 0, 1, 2, 3, 3, 2, 2, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0
}; //1.4kbih

fract16 np_3000hz[101] = {
0, 0, 0, 0, 0, 0, 0, 0, -1, -1, -2, -2, -2, -1, 0, 3, 7, 13, 20, 28, 35, 42, 45, 45, 38, 23, 0, -32, -74, -122, -174, -225, -269, -300, -309, -290, -235, -140, 0, 185, 414, 679, 974, 1286, 1600, 1902, 2177, 2408, 2583, 2693, 2730, 2693, 2583, 2408, 2177, 1902, 1600, 1286, 974, 679, 414, 185, 0, -140, -235, -290, -309, -300, -269, -225, -174, -122, -74, -32, 0, 23, 38, 45, 45, 42, 35, 28, 20, 13, 7, 3, 0, -1, -2, -2, -2, -1, -1, 0, 0, 0, 0, 0, 0, 0, 0
}; //2kbih

fract16 np_4000hz[101] = {
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, -1, -3, -5, -8, -11, -14, -15, -13, -9, 0, 13, 31, 51, 72, 90, 102, 102, 87, 53, 0, -72, -158, -251, -340, -410, -448, -438, -365, -221, 0, 297, 661, 1077, 1522, 1969, 2390, 2755, 3037, 3215, 3276, 3215, 3037, 2755, 2390, 1969, 1522, 1077, 661, 297, 0, -221, -365, -438, -448, -410, -340, -251, -158, -72, 0, 53, 87, 102, 102, 90, 72, 51, 31, 13, 0, -9, -13, -15, -14, -11, -8, -5, -3, -1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
}; //2.4kbih

fract16 np_5000hz[101] = {
0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 3, 5, 5, 5, 3, -1, -8, -17, -27, -35, -40, -39, -28, -7, 23, 63, 106, 144, 170, 174, 146, 83, -16, -145, -290, -431, -540, -591, -556, -413, -150, 233, 725, 1294, 1902, 2499, 3034, 3457, 3729, 3822, 3729, 3457, 3034, 2499, 1902, 1294, 725, 233, -150, -413, -556, -591, -540, -431, -290, -145, -16, 83, 146, 174, 170, 144, 106, 63, 23, -7, -28, -39, -40, -35, -27, -17, -8, -1, 3, 5, 5, 5, 3, 2, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0
}; //2.8kbih

fract16 np_6000hz[101] = {
0, 0, 0, 0, 0, 0, 0, 0, 0, -1, -2, -3, -3, -1, 1, 6, 12, 18, 22, 21, 13, -1, -24, -50, -74, -88, -86, -61, -11, 59, 142, 218, 268, 271, 210, 80, -110, -335, -554, -716, -764, -652, -347, 156, 836, 1637, 2482, 3278, 3929, 4356, 4505, 4356, 3929, 3278, 2482, 1637, 836, 156, -347, -652, -764, -716, -554, -335, -110, 80, 210, 271, 268, 218, 142, 59, -11, -61, -86, -88, -74, -50, -24, -1, 13, 21, 22, 18, 12, 6, 1, -1, -3, -3, -2, -1, 0, 0, 0, 0, 0, 0, 0, 0, 0
}; //3.3kbih


fract16 vp_1000hz[Nh] = {
5, 3, 0, -3, -8, -13, -19, -26, -34, -43, -53, -63, -75, -87, -101, -115, -131, -147, -164, -182, -200, -219, -239, -260, -281, -302, -324, -346, -368, -390, -412, -433, -455, -476, -497, -517, -536, -554, -572, -589, -604, -618, -631, -643, -653, -662, -669, -675, -679, -681, 32078, -681, -679, -675, -669, -662, -653, -643, -631, -618, -604, -589, -572, -554, -536, -517, -497, -476, -455, -433, -412, -390, -368, -346, -324, -302, -281, -260, -239, -219, -200, -182, -164, -147, -131, -115, -101, -87, -75, -63, -53, -43, -34, -26, -19, -13, -8, -3, 0, 3, 5
}; //kaiser beta 3

fract16 vp_2000hz[Nh] = {
36, 43, 50, 56, 63, 69, 76, 81, 86, 89, 91, 92, 91, 89, 84, 76, 67, 54, 39, 21, 0, -24, -51, -81, -115, -151, -190, -231, -275, -321, -369, -418, -468, -519, -570, -621, -671, -721, -769, -815, -859, -900, -938, -972, -1002, -1029, -1051, -1068, -1081, -1088, 31643, -1088, -1081, -1068, -1051, -1029, -1002, -972, -938, -900, -859, -815, -769, -721, -671, -621, -570, -519, -468, -418, -369, -321, -275, -231, -190, -151, -115, -81, -51, -24, 0, 21, 39, 54, 67, 76, 84, 89, 91, 92, 91, 89, 86, 81, 76, 69, 63, 56, 50, 43, 36
}; //kaiser beta 3

fract16 vp_3000hz[Nh] = {
0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 2, 3, 5, 6, 9, 11, 14, 17, 20, 23, 25, 26, 26, 24, 19, 11, 0, -16, -38, -66, -100, -142, -190, -246, -309, -379, -455, -537, -622, -710, -799, -888, -974, -1056, -1131, -1199, -1256, -1303, -1337, -1358, 31402, -1358, -1337, -1303, -1256, -1199, -1131, -1056, -974, -888, -799, -710, -622, -537, -455, -379, -309, -246, -190, -142, -100, -66, -38, -16, 0, 11, 19, 24, 26, 26, 25, 23, 20, 17, 14, 11, 9, 6, 5, 3, 2, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0
}; //bh

fract16 vp_4000hz[Nh] = {
0, 0, 0, 0, 0, 0, 0, 0, 0, -1, -2, -3, -4, -6, -9, -11, -12, -14, -13, -11, -6, 0, 12, 27, 47, 70, 95, 122, 148, 170, 185, 191, 183, 157, 110, 40, -55, -177, -325, -496, -688, -895, -1111, -1329, -1540, -1737, -1911, -2055, -2163, -2230, 30515, -2230, -2163, -2055, -1911, -1737, -1540, -1329, -1111, -895, -688, -496, -325, -177, -55, 40, 110, 157, 183, 191, 185, 170, 148, 122, 95, 70, 47, 27, 12, 0, -6, -11, -13, -14, -12, -11, -9, -6, -4, -3, -2, -1, 0, 0, 0, 0, 0, 0, 0, 0, 0
};//bh

fract16 vp_5000hz[Nh] = {
0, 0, 0, 0, 0, 0, 1, 2, 3, 4, 5, 5, 5, 3, 0, -5, -12, -21, -31, -42, -52, -60, -63, -61, -50, -30, 0, 40, 90, 146, 204, 260, 307, 337, 343, 317, 255, 149, 0, -194, -430, -701, -999, -1310, -1623, -1921, -2190, -2416, -2587, -2694, 30037, -2694, -2587, -2416, -2190, -1921, -1623, -1310, -999, -701, -430, -194, 0, 149, 255, 317, 343, 337, 307, 260, 204, 146, 90, 40, 0, -30, -50, -61, -63, -60, -52, -42, -31, -21, -12, -5, 0, 3, 5, 5, 5, 4, 3, 2, 1, 0, 0, 0, 0, 0, 0
};//kaiser beta 10 2000

fract16 vp_6000hz[Nh] = {
0, 0, 0, 0, 0, 0, 0, 0, -1, -3, -6, -8, -9, -8, -6, 0, 8, 19, 31, 43, 51, 53, 44, 25, -6, -48, -96, -143, -180, -198, -187, -142, -59, 56, 198, 347, 482, 578, 608, 548, 380, 97, -298, -790, -1349, -1937, -2507, -3014, -3412, -3667, 29013, -3667, -3412, -3014, -2507, -1937, -1349, -790, -298, 97, 380, 548, 608, 578, 482, 347, 198, 56, -59, -142, -187, -198, -180, -143, -96, -48, -6, 25, 44, 53, 51, 43, 31, 19, 8, 0, -6, -8, -9, -8, -6, -3, -1, 0, 0, 0, 0, 0, 0, 0, 0
}; //kaiser beta 10 2750




//KRAJ FILTERA

//SPORT0 STARI KOD


volatile short postavkeCodec1836[11] =
{0x0000, 0x1000, 0x23ff, 0x33ff, 0x43ff, 0x53ff, 0x6000, 0x7000, 0xc000, 0xd000, 0xe000};

volatile int primljeno[4];
volatile int zaSlanje[4];

fract16 primljeno_f16[4];
fract16 zaSlanje_f16[4];

void inicijalizacija_DMA1_DMA2(void);
void ukljuci_DMA1_DMA2(void);
void inicijalizacija_prekida(void);
EX_INTERRUPT_HANDLER(Sport0_RX_prekid);


void Init1836(void){
	int i;
	int j;
	//static unsigned char ucActive_LED = 0x01;
	//*pFlashA_PortA_Data = 0x00;
	//*pFlashA_PortA_Data = ucActive_LED;
	for (i=0; i<0xf0000; i++) asm("nop;");
	*pSPI_FLG = FLS4;
	*pSPI_BAUD = 16;
	*pSPI_CTL = 3 | SIZE | MSTR;
	
		*pDMA5_CONFIG = WDSIZE_16;
	*pDMA5_START_ADDR = (void *)postavkeCodec1836;
	*pDMA5_X_COUNT = 11;
	*pDMA5_X_MODIFY = 2;
	*pDMA5_CONFIG = (*pDMA5_CONFIG | DMAEN);
	*pSPI_CTL = (*pSPI_CTL | SPE);
	for (j=0; j<0xaff0; j++) asm("nop;");
	*pSPI_CTL = 0x0000;
}



void Inicijalizacija_Sport0(void)
{
	// vjezba 8
	*pSPORT0_RCR1 = RFSR | RCKFE;
	*pSPORT0_RCR2 = 23 | RSFSE;
	*pSPORT0_TCR1 = TFSR | TCKFE;
	*pSPORT0_TCR2 = 23 | TSFSE;
}
void ukljuci_SPORT0(void)
{
	// vjezba 8
	*pSPORT0_TCR1 = *pSPORT0_TCR1 | TSPEN;
	*pSPORT0_RCR1 = *pSPORT0_RCR1 | RSPEN;
}

void inicijalizacija_prekida(void)
{
*pSIC_IAR1 = 0xffffff0f;
register_handler(ik_ivg7, Sport0_RX_prekid);
*pSIC_IMASK = 0x00000200;
/* funkcija register_handler obavlja postavljanje potrebnog
bita u IMASK registru, tako da nije potrebno to posebno raditi
(kao sto smo nepotrebno radili na prosloj vjezbi) */
}
void inicijalizacija_DMA1_DMA2(void)
{
	*pDMA1_CONFIG = WNR | WDSIZE_32 | DI_EN | FLOW_AUTO;
	*pDMA1_START_ADDR = (void *)primljeno;
	*pDMA1_X_COUNT = 4;
	*pDMA1_X_MODIFY = 4;
	*pDMA2_CONFIG = WDSIZE_32 | FLOW_AUTO;
	*pDMA2_START_ADDR = (void *)zaSlanje;
	*pDMA2_X_COUNT = 4;
	*pDMA2_X_MODIFY = 4;
}

void ukljuci_DMA1_DMA2(void)
{
	*pDMA2_CONFIG = (*pDMA2_CONFIG | DMAEN);
	*pDMA1_CONFIG = (*pDMA1_CONFIG | DMAEN);
}


/* Ovo je funkcija koja obradjuje prekid nastao na kanalu DMA1 (prijem
na SPORT-u 0) */

EX_INTERRUPT_HANDLER(Sport0_RX_prekid)
{
		// obrada prekida
	*pDMA1_IRQ_STATUS = 0x0001;
	
	primljeno_f16[0] = (fract16)(primljeno[0] >> 8);
	primljeno_f16[1] = (fract16)(primljeno[1] >> 8);
	primljeno_f16[2] = (fract16)(primljeno[2] >> 8);
	primljeno_f16[3] = (fract16)(primljeno[3] >> 8);
	
	fir_fr16(primljeno_f16, zaSlanje_f16, 4, &my_filter_state);
	
	

	zaSlanje[0] = ((int) zaSlanje_f16[0] << 8);
	zaSlanje[1] = ((int) zaSlanje_f16[1] << 8);
	zaSlanje[2] = ((int) zaSlanje_f16[2] << 8);
	zaSlanje[3] = ((int) zaSlanje_f16[3] << 8);

	
	/*zaSlanje[0] = ((int) primljeno[0] );
	zaSlanje[1] = ((int) primljeno[1] );
	zaSlanje[2] = ((int) primljeno[2] );
	zaSlanje[3] = ((int) primljeno[3] );*/
}

//KRAJ STAROG KODA

EX_INTERRUPT_HANDLER(Button_Interrupt);
#define pFlashA_PortB_Dir  (volatile unsigned char *)0x20270007
#define pFlashA_PortB_Data (volatile unsigned char *)0x20270005  // PF6 (0b0000000001000000)

int buttonsw = 0;
int freq_up = 0;

void Init_Flash(void);
void Init_EBIU(void);
void init_button(void);
void init_prekid(void);

int Freq = 2;
int Pomak = 1;
int bits = 0x00;

void main(void)
{
	
    // Configure the system
    sysreg_write(reg_SYSCFG, 0x32);
    Init_EBIU();
    Init_Flash();
    Init1836();
    inicijalizacija_prekida();
	inicijalizacija_DMA1_DMA2();
	Init1836();
	Inicijalizacija_Sport0();
	ukljuci_DMA1_DMA2();
	ukljuci_SPORT0();
	init_prekid();
	init_button();
	
	fir_init (my_filter_state, np_2000hz, delay, Nh, 0);
	
	
    

    while(1){}
    
}

void Init_Flash()
{
    // Initialize flash memory direction (verify the correct value)
    *pFlashA_PortB_Dir = 0x3f;
}

void Init_EBIU()
{
    // Initialize the EBIU (verify the correct values)
    *pEBIU_AMBCTL0 = 0x7bb07bb0;
    *pEBIU_AMBCTL1 = 0x7bb07bb0;
    *pEBIU_AMGCTL  = 0x000f;
}

void init_button()
{
	//buttonsw PF11
	*pFIO_DIR = 0xF1FF;
	*pFIO_MASKA_D = 0x0800;
	*pFIO_INEN = 0xFFFF;
	*pFIO_POLAR = 0x0000;
	*pFIO_EDGE = 0x0800;
	
	//PF 10  povecavanje f i PF9 za smanjivanje
	
	*pFIO_MASKA_D |= 0x0E00;
	*pFIO_EDGE |= 0x0E00;
	
	
	*pFlashA_PortB_Data = 0x02;
	
}

void init_prekid()
{
	*pSIC_IAR1 = 0xffffff0f;
	*pSIC_IMASK = 0xFFFFFFFF;
	//register_handler(ik_ivg7, Sport0_RX_Prekid);
	
	*pSIC_IAR2 = 0x66622444;
	register_handler(ik_ivg9, Button_Interrupt);
}

EX_INTERRUPT_HANDLER(Button_Interrupt)
{
	
	if(*pFIO_FLAG_D == 0x0800)
	{
		
		switch(buttonsw)
		{
			case 0:
				*pFlashA_PortB_Data |= 0x20;
				buttonsw = 1; 
				break;
			
			case 1:
				*pFlashA_PortB_Data &= ~0x20;
				buttonsw = 0;
				break;
			default: break;
		}
		*pFIO_FLAG_C = 0x0800;
	}
	
	if(*pFIO_FLAG_D == 0x0400)
	{
		
		bits = *pFlashA_PortB_Data;
		/*switch(freq_up)
		{
			case 0:
				*pFlashA_PortB_Data |= 0x01;
				freq_up = 1; 
				break;
			
			case 1:
				*pFlashA_PortB_Data &= ~0x01;
				freq_up = 0;
				break;
			default: break;
		}
		*pFIO_FLAG_C = 0x0400;*/
		
		if(Freq < 6)
		{
			*pFlashA_PortB_Data &= ~bits;
			bits = bits + 0x01;
			*pFlashA_PortB_Data |= bits;
			Freq+=Pomak;
		}
		
		*pFIO_FLAG_C = 0x0400;
	}
	
	if(*pFIO_FLAG_D == 0x0200)
	{
		
		bits = *pFlashA_PortB_Data;
		/*switch(freq_up)
		{
			case 0:
				*pFlashA_PortB_Data |= 0x01;
				freq_up = 1; 
				break;
			
			case 1:
				*pFlashA_PortB_Data &= ~0x01;
				freq_up = 0;
				break;
			default: break;
		}
		*pFIO_FLAG_C = 0x0400;*/
		
		if(Freq > 1)
		{
			*pFlashA_PortB_Data &= ~bits;
			bits = bits - 0x01;
			*pFlashA_PortB_Data |= bits;
			Freq-=Pomak;
		}
		
		*pFIO_FLAG_C = 0x0200;
	}
	
		switch(Freq)
		{
			case 1:
				if(buttonsw == 0){
					fir_init (my_filter_state, np_1000hz, delay, Nh, 0);
				}
				else{
					fir_init (my_filter_state, vp_1000hz, delay, Nh, 0);	
				}
				break;
			
			case 2:
				if(buttonsw == 0){
					fir_init (my_filter_state, np_2000hz, delay, Nh, 0);
				}
				else{
					fir_init (my_filter_state, vp_2000hz, delay, Nh, 0);	
				}
				break;
				
			case 3:
				if(buttonsw == 0){
					fir_init (my_filter_state, np_3000hz, delay, Nh, 0);
				}
				else{
					fir_init (my_filter_state, vp_3000hz, delay, Nh, 0);	
				}
				break;
			
			case 4:
				if(buttonsw == 0){
					fir_init (my_filter_state, np_4000hz, delay, Nh, 0);
				}
				else{
					fir_init (my_filter_state, vp_4000hz, delay, Nh, 0);
				}
				break;
				
			case 5:
				if(buttonsw == 0){
					fir_init (my_filter_state, np_5000hz, delay, Nh, 0);
				}
				else{
					fir_init (my_filter_state, vp_5000hz, delay, Nh, 0);	
				}
				break;
				
			case 6:
				if(buttonsw == 0){
					fir_init (my_filter_state, np_6000hz, delay, Nh, 0);
				}
				else{
					fir_init (my_filter_state, vp_6000hz, delay, Nh, 0);
				}
				break;
			default: break;
		}
	
}
