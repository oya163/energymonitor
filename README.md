EnergyMonitor
=============

Energy Monitor system is based on Arduino device. We have configured two arduino board in such a way that one handles the Ethernet board and other one handles the GSM board. Both arduinos are communicating with each other using Serial communication.

Basically, we have designed voltage divider circuit to measure current and voltage from using CT sensor and Step-down transformer. Therefore, the primary funciton of this embedded system is to monitor the power level, display it on LCD and log it into the remote database server. Furthermore, It can monitor the server status, generate email, broadcast message, and check the network usage.

Similarly, we have also developed Web APIs in C#/.NET. These Web APIs are consumed by various third-parties like System Administrators or HR. Using the specific code, the Web APIs will transfer the message to the ethernet shield, which will eventually send the message to GSM shield to send SMS to end users.


