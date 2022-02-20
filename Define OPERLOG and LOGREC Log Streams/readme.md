## Defining OPERLOG and LOGREC Log Streams

I practise on a monoplex sandbox z/OS system that has no CICS, DB2, IMS, MQ or any ISV products. I decided to activate OPERLOG and switch LOGREC from data set to DASD-only logstream in my system. 

### Activating OPERLOG

I am going to use OPERLOG as a DASD-only log stream. This method is only suitable for a single system sysplex because a DASD-only log stream is single-sysplex in scope and you can only have one OPERLOG log stream per sysplex. This means that if you make OPERLOG a DASD-only log stream, only one system can access it and in my sandbox environment, I have just one system. 

#### Security Definitions

Before define OPERLOG log stream, the security definitions needed to be made. In my system, RACF is being used as security product. Therefor, I used the definitions below.

    RDEFINE LOGSTRM SYSPLEX.OPERLOG UACC(READ)
    PERMIT SYSPLEX.OPERLOG CLASS(LOGSTRM) ID(OZGUR) ACCESS(UPDATE)
    SETROPTS RACLIST(LOGSTRM) REFRESH  

You need to have UPDATE access on SYSPLEX.OPERLOG profile in the LOGSTRM general resource. 

**Note that** you cannot define any log stream if you don't have the acces below.

    RDEFINE LOGSTRM SYSPLEX.OPERLOG UACC(READ)
    PERMIT MVSADMIN.LOGR CLASS(FACILITY) ID(OZGUR) ACCESS(READ)
    SETROPTS RACLIST(FACILITY) REFRESH  
           
#### Define Log Stream

I defined the DASD-only log stream as below. OPERLOG log stream name have to be 'SYSPLEX.OPERLOG'.

    //DEFLOGRO JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,
    //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)             
    //DEFINELS EXEC PGM=IXCMIAPU                   
    //SYSPRINT DD   SYSOUT=*                       
    //SYSABEND DD   SYSOUT=*                       
    //SYSIN    DD   *                              
         DATA TYPE(LOGR) REPORT(YES)               
         DEFINE LOGSTREAM NAME(SYSPLEX.OPERLOG)    
              HLQ(IXGLOGR)                         
              DASDONLY(YES)                        
              LS_SIZE(128000)                      
              STG_SIZE(64000)                      
              LOWOFFLOAD(0)                        
              HIGHOFFLOAD(80)                      
              STG_DUPLEX(YES)                      
              RETPD(180)                           
              AUTODELETE(YES)  

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/OPERLOG%20log%20stream.png)

After define the log stream, we can also check with MVS command as below. It should be AVAILABLE.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/D%20LOGGER%20Operlog.png)

#### Specify Hardcopy Medium

Hardcopy medium have to be changed manually with 'VARY OPERLOG,HARDCPY' command, to activate OPERLOG.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/v%20operlog%2Chardcpy.png)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/D%20LOGGER%20Operlog%202.png)

Now, OPERLOG can be used by entering 'LOG' command from SDSF.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/OPERLOG.png)

Also, OPERLOG could be turned of by 'VARY OPERLOG,HARDCOPY,OFF' operator command.

#### Update CONSOLxx member

Finally, HARDCOPY statement in CONSOLxx member have to be updated before IPL to keep OPERLOG active.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/CONSOL00.png)

I prefer running with both SYSLOG and OPERLOG as hardcopy.

### Defining LOGREC Log Stream

Like OPERLOG log stream, I am going to define and use a DASD-only log stream for LOGREC recording. I don't implement any automation for copying and clearing the LOGREC data set. So, it can be fulled after a while and some data could be lost. That's why, I am going to change LOGREC recording medium to log stream for now. I am planning to schedule some jobs that will be submitted one a day for copying SYSLOG, OPERLOG, LOGREC and etc. 

#### Security Definitions

Before define LOGREC log stream, the security definitions needed to be made also.

    RDEFINE LOGSTRM SYSPLEX.LOGREC.ALLRECS UACC(READ)
    PERMIT SYSPLEX.LOGREC.ALLRECS CLASS(LOGSTRM) ID(OZGUR) ACCESS(UPDATE)
    SETROPTS RACLIST(LOGSTRM) REFRESH  

