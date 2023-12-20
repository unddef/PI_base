# PI_base
PIbase is designed as "base board" for an RaspberryPI. This means a Pi can be mounted ontop and is connected to the base.
This extension provides a bunch of functionality to interconnect with real world applications: power supply, input and output channels, bus connectors etc.

# feature list
- wide input supply voltage 7-36V
  - input fuse onboard
  -  can run from 12V or 24V battery, solar systems, 24V industrial supply, old notebook PSUs etc
- 5V/3A DCDC converter to  power the PI from input voltage
- 8 channel OUTPUT
  - max 4A per channel, 10A max in total
  - PMOS switching (true "high-side") - no contact weardown as with relais
- 8 channel INPUT
  - 5-40V input voltage
  - inputs with diode character (optocopler diode input) -> 4-5mA enable current required for improved noise immunity
  - option to provide two groups of true galvanic isolated inputs (2 groups of 4 inputs with islolated common ground)
- I2C bus
  - I2C bus on screw terminals
  - I2C bus isolator to protect PI
  - bus voltage selectable by jumper (3.3V/5V)
  - bus pullups selectable by jumper
- 1Wire bus on screw terminals
- RS485 bus on screw terminals
  - SP3485 half duplex transceiver
  - hardware "transmit enable" circuit to eliminate the need of RX/TX direction control by GPIO pin
  - TX + RX activity LED


- 
# hardwre description
a detailed documentation of the circuit can be found here:

# example setups

Short description
- what its for
- default design 5-24V
- maximum ratings

