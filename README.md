# i2c_io_board
the I2C IO board is designed to provide software independent 8ch IN + 8ch out hardware for 5 to 40V DC load switching.
You can think of it as digital version of a relais board.

Short description
- what its for
- default design 5-24V
- maximum ratings

# HW description
## power supply
### Fuse and crowbar
### 5V DCDC converter
### 3.3V LDO

## I2C Bus
### i2c adresses/registers
### PI I2C connection
### external connector
#### bus pullups
#### bus voltage supply
#### GND isolation 
### I2C Bus isolator 

## RS485
### Termination
## 
## output channels

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
assuming Q1 is fully closed(no leakage current) required hFe of Q2 is 4mA / 0.19mA = 21
```
With the worst binning hFe = 100 we are safe on that side. Also there is room for lower input voltage. 3V  input requires nFe of 44, so it should also work (untested/unsafe!)

#### 40V high voltage input case
In this operation mode the voltage rating(Q2) and the power dissipation(Q2+R2) is the critical factor. There are two current paths to account for:
```
R2 -> Q1:
V_Q1_CE is 1.2V so all voltage drops across R2: U_R2 = U_supply - 1.2V = 38.8V
P_R2 = U * I = U^2 / R = (38.8V)^2 / 20kohm = 75mW    (58.8V => 173mW).

Max dissipation of 0805 is 125mW. To be on the save side the 20k are placed as 2x10k.

LED -> Q2 -> R1:
U_LED = 2V ; U_R1 = 0.6V
P_Q2 = (40V-2.6V) * 4mA = 150mW      (leaves room up to 70V =>270mW)
```
#### 40V reverse polarity
for protection in this error state a scottky diode is placed in the input path. so any reverse voltage can be blocked because the optocoppler diode has a reverse breakdown voltage of -6V

### optocoupler digital input stage
the phototransistor of U1 is supplied by the 3.3V system voltage. R5 acts as a current limiter(<49mA) to protect U1. R3 acts as a pulldown. Signal for digital input stage is taken from phototransistor output (= top of R3). This also drives M1's base (NMOS) to turn on a LED(~9.5mA) on activated input.

