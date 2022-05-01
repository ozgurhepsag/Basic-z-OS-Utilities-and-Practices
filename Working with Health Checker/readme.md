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

    LOGGER=ON,LOGSTREAMNAME=HZS.HEALTH.CHECKER.HISTORY 

