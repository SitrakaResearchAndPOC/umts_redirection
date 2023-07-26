# umts_redirection

## For redirection for the first method rrcconnection_request (no code reject):  
* Open  URRCMessages.cpp  
* Search : void handleRrcConnectionRequest(ASN::RRCConnectionRequest_t *msg)  
* And change : sendRrcConnectionSetup(uep,&msg->initialUE_Identity);  
* By sendRrcConnectionReject(uep,&msg->initialUE_Identity);  

    
## For redirection on the second method rrcconnection_release (combined with attach reject with code reject) : 
* Open  URRCMessages.cpp  
* Change void sendRrcConnectionReject(UEInfo *uep) as on diff file : https://github.com/SitrakaResearchAndPOC/umts_redirection/blob/main/URRCMessages.cpp_diff.txt  
  
    
    
## For doing reject search all words reject and uncomment the code  : 
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/Sgsn.cpp  
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/GPRSL3Messages.cpp  
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/SGSNGGSN/Ggsn.cpp  
  
  
  
## List of reject cause : 
* https://github.com/RangeNetworks/OpenBTS-UMTS/blob/master/apps/GetConfigurationKeys.cpp 

