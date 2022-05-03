## Working with Health Checker

I practise on a monoplex sandbox z/OS system that has no CICS, DB2, IMS, MQ or any ISV products. I wanted to do some basic things on the IBM Health Checker.
</br> </br>
The objective of IBM Health Checker for z/OS is to identify potential problems before they impact your availability. If you omit the output of the checks, even outage could happen. Firstly, I want to learn about the configuration of the Health Checker on my system.

### Security Definitions 

Before start working, I made a generic security definition for my user (OZGUR) which belongs to SYS1 group. You can see below the RACF commands that I entered.

    PERMIT HZS.** CLASS(XFACILIT) ID(SYS1) ACCESS(CONTROL)
    SETROPTS RACLIST(XFACILIT) REFRESH
    RL XFACILIT HZS.** ALL  

If you don't have a proper access on XFACILIT resource, you can get the error message about your security access while you want to do something on the Health Checker. For example, when I wanted to refresh a check before security definition, I got the message below.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Working%20with%20Health%20Checker/Images/HZS%20sec.png)

Access Allowed for me was READ because in my environment HZS.** profile has UACC(READ).

### HZSPROC Address Space

With a system address space called HZSPROC, Health Check services is provided. So, HZSPROC must be located in your PROCLIB. Here is my HZSPROC member.

    //HZSPROC  PROC HZSPRM='00'                           
    //HZSSTEP  EXEC   PGM=HZSINIT,REGION=0K,TIME=NOLIMIT, 
    //      PARM='SET PARMLIB=&HZSPRM'                    
    //HZSPDATA DD   DSN=SYS1.VBT1.HZSPDATA,DISP=OLD       
    //        PEND                                        
    //        EXEC HZSPROC                                

It uses HZSPRM00 member in my PARMLIB to start address space. However in my system, HZSPRM00 member is empty. I want to make some changes on that.

### HZSPRMxx Member

I want to add a few statements about Logstream for storing some check records and a policy update. 

#### HZS Logstream

If you want to retain a historical record of check results, which is optional, but a good idea, you can define and connect to a log stream. When you have a log stream connected, the system writes check results to the log stream every time a check completes. Name of that logstream must start with 'HZS'.

    //DEFLOGRC JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,       
    //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)                             
    //DEFINELS EXEC PGM=IXCMIAPU                                   
    //SYSPRINT DD   SYSOUT=*                                       
    //SYSABEND DD   SYSOUT=*                                       
    //SYSIN    DD   *                                              
         DATA TYPE(LOGR) REPORT(YES)                               
         DEFINE LOGSTREAM NAME(HZS.HEALTH.CHECKER.HISTORY)         
              DASDONLY(YES)                                        
              MAXBUFSIZE(65532)                                    
              HIGHOFFLOAD(80)                                      
              LOWOFFLOAD(20)                                       
              STG_SIZE(64000)                                      
              LS_SIZE(32000)                                       
              RETPD(180)                                           
              AUTODELETE(YES)  

I made a plan for setting up a DASD log stream, then activate it.

    F HZSPROC,LOGGER=ON,LOGSTREAMNAME=HZS.HEALTH.CHECKER.HISTORY 
    
    NC0000000 VBT1     22121 21:20:20.03 MCONVBT1 00000290  F HZSPROC,LOGGER=ON,LOGSTREAMNAME=HZS.HEALTH.CHECKER.HISTORY            
    N 0000000 VBT1     22121 21:20:20.70          00000290  IEF196I IGD101I SMS ALLOCATED TO DDNAME (SYS00009)                      
    N 0000000 VBT1     22121 21:20:20.70          00000290  IEF196I         DSN (IXGLOGR.HZS.HEALTH.CHECKER.HISTORY.VBTPLEX  )      
    N 0000000 VBT1     22121 21:20:20.71          00000290  IEF196I         STORCLAS (SCLOGR) MGMTCLAS (        ) DATACLAS (        
    N 0000000 VBT1     22121 21:20:20.71          00000290  IEF196I )                                                               
    N 0000000 VBT1     22121 21:20:20.71          00000290  IEF196I         VOL SER NOS FOR DATA COMPONENT= LOGR01                  
    M 0000000 VBT1     22121 21:20:21.52          00000290  IXG283I STAGING DATASET IXGLOGR.HZS.HEALTH.CHECKER.HISTORY.VBTPLEX 602  
    D                                         602 00000290  ALLOCATED NEW FOR LOGSTREAM HZS.HEALTH.CHECKER.HISTORY                  
    E                                         602 00000290  CISIZE=4KB, SIZE=250MB,320KB                                            
    N 0000000 VBT1     22121 21:20:21.64          00000290  IEF196I IGD103I SMS ALLOCATED TO DDNAME SYS00010                        
    M 0000000 VBT1     22121 21:20:22.25 STC05049 00000090  HZS0344I THE LOGGER REQUEST HAS COMPLETED. 604                          
    E                                         604 00000090  LOG STREAM HZS.HEALTH.CHECKER.HISTORY IS CONNECTED                      

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Working%20with%20Health%20Checker/Images/HZS%20logstream%20list.png)

I added the statement below to HZSPRM to make this change permanent. 

    LOGGER=ON,LOGSTREAMNAME=HZS.HEALTH.CHECKER.HISTORY 

Normally before this change, when you want to type 'L' (ListHistory) line command from CK panel on SDSF, it is not available. Now, after adding and activating the HZS logstream, you can see the history of the specified check.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Working%20with%20Health%20Checker/Images/Check%20History.png)
                                    
#### Update Policy

Because I am working on a monoplex sandbox environment, I don't want to see the XCF_CDS_SPOF check as a red WTOR on my log and console.
</br> </br>
I see the 'XCF_CDS_SPOF' check in the console and SYSLOG as below.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Working%20with%20Health%20Checker/Images/XCF_CDS_SPOF%20before.png)

I do not want to see this check like that in my console. It also sticks on my console until I delete it (or resolve the check). Therefore, I updated the policy for that check like below in my HZSPRM parmlib member.

       ADDREPLACE POLICY STMT(SYSLOG01)              
         UPDATE CHECK(IBMXCF,XCF_CDS_SPOF)           
         WTOTYPE(INFORMATIONAL)                      
         REASON('Sandbox System')                    
         DATE(20220501)                              

Then, I entered the command below to make this change permanent.

    F HZSPROC,REPLACE,PARMLIB=00 (or F HZSPROC,ADD,PARMLIB=xx)

Now, I don't see XCF_CDS_SPOF check as red WTO message in my console.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/Working%20with%20Health%20Checker/Images/XCF_CDS_SPOF%20after.png)

## References

https://www.ibm.com/docs/en/zos/2.3.0?topic=guide-using-health-checker-zos </br>
