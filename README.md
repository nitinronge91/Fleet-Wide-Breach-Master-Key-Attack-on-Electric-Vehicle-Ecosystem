# Master_keyFob_Attack

# Abstract

We demonstrated master key fob attack on different vehicle models of electric car manufacturing OEM. In this attack we exploited vulnerability in the key fob pairing process and UDS (Unified Diagnostic Services) service implemented. For master key attack we were able to unpair the existing paired vehicle owner key fob and paired the new key fob with UDS attack and RF replay attack.
Similar key fob pairing technique is used in all vehicle models of an OEM.So, we unpaired the existing paired vehicle owner key fob and paired our key fob with all different vehicle models of same OEM. POC is performed on component level (on ECU) and real-life attack scenario is performed on vehicles. Attacker can access the different vehicle models of an OEM using master key. So, single key fob can work as Master key for all vehicle models.


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
We checked the effect of routine identifiers on ECU, and we came to know that after executing  routine 0x0207 Body control module is not responding to lock and unlock button press  from key fob i.e Body control module ECU is not switching the relay which are responsible for vehicle locking and unlocking. We confirmed the routine identifier 0x0207 is responsible for key pairing with vehicle.
so, attacker can unpair existing owner key fob and which can cause inconvenience for owner. CAN communication log is captured and shown in below image.

    System supplier specific session request 	   :6xx 0x02 0x10 0x60
    Positive response			           :6yy 0x06 0x50 0x60 0x00 0x32 0x00 0xc8 
    Start erase key fob routine request 		   :6xx 0x04 0x31 0x01 0x02 0x07
    Positive response 		                   :6yy 0x04 0x71 0x01 0x02 0x07



![image](https://github.com/user-attachments/assets/8a43531f-8bbb-4d5f-8686-9239625a4363)
                                Unpair existing Keyfob using routine control service

Also, we confirmed the key fob is unpaired from Body control module ECU by using UDS read DID service($22). Read DID 0xFD02 and ECU responded with positive response with data as 0x00. It means no key paired with ECU currently.


    Read number of key fob paired with Vehicle DID 	:6xx 0x03 0x22 0xFD 0x02
    Positive response 			        :6yy 0x04 0x62 0xFD 0x02 0x00

 
 <img width="1175" alt="unpair_confirm" src="https://github.com/user-attachments/assets/9b3c9d2b-6998-42f4-9bc8-0c2867e8caa5">
                                   Confirmed the key fob is unpaired from ECU


 # Pairing Attacker key fob with vehicle:
As stated in section ‘Key Fob pairing with Vehicle’ , for pairing the key fob after UDS routine started, user has to press lock and unlock button from key fob. We pressed lock and unlocked buttons from key fob at the same time and captured the signal using HackRF device and analysed the spectrum. We referred it as **‘key pairing signal’**. 

![MicrosoftTeams-image (19)](https://github.com/user-attachments/assets/7c57cf99-7271-48c4-be0b-58339cf53827)
                            Captured key pairing signal.

After capturing the key pairing signal, we unpaired the paired key fob using UDS routine control services with routine identifier 0x0207 as stated in above section and started key pairing UDS routine with routine identifier 0x0206. Once the routine started, we replayed the captured key pairing signal using HackRF device. This signal can be paired from a distance up to 5 meters.

![MicrosoftTeams-image_](https://github.com/user-attachments/assets/3cbd0b10-16a7-4c89-84db-7d4d836c422c)
                  Replayed the captured key pairing signal to the Body control module ECU for key pairing


Upon completion of replaying captured signal from HackRF device, we confirmed whether key fob paired with Body control module ECU or not, by pressing buttons from key fob. After pressing lock or unlock button from key fob, relays on Body control module ECU cranked. it indicates that key fob is paired with vehicle. Also, we verified it by UDS service read Data identifier($22) with identifier value 0xFD02. ECU responded with positive response with data as 0x01, it shows that one key fob is paired with vehicle.

    System supplier specific session request 	   :6xx 0x02 0x10 0x60
    Positive response				   :6yy 0x06 0x50 0x60 0x00 0x32 0x00 0xc8 
    Start key pairing routine 	 		   :6xx 0x04 0x31 0x01 0x02 0x06
    Positive response 				   :6yy 0x04 0x71 0x01 0x02 0x06


![image](https://github.com/user-attachments/assets/c069c5ab-47c1-42fe-b3ff-81a4387c46b5)
Pair the key fob using routine control service