#### Define DASD-only Log Stream

I defined the DASD-only log stream as below. OPERLOG log stream name have to be 'SYSPLEX.LOGREC.ALLRECS'.

        //DEFLOGRC JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,   
        //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)                         
        //DEFINELS EXEC PGM=IXCMIAPU                               
        //SYSPRINT DD   SYSOUT=*                                   
        //SYSABEND DD   SYSOUT=*                                   
        //SYSIN    DD   *                                          
             DATA TYPE(LOGR) REPORT(YES)                           
             DEFINE LOGSTREAM NAME(SYSPLEX.LOGREC.ALLRECS)         
                  HLQ(IXGLOGR)                                     
                  DASDONLY(YES)                                    
                  LS_SIZE(64000)                                   
                  STG_SIZE(32000)                                  
                  LOWOFFLOAD(0)                                    
                  HIGHOFFLOAD(80)                                  
                  STG_DUPLEX(YES)                                  
                  RETPD(180)                                       
                  AUTODELETE(YES)  
                  
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/logrec%20log%20stream.png)

#### Change LOGREC Recording Medium

Normally, I used LOGREC data set before changing recordin medium. You can check your LOGREC as below (with DISPLAY LOGREC command).

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/D%20LOGREC.png)

Now, we are ready to change the LOGREC recording medium with the 'SETLOGRC LOGSTREAM' operator command.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/logrec%20log%20stream%20medium.png)

After changing the recording medium, the log stream that we defined for LOGREC became in use.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/logrec%20log%20stream%20inuse.png)

        14:34:56.45 OZGUR    00000290  SETLOGRC LOGSTREAM                                                 
        14:34:57.09          00000290  IEF196I IGD101I SMS ALLOCATED TO DDNAME (SYS00008)                 
        14:34:57.09          00000290  IEF196I         DSN (IXGLOGR.SYSPLEX.LOGREC.ALLRECS.VBTPLEX      ) 
        14:34:57.09          00000290  IEF196I         STORCLAS (SCLOGR) MGMTCLAS (        ) DATACLAS (   
        14:34:57.09          00000290  IEF196I )                                                          
        14:34:57.09          00000290  IEF196I         VOL SER NOS FOR DATA COMPONENT= LOGR01             
        14:34:57.90          00000290  IXG283I STAGING DATASET IXGLOGR.SYSPLEX.LOGREC.ALLRECS.VBTPLEX 734 
                         734 00000290  ALLOCATED NEW FOR LOGSTREAM SYSPLEX.LOGREC.ALLRECS                 
                         734 00000290  CISIZE=4KB, SIZE=125MB,160KB                                       
        14:34:58.02          00000290  IEF196I IGD103I SMS ALLOCATED TO DDNAME SYS00009                   
        14:34:58.63          00000290  IEF196I IEF285I   SYS1.VBT1.LOGREC                             KEPT
        14:34:58.64          00000290  IEF196I IEF285I   VOL SER NOS= PAG001.                             
        14:34:58.64 OZGUR    00000090  IFB097I LOGREC RECORDING MEDIUM CHANGED FROM DATASET   TO LOGSTREAM

#### Update IEASYSxx member

Lastly, LOGREC parameter in the IEASYSxx member should be altered to LOGSTREAM (from SYS1.&SYSNAME..LOGREC dataset name) like below. That keeps LOGREC log stream in use after IPLs.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Define%20OPERLOG%20and%20LOGREC%20Log%20Streams/Images/logrec%3Dlogstream.png)

## References

https://www.ibm.com/docs/en/zos/2.3.0?topic=operlog-steps-setting-up </br>
https://www.ibm.com/docs/en/zos/2.3.0?topic=medium-using-operlog </br>
https://www.ibm.com/docs/en/zos/2.4.0?topic=utility-define-logstream-keywords-parameters </br>
https://www.ibm.com/docs/en/zos/2.3.0?topic=in-defining-activating-logrec-log-stream
