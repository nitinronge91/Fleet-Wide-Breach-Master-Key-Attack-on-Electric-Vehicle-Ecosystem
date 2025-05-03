# Fleet-Wide Breach: Master Key Attack on Electric Vehicle Ecosystem

# Abstract

We conducted a demonstration of a master key fob attack across multiple vehicle models from a major electric vehicle OEM. This attack leveraged vulnerabilities in both the key fob pairing process and the implementation of Unified Diagnostic Services (UDS). By exploiting these weaknesses, we were able to unpair the legitimate vehicle owner’s key fob and successfully pair a new, unauthorized key fob(from the same OEM) using a combination of UDS-based attacks and RF replay techniques.

Notably, the key fob pairing mechanism is consistent across all vehicle models from the same OEM. As a result, we were able to replicate this attack on various models, highlighting a systemic vulnerability. Our proof-of-concept (PoC) was initially validated at the ECU (Electronic Control Unit) level and later executed in real-world scenarios on actual vehicles. This demonstrates that an attacker could gain access to multiple vehicle models using a single unauthorized key fob, effectively creating a “**Master key**” applicable across the OEM's entire lineup.
We responsibly disclosed this vulnerability to the OEM, and the issue has since been remediated across all affected vehicle models.


# Introduction
The automotive industry has evolved from providing mechanical keys to unlock vehicles to fobs with buttons that can unlock vehicles. 
Now, the most common form of car access revolves around passive entry passive start (PEPS) systems, enabling drivers to enter their car but also start the engine without using a physical key. With the evolution in driver authentication systems, there are many security threats present in RF based key system and attacker had exploited these open loops using relay and replay attacks in recent time.
In our analysis we studied the key fob pairing process with vehicle. For pairing process handshaking between Body control module and key fob take place. Key pairing routine using UDS routine control service need to start and then lock and unlock button from key fob need to press at the same time for pairing key fob for few seconds. When lock or unlock action triggered from key fob, Body control module ECU drives the relay to unlock and lock the vehicle doors.
We exploited weakness in key pairing process, and we were able to unpair the existing paired owner key and paired new key(attacker key). We tried to pair the same key with other vehicle models, and it worked too! So, at a time we were able to lock and unlock different vehicle models of same OEM using single key.

# Key Fob pairing with Vehicle
We studied the key fob pairing process with vehicle. For pairing process to start, UDS (Unified Diagnostic Service) key fob pairing routine($3101) needs to start with routine identifier 0x0206. Once the pairing routine started, the user must press lock and unlock buttons from key fob at the same time for few seconds. Key fob communicate with Body control module ECU on the vehicle with RF (433Mhz). We can verify the paired key fob using UDS service read DID($22) with DID(data identifier) 0xFD02. It returns how many key fobs are paired currently with vehicle.
Maximum of two key fobs can be paired with the vehicle. But there was no specific limit on single key fob can be paired with how many different vehicles. This is the loophole we identified in requirements, and we exploited it. 

# Unpairing Existing paired vehicle owner key fob
As a part of our grey box PEN test approach, we performed UDS fuzzing on Body control module ECU for session control service($10), and through fuzzing we found system supplier specific session (0x10 0x60). We checked the security access in system supplier specific session, and we found that  no security access was implemented in system supplier specific session so, we were successfully able to fuzz routine control service. In system supplier specific session, performed UDS fuzzing for routine control service($31) and for routine identifiers 0x0207 and 0x0206 ECU responded with positive response. 
We checked the effect of routine identifiers on ECU, and we came to know that after executing  routine 0x0207 Body control module is not responding to lock and unlock button press  from key fob i.e Body control module ECU is not switching the relay which are responsible for vehicle locking and unlocking. We confirmed the routine identifier 0x0207 is responsible for key unpairing with vehicle.
so, attacker can unpair existing owner key fob and which can cause inconvenience for owner. CAN communication log is captured and shown in below image.

    System supplier specific session request 	   :6xx 0x02 0x10 0x60
    Positive response			           :6yy 0x06 0x50 0x60 0x00 0x32 0x00 0xc8 
    Start erase key fob routine request 		   :6xx 0x04 0x31 0x01 0x02 0x07
    Positive response 		                   :6yy 0x04 0x71 0x01 0x02 0x07
<div align="center">
  <img width="691" alt="unpair_keyfob" src="https://github.com/user-attachments/assets/e9d164ff-83d9-4cde-b64c-e54ce9840711" />
</div>
 

Also, we confirmed the key fob is unpaired from Body control module ECU by using UDS read DID service($22). Read DID 0xFD02 and ECU responded with positive response with data as 0x00. It means no key paired with ECU currently.


    Read number of key fob paired with Vehicle DID 	:6xx 0x03 0x22 0xFD 0x02
    Positive response 			        :6yy 0x04 0x62 0xFD 0x02 0x00

 <div align="center">
  <img width="611" alt="unpair_confirm" src="https://github.com/user-attachments/assets/d295a4f0-5072-4b44-bef4-3b858fcbc2e9" />
</div>
 


 # Pairing Attacker key fob with vehicle
As stated in section ‘Key Fob pairing with Vehicle’ , for pairing the key fob after UDS routine started, user has to press lock and unlock button from key fob. We pressed lock and unlocked buttons from key fob at the same time and captured the signal using HackRF device and analysed the spectrum. We referred it as **‘key pairing signal’**. 

