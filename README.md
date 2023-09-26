# i2c_io_board
the I2C IO board is designed to provide software independent 8ch IN + 8ch out hardware for 5 to 40V DC load switching.
You can think of it as digital version of a relais board.

Short description
- what its for
- default design 5-24V
- maximum ratings

# HW description
## connectors
### PI connection
### I2C connector
### OneWire connector
### RS485 connector

## power supply
the supply voltage is provided by screw terminal U14. It is limited by a 5x20 fuse (max 10A) and the supplies the DCDC converter and the output stages
### Fuse and crowbar
The input is using a so called "crowbar" configuration. After the fuse there is a diode (D25) in blocking direction between input and GND. If the input voltage is connected with wrong polarity this diode will conduct, short circuit the input voltage and blow the fuse.
![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/e7af824d-4be5-4e5e-b579-71d84d57db8b)

### 5V DCDC converter
the DCDC converter is providing 5V (up to 3A) to power the raspberryPI. This is done by LMR33630 (U3). The design of the circuit is done mostly by TI's webbench tool. Input and output capacitors were increased a bit. Also there is a elko foreseen on the input side but not placed by default. This could be necessary with higher impedance sources.
![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/44d234a6-0c98-4f36-afb8-aa0dba3ddea5)

### 3.3V LDO
3.3V LDO is providing power (VCC) to the TCA9535 (powering output stage drivers + LEDs) and input stage LEDs. Also RS485 driver IC and I2C isolator are powered by VCC.
![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/3d7f291f-130a-4452-b777-6a9e19991fbc)
The LDO used (AMS1117) is nothing special in any way. 
A "power good" LED (LED19) is provided to give visible feedback.

## I2C Bus
### i2c adresses/registers
#### bus pullups
#### bus voltage supply
#### GND isolation 
### I2C Bus isolator 

## RS485
### Termination
 
## output channels
the output channels are designed as PMOS (high side switched) stages. As main switch the PMOS SI4401 is used in a SO8 package. The PMOS is controlled by a NMOS driver stage which in turn is driven by the output of the TCA9535

![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/f72a22b6-0c97-4ae1-991f-f4bb4c2ae579)

The output of the TCA9535 (-> represented by V2) is directly driving the LED (D3) and is controlling the gate of M2. R5 is limiting the recharge current to about 50mA. R3 is a pulldown to provide a definded low level in off state.
M2 is controlling the driving stage for M1. The driving stage has to cope with variable input voltage (V1, 7-36V). For input voltages below 12V D2 is inactive. R1 and R2 form a voltage divider pulling the base of M1 very close to GND when M2 is active.
For input voltages higher then 12V D2 (= Zener diode) starts conducting. R1 becomes inactive. Therefore the base of M1 is clamped at 12V below V1 (this protects the base as more then 20V will destroy the PMOS)
Finally D1 forms a "free wheeling" diode for inductive loads. This is another measure to protect M1 when switching inductive loads like pumps.

## input channels
the input channels operate in the voltage range of 5V - 40V. Each channel is designed as a current driven optocoppler diode set for about 4.5 mA.
Designed in reference to https://electronics.stackexchange.com/questions/441277/optocoupler-circuit-accept-input-between-3-50-volt

![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/24c5da66-366c-436f-954b-165b17396760)

### current source LED driver
the led of the input optocoppler is driven by a npn transistor current source to support variable input voltage. Q2 is changing resistance according to input voltage to maintain diode current at 4.5mA.
Current is set by R1. Current will regulate so that V_R1 = 0.6V ( =Vbe of Q1)

#### 5V low voltage input case
R2 has to source enough current to supply Q2 base. Question: what hFe is needed?
```
V_Q2_base = 1.2V
V_R2 = 5V - 1.2V = 3.8V  --> Ir2 = 3.8V / 20kohm = 0.19mA
assuming Q1 is fully closed(no leakage current) required hFe of Q2 is 4.5mA / 0.19mA = 24

[3V case: Ir2 = (3V-1.2V) /20kohm = 0.09mA -> needed hFe > 50]
```
With the worst binning hFe = 100 we are safe on that side. Also there is room for lower input voltage. 3V  input requires nFe of 50, so it should also work (untested/unsafe!)

#### 40V high voltage input case
In this operation mode the voltage rating(Q2) and the power dissipation(Q2+R2) is the critical factor. There are two current paths to account for:
```
R2 -> Q1:
V_Q1_CE is 1.2V so all voltage drops across R2: U_R2 = U_supply - 1.2V = 34.8V
P_R2 = U * I = U^2 / R = (34.8V)^2 / 20kohm = 61mW    (58.8V => 173mW).

Max dissipation of 0805 is 125mW. To be on the save side (and leave room for more;) the 20k are placed as 2x10k.

LED -> Q2 -> R1:
U_LED = 2V ; U_R1 = 0.6V
P_Q2 = (40V-2.6V) * 4mA = 150mW      (leaves room up to 70V =>270mW)
```
#### 40V reverse polarity
for protection in this error state a scottky diode is placed in the input path. so any reverse voltage can be blocked because the optocoppler diode has a reverse breakdown voltage of -6V

### optocoupler digital input stage
the phototransistor of U1 is supplied by the 3.3V system voltage. R5 acts as a current limiter(<49mA) to protect U1. R3 acts as a pulldown. Signal for digital input stage is taken from phototransistor output (= top of R3). This also drives M1's base (NMOS) to turn on a LED(~9.5mA) on activated input.

#### max frequency
the propagation delay of the input stage lays in the range of 8us (rise) 
![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/e3de92a7-b995-49b4-a995-59a59334d5b3)
and about 60us in case of falling edge:
![grafik](https://github.com/unddef/i2c_io_board/assets/27676292/2f7c6752-4496-487a-a1c9-ee845cab159b)

In an practical application he maximum sampling capability is limited by the polling frequency on the I2C bus. As by default there is no interrupt when an input changes, the change is only detected after the next polling of input register by the uC. But TCA9535 provieds an interrupt pin which is exposed on the board as a one pin header to provide access for further improvement.
