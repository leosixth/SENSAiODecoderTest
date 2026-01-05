Copyright © EDGE TECHNOLOGIES SAS

This repository presents the SENSA.iO LoRaWAN payloads, both for uplinks and downlinks. A Javascript code is provided to be used within a LoRaWAN platform. 

Below is a detailed description of the payloads. 

# Data formating

<u><b><i>Note:</b></u> 
Acording to the RP002-1.0.4 Regional Parameters specification, the maximum LoRaWAN uplink payload size depends on the region and the spread factor (SF) settings. Below is an extract of the specification, please refer to <a href="https://resources.lora-alliance.org/technical-specifications/rp002-1-0-4-regional-parameters">the full document</a> for more details.

-  EU868 / AU915 / KR920 / IN868 / RU868:
    - SF12-SF10: 51 bytes
    - SF9: 115 bytes
    - SF8-SF7: 222 bytes
-  AS923:
    - SF12-SF11: 51 bytes
    - SF10-SF9: 115 bytes
    - SF8-SF7: 222 bytes
-  US915:
    - SF10: 11 bytes
    - SF9: 53 bytes
    - SF8: 125 bytes
    - SF7: 222 bytes

These restrictions imply that the received sensor payload may not be complete depending on the sensor type (and its payload size), vs. the max payload size authorized on the network. 
As an example, in the US915 region in DR0 (SF10), the payload is limited to 11 bytes, and the pressure sensor (nomally a 15 bytes payload) will not return the last 4 bytes (temperature value). 
</i>

## Data Uplink

The data uplink payload is composed of a 7-byte section common for all sensors, and a sensor specific section.

### Common to all sensors
        
<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>0</b></td>
            <td align="center" colspan="8">8-bit Sensor Type <i>(see table below for sensor type encoding)</i></td>
        </tr>
        <tr>
            <td align="center"><b>1</b></td>
            <td align="center" colspan="6">7-bit Battery Level (6-bit LSB)</td>
            <td align="center" colspan="2">2-bit Msg Type <i>(see below)</i></td>
        </tr>
        <tr>
            <td align="center"><b>2</b></td>
            <td align="center" colspan="1">3-bit HwVersion (1-bit LSB)</td>
            <td align="center" colspan="3">3-bit FwMajorVersion</td>
            <td align="center" colspan="3">3-bit FwMinorVersion</td>
            <td align="center" colspan="1">7-bit Battery Level (1-bit MSB)</td>
        </tr>
        <tr>
            <td align="center"><b>3</b></td>
            <td align="center" colspan="6">9-bit DeviceTemp (6-bit LSB)</td>
            <td align="center" colspan="2">3-bit HwVersion (2-bit MSB)</td>
        </tr>
        <tr>
            <td align="center"><b>4</b></td>
            <td align="center" colspan="5">13-bit DeviceShock (5-bit LSB)</td>
            <td align="center" colspan="3">9-bit DeviceTemp (3-bit MSB)</td>
        </tr>
        <tr>
            <td align="center"><b>5</b></td>
            <td align="center" colspan="8">13-bit DeviceShock (8-bit MSB)</td>
        </tr>
        <tr>
            <td align="center"><b>6</b></td>
            <td align="center" bgcolor="#DDDDDD"><i>User Accept Degraded Calibration Flag</i></td>
            <td align="center">Swap Failure Flag</td>
            <td align="center">Sensor warning Flag</td>
            <td align="center">QoL Flag</td>
            <td align="center">Memory Failure Flag</td>
            <td align="center">Sensor Failure Flag</td>
            <td align="center">LoRaWAN Failure Flag</td>
            <td align="center">Secure Failure Flag</td>
        </tr>
        <tr>
            <td align="center"><b>7+</b></td>
            <td align="center" colspan="8"><b>Sensor specific (see below sections)</b></td>
        </tr>
    </tbody>
</table>

