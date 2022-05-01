## Working with Health Checker

I practise on a monoplex sandbox z/OS system that has no CICS, DB2, IMS, MQ or any ISV products. I wanted to some basic things on the z/OS Health Checker.
</br> </br>
The objective of IBM Health Checker for z/OS is to identify potential problems before they impact your availability. If you omit the output of the checks, even outage could be happened. Firstly, I want to learn about the configuration of the Health Checker on my system.

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
</br> </br>
If you want to retain a historical record of check results, which is optional, but a good idea, you can define and connect to a log stream. When you have a log stream connected, the system writes check results to the log stream every time a check completes.

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

I added the statement below to make this change permanent. 

    LOGGER=ON,LOGSTREAMNAME=HZS.HEALTH.CHECKER.HISTORY 

                                    
