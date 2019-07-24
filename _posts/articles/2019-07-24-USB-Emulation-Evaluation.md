---
published: false
---
# Summary
{:.no_toc}

* toc
{:toc}


# BadUSB

BadUSB was a talk at blackhat about how USBs can be reprogrammed to become malware. The focus is on overwriting the original firmware of the USB device controller with customized firmware. The USB device controller acts as the interface between the host and the USB device. Customizing the firmware on the USB device controller offers numerous opportunities. By controlling the USB device controller, the user (person programming the firmware) can choose how he/she wants the device to interact with the host. Parts of the USB device can be hidden, for instance, if it is a mass storage device or emulating a mass storage device, parts of the storage can be hidden from the host. If done correctly, the host should not be able to tell what the device really is. You could spoof the firmware so that if the host asks for it, the host is given the firmware the device wants you to think it has on it, rather than the firmware that is actually on it.

It was also mention that badUSB can infect other USB devices and overwrite those devices' firmware as well. This part I am hazy on. I don't understand how exactly the original device can overwrite the USB device controller firmware of a separate USB device. This capability could greatly extend the scope of badUSB. It could be used as gap-jumping malware that infects the peripherial USB devices on a computer making it difficult to detect and get rid of.

Something worth looking into is utilizing the hidden partitions of a mass storage device combined with the USBRubberDucky capabilities to be able to both input and output to a host. Hiding various features from the host is very beneficial for the device, and keyboard emulation can be used for controlling a computer under the radar.

 

# USBRubberDucky

USBRubberDucky is a keyboard emulator. It sends keystrokes to the host as if the device was a keyboard that someone was typing on. The device has the same capabilities as a keyboard which is both good and bad. If a keyboard cannot do something, then the USBRubberDucky cannot do it either. It does allow for the key typing process to be automated making it faster and more accurate than someone typing. The downside is that it is automated, so it cannot react like a human could if something went wrong or something unexpected occurred. A point to highlight is that by acting as a keyboard, the computer trusts the USB device once it is connected, allowing the device to bypass virus scanners. This can enable you to type out an executable and then run it.

The USBRubberDucky is well crafted both through its software and hardware. The software has a nice interface for users and comes with a good amount of documentation/examples online. The hardware comes with an SD card that can be removed, making it easy to add new payloads that will be automated upon connection to a target computer/device. There is also a button that reruns the device so that it could reexecute (retype in the case of keyboard emulation) the payload.

 

# Goodfet / Facedancer

The Facedancer21 has source code provided for various USB capabilities. The ones I have worked with are the keyboard and FTDI emulation. The firmware allows for many different clients to be developed in python. This requires a computer containing the client code to be connected the board, so that the client can be executed from the the host (controlling) computer passing information to the board of what to send to the target computer. Requiring a host computer to tell the board what to do isn't the best way idea of a final product to be used in the field but this could help with Proof of Concept work.

I further developed the keyboard and FTDI client to have more functionality. The keyboard client takes a format file on the host and sends the keystrokes to the target. Moving forward, I would suggest using the USBRubberDucky technology/code for keyboard emulation, because it has been developed much more than the facedancer-keyboard code.

Pros: The facedancer21 has the ability to run many different clients.

Cons: On the current setup, all the clients are in python and are made to interface with the board from the host. That makes it difficult to take the existing python client code and flash it on the board so that the client can be automated on connection to a target (not requiring a host computer to also be connected to the board). Therefore, for automation and not needing a host to be connected, the firmware will need to be changed.

Possibly look into being able to flash the firmware with totally different code so that the board can run one client by itself. Check how power is supplied to the board. The host USB connection supplies power to the board, and the target USB connection may or may not supply power to the board. Understanding how the board gets flashed with the firmware would be very helpful (knowing how to flash multiple files and being able to tweak the flashing process).

See the Facedancer21 UserGuide for more information.

 

# Future

From what I have seen and read, I see a good amount of potential from the BadUSB concept/technology. I think the ideal plan is to know exactly what we want our device to do first and build a board that is customized for its purposes. In reference to keyboard emulation alone, I think USBRubberDucky has a very good setup in regard to their board and software, as I have previously mentioned. If we want to add on to keyboard emulation, then I believe we should consider including the same capabilities and code that the USBRubberDucky uses (ease of changing payloads, possibly a button for rerun, and definitely the encoder/source code). A more general plan is to have a USB device much like a USB thumb drive that we can easily change the USB device controller firmware. Unlike the USBRubberDucky with its SD, the Facedancer21 does not have an offchip storage component. Having a storage component on the device would be preferred so that it could appear to be a normal USB thumb drive if it needs to.
