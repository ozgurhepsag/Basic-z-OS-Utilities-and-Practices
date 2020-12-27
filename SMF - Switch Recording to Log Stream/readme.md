## Switch SMF Recording to Log Stream

This is the short explanation of my practice. 

### Overview

I have small z/OS system that has no CICS, DB2, IMS, MQ or any ISV products. My target is to switch SMF recording from data set to DASD-only log stream. So, we have to create new PARMLIB member. We should decide name of the log streams, record types that we want to record, SMF parameters, SMF exits and etc for new member. Then, policies need to be defined while creating log streams using IXCMIAPU utility. Our log streams are ready to switch use at the end.

### Prerequisites

There are two prerequisites so that in your system you can perform this practice. These are:

- Couple data sets have already defined in the system.
- SMS have to be installed and activeted. System logger requires that.

### Phases

#### 1- Create New PARMLIB Member

Previously, I have been using MAN data sets in my system to record SMF records.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/man%20datasets.PNG)

Firstly, new member must be created in the PARMLIB. I named this new member 'SMFPRMLS' (SMFPRMxx). Key thing in this member is to code 'RECORDING(LOGSTREAM)'. Then, I put 2 log streams. One of these is default log stream that records all of the SMF records and other is to keep records that records just SMF records for SCRT report.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/SMFPRMLS.png)

Other parameters (record types, exits, interval parameters and etc.) nearly same with my old SMFPRM member. You can obtain two of these members in this repository. You can go to [IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/SSLTBW_2.1.0/com.ibm.zos.v2r1.ieae200/smfparm.htm) to get further information for the SMFPRMxx parameters.

#### 2- Create Couple Data Sets

I already have couple datasets in my system, so I don't do anything for this step. But, I put here my JCL to create couple datasets. (Don't forget to code couple data sets in COUPLExx member.)

    //DCDSLOGR JOB (SPG1,1),'DEFINE CDS',CLASS=A,MSGLEVEL=(1,1),MSGCLASS=X,  
    // NOTIFY=&SYSUID,REGION=0M                                              
    //******************************************************************     
    //STEP10   EXEC PGM=IXCL1DSU                                             
    //STEPLIB  DD   DSN=SYS1.MIGLIB,DISP=SHR                                 
    //SYSPRINT DD   SYSOUT=*                                                 
    //SYSIN    DD   *                                                        
         DEFINEDS SYSPLEX(VBTPLEX)                                            
                  DSN(SYS1.VBT1.LOGR.CDS01) VOLSER(CPL001)                 
                  CATALOG STORCLAS(SCNONSMS)                                 
              DATA TYPE(LOGR)                                                
                  ITEM NAME(LSR) NUMBER(2000)                               
                  ITEM NAME(LSTRR) NUMBER(10)                               
                  ITEM NAME(DSEXTENT) NUMBER(10)                            
                  ITEM NAME(SMDUPLEX) NUMBER(1)                             
         DEFINEDS SYSPLEX(VBTPLEX)                                            
                  DSN(SYS1.VBT1.LOGR.CDS02) VOLSER(CPL002)                 
                  CATALOG STORCLAS(SCNONSMS)                                 
              DATA TYPE(LOGR)                                                
                  ITEM NAME(LSR) NUMBER(2000)                 
                  ITEM NAME(LSTRR) NUMBER(10)                 
                  ITEM NAME(DSEXTENT) NUMBER(10)              
                  ITEM NAME(SMDUPLEX) NUMBER(1)
    
#### 3- Prepare DFSMS

SMS manages the DASD log data sets and staging data sets, because system logger uses SMS to allocate data sets, such as the DASD log stream and staging data sets. 

Therefore, I created just a storage group and a storage class for log streams and ACS routines need to be updated so that right classes assign to log streams. Then, SCDS have to be translated, validated and activated. (I want to prepare another basic practice for this subject in the future.)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/storclas.png)

IXGLOGR is the default HLQ for all of the log streams.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/storgrp.png)

SCLOGR storage class and SGLOGR storage group was created for the log streams and LOGR01 volume was initialized.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/add%20volumes.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/storage%20groups.png)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/listvolumes%20in%20stglgr.png)

#### 4- Define Log Streams and Policies

I defined two log streams that names is the same with the ones in the SMFPRMLS member except HLQ. HLQ must be 'IFASMF' for log streams. You can see below my JCL for this step.

    //DEFLOGR  JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,        
    //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)                              
    //DEFINELS EXEC PGM=IXCMIAPU                                    
    //SYSPRINT DD   SYSOUT=*                                        
    //SYSABEND DD   SYSOUT=*                                        
    //SYSIN    DD   *                                               
        DATA TYPE(LOGR) REPORT(YES)                                
        DEFINE LOGSTREAM NAME(IFASMF.VBT1.SCRT)                    
            DESCRIPTION(SYS_SCRT_RECORDS)                          
            DASDONLY(YES)                                          
            STG_SIZE(2000)                                         
            LS_SIZE(10000)                                         
            AUTODELETE(YES)                                        
            RETPD(180)                                             
        DEFINE LOGSTREAM NAME(IFASMF.VBT1.DEFAULT)                 
            DESCRIPTION(SYSTEM_OTHER_REC)                          
            DASDONLY(YES)                                          
            STG_SIZE(50000)                                        
            LS_SIZE(250000)                                        
            AUTODELETE(YES)        
            RETPD(60)              
    
Because I want to use DASD-only log stream, DASDONLY(YES) parameter should not be forgotten.

There are many parameter to define log stream. Here is the description of the parameters that I use: <br> <br>
**NAME:** Name of the log stream <br>
**DASDONLY:** Type of log stream <br>
**RETPD:** Number of days that the system logger retains messages before it deletes them <br>
**AUTODELETE:** Automatic delete option <br>
**LG_SIZE:** Size, in 4 KB blocks, of the offload data sets for the log stream <br>
**STG_SIZE:** Size, in 4 KB blocks, of the staging data sets for the log stream <br>

After log stream defined, you can see out log streams are ready to use. Here is the output of the 'D LOOGER,L' command:

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/d%20logger%2Cl.PNG)

#### 5- Switch SMF PARMLIB Member

Now, we can use the our new PARMLIB member (SMFPRMLS) after enter 'SET SMF=LS' command from SDSF or console.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/set%20smf.PNG)

Here is some output of the console commands related to log streams after switch:

    D SMF
    
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/d%20smf.PNG)

    D LOGGER

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/d%20logger%2Cl%20v2.PNG)

    D LOGGER,CONNECTION
    
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/d%20logger%2Cconnection.PNG)

In IEASYSxx (IEASYS00 in my system) member's SMF parameter should be updated like 'SMF=LS', if we want to use log streams after IPLs.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/IEASYS.png)

Do not delete your old MAN data sets and SMF PARMLIB member that used to recording SMF records to data sets. Because, you can switch recording types via 'SET SMF=LS' or 'SET SMF=DS' operator commands.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/SMF%20-%20Switch%20Recording%20to%20Log%20Stream/Images/PARMLIB.png)
