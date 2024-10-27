---

title: "Dissecting USB PCAP Traffic"
date: 2024-10-27 01:09:33 +0300
author: [oste, m4lw0r3, d3xt3r]
description: This blog post explores USB packet capture (Pcap) traffic analysis, focusing on what occurs when a keyboard is plugged in. It covers packet dissection, traffic filtering, and decoding of keyboard keystrokes.
image: /assets/img/Posts/usbpcap.png
categories: [DFIR]
tags:
  [usb, pcap, wireshark, hid data, keycodes, urb]
---


Dear reader, welcome back to this blog post where we dissect USB PCAP traffic. Ever wondered what happens under the hood when you plug your keyboard in your laptop? Or how keyboard keystrokes are translated by the computer? Have you found yourself playing a CTF and you've been given USB related traffic to retrieve a flag and you're like:



![ohgodno](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExNm41Zjh4MzZ0czR5YjJ3NGJ4NmYwbjBwNDU0ZjY1MHlxbHJnZDh6bSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/8vUEXZA2me7vnuUvrs/giphy.webp){: .left }

![iunderstandnothing](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExdzlkZ2Vwa2tkNXA3dGNud2Z1YjhmcTQ0dGk4dHRhM2I1Z3pwMjR6cSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/SAAMcPRfQpgyI/giphy.webp){: .normal }



