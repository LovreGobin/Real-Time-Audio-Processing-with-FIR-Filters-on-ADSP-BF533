**VisualDSP++ Real-Time Audio Processing**<br>
Faculty of Electrical Engineering, Mechanical Engineering and Naval Architecture
Authors: Mateo Franjić, Lovre Gobin, Marko Grubišić

**Overview**<br>
This project implements real-time audio processing using FIR (Finite Impulse Response) filters on the ADSP-BF533 DSP processor, running on an EZ-KIT Lite development board. The system filters an incoming analog audio signal through an AD1836 codec, applying selectable low-pass and high-pass filters with configurable cutoff frequencies.

**Hardware**<br>
  DSP: Analog Devices ADSP-BF533 (Blackfin)
  Board: EZ-KIT Lite
  Codec: AD1836 (analog-to-digital and digital-to-analog conversion)
  Interface: SPORT0, DMA1/DMA2 for data transfer


<img width="1383" height="778" alt="image" src="https://github.com/user-attachments/assets/b4688dfd-909e-45a3-9bcb-e082dbd7ad72" /><br>

**Tools & Software**<br>
ToolPurposeVisualDSP++Main IDE for DSP programming and deploymentMATLAB Filter DesignerFIR coefficient generationWaveform GeneratorTest signal generation (sinusoidal)AudacityOutput signal analysis (time domain & frequency spectrum)

**FIR Filter Design**<br>
Coefficients were generated using MATLAB's Filter Designer extension:
  Filter type: FIR (low-pass and high-pass)
  Number of coefficients: 101
  Sampling frequency: 48 kHz
  Available filters: 6 low-pass and 6 high-pass, with cutoff frequencies ranging from 1 kHz to 6 kHz

<img width="844" height="534" alt="image" src="https://github.com/user-attachments/assets/96c26bd6-27c7-408f-a4bd-011147223bec" /><br>



**Implementation**<br>

**Initialization**<br>
On startup, the system initializes the following peripherals and modules:
  EBIU and Flash memory
  AD1836 codec
  SPORT0 communication port
  DMA1 and DMA2 channels for audio data transfer
  Button interrupt handlers for real-time filter switching

<img width="786" height="605" alt="image" src="https://github.com/user-attachments/assets/45c799cf-b7be-4a0e-92e7-1a5dfee02c9b" /><br>


**Filtering*<br>
Audio samples received via DMA are converted to fract16 format and processed using the fir_fr16() function. The filtered output is written back to the output buffer for playback through the codec.

<img width="645" height="540" alt="image" src="https://github.com/user-attachments/assets/67f43155-b964-408a-a88f-621c5538b985" /><br>

<img width="414" height="210" alt="image" src="https://github.com/user-attachments/assets/23d5e1f5-dd2a-4f3e-8ec8-09606b9a2aa0" /><br>

<img width="314" height="25" alt="image" src="https://github.com/user-attachments/assets/176f3267-8538-416e-b456-5e417ae0e7f1" /><br>

<img width="352" height="863" alt="image" src="https://github.com/user-attachments/assets/86c2b2b0-9d8b-4d17-86a2-8cfb9fa6759f" /><br>

<img width="551" height="539" alt="image" src="https://github.com/user-attachments/assets/2c99a3f6-956b-48eb-8cf7-e5ab11a29b8c" /><br>

<img width="578" height="574" alt="image" src="https://github.com/user-attachments/assets/7085c2fe-0909-4ae8-abb7-0009f169f9cc" /><br>


**Filter Selection (Buttons)**<br>

The active filter can be changed in real time using the physical buttons on the board. PF8 and PF11 are used to toggle the filter type between low-pass and high-pass, while PF10 and PF9 are used to increase or decrease the cutoff frequency, respectively.

**Testing**<br>

<img width="781" height="422" alt="image" src="https://github.com/user-attachments/assets/e2dfe8cb-77b2-492a-8e83-8d6963c1593e" /><br>
<img width="477" height="403" alt="image" src="https://github.com/user-attachments/assets/ea4e2322-0a3f-4bb8-926e-3a0d59b57c0f" /><br>

Test Signals<br>

1500 Hz sinusoidal signal
2500 Hz sinusoidal signal
White noise (for frequency response measurement)

<img width="610" height="469" alt="image" src="https://github.com/user-attachments/assets/24b9a91b-f467-44e9-82ef-1ba7fb0f2099" /><br>
<img width="602" height="469" alt="image" src="https://github.com/user-attachments/assets/38b29314-1bb8-4ec0-a12e-20fcb403f52a" /><br>



**Results**<br>

A 1500 Hz sinusoidal test signal was run through several filter configurations. 

<img width="934" height="288" alt="image" src="https://github.com/user-attachments/assets/9e33de9b-9918-4f6f-95be-df05f0588d8c" /><br>

When passed through a low-pass filter with a 2 kHz cutoff, the signal came through nearly unchanged, as expected since it falls well below the cutoff. 

<img width="934" height="427" alt="image" src="https://github.com/user-attachments/assets/fe2e4db5-bf70-4e25-a72e-cce28a8aa32d" /><br>

Switching to a high-pass filter with a 4 kHz cutoff attenuated the signal noticeably, and raising the cutoff to 6 kHz attenuated it significantly — both consistent with the 1500 Hz input being below the respective cutoff frequencies.

<img width="934" height="575" alt="image" src="https://github.com/user-attachments/assets/4016d2d7-2490-4762-9476-ccd8b59eb7d1" /><br>

Frequency response measurements confirmed expected filter behavior. The low-pass filter showed a clean characteristic, while the high-pass filter exhibited a slightly less ideal rolloff.

**Conclusion**<br>
The project successfully demonstrates real-time FIR filtering on an embedded DSP platform. Both low-pass and high-pass filters perform as expected, with minor deviations attributable to the constraints of real-time processing on the ADSP-BF533.
