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



![image](https://github.com/user-attachments/assets/332f3f48-9f41-46ad-b892-68105964e63c)
Unpair existing Keyfob using routine control service

Also, we confirmed the key fob is unpaired from Body control module ECU by using UDS read DID service($22). Read DID 0xFD02 and ECU responded with positive response with data as 0x00. It means no key paired with ECU currently.


    Read number of key fob paired with Vehicle DID 	:6xx 0x03 0x22 0xFD 0x02
    Positive response 			        :6yy 0x04 0x62 0xFD 0x02 0x00

 ![image](https://github.com/user-attachments/assets/ab45e316-3bab-41d9-b458-0e26073daeb5)
Confirmed the key fob is unpaired from ECU


 # Pairing Attacker key fob with vehicle:
As stated in section ‘Key Fob pairing with Vehicle’ , for pairing the key fob after UDS routine started, user has to press lock and unlock button from key fob. We pressed lock and unlocked buttons from key fob at the same time and captured the signal using HackRF device and analysed the spectrum. We referred it as **‘key pairing signal’**. 

![image](https://github.com/user-attachments/assets/7f681d41-f229-4339-b69d-da332b6720bd)
Captured key pairing signal.

After capturing the key pairing signal, we unpaired the paired key fob using UDS routine control services with routine identifier 0x0207 as stated in above section and started key pairing UDS routine with routine identifier 0x0206. Once the routine started, we replayed the captured key pairing signal using HackRF device. This signal can be paired from a distance up to 5 meters.

![image](https://github.com/user-attachments/assets/f0a40363-06c1-4d24-85fd-cfccaccc17e9)
Replayed the captured key pairing signal to the Body control module ECU for key pairing


Upon completion of replaying captured signal from HackRF device, we confirmed whether key fob paired with Body control module ECU or not, by pressing buttons from key fob. After pressing lock or unlock button from key fob, relays on Body control module ECU cranked. it indicates that key fob is paired with vehicle. Also, we verified it by UDS service read Data identifier($22) with identifier value 0xFD02. ECU responded with positive response with data as 0x01, it shows that one key fob is paired with vehicle.

    System supplier specific session request 	   :6xx 0x02 0x10 0x60
    Positive response				   :6yy 0x06 0x50 0x60 0x00 0x32 0x00 0xc8 
    Start key pairing routine 	 		   :6xx 0x04 0x31 0x01 0x02 0x06
    Positive response 				   :6yy 0x04 0x71 0x01 0x02 0x06



![image](https://github.com/user-attachments/assets/d9c0a950-ffa1-4797-8aed-7e89b04e53a5)

Pair the key fob using routine control service

Confirm key fob paired with read DID 0xFD02.

        Read number of key fob paired with Vehicle DID 	:6xx 0x03 0x22 0xFD 0x02
        Positive response 			     		:6yy 0x04 0x62 0xFD 0x02 0x01


![image](https://github.com/user-attachments/assets/09b7f08f-e642-480e-aaf4-c9d84878c3b2)
Confirm key fob paired with ECU

As there is no security access implemented in system specific session, we were able to start key pairing routine and for key pairing there is no freshness counter or rolling code implemented so, captured pairing signal is valid for any time duration. whenever we replay the captured signal, the key fob gets paired with vehicle. The authentication algorithm  for LF-RF communication between key fob and Body control module ECU is weak. So, after replaying pairing signal from HackRF device original key fob is paired with Body control module ECU. 

# Pairing Attacker key fob with multiple ECU/Vehicle
After pairing key fob with captured key pairing signal with single ECU. We took Body control module ECU from other two different vehicle models of same OEM and performed the same procedure of unpairing the paired key fob and pairing the same key fob by replaying captured key pairing signal with the help of HackRF device.

![IMG-20240313-WA0005](https://github.com/user-attachments/assets/5cb0dcdc-5554-436c-b8b6-baf4c2cda85d)

Lab set up for two Body control module ECU from different vehicle models,single key fob paired with two different ECU by replaying captured key fob signal.

We were successfully able to pair same single key fob with three different Body control module ECUs from different vehicle models.

Attacker can pair key fob with any vehicle of these vehicle models with the help of captured key fob signal and can gain unauthorised access to the vehicle. 


# Real Life Attack scenario:
In real life this attack is possible on these vehicle models. For this attack attacker needs physical access of OBDII port to connect bluetooth dongle and key fob of any of these vehicle models which will be easily available in market. For physical access of OBDII port, attacker can bribe service technician at garage can get physical access to OBD port. Once he gets access to OBD port, attacker can connect bluetooth dongle to OBD port which will execute the UDS commands remotely. When the vehicle owner is not near by the vehicle proximity, attacker can execute the UDS commands to start UDS pairing routine remotely. Once the pairing routine started, he can replay the key pairing signal using HackRF device remotely. Key fob can be paired from distance up to 5 meters from the vehicle.
Once the key fob is paired with vehicle, attacker can gain unauthorised access to vehicle. With this access attacker can steal the vehicle or belongings of the owner kept inside the vehicle. 


# Countermeasures:
We reported this vulnerability to the vehicle manufacturer and proposed the mitigation measures that could minimize the risk of this attack. Below are the mitigation measures.
•	Use Advanced Encryption: Choose key fob systems that employ strong encryption methods, such as AES (Advanced Encryption Standard). These encryption techniques make it extremely difficult for attackers to intercept and clone the signals. 
•	Rolling Code Technology: This technology changes the code transmitted by the key fob with each use. Even if an attacker intercepts one code, it won't work again, making it very secure. Most modern key fobs use this technology. 
•	Secure Pairing: Ensure that your key fob and the receiver have a secure initial pairing process with protected UDS services. If the initial pairing isn't secure, attackers might be able to mimic your key fob.