<u>In byte 0, the <i>Sensor Type</i> is encoded on 8 bits, and can be one of the following:</u>

 - 0x00: Test <i>(internal)</i>
 - 0x01: Pressure sensor
 - 0x02: Temperature RTD sensor
 - 0x03: Motion sensor
 - 0x04: Temperature TCK sensor
 - 0x05: Gas sensor <i>(ongoing)</i>
 - 0x06: Vibration sensor
 - 0x07: Loadcell sensor <i>(ongoing)</i>
 - 0x08: Differential pressure sensor
 - 0x09: Acoustic Inline <i>(ongoing)</i>
 - 0x0A: Acoustic Clamp-On

<u>In byte 1, the <i>Msg Type</i> is encoded on 2 bits, and can be one of the following:</u>

 - 0x0: First message after reboot/BLE Wakeup
 - 0x1: Periodic
 - 0x2: Alert
 - 0x3: Log

#### Log message type description

They are send in response of a log request downlink <i>(see below)</i>. It is useful to recover the latest data sent, in case of temporary network failure for instance. It returns the longest LoRaWAN payload possible with the last uplinks contents. The structure is the same as the Data uplink. Only the pattern of the sensor dedicated field differs from regular data uplinks : It is a list of elements formed as follow: 
 
<i>[(timestamp<sub>1</sub>, data<sub>1</sub>), (timestamp<sub>2</sub>, data<sub>2</sub>), … (timestamp<sub>n</sub>, data<sub>n</sub>)]</i>
 
 + Each timestamp is on 4 bytes (LSB), corresponding to the data sampling time (GPS time, see <a href="http://leapsecond.com/java/gpsclock.htm">GPS, UTC, and TAI Clocks</a> for details). 
 + The data<sub>i</sub> is the regular sensor dedicated data content (2 floats for pressure sensor, 1 float for temperature sensor, ...). 

### Pressure sensor specific payload
        
