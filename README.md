# Fiat_500e_Cabin_Heater
Info and code (eventually) on controlling a Fiat 500e cabin heater for use in electric car conversions

The 2013-2019 Fiat 500e uses a resistive air heater driven off the ~400V (97s) HV battery pack (P/N 68140376AH).  It's capable 
of 5 kW at full output.  Internally, it is PWM driven by a microcontroller driving 3 IGBTs.  There are 5 connections to the 
cabin heater: HV+ (orange), HV- (orange), +12V (red/white), LIN (white/dark blue) and GND (black).  The 3 low voltage 
connections are made at one connector with 20 AWG wiring and the HV connections are done on a separate connector.  +12V power 
is fused at 5A and is not switched (always on).  The HV wiring is fused at 30A.

The LIN 2.0 bus is a single wire serial interface and communicates with the body control module (BCM).  The BCM calculates the 
target PWM duty cycle based on fan speed, commanded temperature, etc. and sends this data over the LIN line to the heater at 
19.2 kbps.  The BCM is the master and the heater is the slave.

The heater has 3 internal temperature sensors (IGBT temperature, core sensor A and core sensor B) as well as HV voltage and HV 
current sensing.  It appears that some or all of this data is available to the BCM on the LIN bus.
The BCM sends a signal on the LIN bus every 50 ms, alternating between PID 1A and 99.  Both of these frames use enhanced 
checksum per LIN 2.0.  The PID 1A is a request for data and looks something like this:

Frame ID      Data1   Data2    Data3   Data4  Data5   Data6   Data7    Data8
1A          		00    	00    	8A     	01    	00    	3B    	5B      	00

Data1, Data4 and Data8 are not used.  The rest of the data appears to be sensor data but the actual data and scaling is 
unknown.  PID 99 looks like this:

Frame ID      Data1   Data2    Data3   Data4  Data5   Data6   Data7    Data8
99		         FF     	FF     	46      FF    	FF    	FF    	FF    	FF

Where Data3 is the requested PWM duty cycle and varies between 0x00 (0%) and 0x64 (100%) and is just the hex value converted 
to decimal with no scaling or offset.

When the key is turned off, the BCM commands the heater unit to sleep with the sleep command:

Frame ID      Data1   Data2    Data3   Data4  Data5   Data6   Data7    Data8
3C          		00    	FF     	FF    	FF    	FF    	FF    	FF    	FF

Since this ID is used for diagnostics, it uses the classic checksum.

For more information on LIN, see the latest published spec here: https://www.lin-cia.org/standards/.  There are now ISO and 
SAE specs that cover LIN, but they aren't freely available.

All the LIN data was obtained using a Microchip LIN Serial Analyzer APGDT001 module and the Microchip LIN Serial Analyzer 
v3.0.0 software.

Although the 500e was built as a compliance car and only sold in Oregon and California, they are rather plentiful in the 
junkyards there.  The air heaters can be found as "heater cores" on sites like car-part.com.  The cabin heater can be had for 
as little as $130 shipped, or perhaps less if you can find one from a pick and pull yard, and may come with the entire 
interior HVAC assembly.
