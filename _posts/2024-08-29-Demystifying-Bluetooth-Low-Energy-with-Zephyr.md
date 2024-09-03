# Demystifying Bluetooth Low Energy with Zephyr

# Marvin Ouma
27/08/2024

# What is Bluetooth Low Energy?

Wireless technology standard\, designed from ground up\.

Ultra Low Power\.

Small bursts of data transmission\.

Fast connection and teardown

Low cost\.

Works on free 2\.4 Ghz band\.

Ideal for sensors/ IoT\.

BLE Protocol:  _[https://software\-dl\.ti\.com/lprf/simplelink\_cc2640r2\_sdk/1\.00\.00\.22/exports/docs/blestack/html/ble\-stack/index\.html](https://software-dl.ti.com/lprf/simplelink_cc2640r2_sdk/1.00.00.22/exports/docs/blestack/html/ble-stack/index.html)_

# Stack Architecture

* __Two main sections:__
  * <span style="color:#92D050">Controller</span>
  * <span style="color:#0070C0">Host</span>

![](https://private-user-images.githubusercontent.com/36479602/362590049-f771c3cb-3234-4b44-933e-da21d637cdd9.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjQ5MTkyMjMsIm5iZiI6MTcyNDkxODkyMywicGF0aCI6Ii8zNjQ3OTYwMi8zNjI1OTAwNDktZjc3MWMzY2ItMzIzNC00YjQ0LTkzM2UtZGEyMWQ2MzdjZGQ5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MjklMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODI5VDA4MDg0M1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTUwMzQ0OWVkNDEwNTRiOTU1MmQxZWMwNTU3NWI4YjExNjM2MzVlMjc4MTBmYzM1ZGVjYzI2YWZhYWZkYWJkODAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.geibFRxXCK0tFuHKZcKLlZ_rWk8bYpeBSsEByRcTbjg)

# Generic Access Profile (GAP)

* Controls connection and advertising processes\.
* Makes a device visible to the world
* Determines how two devices can/cannot interact\.
* Defines the role of a device\.
  * __Connection\-oriented roles__
    * __Central \(\.e\.g\. Mobile phone\)__
    * __Peripheral \(e\.g\. sensor device\)__
  * __Connection\-less roles__
    * __Broadcaster \(sends out BLE advertisements\)__
    * __Observer \(scans for BLE advertisements\)__
* Each role comes with its own build\-time configuration option in Zephyr\.

# Generic Attribute Profile (GATT)

Defines how two BLE devices transfer data back and forth using concepts like Services and Characteristics\.

GATT works only after a dedicated connection between two devices has been established\.

Connections are exclusive\, i\.e\. a peripheral can only connect to one central device\.

Communication is two\-way\.

![](https://private-user-images.githubusercontent.com/36479602/362590075-6ab01500-a229-4574-b784-6bd19f8a854e.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjQ5MTkyMjMsIm5iZiI6MTcyNDkxODkyMywicGF0aCI6Ii8zNjQ3OTYwMi8zNjI1OTAwNzUtNmFiMDE1MDAtYTIyOS00NTc0LWI3ODQtNmJkMTlmOGE4NTRlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MjklMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODI5VDA4MDg0M1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWIyZDgxMDViMDU5NzY5ZGU0ZGZmNTljMWJkZmMwNGI2MTE3OTVmMWY5ZDk2NDU3MTI4NWYyZDM2MjRjYjU5ZDImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.bM512waDbsBEmO5ArfHFNy_D2uwRBFzSRm0CnLvZ-0c)

# Zephyr Comes In

* Zephyr comes with a fully integrated Bluetooth Low Energy \(BLE\) stack\.
* Supports various BLE roles\, including Central\, Peripheral\, Broadcaster\, and Observer\.
* Modular design allows for easy customization and extension\.
* Core Components:
  * <span style="color:#FF0000"> _Host Controller Interface \(HCI\)_ </span>  __: Provides a communication interface between the BLE host and the controller\.__
  * <span style="color:#FF0000"> _BLE Controller_ </span>  __: Handles low\-level BLE tasks like link layer management and packet handling\.__
  * <span style="color:#FF0000"> _BLE Host_ </span>  __: Manages higher\-level BLE functions such as the Logical Link Control and Adaptation Protocol \(L2CAP\)\, Attribute Protocol \(ATT\)\, and Security Manager Protocol \(SMP\)\.__

# Whats next

Zephyr has great examples\, in this case we will be looking at the peripheral example\.

Most of the Zephyr BLE devices that are built will most likely be peripheral\-role devices\.

Use a BLE scanner app for BLE connectivity testing\.
