# umts_redirection
## ASN file
https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/ASN/rrc.asn1

## REDIRECT V1 : For redirection for the first method rrcconnection_request (no code reject):  
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



* diff file at : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages_diff0.txt
* cpp modified : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_reject0

    
## For redirection on the second method rrcconnection_release (combined with attach reject with code reject) : 
* Open  URRCMessages.cpp  
* Change void sendRrcConnectionReject(UEInfo *uep) as on diff file : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_diff.txt
* cpp modified : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_reject
    
    
## For doing reject search all words reject and uncomment the code  : 
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/Sgsn.cpp  
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/GPRSL3Messages.cpp  
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/Ggsn.cpp  
  
  
  
## List of reject cause : 
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/apps/GetConfigurationKeys.cpp 

## Doing diff file 
* https://www.linuxtricks.fr/wiki/utiliser-diff-et-patch-sous-linux#:~:text=Introduction,les%20diff%C3%A9rences%20ligne%20par%20ligne.
