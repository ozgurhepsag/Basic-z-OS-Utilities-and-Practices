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

I defined the DASD-only log stream as below. OPERLOG log stream name have to be 'SYSPLEX.OPERLOG'

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


## References

https://www.ibm.com/docs/en/zos/2.3.0?topic=operlog-steps-setting-up
https://www.ibm.com/docs/en/zos/2.3.0?topic=medium-using-operlog
https://www.ibm.com/docs/en/zos/2.4.0?topic=utility-define-logstream-keywords-parameters
https://www.ibm.com/docs/en/zos/2.3.0?topic=in-defining-activating-logrec-log-stream