![image](https://github.com/user-attachments/assets/7f681d41-f229-4339-b69d-da332b6720bd)
*Captured key pairing signal*

After capturing the key pairing signal, we unpaired the paired key fob using UDS routine control services with routine identifier 0x0207 as stated in above section and started key pairing UDS routine with routine identifier 0x0206. Once the routine started, we replayed the captured key pairing signal using HackRF device. This signal can be paired from a distance up to 5 meters.

![image](https://github.com/user-attachments/assets/f0a40363-06c1-4d24-85fd-cfccaccc17e9)
*Replayed the captured key pairing signal to the Body control module ECU for key pairing*


Upon completion of replaying captured signal from HackRF device, we confirmed whether key fob paired with Body control module ECU or not, by pressing buttons from key fob. After pressing lock or unlock button from key fob, relays on Body control module ECU cranked. it indicates that key fob is paired with vehicle. Also, we verified it by UDS service read Data identifier($22) with identifier value 0xFD02. ECU responded with positive response with data as 0x01, it shows that one key fob is paired with vehicle.

    System supplier specific session request 	   :6xx 0x02 0x10 0x60
    Positive response				   :6yy 0x06 0x50 0x60 0x00 0x32 0x00 0xc8 
    Start key pairing routine 	 		   :6xx 0x04 0x31 0x01 0x02 0x06
    Positive response 				   :6yy 0x04 0x71 0x01 0x02 0x06


<div align="center">
  <img width="533" alt="pair_keyfob" src="https://github.com/user-attachments/assets/00357ccd-9388-463b-b77c-813fdd08cb6c" />

</div>


Confirm key fob paired with read UDS service using  DID 0xFD02.ECU responded with positive response with data as 0x01, it shows that one key fob is paired with vehicle.

        Read number of key fob paired with Vehicle DID 	:6xx 0x03 0x22 0xFD 0x02
        Positive response 			     		:6yy 0x04 0x62 0xFD 0x02 0x01


<div align="center">
  <img width="508" alt="pair_confirm" src="https://github.com/user-attachments/assets/af01c7a3-0feb-4aea-bd83-5fea86704275" />


</div>


As there is no security access implemented in system specific UDS session, we were able to start key pairing routine and for key pairing there is no freshness counter or rolling code implemented so, captured pairing signal is valid for any time duration. whenever we replay the captured signal, the key fob gets paired with vehicle. The authentication algorithm  for LF-RF communication between key fob and Body control module ECU is weak. So, after replaying pairing signal from HackRF device key fob is paired with Body control module ECU.

# Pairing Attacker key fob with multiple ECU/Vehicle
After pairing key fob with captured key pairing signal with single ECU. We took Body control module ECU from other two different vehicle models of same OEM and performed the same procedure of unpairing the paired key fob and pairing the same key fob by replaying captured key pairing signal with the help of HackRF device.

![IMG-20240313-WA0005](https://github.com/user-attachments/assets/5cb0dcdc-5554-436c-b8b6-baf4c2cda85d)
*Lab set up for two Body control module ECU from different vehicle models,single key fob paired with two different ECU by replaying captured key fob signal*


We were successfully able to pair same single key fob with three different Body control module ECUs from different vehicle models.
Attacker can pair key fob with any vehicle of these vehicle models with the help of captured key fob signal and can gain unauthorised access to the vehicle. 


# Real Life Attack scenario
This attack is feasible in real-world scenarios across the affected vehicle models. To execute the attack, the adversary requires physical access to the vehicle’s OBD-II port, a compatible Bluetooth dongle, and a key fob model commonly available in the aftermarket.

Physical access to the OBD-II port may be obtained by compromising a third party, such as bribing a service technician at a garage. Once access is granted, the attacker can connect a Bluetooth-enabled dongle to the OBD-II port, enabling remote execution of UDS (Unified Diagnostic Services) commands.

When the legitimate vehicle owner is not within proximity (i.e., outside the keyless entry range), the attacker can remotely initiate the UDS pairing procedure. During this window, the attacker uses a software-defined radio device (e.g., HackRF) to replay the pairing signal. The key fob can be successfully paired from a distance of up to 5 meters from the vehicle.

Once the unauthorized key fob is paired, the attacker gains full access to the vehicle, allowing them to unlock doors, start the engine, and potentially steal the vehicle or valuables stored inside.

<div align="center">
  
![reduced_image](https://github.com/user-attachments/assets/ae793ac6-0f01-4880-8c87-27fea05491ff)


</div>


# Countermeasures
We reported this vulnerability to the vehicle manufacturer and provided a set of recommended mitigation measures to reduce the risk of exploitation. These proposed countermeasures are as follows:

#Implement Strong Encryption: Adopt key fob systems that utilize robust encryption algorithms such as AES (Advanced Encryption Standard). Advanced encryption significantly increases the difficulty for attackers to intercept, analyze, or clone key fob signals.

#Use Rolling Code Technology: Incorporate rolling code (hopping code) mechanisms, which generate a unique transmission code with every use. This ensures that even if an attacker captures a signal, it cannot be reused, thereby preventing replay attacks. This technology is widely adopted in modern key fob systems due to its effectiveness.

#Secure the Pairing Process: Strengthen the initial pairing process between the key fob and the vehicle by implementing secure UDS (Unified Diagnostic Services) procedures. If this process is not properly secured, it may allow unauthorized actors to spoof or re-pair key fobs, compromising vehicle security.







