java cSchool of Engineering
1 Digital Communications 4/M: Bit Errors and Parity Check ing
1.1 Introduction
This laboratory project will introduce you to some issues that occur in digital communication
channels. In particular we will study the effects of additive white gaussian noise on the com munication channel and how its effects can be mitigated using a simple parity checking and
Automatic Repeat-reQuest (ARQ). In your previous laboratories on this course you will have
studied modulation formats. Here we will circumvent some of the low-level coding by using the
komm Python library https://pypi.org/project/komm/ to provide the appropriate func tionality. If you are using your own computer, make sure the Python libraries scipy, numpy,
matplotlib and pillow as well as komm version 0.16.1 or later are installed, and your python
version is 3.10 or later. It is recommended that you use a suitable IDE for your project, such as
Spyder.
Each project will be scheduled over a two week period, within which there will be 2 scheduled
online consultation sessions where you will be able to ask teaching staff for guidance. The project
should be written up as a short report describing what you’ve done and the results you have
taken along with any conclusions that you draw. Include your python code(s) in the appendices.
Make sure your name and student ID number is on the report. The report should be uploaded to
the Moodle assignment by the stated deadline, either using Moodle’s inbuilt html editor, or as a
single PDF file.
1.2 Obtaining Digital Data
A number of 8 bit depth grayscale images of various sizes have been provided for you to use
in this laboratory project. You may also consider the use of the numpy.random.randint()
command to create random binary streams for testing purposes. As the runtime depends on the
size of the data, you should generally use the smallest data set first, although in terms of accuracy
it may be advisable to use the larger datasets where the smallest images result in a small number
of bit errors, say Preferences>IPython console>Graphics>Backend
from Inline to Automatic. The unpackbits() command converts the integer values into a
flattened (1D) binary array.
Figure 1: example 150×200 and 250×376 grayscale images
import numpy as np
from PIL import Image
from matplotlib import pyplot as plt
import komm
tx_im = Image.open("Lord Kelvin’s house 011.pgm")
Npixels = tx_im.size[1]*tx_im.size[0]
plt.figure()
plt.imshow(np.array(tx_im),cmap="gray",vmin=0,vmax=255)
plt.show()
tx_bin = np.unpackbits(np.array(tx_im))
1.3 Noisy Channel Simulation
Now we turn to the channel simulation which consists of 3 parts, namely (1) the channel coding,
(2) the transmission and reception over a noisy channel, and (3) the channel decoding. We shall
start with binary phase-shift keying (BPSK) which has two symbols. We will examine other
keying schemes with more symbols later.
The komm library provides functions for PSK modulation and demodulation and for simu-
lating an additive white gaussian noise source, as shown below. The modulated psk waveform
is by default unit average power. The scalar snr specifies the (linear) signal-to-noise ratio for
the channel, and the example below corresponds to 6 dB. Note that as of komm version 0.9.1
the komm.AWGNChannel requires signal_power to be defined. tx_data and rx_data will
consist of complex arrays.
psk = komm.PSKModulation(2)
awgn = komm.AWGNChannel(signal_power=1.0, snr=10**(6./10.))
tx_data = psk.modulate(tx_bin)
rx_data = awgn(tx_data)
rx_bin = psk.demodulate_hard(rx_data)
In Spyder, use the variable explorer and write down the value of Npixels, and the data type
and sizes of tx_bin, tx_data, rx_data and rx_bin. Check that the data types are correct,
and the comparison of the sizes of tx_data, rx_data to tx_bin, rx_bin corresponds to the
number of bits per symbol for the modulation scheme used.
1.4 Measuring Bit Error Ratio
Here we will investigate the influence of noise on the received data. An I-Q constellation
diagram can be displayed and inspected visually using the following where just the first 10000
elements from rx_data is used so that the detail of the constellation is not obscured.
plt.figure()
plt.axes().set_aspect("equal")
plt.scatter(rx_data[:10000].real,rx_data[:10000].imag,s=1,marker=".")
plt.show()
You can also visually inspect the recovered image by rearranging the received data into a
decimal valued matrix corresponding to the original image dimensions, and using imshow as
you did previously.
rx_im = np.packbits(rx_bin).reshape(tx_im.size[1],tx_im.size[0])
To determine the number of bit errors a bitwise comparison of the transmitted and received
binary matrices should be made, summing the case of unequal elem代 写program、Python
代做程序编程语言ents. The bit error ratio (ber)
is obtained by dividing the number of errors by the total number of bits, 8*Npixels. Calculate
the ber for a range of dB values for the signal-to-noise ratio and plot them as 𝑏𝑒𝑟 (logarithmic
axis) vs 𝑠𝑛𝑟 in dB. The following python code excerpt can be used to plot the data contained in
x and y arrays (same length) with a logarithmic vertical axis.
plt.figure()
plt.scatter(x, y) #plot points
plt.plot(x, y) #plot lines
plt.yscale("log")
plt.grid(True)
plt.show()
Include a curve showing the theoretical dependence for BPSK (or QPSK),
1
2
erfc√︂
10𝑠𝑛𝑟(dB)/10
𝑘
where erfc is the complementary error function (available in the python scipy library, i.e.
scipy.special.erfc(x)), 𝑠𝑛𝑟 is the signal to noise in dB and 𝑘 is the number of bits per
symbol (𝑘 = 1 for BPSK).
1.5 Parity Check
In this task consider the data as consisting Npixels× 8-bit words and replace the least significant
bit of each 8-bit word with a parity bit, which doesn’t make any discernable difference to your
view of the grayscale image. You are free to select whether you use even parity or odd parity:
setting the 8th bit to the modulo 2 (%2 in python) sum of the previous 7 bits corresponds to even
parity.
The parity test of the received data is done by looking at the modulo 2 sum of each 8-bit
word. If this is different from the parity setting you should do an Automatic Repeat-reQuest
(ARQ) and resend the word. Therefore you should amend your code to simulate the transmission
of an 8-bit word at a time as a step within a loop. Place a counter in your code to add up the
total number of ARQs.
Visually inspect the received image and determine the bit error ratio. On a single graph, plot
the 𝑏𝑒𝑟 as a function of 𝑠𝑛𝑟 in dB as before, along with the theoretical curve for uncorrected
𝑏𝑒𝑟 and the ratio of the total number of ARQs to the number of bits. What can you conclude
from this graph?
Once you have obtained satisfactory results for BPSK format, repeat the exercise for Quadra ture Phase Shift Keying (QPSK) by setting
psk = komm.PSKModulation(4,phase_offset=np.pi/4)
Check that psk.bits_per_symbol is commensurate with your data array sizes. Note also that
the komm implementation uses gray coding by default (so that symbol errors will give rise to
mostly single-bit errors), which you can verify by inspecting psk.labeling.
1.6 4-QAM and 16-QAM
Now adapt your code to undertake similar simulations with square QAM: 4-QAM (identical to
QPSK so verify that this matches the previous exercise). Again, the komm implementation uses
gray coding by default, which you can verify by inspecting psk.labeling. The following code
excerpts demonstrates the implementation of 4-QAM.
qam = komm.QAModulation(4,base_amplitudes=1./np.sqrt(2.))
print(qam.energy_per_symbol) # this should be unity
tx_data = qam.modulate(...)
Check that qam.bits_per_symbol is commensurate with your data array sizes. base_amplitudes
is unity by default and correspondingly the closest points to the origin of the I-Q constellation
are (±1, ±1). However, to draw appropriate comparisons with PSK we should be consis tent with the average power per symbol (unity by default with PSK). Therefore the value for
base_amplitudes needs to be renormalised, as shown above for 4-QAM. One method you can
use to do this is to inspect the value qam.energy_per_symbol and then set base_amplitudes
to a value such that qam.energy_per_symbol becomes unity.
Repeat the exercise for 16-QAM.
1.7 Higher order QAM - Mandatory for ENG5336/ Optional for ENG4052
Adapt your code to extend your study to 64-QAM (you may need to add dummy bits so that the
total bits is a multiple of 6) and 256-QAM, remembering to appropriately renormalise the value
for base_amplitudes in each case. You will need to use significantly higher signal-to-noise
ratios than previously. Identify the approximate increase in signal-to-noise ratio to achieve
equivalent 𝑏𝑒𝑟 compared to 4-QAM and 16-QAM. Examine the constellation plots and note any
significant pattern; what is your explanation for this pattern?
1.8 Documentation
Getting the python libraries
If you are using your own computer, make sure the Python libraries scipy, numpy, matplotlib
and pillow are installed. These libraries are installed by default with the Anaconda python
distribution. It is recommended that you use a suitable IDE for your project, such as Spyder. The
komm Python library, which requires python3.10 or newer, is available from PyPI repository
and, if required, can be installed using pip. From a python-activated command line (such as
Anaconda prompt), use the following command to install in your user App config
pip install komm --user

         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
