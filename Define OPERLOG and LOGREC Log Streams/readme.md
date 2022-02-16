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



## References

https://www.ibm.com/docs/en/zos/2.3.0?topic=operlog-steps-setting-up
https://www.ibm.com/docs/en/zos/2.3.0?topic=medium-using-operlog
https://www.ibm.com/docs/en/zos/2.4.0?topic=utility-define-logstream-keywords-parameters
https://www.ibm.com/docs/en/zos/2.3.0?topic=in-defining-activating-logrec-log-stream
