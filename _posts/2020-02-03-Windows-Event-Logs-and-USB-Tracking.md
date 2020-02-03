---
layout: post
title:  "Windows Event & USB Tracking"
description: "A brief post on tracking USB devices natively with the Windows Event Logs"
date:   2020-02-03 11:00:01 +0000
tags: [Blue-Team, SOC, Monitoring, Logging]
---

![USBs](/assets/USBs.jpg)

## Inspiration
As a small business we are signed up to the UK's CiSP forum, a place where we can engage with other infosec individuals, swap intelligence, ask questions, and gain advice and experience from others within our industry and community.  

For those that do not know of CiSP it describes itself as:

*The Cyber Security Information Sharing Partnership (CiSP) is a joint industry and government initiative set up to exchange cyber threat information in real time, in a secure, confidential and dynamic environment, increasing situational awareness and reducing the impact on UK business.*

Should you be a UK business and wish to sign up? you can do so here:
* [Sign up to CiSP](https://www.ncsc.gov.uk/information/cyber-security-information-sharing-partnership--cisp-)

## The Question

*I was looking to monitor the data/traffic usage of USB devices across the network. Would any one have any advise as to how this may be best achieved in their experience. Currently I have restricted USB usage through Group Policies however I was looking to sample the traffic for those that do have legitimate access rights.  Thanks you in advance for your thoughts.*

We already briefly provided an answer on the forum, but we thought it would made sense to publish a blog post so that all could benefit from our experience.

### Using the Windows Event Logs to Track USBs
Not everyone knows this but you can track USB events inside the normal Windows Event Logging mechanism.  
We can describe how to do this from Windows XP onwards, but since Windows 7 is now deprecated as of 14th Jan 2020, we will stick to modern systems (2012+).

The specific log file is located at:
```
%systemroot%\System32\Winevt\Logs\Microsoft-Windows-DriverFrameworks-UserMode%4Operational.evtx
```
Or more easily accessible from Eventviewer (eventvwr.exe):
```
Application and Services Logs > Microsoft > Windows > DriverFrameworks-UserMode > Operational
```
**However, this particular log is not enabled by default.** 

As such, you need to enable it first 
 * by drilling down to DriverFrameworks-UserMode; 
 * right-clicking on the Operational Log; 
 * and then selecting Properties from the context menu; 
 * when the Log Properties - Operational dialog appears; 
 * select the Enable Logging check box.

![USB Logging](/assets/USB_1.jpg)

You may want to increase the log file size to something more appropriate to your needs.
 
### Note: Extra steps for Server Versions (2012,2016,2019):

Enabling the log is not enough on server editions, **you also need to enable the feature.**
 * Open Roles and Features
 * **User Interfaces and Infrastructure**
 * **Desktop Experience**
 
![Win12 Features](/assets/desktop-experience.png)

## Tracking a USB flash drive connection 
When you connect a USB flash drive to your system, a number of Information and Verbose Level event records are generated in the Operational Log. These records will consist of the following Event IDs:
 * 2003 - Query to load USB Drivers
 * 2004 - Loading Drivers for new Device
 * 2006 - Loading Drivers for new Device
 * 2010 - USB Device Power Event
 * 2100 - Power Operation for USB Device
 * 2101 - Power Operation for USB Device
 * 2102 - Power Operation for USB Device
 * 2103 - Error for Power Operation for USB Device
 * 2105 - Power Operation for USB Device
 * 2106 - Power Operation for USB Device

### Example: Connecting a typical USB flash drive

You should observe the following event ids when connecting a USB flash drive
 * 2100
 * 2003
 * 2004
 * 2006
 
![USB connect](/assets/USB_2.jpg)

As you can observe - various USB event records

### Example: Disconnection Events

You should observe the following events when removing a USB device:
 * 2100
 * 2102

![USB detach](/assets/USB_3.jpg) 

You can clearly see the new events at the top of the log.

## Caveats
While the Operational Log shows USB flash drive connect and disconnect events, that's not the only USB device information this log displays. It may show event records for other USB devices as well. So just be aware of that as you look through the event records.

If you find an Event ID 2003 event record for a specific USB flash drive but don't find a corresponding Event ID 2102 event record, that either means that the USB flash drive is still attached to the system or the system was shut down before the device was removed. The latter makes tracking a disconnect event a bit more tricky, but not impossible. You can investigate recent shutdowns as a means of determining when a USB flash drive was disconnected. You can track recent shutdowns by creating a Custom View and specifying Windows > System as the Event log, User32 as the Event source, and 1074 as the Event ID.

## SIEM / Elastic
Of course its easier to track USB events with the use of a Security Incident and Event Management (SIEM) system.

But we will leave the configuration of SIEM / Elastic searching capabilities to the resident Blue Team.