<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>7</b></td>
            <td align="center" rowspan="4" colspan="8">32-bit Pressure value (Pa, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>8</b></td>
        </tr>
        <tr>
            <td align="center"><b>9</b></td>
        </tr>
        <tr>
            <td align="center"><b>10</b></td>
        </tr>
        <tr>
            <td align="center"><b>11</b></td>
            <td align="center" rowspan="4" colspan="8">32-bit Temperature value (K, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>12</b></td>
        </tr>
        <tr>
            <td align="center"><b>13</b></td>
        </tr>
        <tr>
            <td align="center"><b>14</b></td>
        </tr>
    </tbody>
</table>

### Temperature (TCK and RTD) sensor specific payload
        
<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>7</b></td>
            <td align="center" rowspan="4" colspan="8">32-bit Temperature value (K, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>8</b></td>
        </tr>
        <tr>
            <td align="center"><b>9</b></td>
        </tr>
        <tr>
            <td align="center"><b>10</b></td>
        </tr>
    </tbody>
</table>

### Motion sensor specific payload
        
<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>7</b></td>
            <td align="center" colspan="8">8-bit opening percentage ([0:100]%, unsigned integer)</td>
        </tr>
        <tr>
            <td align="center"><b>8</b></td>
            <td align="center" colspan="8">8-bit opening time (s, unsigned integer)</td>
        </tr>
    </tbody>
</table>
  

### Differential pressure sensor specific payload

#### With static pressure option
        
<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>7</b></td>
            <td align="center" colspan="4"> 0x1</td>
            <td align="center" colspan="4"> 4-bit sub type ID (0=delta pressure, 1=liquid level, 2=mass flow rate)</td>
        </tr>
        <tr>
            <td align="center"><b>8</b></td>
            <td align="center" rowspan="4" colspan="8"> Differential pressure (Pa, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>9</b></td>
        </tr>
        <tr>
            <td align="center"><b>10</b></td>
        </tr>
        <tr>
            <td align="center"><b>11</b></td>
        </tr>
        <tr>
            <td align="center"><b>12</b></td>
            <td align="center" rowspan="4" colspan="8"> Static pressure (Pa, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>13</b></td>
        </tr>
        <tr>
            <td align="center"><b>14</b></td>
        </tr>
        <tr>
            <td align="center"><b>15</b></td>
        </tr>
        <tr>
            <td align="center"><b>16</b></td>
            <td align="center" rowspan="4" colspan="8"> If sub type ID = 1: Liquid level (m, float LSB)</br>If sub type ID = 2: Mass flow rate (kg/s, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>17</b></td>
        </tr>
        <tr>
            <td align="center"><b>18</b></td>
        </tr>
        <tr>
            <td align="center"><b>19</b></td>
        </tr>
    </tbody>
</table>
  
#### Without static pressure option
        
<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>7</b></td>
            <td align="center" colspan="4"> 0x0</td>
            <td align="center" colspan="4"> 4-bit sub type ID (0=delta pressure, 1=liquid level, 2=mass flow rate)</td>
        </tr>
        <tr>
            <td align="center"><b>8</b></td>
            <td align="center" rowspan="4" colspan="8"> Differential pressure (Pa, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>9</b></td>
        </tr>
        <tr>
            <td align="center"><b>10</b></td>
        </tr>
        <tr>
            <td align="center"><b>11</b></td>
        </tr>
        <tr>
            <td align="center"><b>12</b></td>
            <td align="center" rowspan="4" colspan="8"> If sub type ID = 1: Liquid level (m, float LSB)</br>If sub type ID = 2: Mass flow rate (kg/s, float LSB)</td>
        </tr>
        <tr>
            <td align="center"><b>13</b></td>
        </tr>
        <tr>
            <td align="center"><b>14</b></td>
        </tr>
        <tr>
            <td align="center"><b>15</b></td>
        </tr>
    </tbody>
</table>
  

### Vibration and Acoustic sensor specific payload

<table table-layout="fixed">
    <thead>
    <tr>
        <th rowspan="2" align="center">Byte</th>
        <th colspan="8" align="center">Bit</th>
    </tr>
    <tr>
        <th align="center">7</th>
        <th align="center">6</th>
        <th align="center">5</th>
        <th align="center">4</th>
        <th align="center">3</th>
        <th align="center">2</th>
        <th align="center">1</th>
        <th align="center">0</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><b>7</b></td>
            <td align="center" colspan="8"> AI Output size (n)</td>
        </tr>
        <tr>
            <td align="center"><b>8</b></td>
            <td align="center" rowspan="3" colspan="8"> AI Output (each byte is a tag probability, in range [0:100]%)</td>
        </tr>
        <tr>
            <td align="center"><b>...</b></td>
        </tr>
        <tr>
            <td align="center"><b>8+n-1</b></td>
        </tr>
        <tr>
            <td align="center"><b>8+n</b></td>
            <td align="center" rowspan="2" colspan="8"> Total energy (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+1</b></td></tr>
        <tr>
            <td align="center"><b>8+n+2</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 1: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+3</b></td></tr>
        <tr>
            <td align="center"><b>8+n+4</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 1: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+5</b></td></tr>
        <tr>
            <td align="center"><b>8+n+6</b></td>
            <td align="center" colspan="8"> Peak 1: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+7</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 2: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+8</b></td></tr>
        <tr>
            <td align="center"><b>8+n+9</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 2: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+10</b></td></tr>
        <tr>
            <td align="center"><b>8+n+11</b></td>
            <td align="center" colspan="8"> Peak 2: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+12</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 3: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+13</b></td></tr>
        <tr>
            <td align="center"><b>8+n+14</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 3: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+15</b></td></tr>
        <tr>
            <td align="center"><b>8+n+16</b></td>
            <td align="center" colspan="8"> Peak 3: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+17</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 4: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+18</b></td></tr>
        <tr>
            <td align="center"><b>8+n+19</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 4: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+20</b></td></tr>
        <tr>
            <td align="center"><b>8+n+21</b></td>
            <td align="center" colspan="8"> Peak 4: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+22</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 5: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+23</b></td></tr>
        <tr>
            <td align="center"><b>8+n+24</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 5: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+25</b></td></tr>
        <tr>
            <td align="center"><b>8+n+26</b></td>
            <td align="center" colspan="8"> Peak 5: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+27</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 6: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+28</b></td></tr>
        <tr>
            <td align="center"><b>8+n+29</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 6: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+30</b></td></tr>
        <tr>
            <td align="center"><b>8+n+31</b></td>
            <td align="center" colspan="8"> Peak 6: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+32</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 7: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+33</b></td></tr>
        <tr>
            <td align="center"><b>8+n+34</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 7: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+35</b></td></tr>
        <tr>
            <td align="center"><b>8+n+36</b></td>
            <td align="center" colspan="8"> Peak 7: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+37</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 8: Frequency (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+38</b></td></tr>
        <tr>
            <td align="center"><b>8+n+39</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak 8: Magnitude (mg, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+40</b></td></tr>
        <tr>
            <td align="center"><b>8+n+41</b></td>
            <td align="center" colspan="8"> Peak 8: Ratio (%)</td>
        </tr>
        <tr>
            <td align="center"><b>8+n+42</b></td>
            <td align="center" rowspan="2" colspan="8"> Peak integration width (Hz, 2 bytes LSB)</td>
        </tr>
        <tr><td align="center"><b>8+n+43</b></td></tr>
        <tr>
            <td align="center"><b>8+n+44</b></td>
            <td align="center" rowspan="2" colspan="8"> Temperature (*7.8125m°C, 2 bytes LSB)<br/><i>(only Vibration sensor)</i></td>
        </tr>
        <tr><td align="center"><b>8+n+45</b></td></tr>
    </tbody>
</table>
  
## Configuration uplink

This payload is triggered by an internal change of the LoRaWAN or alarm settings, or in response of the LoRaWAN or Alarm configuration downlink <i>(see the downlink section below)</i>

<u><b>Note:</b></u> Configuration uplinks are only valid for devices with FW version 1.1.2 and above. 

### Alarm configuration uplink (deprecated)

<u><b>Note:</b></u> Alarm configuration format has changed in firmware version 1.2.0+. We strongly recommend to upgrade the devices with the latest firmware, and use the extended alarm configuration <i>(see below)</i> formatting. 

Refer to the alarm dedicated documentation for the data details.

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0</b></td>
        <td>0xFF</td>
        <td>Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>1</b></td>
        <td>0x05</td>
        <td>Alarm Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>2</b></td>
        <td>Status</td>
        <td>0 = OK, when triggering configuration downlink was accepted, >0 otherwise</td>
    </tr>
    <tr>
        <td align="center"><b>3-6</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>7-10</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>11-14</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>15-18</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>19</b></td>
        <td>Active alarms</td>
        <td>0x00=No active alarm, 0x01=High alarm active, 0x02=Low alarm active, 0x03:High and low alarm active</td>
    </tr>
    <tr><td colspan="3" align="center"><i>And <b>only</b> for the pressure sensor, a second alarm (for the temperature)</i></td></tr>
    <tr>
        <td align="center"><b>20-23</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>24-27</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>28-31</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>32-35</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>36</b></td>
        <td>Active alarms</td>
        <td>0x00=No active alarm, 0x01=High alarm active, 0x02=Low alarm active, 0x03:High and low alarm active</td>
    </tr>
    </tbody>
</table>

### Extended alarm configuration uplink

From firmware version 1.2.0 and above. 

Refer to the alarm dedicated documentation for the data details.

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0</b></td>
        <td>0xFF</td>
        <td>Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>1</b></td>
        <td>0x07</td>
        <td>Extended alarm Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>2</b></td>
        <td>Status</td>
        <td>0 = OK, when triggering configuration downlink was accepted, >0 otherwise</td>
    </tr>
    <tr>
        <td align="center"><b>3-6</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>7-10</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>11-14</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>15-18</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>19</b></td>
        <td>Parameters</td>
        <td>
        Bit0: High alarm threshod active</br>
        Bit1: Low alarm threshod active</br>
        Bit2: Variation alarm active</br>
        Bit3: <i>Reserved</i>
        Bit7-4: Wakeup period <i>(see below for authorized values)</i>
        </td>
    </tr>
    <tr>
        <td align="center"><b>20</b></td>
        <td> Variation alarm value</td>
        <td>8-bit integer [1:255]</td>
    </tr>
    <tr>
        <td align="center"><b>21</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>22</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr><td colspan="3" align="center"><i>And <b>only</b> for the pressure sensor, a second alarm (for the temperature)</i></td></tr>
    <tr>
        <td align="center"><b>23-26</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>27-30</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>31-34</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>35-38</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>39</b></td>
        <td>Parameters</td>
        <td>
        Bit0: High alarm threshod active</br>
        Bit1: Low alarm threshod active</br>
        Bit2: Variation alarm active</br>
        Bit3: <i>Reserved</i>
        Bit7-4: Wakeup period <i>(see below for authorized values)</i>
        </td>
    </tr>
    <tr>
        <td align="center"><b>40</b></td>
        <td> Variation alarm value</td>
        <td>8-bit integer [1:255]</td>
    </tr>
    <tr>
        <td align="center"><b>41</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>42</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    </tbody>
</table>

<u>Allowed alarm wakeup period encodings:</u>
<ul>
<li>0x0: 15 seconds</li>
<li>0x1: 30 seconds</li>
<li>0x2: 1  minute, default</li>
<li>0x3: 2  minutes</li>
<li>0x4: 5  minutes</li>
<li>0x5: 15 minutes</li>
<li>0x6: 30 minutes</li>
<li>0x7: 1  hour</li>
<li>0x8: 3  hours</li>
<li>0x9: 6  hours</li>
<li>0xA: 12 hours</li>
</ul>

### Extended alarm configuration uplink

From firmware version 1.3.2 and above. applied in Pressure and Temperature Transmitter. 

The latest alarm can choose for relative alarm or absolute alarm.

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0</b></td>
        <td>0xFF</td>
        <td>Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>1</b></td>
        <td>0x07</td>
        <td>Extended alarm Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>2</b></td>
        <td>Status</td>
        <td>0 = OK, when triggering configuration downlink was accepted, >0 otherwise</td>
    </tr>
    <tr>
        <td align="center"><b>3-6</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>7-10</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>11-14</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>15-18</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>19</b></td>
        <td>Parameters</td>
        <td>
        Bit0: High alarm threshod active</br>
        Bit1: Low alarm threshod active</br>
        Bit2: Variation alarm active</br>
        Bit3: <i>Reserved</i>
        Bit7-4: Wakeup period <i>(see below for authorized values)</i>
        </td>
    </tr>
    <tr>
        <td align="center"><b>20</b></td>
        <td> Variation alarm value</td>
        <td>8-bit integer [1:255]</td>
    </tr>
    <tr>
        <td align="center"><b>21</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>22</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr><td colspan="3" align="center"><i>And <b>only</b> for the pressure sensor, a second alarm (for the temperature)</i></td></tr>
    <tr>
        <td align="center"><b>23-26</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>27-30</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>31-34</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>35-38</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>39</b></td>
        <td>Parameters</td>
        <td>
        Bit0: High alarm threshod active</br>
        Bit1: Low alarm threshod active</br>
        Bit2: Variation alarm active</br>
        Bit3: <i>Reserved</i>
        Bit7-4: Wakeup period <i>(see below for authorized values)</i>
        </td>
    </tr>
    <tr>
        <td align="center"><b>40</b></td>
        <td> Variation alarm value</td>
        <td>8-bit integer [1:255]</td>
    </tr>
    <tr>
        <td align="center"><b>41</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>42</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    </tbody>
</table>

<u>Allowed alarm wakeup period encodings:</u>
<ul>
<li>0x0: 15 seconds</li>
<li>0x1: 30 seconds</li>
<li>0x2: 1  minute, default</li>
<li>0x3: 2  minutes</li>
<li>0x4: 5  minutes</li>
<li>0x5: 15 minutes</li>
<li>0x6: 30 minutes</li>
<li>0x7: 1  hour</li>
<li>0x8: 3  hours</li>
<li>0x9: 6  hours</li>
<li>0xA: 12 hours</li>
</ul>


### LoRaWAN configuration uplink

Below is the structure of the LoRaWAN configuration uplink

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0</b></td>
        <td>0xFF</td>
        <td>Configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>1</b></td>
        <td>0x04</td>
        <td>LoRaWAN configuration message token</td>
    </tr>
    <tr>
        <td align="center"><b>2</b></td>
        <td>Status</td>
        <td>0 = OK, when triggering configuration downlink was accepted, >0 otherwise</td>
    </tr>
    <tr>
        <td align="center"><b>3</b></td>
        <td>TxDatarate</td>
        <td>8-bit unsigned</td>
    </tr>
    <tr>
        <td align="center"><b>4</b></td>
        <td><i>Reserved</i></td>
        <td>8-bit unsigned</td>
    </tr>
    <tr>
        <td align="center"><b>5</b></td>
        <td>TxRetries</td>
        <td>8-bit unsigned</td>
    </tr>
    <tr>
        <td align="center"><b>6</b></td>
        <td>AppPort</td>
        <td>8-bit unsigned</td>
    </tr>
    <tr>
        <td align="center"><b>7-10</b></td>
        <td>TxPeriodicity</td>
        <td>LoRaWAN TX Period (32-bit unsigned LSB)</td>
    </tr>
    <tr>
        <td align="center"><b>11</b></td>
        <td>IsTxConfirmed</td>
        <td>8-bit unsigned</td>
    </tr>
    <tr>
        <td align="center"><b>12</b></td>
        <td>AdrEnable</td>
        <td>8-bit unsigned</td>
    </tr>
    <tr>
        <td align="center"><b>13</b></td>
        <td>PublicNetworkEnable</td>
        <td>8-bit unsigned</td>
    </tr>
    </tbody>
</table>

## Downlink

Downlinks are used to update the device configuration (alarms or LoRaWAN), or to request the data log. 

Note: An dummy empty configuration downlink on the respective port (4, 5 or 7) will force the device to send its current alarm/LoRaWAN configuration. 

### LoRaWAN configuration downlink

This command updates the LoRaWAN configuration of the device. It must be sent on the AppPort 4

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Type</th>
        <th align="center">Valid range</th>
        <th align="center">Default value</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0</b></td>
        <td>TxDatarate</td>
        <td>8-bit unsigned</td>
        <td>Region dependent. See LoRaWAN specifications</td>
        <td>0 (DR0)</td>
    </tr>
    <tr>
        <td align="center"><b>1</b></td>
        <td><i>Reserved</i></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>2</b></td>
        <td>TxRetries</td>
        <td>8-bit unsigned</td>
        <td>[0;5]</td>
        <td>0 (no retries)</td>
    </tr>
    <tr>
        <td align="center"><b>3</b></td>
        <td>AppPort</td>
        <td>8-bit unsigned</td>
        <td>[1;254]</td>
        <td>2</td>
    </tr>
    <tr>
        <td align="center"><b>4-7</b></td>
        <td>TxPeriodicity</td>
        <td>LoRaWAN TX Period (32-bit unsigned LSB)</td>
        <td>>= 60000 ms ( =1 minute)</td>
        <td>900000 (15 minutes)</td>
    </tr>
    <tr>
        <td align="center"><b>8</b></td>
        <td>IsTxConfirmed</td>
        <td>8-bit unsigned</td>
        <td>[0;1]</td>
        <td>0 (Unconfirmed)</td>
    </tr>
    <tr>
        <td align="center"><b>9</b></td>
        <td>AdrEnable</td>
        <td>8-bit unsigned</td>
        <td>[0;1]</td>
        <td>1 (Adaptative Data Rate Enabled)</td>
    </tr>
    <tr>
        <td align="center"><b>10</b></td>
        <td>PublicNetworkEnable</td>
        <td>8-bit unsigned</td>
        <td>[0;1]</td>
        <td>1 (Public Network enabled)</td>
    </tr>
    </tbody>
</table>

### Alarm configuration downlink (depecrated)

This command updates the alarm configuration of the device. It must be sent on the AppPort 5. 

<u><b>Note:</b></u> Alarm configuration format has changed in firmware version 1.2.0+. We strongly recommend to upgrade the devices with the latest firmware, and use the extended alarm configuration <i>(see below)</i> formatting. 

Refer to the alarm dedicated documentation for the data details.

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0-3</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>4-7</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>8-11</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>12-15</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>16</b></td>
        <td>Active alarms</td>
        <td>0x00=No active alarm, 0x01=High alarm active, 0x02=Low alarm active, 0x03:High and low alarm active</td>
    </tr>
    <tr><td colspan="3" align="center"><i>And <b>only</b> for the pressure sensor, a second alarm (for the temperature)</i></td></tr>
    <tr>
        <td align="center"><b>17-20</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>21-23</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>24-27</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>28-31</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>32</b></td>
        <td>Active alarms</td>
        <td>0x00=No active alarm, 0x01=High alarm active, 0x02=Low alarm active, 0x03:High and low alarm active</td>
    </tr>
    </tbody>
</table>

### Extended alarm configuration downlink

From firmware version 1.2.0 and above. 

This command updates the alarm configuration of the device. It must be sent on the AppPort 7.

Refer to the alarm dedicated documentation for the data details.

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Name/Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0-3</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>4-7</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>8-11</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>12-15</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>16</b></td>
        <td>Parameters</td>
        <td>
        Bit0: High alarm threshod active</br>
        Bit1: Low alarm threshod active</br>
        Bit2: Variation alarm active</br>
        Bit3: <i>Reserved</i>
        Bit7-4: Wakeup period <i>(see below for authorized values)</i>
        </td>
    </tr>
    <tr>
        <td align="center"><b>17</b></td>
        <td> Variation alarm value</td>
        <td>8-bit integer [1:255]</td>
    </tr>
    <tr>
        <td align="center"><b>18</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>19</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr><td colspan="3" align="center"><i>And <b>only</b> for the pressure sensor, a second alarm (for the temperature)</i></td></tr>
    <tr>
        <td align="center"><b>20-23</b></td>
        <td>High Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>24-27</b></td>
        <td>High Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>28-31</b></td>
        <td>Low Alarm Threshold Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>32-35</b></td>
        <td>Low Alarm Hystersis Value</td>
        <td>32-bit float LSB</td>
    </tr>
    <tr>
        <td align="center"><b>36</b></td>
        <td>Parameters</td>
        <td>
        Bit0: High alarm threshod active</br>
        Bit1: Low alarm threshod active</br>
        Bit2: Variation alarm active</br>
        Bit3: <i>Reserved</i>
        Bit7-4: Wakeup period <i>(see below for authorized values)</i>
        </td>
    </tr>
    <tr>
        <td align="center"><b>37</b></td>
        <td> Variation alarm value</td>
        <td>8-bit integer [1:255]</td>
    </tr>
    <tr>
        <td align="center"><b>38</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    <tr>
        <td align="center"><b>39</b></td>
        <td bgcolor="#DDDDDD"><i>Reserved</i></td>
        <td></td>
    </tr>
    </tbody>
</table>

<u>Allowed alarm wakeup period encodings:</u>
<ul>
<li>0x0: 15 seconds</li>
<li>0x1: 30 seconds</li>
<li>0x2: 1  minute, default</li>
<li>0x3: 2  minutes</li>
<li>0x4: 5  minutes</li>
<li>0x5: 15 minutes</li>
<li>0x6: 30 minutes</li>
<li>0x7: 1  hour</li>
<li>0x8: 3  hours</li>
<li>0x9: 6  hours</li>
<li>0xA: 12 hours</li>
</ul>

### Log request downlink

This command triggers the send of a data payload of type LOG. It must be sent on the AppPort 6 with the following payload:

<table align="center">
    <thead>
    <tr>
        <th align="center">Byte</th>
        <th align="center">Value</th>
        <th align="center">Description</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td align="center"><b>0</b></td>
        <td>0x52</td>
        <td>'R' ASCII char</td>
    </tr>
    <tr>
        <td align="center"><b>1</b></td>
        <td>0x45</td>
        <td>'E' ASCII char</td>
    </tr>
    <tr>
        <td align="center"><b>2</b></td>
        <td>0x50</td>
        <td>'P' ASCII char</td>
    </tr>
    <tr>
        <td align="center"><b>3</b></td>
        <td>0x4C</td>
        <td>'L' ASCII char</td>
    </tr>
    <tr>
        <td align="center"><b>4</b></td>
        <td>0x41</td>
        <td>'A' ASCII char</td>
    </tr>
    <tr>
        <td align="center"><b>5</b></td>
        <td>0x59</td>
        <td>'Y' ASCII char</td>
    </tr>
    <tr>
        <td align="center"><b>6</b></td>
        <td>0x00</td>
        <td>NULL terminating ASCII string</td>
    </tr>
    </tbody>
</table