Well, me and my friends ( [m4lw0r3](https://x.com/m4lw0r3) & [d3xt3r](https://x.com/alvin_kidwiz) ) embarked on a 2 week research on the same and thought we'd share our learnings with the rest of the world. Hopefully you will learn a thing or two 😎. 


> I highly encourage you to first read and understand the theory explained first before diving into the PRACTICAL section. This way, you will understand better what is included in the screenshots attached.
{: .prompt-tip }


With that said, lets get started.


## What is USBPcap?

USBPcap is a Windows-based tool that allows you to capture USB traffic (data exchanged between USB devices and your computer). Wireshark can analyze this captured traffic, helping you to understand what data is being transferred over USB. 

> Please note that in this study, we will be using Windows
{: .prompt-info }


## URB (USB Request Block)

URBs represent USB requests or responses and contain information about the direction (from or to the USB device), the type of request (control, interrupt, bulk, or isochronous transfer), and the data payload.

Key fields to pay attention to:

- **URB ID**: A unique identifier for the request/response.
- **URB type**: The type of transaction (control, interrupt, bulk, etc.).
- **Endpoint**: Represents the specific endpoint on the USB device (input/output) the communication is targeting.
- **Data**: The actual data being transferred, often seen as hex data.

## USB Transfer Types:

### TL;DR

- **Control Transfer**
- **Interrupt Transfer**
- **Bulk Transfer**
- **Isochronous Transfer**

| Transfer Type   | Data Size    | Timing Sensitivity | Error Handling   | Common Devices                        |
| --------------- | ------------ | ------------------ | ---------------- | ------------------------------------- |
| **Control**     | Small        | Low                | Retries on error | USB initialization, configuration     |
| **Interrupt**   | Small        | High               | Retries on error | Keyboards, mice, game controllers     |
| **Bulk**        | Large        | Low                | Retries on error | USB flash drives, printers            |
| **Isochronous** | Large/Medium | High               | No retries       | USB audio, webcams, real-time devices |

### Deep Dive

![ohgodno](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExdW90ejloNHdrM280YjRtNWNrZTlndjZpdWw5dWg4cDM2cG5jNnI5bSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/WsNbxuFkLi3IuGI9NU/giphy.webp){: .normal }

----
#### 1. Control Transfer

Control transfers are primarily used to send commands or configuration data to the USB device. They are essential for device setup and management, especially when a device is first connected.

##### Characteristics:

- **Direction**: Can be bi-directional, meaning they can both send and receive data.
- **Data Size**: They are limited to small amounts of data, typically used for configuration or setup purposes.
- **Guaranteed Delivery**: Control transfers ensure that data is successfully delivered. If there’s an error, the transfer is retried.

##### Control Transfer Stages:

A control transfer typically consists of three stages:

1. **Setup Stage**: This stage contains the control request, which defines the action the device should take (e.g., a request to read the device's descriptor). It includes information like:
    - Request type (Standard, Class-specific, Vendor-specific)
    - Request code (like "Get Descriptor" or "Set Configuration")
    - Value and Index fields (to provide additional information for the request)
2. **Data Stage**: This stage is optional. It is used if the control request requires data transfer. For example, when requesting a device descriptor, the data stage will carry the actual descriptor information back from the device.
    - This stage could either be IN (device to host) or OUT (host to device).
3. **Status Stage**: This stage signals the end of the transfer, indicating whether it was successful or there was an error. The host acknowledges that the control transfer is complete.

##### Use cases:

- **Device Enumeration**: When a USB device is plugged in, control transfers are used to read the device’s descriptors (such as the Vendor ID and Product ID).
- **Configuration**: Control transfers are used to configure the device, such as setting the device’s operational parameters.

---------

#### 2. Interrupt Transfer

Interrupt transfers are designed for short, low-latency communications, often for devices that need to send or receive small, time-critical data (like button presses or sensor data).

##### Characteristics:

- **Direction**: Interrupt transfers can be either IN (from device to host) or OUT (from host to device).
- **Guaranteed Delivery**: Like control transfers, interrupt transfers guarantee that data will be delivered. If there’s an error, the transfer will be retried.
- **Polling Interval**: The host polls the device at regular intervals to check if any new data is available. The polling interval is specified by the device.
- **Small Data Size**: Interrupt transfers typically transfer small packets of data (e.g., a few bytes).

##### Use cases:

- **Keyboards and Mice**: For example, when you press a key on a USB keyboard, an interrupt transfer sends a packet from the keyboard to the host containing the key press event.
- **Game Controllers**: Similar to a keyboard, game controllers use interrupt transfers to report button presses or joystick movements.

----
#### 3. Bulk Transfer

Bulk transfers are designed for larger amounts of data that don’t need to be transferred immediately. They are used when the timing of the transfer isn’t as critical, but the data itself is important, such as in file transfers.

##### Characteristics:

- **Direction**: Bulk transfers can be either IN or OUT.
- **Data Size**: There is no predefined limit on the amount of data for a bulk transfer, making it ideal for transferring large amounts of data.
- **No Guaranteed Timing**: Unlike interrupt and isochronous transfers, there’s no guarantee when bulk transfers will be completed. However, they are guaranteed to be delivered eventually. If the bus is busy, bulk transfers may be delayed.
- **Error Handling**: Bulk transfers have built-in error detection and correction mechanisms. If an error occurs, the transfer is retried.

##### Use cases:

- **USB Flash Drives (Mass Storage Devices)**: When you copy files to or from a USB flash drive, bulk transfers are used to move the data. The drive reads or writes the data in large chunks, but the timing of these transfers is not critical.
- **Printers**: Bulk transfers are also used to send large documents to a printer.

----
#### 4. Isochronous Transfer

Isochronous transfers are designed for time-sensitive data, where maintaining a steady data flow is more important than ensuring every bit of data is transmitted perfectly. These transfers are often used for real-time audio or video streams.

##### Characteristics:

- **Direction**: Isochronous transfers can be either IN or OUT.
- **Guaranteed Bandwidth**: Isochronous transfers are guaranteed a certain amount of bandwidth on the USB bus to ensure that data is delivered at regular intervals.
- **No Error Retries**: Unlike the other transfer types, isochronous transfers do not include error correction. This is because real-time applications often prioritize timely delivery over perfect data integrity. For instance, in a video or audio stream, a small glitch is less noticeable than a significant delay.
- **Time-Sensitive**: Isochronous transfers are used when maintaining a consistent flow of data is crucial. The host guarantees that the data will be transferred within a certain time frame.

##### Use cases:

- **Audio Devices**: USB microphones or audio interfaces often use isochronous transfers to ensure that audio is streamed in real-time without any delay.
- **Webcams**: When streaming live video from a USB webcam, isochronous transfers are used to maintain smooth video playback.
- **Medical Devices**: Some medical equipment uses isochronous transfers to send real-time data such as heart rate or EEG data to a computer for immediate analysis.


## PRACTICAL

In this section, we are going to look at what happens when you first plug in a keyboard for the first time. Also, to better understand the USB Transfer types discussed above, we will focus of *Control Transfer* as well as *Interrupt Transfers*

> I highly encourage you to try this excerice yourself after reading this section. Information about how to capture & analyze traffic can be found in later sections of this blog post.
{: .prompt-tip }

For now we will analyze traffic that has already been captured.

### 1st packet Frame

This packet shows the host sending a **GET DESCRIPTOR** request, asking for the device descriptor from the USB device. This is usually one of the first things a host does when communicating with a USB device, as it needs the device descriptor to identify the device's capabilities, vendor, and product IDs.
#### 1. Packet List frame

Represents a **USB Control Transfer**, specifically the "`GET DESCRIPTOR`" request from the host to the USB device.

![image](https://gist.github.com/user-attachments/assets/a3078b74-3ea3-48ac-85dc-75c607e3a40f)


#### 2. Packet Overview:

- **Source**: `host`
- **Destination**: `1.25.0`  - *This is the address of the USB device*
- **URB Function**: `URB_FUNCTION_GET_DESCRIPTOR_FROM_DEVICE` - *This indicates the type of USB operation, in this case, getting a descriptor from the device*
- **URB Transfer Type**: `URB_CONTROL` - *This indicates it's a control transfer*

#### 3.  Control Transfer Setup Phase:

Remember earlier we said control transfers have three phases: `Setup`, `Data`, and `Status`. In this case, we're looking at the **Setup Stage**:

- **bmRequestType (0x80)**:
    - **Direction**: `Device-to-host` (bit 7 = 1, meaning the device is sending data back to the host)
    - **Type**: `Standard` (bit 5:6 = 00, meaning it's a standard request, as opposed to class or vendor-specific)
    - **Recipient**: `Device` (bit 0:4 = 00000, meaning the request is directed to the device itself, rather than an interface or endpoint)
- **bRequest (0x06)**:
    - This is the "GET DESCRIPTOR" request, which is a standard USB request used to retrieve a descriptor (such as device, configuration, string, etc.) from the device.
- **wValue (0x0100)**:
    - This field provides additional information about the descriptor being requested.
    - **Descriptor Type**: `DEVICE` (0x01 indicates a request for the device descriptor)
    - **Descriptor Index**: 0x00 (This is the first instance of the descriptor, which means the host is requesting the main device descriptor)
- **wIndex (0x0000)**:
    - This is the language ID for string descriptors or the interface index for specific requests. In this case, it’s 0, meaning it’s not relevant for this type of request.
- **wLength (0x0012)**:
    - This indicates the length of the descriptor being requested, 18 bytes in this case. **This is the typical length of a USB device descriptor.**


### 2nd packet frame

![image](https://gist.github.com/user-attachments/assets/1dce51a8-535a-4e09-943e-0d9bc7e34083)

#### 2. **Packet Overview:**

- **URB Function**: `URB_FUNCTION_CONTROL_TRANSFER` - *Indicating this is a response to the control transfer initiated by the host*
- **IRP Status**: `USBD_STATUS_SUCCESS` - *This indicates that the request was successfully processed, and the device has sent the data*

#### 3. **URB Details:**

- **Device Address**: 25 - *This is the address assigned to the USB device during enumeration*
- **Endpoint**: `0x80, Direction: IN` - *This shows that the data is being sent from the device to the host*
- **URB Transfer Type**: `URB_CONTROL` - *Confirming it's a control transfer*
- **Packet Data Length**: 18 bytes - *The response contains 18 bytes, which matches the length of the device descriptor requested*

#### 4. **Control Transfer Stage**: Complete (3)

This indicates that the control transfer (**Setup -> Data -> Status**) is complete. In this case, we're in the **Data Stage**, where the device sends the descriptor.

#### 5. **Device Descriptor**: (18 Bytes of Data)

This section provides crucial information about the USB device. Let's go through each field:

1. **bLength**: `18` - *This indicates that the descriptor length is 18 bytes*
2. **bDescriptorType**: `0x01` - *This confirms that the descriptor type is "DEVICE"*
3. **bcdUSB**: `0x0110` - *This indicates the USB version supported by the device, which is **USB Version**: **1.1***

Other **bcdUSB** Examples

| bcdUSB Value | USB Version | Max Speed       | Release Year | Common Use Cases                                         |
| ------------ | ----------- | --------------- | ------------ | -------------------------------------------------------- |
| 0x0110       | USB 1.1     | 12 Mbps (Full)  | 1998         | Keyboards, mice, older devices                           |
| 0x0200       | USB 2.0     | 480 Mbps (High) | 2000         | Flash drives, printers, most common devices              |
| 0x0300       | USB 3.0     | 5 Gbps (Super)  | 2008         | External hard drives, SSDs, cameras, fast peripherals    |
| 0x0310       | USB 3.1     | 10 Gbps         | 2013         | Modern external storage, USB-C, high-performance devices |
| 0x0320       | USB 3.2     | 20 Gbps         | 2017         | Ultra-high-performance devices, external SSDs, graphics  |
| 0x0400       | USB 4.0     | 40 Gbps         | 2019         | External GPUs, high-speed storage, Thunderbolt           |

4. **bDeviceClass**: `0x00` - *Device class is unspecified at the device level. This means that class information will be provided in the interface descriptor*
5. **bDeviceSubClass**: `0x00` - *The device does not specify a subclass at the device level*
6. **bDeviceProtocol**: `0x00` - *The device does not specify a protocol at the device level*
7. **bMaxPacketSize0**: `64` - *This indicates the maximum packet size for endpoint 0, which is used for control transfers. A size of 64 bytes is typical for high-speed devices*
8. **idVendor**: `0x0f04` - *identifies the device’s manufacturer. Based on this Vendor ID, the manufacturer could be identified (in this case, it appears to be a generic or less common vendor, as it is unknown from the provided information).*
9.  **idProduct**: `0x2f05` - *This is the **Product ID**. It identifies the specific product model made by the vendor. Similar to the Vendor ID, this Product ID is unknown, possibly indicating a lesser-known device or a custom product.*
10. **bcdDevice**: `0x0110` - *This is the device's release number in binary-coded decimal. The device version is **1.10**, likely representing its hardware or firmware version.*
11. **iManufacturer**: `0x01` - *This index refers to the **Manufacturer String Descriptor**, which the host can retrieve to get the name of the manufacturer. The value `1` points to the specific string descriptor that contains this information.*
12. **iProduct**: `0x02` - *This index refers to the **Product String Descriptor**, which the host can request to retrieve the product’s name. The value `2` points to the descriptor that contains the product name.*
13. **iSerialNumber**: `0x00` - *This value indicates that the device does not provide a serial number string descriptor.*
14. **bNumConfigurations**: `1` - *This field indicates that the device has **one configuration**. This means that the device supports only one set of interfaces and endpoints, which will be used after the configuration is selected.*


I tried the same with Logitech headphones:

![image](https://gist.github.com/user-attachments/assets/9ebfb4d6-4bf5-471b-9858-c301197ebe7f)


- **bcdUSB**: `0x0200` *This indicates the USB version supported by the device, which is **USB 2.0***
- **idVendor**: `0x046d` - *This is the **Vendor ID** assigned by the USB-IF to **Logitech, Inc.***
- **idProduct**: `0x0a38` - *In this case, it corresponds to the **Logitech H340 USB Headset***
- **bcdDevice**: `0x0117` - *This is the device's release number in binary-coded decimal, indicating version **1.17***
- **iManufacturer**: `0x01` ***Logitech, Inc.***
- **iProduct**: `0x02`  ***"H340 USB Headset"***


### 3rd Packet

![image](https://gist.github.com/user-attachments/assets/a588783c-0319-4e27-9c1d-f35176b5edf9)

#### **Packet Details:**

- **URB Transfer Type**: `URB_CONTROL` - *Control transfer, used for setup and configuration, such as requesting and setting descriptors*
- **Packet Data Length**: `8 bytes` - *The length of the setup packet*

#### **USB URB (USB Request Block)**:

This section describes the actual USB transaction in detail.

- **IRP Status**: `USBD_STATUS_SUCCESS (0x00000000)`
    - *This indicates that the request was successful, meaning that the control transfer setup stage was sent correctly.*
- **URB Function**: `URB_FUNCTION_GET_DESCRIPTOR_FROM_DEVICE (0x000b)`
    - *This indicates that the host is asking for a **descriptor** from the device, which is part of the standard enumeration process.*
- **Direction**: `FDO -> PDO (0x0)`
    - **FDO (Function Driver Object)** to **PDO (Physical Device Object)**. *This means the direction of the control transfer is from the host to the device (outgoing setup stage).*
- **Endpoint**: `0x80, Direction: IN`
    - **IN** *direction indicates that after the setup stage, the response (data) is expected from the device to the host.*
    - **Endpoint 0** *is used for control transfers (as is always the case in USB control transfers).*
- **URB Transfer Type**: `URB_CONTROL (0x02)`
    - *This is a control transfer type request, typical for descriptor requests.*

#### **Control Transfer Stage**: Setup (0)

This is the **Setup Stage** of a control transfer, where the host sends the initial setup packet to the device, asking for a descriptor.

#### **Setup Data**:

This section describes the setup data, which provides details about the specific request.

- **bmRequestType**: `0x80`
    - **Direction**: `Device-to-host` - *The device will send data to the host after this setup packet, which is expected in the data stage of the control transfer*
    - **Type**: `Standard` - *This is a standard USB request, as opposed to class-specific or vendor-specific requests*
    - **Recipient**: `Device` - *The request is directed to the USB device itself.*
- **bRequest**: `GET DESCRIPTOR (6)`
    - *The host is requesting a descriptor from the device. The specific descriptor is indicated in the next fields.*
- **Descriptor Index**: `0x00`
    - *This indicates the **first (and only) configuration** descriptor of the device. Many devices only have one configuration, which is why the index is `0x00`.*
- **bDescriptorType**: `CONFIGURATION (0x02)`
    - *This indicates that the host is requesting the **Configuration Descriptor**, which provides information about the device's configurations, interfaces, endpoints, and power requirements.*
- **Language ID**: `No language specified (0x0000)`
    - *This field is relevant only for string descriptor requests. Since this is a **Configuration Descriptor** request, the **language ID** is set to zero, meaning it's not applicable in this case.*
- **wLength**: `9`
    - *The host is requesting **9 bytes** of data. This is the length of the initial portion of the **Configuration Descriptor**. In many cases, the full configuration descriptor may be longer, but the host starts by requesting just the first part. Once it gets this initial part, it can issue additional requests to retrieve the full descriptor.*


### 4th Packet

![image](https://gist.github.com/user-attachments/assets/077f5a64-3166-4fe5-96c2-b06bd9e50ce6)

#### **Packet Overview:**

- **Packet Data Length**: `9 bytes` (This matches the requested length from packet 3)

#### **USB URB (USB Request Block)**:

- **IRP Status**: `USBD_STATUS_SUCCESS (0x00000000)`
    - *The response was successful, and the device sent the requested data.*
- **Packet Data Length**: `9 bytes`
    - *This matches the request in packet 3, where the host asked for 9 bytes, which is the size of the **Configuration Descriptor** header.*

#### **Configuration Descriptor (9 Bytes)**:

This descriptor provides details about the device's configuration. Let's dissect it field by field:

- **bLength**: `9`
    - *This indicates that the length of this descriptor is 9 bytes, which is the size of the standard **Configuration Descriptor** header.*
- **bDescriptorType**: `0x02 (CONFIGURATION)`
    - *This confirms that this is a **Configuration Descriptor**.*
- **wTotalLength**: `59`
    - *This field shows the **total length** of the configuration, including all related descriptors (such as interface and endpoint descriptors). In this case, the total length is 59 bytes.*
    - *This means that after this initial 9-byte header, the host can request additional data (up to 59 bytes) to retrieve the rest of the configuration, including interfaces and endpoints.*
- **bNumInterfaces**: `2`
    - *This indicates that the device has **2 interfaces**. An interface defines a functional unit in the device, and multiple interfaces can be used for different purposes.*
- **bConfigurationValue**: `1`
    - *This is the value used by the host to select this configuration. If the device has multiple configurations, the host uses this value to choose the desired one.*
- **iConfiguration**: `0`
    - *This field refers to the **string descriptor index** that provides a human-readable description of the configuration. Since it’s `0`, it means that no string descriptor is available for this configuration.*
- **bmAttributes**: `0xA0`
    - This is a bit field that provides information about the device’s power capabilities and features:
        - **0x80**: *Must be set for all devices compliant with USB 1.1 or higher.*
        - **0x20**: *Indicates that the device **supports remote wakeup**, meaning it can wake the host from a suspended state.*
        - **NOT self-powered**: *The device does **not have its own power source**; it is powered from the USB bus.*
- **bMaxPower**: `49 (98 mA)`
    - *This indicates the maximum amount of power the device will draw from the USB bus. The value `49` means **49 units of 2 mA**, so the device will draw **98 mA** at most.*
    - *This is a relatively low power consumption, which is typical for devices powered directly by the USB bus and not requiring much power.*


### 5th Packet

The **5th packet** in the capture represents a continuation of the USB enumeration process. The host is sending a **GET DESCRIPTOR Request** to the device, asking for the full **Configuration Descriptor**.

![image](https://gist.github.com/user-attachments/assets/1684be7c-4b10-49e6-8235-ef42a0b68a33)


#### **Control Transfer Stage**: Setup (0)

This is the **Setup Stage** of a control transfer, where the host sends the request and awaits the device's response.

#### **Setup Data**:

This part defines the specific request being made by the host:

- **wLength**: `59`
    - This indicates the **total length** of the descriptor that the host is requesting, which is **59 bytes** in this case. The previous request in packet 3 only asked for 9 bytes (the header of the configuration descriptor), while this request is for the full descriptor, which includes the entire configuration (interfaces, endpoints, etc.).

### 6th packet

The **6th packet** in this capture contains the response from the device to the **GET DESCRIPTOR Request (Configuration)** sent in the 5th packet. The host requested a full **Configuration Descriptor**, and in this packet, the device responds with the entire **59 bytes** of the requested descriptor.

![image](https://gist.github.com/user-attachments/assets/0e73586b-640d-483f-9c58-c40eee2d8387)

![image](https://gist.github.com/user-attachments/assets/39a91867-492d-490b-94eb-eea0b3bb3edb)
#### **Packet Overview:**

- **Packet Data Length**: `59 bytes` (This matches the length requested by the host in the previous packet)

#### **USB URB (USB Request Block)**:

- **IRP Status**: `USBD_STATUS_SUCCESS (0x00000000)`
    - The response was successful, and the device sent the requested 59 bytes.
- **Packet Data Length**: `59 bytes`
    - This confirms that the device is sending 59 bytes in response to the request.

#### **Descriptor Analysis**:

The response includes both the **Configuration Descriptor** and related **Interface**, **HID**, and **Endpoint Descriptors**. Let’s break them down:

##### **1. Configuration Descriptor**:

- **bLength**: `9` (Length of this descriptor is 9 bytes)
- **bDescriptorType**: `0x02 (CONFIGURATION)`
- **wTotalLength**: `59` (Total length of the configuration descriptor, including all related interface and endpoint descriptors)
- **bNumInterfaces**: `2` (The device has two interfaces)
- **bConfigurationValue**: `1` (The value used to select this configuration)
- **iConfiguration**: `0` (No string descriptor for configuration)
- **bmAttributes**: `0xA0` (This device is **bus-powered** and supports **remote wakeup**)
- **bMaxPower**: `49` (The device can draw up to **98 mA** from the bus)

##### **2. Interface Descriptor (0.0): Class HID**:

- **bLength**: `9`
- **bDescriptorType**: `0x04 (INTERFACE)`
- **bInterfaceNumber**: `0` (The first interface)
- **bAlternateSetting**: `0` (No alternate setting)
- **bNumEndpoints**: `1` (One endpoint is associated with this interface)
- **bInterfaceClass**: `HID (0x03)` (This is a **Human Interface Device**)
- **bInterfaceSubClass**: `Boot Interface (0x01)`
- **bInterfaceProtocol**: `Keyboard (0x01)` (This is a USB **keyboard** interface)
- **iInterface**: `0` (No string descriptor for this interface)

##### **3. HID Descriptor**:

- **bLength**: `9`
- **bDescriptorType**: `0x21 (HID)`
- **bcdHID**: `0x0110` (This indicates compliance with **HID 1.1**)
- **bCountryCode**: `Not Supported (0x00)` (The device does not support a specific country code)
- **bNumDescriptors**: `1`
- **bDescriptorType**: `HID Report (0x22)`
- **wDescriptorLength**: `54` (The HID Report Descriptor is **54 bytes** long)

##### **4. Endpoint Descriptor (IN)**:

- **bLength**: `7`
- **bDescriptorType**: `0x05 (ENDPOINT)`
- **bEndpointAddress**: `0x81 (IN endpoint 1)`
- **bmAttributes**: `0x03` (This is an **interrupt** endpoint)
- **wMaxPacketSize**: `8` (Maximum packet size is 8 bytes)
- **bInterval**: `10` (The device polls this endpoint every 10 ms)

##### **5. Interface Descriptor (1.0): Class HID**:

- **bLength**: `9`
- **bDescriptorType**: `0x04 (INTERFACE)`
- **bInterfaceNumber**: `1` (The second interface)
- **bAlternateSetting**: `0` (No alternate setting)
- **bNumEndpoints**: `1` (One endpoint is associated with this interface)
- **bInterfaceClass**: `HID (0x03)` (This is a Human Interface Device)
- **bInterfaceSubClass**: `No Subclass (0x00)`
- **bInterfaceProtocol**: `0x00` (No specific protocol)
- **iInterface**: `0` (No string descriptor for this interface)

##### **6. HID Descriptor** (Second HID Interface):

- **bLength**: `9`
- **bDescriptorType**: `0x21 (HID)`
- **bcdHID**: `0x0110` (This indicates compliance with **HID 1.1**)
- **bCountryCode**: `Not Supported (0x00)` (No specific country code support)
- **bNumDescriptors**: `1`
- **bDescriptorType**: `HID Report (0x22)`
- **wDescriptorLength**: `50` (The HID Report Descriptor is **50 bytes** long)

##### **7. Endpoint Descriptor (IN)**:

- **bLength**: `7`
- **bDescriptorType**: `0x05 (ENDPOINT)`
- **bEndpointAddress**: `0x82 (IN endpoint 2)`
- **bmAttributes**: `0x03` (This is an **interrupt** endpoint)
- **wMaxPacketSize**: `8` (Maximum packet size is 8 bytes)
- **bInterval**: `10` (The device polls this endpoint every 10 ms)

### 7th packet frame

![image](https://gist.github.com/user-attachments/assets/64b5355f-032e-48fd-9bab-f64ee893bb23)

#### **USB URB (USB Request Block)**:


- **URB Function**: `URB_FUNCTION_SELECT_CONFIGURATION (0x0000)`
    - The function being performed here is selecting a configuration for the device.

#### **Control Transfer Stage**: Setup (0)

- This is the **setup stage** of a control transfer. The host sends a setup packet to the device, which will later be acknowledged by the device.

#### **Setup Data**:

- **bRequest**: `SET CONFIGURATION (9)`
    - This is the standard **SET CONFIGURATION** request, which tells the device to switch to a specific configuration.
- **bConfigurationValue**: `1`
    - The **Configuration Value** is set to **1**, meaning the host is selecting **Configuration 1** for the device. This value corresponds to the configuration descriptor retrieved in earlier packets (such as packet 6), which described how the device is structured (e.g., interfaces, endpoints, etc.).

### 8th Packet frame

The **8th packet** is the response from the USB device to the **SET CONFIGURATION Request** sent by the host in the **7th packet**. In this packet, the device acknowledges the configuration change. Let’s break it down:

![image](https://gist.github.com/user-attachments/assets/79bfd677-cd88-4f2c-ae7a-d17b5eb86238)
#### **Packet Overview:**

- **Packet Data Length**: `0 bytes` (No data payload, only an acknowledgment)

#### **USB URB (USB Request Block)**:

- **IRP Status**: `USBD_STATUS_SUCCESS (0x00000000)`
    - *This indicates that the configuration change request was successfully processed by the device, and it is now configured according to the host’s instructions.*
- **URB Function**: `URB_FUNCTION_SELECT_CONFIGURATION (0x0000)`
    - *The function performed here is confirming the selection of the device’s configuration.*
- **Control Transfer Stage**: Complete (3)
	- *This indicates that the control transfer, which started with the **SET CONFIGURATION** request, is now complete. The device is fully configured according to the instructions from the host.*
- **Packet Data Length**: `0`
	- *The device sends **0 bytes** as part of this acknowledgment. This is typical in control transfers where the response is simply an acknowledgment of successful completion.*
- **Time from Request**: `0.000972000 seconds`
	- *The device responded very quickly, in less than 1 millisecond, indicating that the configuration was applied almost instantaneously.*

### 9-12th packet frames

In packets 9 to 12, we see transfers with the descriptor type `0xff` and a note about an **Unknown type 7f**. This indicates **vendor-specific commands** or unsupported requests. This is typical when the host or device is trying to perform custom operations or when Wireshark doesn’t recognize the specific packet format.

![image](https://gist.github.com/user-attachments/assets/16617171-827e-4aa3-9e90-963c35ec093f)


### 13-20 packet frames

- Starting from packet 13, the host is requesting **String Descriptors** (e.g., for the manufacturer, product name, etc.):
    - **English (United States)** indicates the **language ID** for the string descriptor.
    - The responses include details like "U" (which is likely a part of the string response), and in later responses, the full string "Usb KeyBoard" is shown, which is likely the **Product String Descriptor** for the USB device.
- String descriptors help identify the device by providing human-readable text for the **manufacturer** and **product name**.

![[Pasted image 20241020120709.png]]

### 21-24 Packet frames

Starting from packet 21, we see communication related to the **HID (Human Interface Device)** class, which is typical for devices like **keyboards or mice**. These packets show:

![image](https://gist.github.com/user-attachments/assets/1a72826f-6822-4875-922f-71d562258fd8)

#### Packets 21 and 22

- **SET_IDLE Request and Response (Packets 21 and 22)**: This command is used to control the device’s idle state. For instance, if the host doesn’t want to receive updates from the device unless there's a change, it can set the idle time. The response packet indicates that the device successfully processed the request.

##### Packets 21

- **bRequest**: `SET_IDLE (0x0a)`
	- This indicates that the host is sending a **SET_IDLE** request. This is used to set the idle rate for the HID device, meaning how often it should send updates to the host (if at all) when no changes occur (e.g., no key presses on a keyboard).
- **wValue**: `0x0000`
	- This sets the **idle duration** to **0**, which typically means that the device should send reports only when there’s a change (i.e., no idle behavior).

![image](https://gist.github.com/user-attachments/assets/00f48bf8-2b18-4ccd-94ef-be83dead9141)

##### Packets 22

![image](https://gist.github.com/user-attachments/assets/ec9fb3e0-a724-4b87-9797-369a25e16e2d)

- **Packet Data Length**: `0`
	- The response contains no data, only an acknowledgment that the request was successfully processed.

#### Packet 23 & 24

- **GET DESCRIPTOR Request HID Report (Packet 23)**: The host is requesting the **HID report descriptor**, which tells the host how the device will send its input data (e.g., key presses )
- **GET DESCRIPTOR Response HID Report (Packet 26)**: The device responds with the **HID report descriptor**, typically a binary format detailing the report structure.

![image](https://gist.github.com/user-attachments/assets/1090dbf7-ec42-4d1d-bff6-1dc5e46fcb90)


### Interrupt transfer frames

From **Frame 45**, we see an **interrupt transfer** from a USB HID device to the host.

![image](https://gist.github.com/user-attachments/assets/ca49aadb-2a2c-4d64-af53-dd4b5894214e)

- **URB Transfer Type**: `URB_INTERRUPT in` - *This is an **interrupt IN transfer**, meaning that the device is sending data to the host.*
- **URB Function**: `URB_FUNCTION_BULK_OR_INTERRUPT_TRANSFER (0x0009)` - *This indicates that the transfer is either a bulk or interrupt transfer, and in this case, it’s specifically an interrupt transfer .*
- **HID Data**: `0200000000000000` - *This is the actual data being sent from the HID device to the host.*


## Filtering data

![tldr](https://y.yarn.co/bd8cc9c5-f4cc-4951-a4db-5a925b50c7f1_text.gif){: .normal }

Having learnt the theory part, its time for doing stuff practically.

![letsgo](https://media1.tenor.com/m/AGOS7ooBUxwAAAAC/yes.gif){: .normal }

For this test, I'm going to type: `flag : {W3ll_d0n3_s33_Y0u_m@d3_1t}` on a blank notepad.

- Open Wireshark and locate the `USBPcap` interface and double click on it.

![image](https://gist.github.com/user-attachments/assets/e427778a-0b44-407b-8b9e-4fbbe63e5c22)

- Start typing on notepad (any other editor anyway) while capturing the traffic.

![image](https://gist.github.com/user-attachments/assets/d405d2fa-a13b-4ea8-969a-37414a40dc3e)

- I successfully captured 17308 packets. Most of this frames are from different devices plugged into my laptop. Specifically I want to focus on frames related to the keyboard. So click on the search icon, select packet details and search for the keyword `keyboard`. When you click on find, you get a hit on packet 4476.

![image](https://gist.github.com/user-attachments/assets/325601b8-dd0e-4b57-84d8-e014643c401f)

- Locate the `Device Address` id of the keyboard in the packet details. Right click on it and apply as a filter. (*Display filter:* `usb.device_address == 6`)

![image](https://gist.github.com/user-attachments/assets/d0632789-cfe0-4374-9306-e210f2c55ae5)

You will now see packets related to keyboard: 

![image](https://gist.github.com/user-attachments/assets/5fc15e97-a49f-44f8-99cc-3fc87016c3a7)

- Specifically, we want to also filter out packets that contain data (keystrokes captured). In this case, the display filter would be (`usb.transfer_type == 0x01`).

```bash
(usb.device_address == 6) && (usb.transfer_type == 0x01)
```
{: .nolineno }

![image](https://gist.github.com/user-attachments/assets/7d7830e5-117a-4889-b8d1-7b61daa68abf)

- Notice that packets with a data length of 8, have some HID data: so to filter for this specific packets, append this filter to the previous filter (`usb.data_len == 8`).

```bash
((usb.device_address == 6) && (usb.transfer_type == 0x01)) && (usb.data_len == 8)
```
{: .nolineno }


![image](https://gist.github.com/user-attachments/assets/12b043c7-34ff-4a79-8ba7-1cfb90d7a401)

- Now this is what we're interested in. There are two ways to go about decoding the keystrokes:

### Method 1

- In the PCAP containing different USB devices, you can use `tshark` to export this packets using the following commands:

```bash
# Linux
tshark -r capture.pcapng -Y '((usb.device_address == 6) && (usb.data_len == 8)) && (usb.transfer_type == 0x01)' -T fields -e usbhid.data > data.txt

# Windows
.\tshark.exe -r capture.pcapng -Y '((usb.device_address == 6) && (usb.data_len == 8)) && (usb.transfer_type == 0x01)' -T fields -e usbhid.data > data.txt
```
{: .nolineno }


Where:

- `-r capture.pcapng`: Specifies the input capture file.
- `-Y 'filter_expression'`: Display filter.
- `-T fields`: Specifies that we want to extract specific fields
- `-e usbhid.data`: Extracts the raw USB data.
- `> data.txt`: Exports the result to a text file.


Eg:

- Save the pcap (File > Save As)
- On a terminal, navigate to `C:\Program Files\Wireshark` (Windows) and export the data as shown: 

```powershell
PS C:\Program Files\Wireshark> .\tshark.exe -r D:\CTFs\Capture\usbtraffic.pcapng -Y '((usb.device_address == 6) && (usb.transfer_type == 0x01)) && (usb.data_len == 8)' -T fields -e usbhid.data


0000000000000000
0000090000000000
0000000000000000
00000f0000000000
0000000000000000
0000040000000000
0000000000000000
00000a0000000000
0000000000000000
00002c0000000000
0000000000000000
2000000000000000
2000330000000000
0000000000000000
00002c0000000000
0000000000000000
2000000000000000
20002f0000000000
2000000000000000
0000000000000000
0200000000000000
02001a0000000000
00001a0000000000
0000000000000000
0000200000000000
0000000000000000
00000f0000000000
0000000000000000
00000f0000000000
0000000000000000
2000000000000000
20002d0000000000
2000000000000000
0000000000000000
0000070000000000
0000000000000000
0000270000000000
0000000000000000
0000110000000000
0000000000000000
0000200000000000
0000000000000000
2000000000000000
20002d0000000000
2000000000000000
0000000000000000
0000160000000000
0000000000000000
0000200000000000
0000000000000000
0000200000000000
0000000000000000
2000000000000000
20002d0000000000
2000000000000000
0000000000000000
0200000000000000
02001c0000000000
0000000000000000
0000270000000000
0000000000000000
0000180000000000
0000000000000000
2000000000000000
20002d0000000000
2000000000000000
0000000000000000
0000100000000000
0000000000000000
0200000000000000
02001f0000000000
00001f0000000000
0000000000000000
0000070000000000
0000000000000000
0000200000000000
0000000000000000
2000000000000000
20002d0000000000
2000000000000000
0000000000000000
00001e0000000000
0000000000000000
0000170000000000
0000000000000000
2000000000000000
2000300000000000
2000000000000000
0000000000000000
```
{: .nolineno }


### Method 2

Alternatively, on Wireshark, you can choose to save only packets related to HID data as shown.

- After applying the display filter, press `ctrl + a` to select all packets
- `ctrl + m` to mark selected packets. Alternatively, click on the edit tab and click on **Mark/Unmark selected** 

![image](https://gist.github.com/user-attachments/assets/c82d6bbc-918a-44ff-ad68-f3e3d049d248)

-  File > Export Specified Packets

![image](https://gist.github.com/user-attachments/assets/2255495c-19bb-45b5-bca1-c0cced2ec06e)

- Name your pcap, export marked packets only and click save:

![image](https://gist.github.com/user-attachments/assets/c4da376b-989b-47de-b655-427eec34c402)

- Load the new pcap on wireshark. Notice it only contains 89 Left over capture data. Also notice that the field you will use to filter using tshark changes to `usb.capdata`. Initially it was `usbhid.data`

![image](https://gist.github.com/user-attachments/assets/b2130473-08ac-461e-ab71-4c30ff1f4581)

- To export packets from this pcap, simply:

```bash
# Linux
tshark -r capture.pcapng -Y '((usb.device_address == 6) && (usb.data_len == 8)) && (usb.transfer_type == 0x01)' -T fields -e usb.capdata > rawdata.txt

# Windows
.\tshark.exe -r capture.pcapng -Y '((usb.device_address == 6) && (usb.data_len == 8)) && (usb.transfer_type == 0x01)' -T fields -e usb.capdata > rawdata.txt
```
{: .nolineno }


eg:

![image](https://gist.github.com/user-attachments/assets/99f2ad4e-24c1-4b39-80a3-4b157dbeb959)

## Decoding HID Data

Once you have exported the HID data, you need to decode it to understand what it represents. In this case, you need to understand the **USB Keycodes**. USB keycodes are standardized values defined by the USB HID (Human Interface Device) protocol that represent individual keys on a keyboard or other input devices. When you press a key, the device sends a corresponding keycode to the computer, which interprets it based on the keycode’s value.

You can refer to the [**USB HID usage tables**](https://usb.org/sites/default/files/documents/hut1_12v2.pdf) (Page 53) for detailed information on key codes.

![image](https://gist.github.com/user-attachments/assets/39e39a80-735e-41cb-91ec-cd903c62d371)

As shown in the screenshot above, Each key on a keyboard corresponds to a specific hexadecimal keycode in the USB HID standard. For example:

- `0x04` represents the letter `A`
- `0x05` represents the letter `B`

Considering manually decoding 89 keystrokes is a tedious process, I thought of way's to automate this process. After doing extensive research, I came across scripts from the following resources:

- [TeamRocketIST CTF Team - *# Hackit 2017 - USB ducker*](https://teamrocketist.github.io/2017/08/29/Forensics-Hackit-2017-USB-ducker/) by the Portuguese Computer Science Students
- [# kaizen-ctf 2018 — Reverse Engineer usb keystrok from pcap file](https://abawazeeer.medium.com/kaizen-ctf-2018-reverse-engineer-usb-keystrok-from-pcap-file-2412351679f4) by [AliBawazeEer](https://abawazeeer.medium.com/?source=post_page-----2412351679f4--------------------------------)

So I decided to slightly advance/modify the script a little with the help of my buddies.


```python
# Import necessary modules
import os
import argparse


# USB HID Keycode to character mapping
KEY_CODES = {
    0x00: ['', ''],  # Reserved (no event indicated)
    0x04: ['a', 'A'],
    0x05: ['b', 'B'],
    0x06: ['c', 'C'],
    0x07: ['d', 'D'],
    0x08: ['e', 'E'],
    0x09: ['f', 'F'],
    0x0A: ['g', 'G'],
    0x0B: ['h', 'H'],
    0x0C: ['i', 'I'],
    0x0D: ['j', 'J'],
    0x0E: ['k', 'K'],
    0x0F: ['l', 'L'],
    0x10: ['m', 'M'],
    0x11: ['n', 'N'],
    0x12: ['o', 'O'],
    0x13: ['p', 'P'],
    0x14: ['q', 'Q'],
    0x15: ['r', 'R'],
    0x16: ['s', 'S'],
    0x17: ['t', 'T'],
    0x18: ['u', 'U'],
    0x19: ['v', 'V'],
    0x1A: ['w', 'W'],
    0x1B: ['x', 'X'],
    0x1C: ['y', 'Y'],
    0x1D: ['z', 'Z'],
    0x1E: ['1', '!'],
    0x1F: ['2', '@'],
    0x20: ['3', '#'],
    0x21: ['4', '$'],
    0x22: ['5', '%'],
    0x23: ['6', '^'],
    0x24: ['7', '&'],
    0x25: ['8', '*'],
    0x26: ['9', '('],
    0x27: ['0', ')'],
    0x28: ['\n', '\n'],  # Enter
    0x2C: [' ', ' '],    # Space
    0x2D: ['-', '_'],    # Hyphen and Underscore
    0x2E: ['=', '+'],    # Equal and Plus
    0x2F: ['[', '{'],    # Left Square Bracket and Left Curly Bracket
    0x30: [']', '}'],    # Right Square Bracket and Right Curly Bracket
    0x33: [';', ':'],    # Semicolon and Colon
    0x34: ["'", '"'],    # Single Quote and Double Quote
    0x36: [',', '<'],    # Comma and Less Than
    0x37: ['.', '>'],    # Period and Greater Than
    0x38: ['/', '?'],    # Slash and Question Mark
    0x29: ['ESC', 'ESC'],  # Escape
    0x2A: ['DEL', 'DEL'],  # Delete (Backspace)
    0x31: ['\\', '|'],   # Backslash and Vertical Bar
    0x35: ['`', '~'],    # Grave Accent and Tilde
    0x4F: ['→', '→'],    # Right Arrow
    0x50: ['←', '←'],    # Left Arrow
    0x51: ['↓', '↓'],    # Down Arrow
    0x52: ['↑', '↑'],    # Up Arrow
    # Additional keycodes for function keys, etc.
    0x39: ['Caps Lock', 'Caps Lock'],
    0x3A: ['F1', 'F1'], 0x3B: ['F2', 'F2'], 0x3C: ['F3', 'F3'], 0x3D: ['F4', 'F4'],
    0x3E: ['F5', 'F5'], 0x3F: ['F6', 'F6'], 0x40: ['F7', 'F7'], 0x41: ['F8', 'F8'],
    0x42: ['F9', 'F9'], 0x43: ['F10', 'F10'], 0x44: ['F11', 'F11'], 0x45: ['F12', 'F12'],
    0x46: ['PrintScreen', 'PrintScreen'], 0x47: ['Scroll Lock', 'Scroll Lock'],
    0x48: ['Pause', 'Pause'], 0x49: ['Insert', 'Insert'], 0x4A: ['Home', 'Home'],
    0x4B: ['PageUp', 'PageUp'], 0x4C: ['Delete Forward', 'Delete Forward'],
    0x4D: ['End', 'End'], 0x4E: ['PageDown', 'PageDown'], 0x53: ['Num Lock', 'Num Lock'],
    # Numeric Keypad keys
    0x54: ['/', '/'], 0x55: ['*', '*'], 0x56: ['-', '-'], 0x57: ['+', '+'],
    0x58: ['Enter', 'Enter'], 0x59: ['1', '1'], 0x5A: ['2', '2'], 0x5B: ['3', '3'],
    0x5C: ['4', '4'], 0x5D: ['5', '5'], 0x5E: ['6', '6'], 0x5F: ['7', '7'],
    0x60: ['8', '8'], 0x61: ['9', '9'], 0x62: ['0', '0'], 0x63: ['.', '.'],
    # Control Keys
    0xE0: ['Left Ctrl', 'Left Ctrl'], 0xE1: ['Left Shift', 'Left Shift'],
    0xE2: ['Left Alt', 'Left Alt'], 0xE3: ['Left GUI', 'Left GUI'],
    0xE4: ['Right Ctrl', 'Right Ctrl'], 0xE5: ['Right Shift', 'Right Shift'],
    0xE6: ['Right Alt', 'Right Alt'], 0xE7: ['Right GUI', 'Right GUI']
}


# Initialize argument parser
parser = argparse.ArgumentParser(description="USB Keyboard Decoder")
parser.add_argument("-f", "--file", required=True, help="Input file to decode keystrokes (e.g., hope.txt)")
args = parser.parse_args()

input_file = args.file  # File name is passed via the command line argument

# Check if the file exists
if not os.path.isfile(input_file):
    print(f"Error: The file '{input_file}' does not exist.")
    exit()

# Read the data from the file, ignoring non-ASCII characters
with open(input_file, 'rb') as file:
    data = file.read().decode('utf-16', errors='ignore')  # Decode as UTF-16 and ignore errors
    lines = data.splitlines()

output = ''  # Initialize an empty string for the decoded keystrokes
previous_keycode = None  # To avoid repeated characters

# Iterate through each line of the data file
for line in lines:
    if len(line) < 16:  # Skip any lines that are too short
        continue

    try:
        # Extract the modifier and key from the data (in hex)
        modifier = int(line[:2], 16)  # First byte for shift/ctrl
        keycode = int(line[4:6], 16)  # Third byte for the actual key
    except ValueError:
        # Skip lines that cannot be converted to integers (invalid data)
        continue

    if keycode == 0:
        continue  # Skip if no key was pressed

    # Handle Backspace (Delete)
    if keycode == 0x2A:  # Backspace
        output = output[:-1]  # Remove the last character
        continue

    # Ignore Arrow Keys
    if keycode in [0x4F, 0x50, 0x51, 0x52]:  # Arrow keys
        continue

    # Prevent repeated characters if the same keycode occurs twice
    if keycode == previous_keycode:
        continue

    previous_keycode = keycode  # Track the last processed keycode

    # Determine if Shift is held (bit 1 in modifier byte)
    shift = (modifier & 0x22) > 0  # Check for both Left Shift and Right Shift (0x02 and 0x20)

    # Translate the keycode to the actual character
    if keycode in KEY_CODES:
        output += KEY_CODES[keycode][shift]

# Print the final decoded output
print("Decoded Keystrokes:")
print(output)
```
{: .nolineno }


> The script might not be 100% accurate as we have noticed an issue with repeated characters (*Line 131-135*). Its probably an issue with some keycode bindings but we're looking into it.  If you have any suggestions on improving/optimizing the script further upon testing, your welcome to share your version on this [Github gist](https://gist.github.com/05t3/2025e52a7b09de1230627a004a135b32).  
{: .prompt-warning }


When you run the script, you get the clear text keystrokes:


```bash
PS D:\CTFs\Capture> python3.11.exe .\final.py -f data.txt
Decoded Keystrokes:
flag : {W3ll_d0n3_s33_Y0u_m@d3_1t}
```
{: .nolineno }


![thatwasntgettingeasy](https://y.yarn.co/5f3959c0-5d8a-4751-b66b-18c8c08fa421_text.gif){: .normal }

## Closing remarks

I hope this blog post was insightful as we put in alot of research and efforts on it. 😊 For challenge creators, I hope you also learnt a thing or two. In the next blog post, we are going to dive into the remaining USB transfer types (Isochronous & Bulk Transfers). 

For example, I'm just thinking to myself *"What happens when i plug in a headset using USB on my computer and maybe play music? Can i recreate the packets and maybe convert data into sound output?"* or, "*What happens when you drag your mouse in a certain motion? Perhaps drawing?*", or "*What happens when i plug in a flash drive in my computer and transfer a file to it? Can I reconstruct the file data and maybe read its content?*"

![thinking](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExMjd3ZXZxNzdwNDRkdmRxc3I3cWhrM2N2cTZ2N3N1ZGY0dWNxbGh3cCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/ZBK7b4vHYyb0n70zJq/200.webp){: .normal }

Perhaps if you are as curious as I am, stay tuned for a new blog post on the same. Do let us know what you think in the comment section. Feel free to share this post in your circles/pages.


## Resources

- [TeamRocketIST CTF Team - *# Hackit 2017 - USB ducker*](https://teamrocketist.github.io/2017/08/29/Forensics-Hackit-2017-USB-ducker/) by the Portuguese Computer Science Students
- [# kaizen-ctf 2018 — Reverse Engineer usb keystrok from pcap file](https://abawazeeer.medium.com/kaizen-ctf-2018-reverse-engineer-usb-keystrok-from-pcap-file-2412351679f4) by [AliBawazeEer](https://abawazeeer.medium.com/?source=post_page-----2412351679f4--------------------------------)
- [# Techniques for analyzing USB protocols](https://github.com/liquidctl/liquidctl/blob/main/docs/developer/techniques-for-analyzing-usb-protocols.md) by [liquidctl](https://github.com/liquidctl)
- [# Capturing USB traffic](https://github.com/liquidctl/liquidctl/blob/main/docs/developer/capturing-usb-traffic.md) by [liquidctl](https://github.com/liquidctl)
- [# Sniffing from USB ports](https://wiki.wireshark.org/USB) by [wireshark](https://wiki.wireshark.org/)
- [USB in a NutShell - # Endpoint Types](https://www.beyondlogic.org/usbnutshell/usb4.shtml#Interrupt) by [beyond logic](https://www.beyondlogic.org/)
- [Universal Serial Bus HID Usage Tables](https://usb.org/sites/default/files/documents/hut1_12v2.pdf) (Page 53)
- [Display Filter Reference: USB](https://www.wireshark.org/docs/dfref/u/usb.html) by [wireshark](https://wiki.wireshark.org/)
- [List of USB ID's](https://www.linux-usb.org/usb.ids) by [Linux USB Project](https://www.linux-usb.org/)
- [ChatGPT](https://chatgpt.com/)

