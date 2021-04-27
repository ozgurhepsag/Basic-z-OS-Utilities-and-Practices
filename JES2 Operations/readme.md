
I have a monoplex that has no CICS, DB2, IMS, MQ or any ISV products. I decided to begin my practice to add one more spool volume for JES2 and then replace my check point data sets both.

## Adding SPOOL Volume

#### Display Current Configuration

Before this practise, I just have one model-9 volume for JES2 spool data set. Volume serial mask of the spool is 'SPL0' in my system (as you can see below VOLUME=SPL0) and my volume serial is SPL001.

    $D SPL,LONG

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/dspl%2Clong.PNG)

You can see below my spool definition. Track Group Space (TGSPACE) is important here. Track Group is simply an allocation unit for spool (You can see the definitions of the SPOOLDEF parameters from [IBM Knowledge Center](https://www.ibm.com/docs/en/zos/2.4.0?topic=definition-parameter-description-spooldef)). You define DASD data sets in tracks or cylinders. Spool is allocated in Track Groups, by default one Group has three tracks. The more TG space, the more space for the spool.

    $D SPOOLDEF

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24dspooldef.PNG)

#### Add New Volume

I added model-27 volume with VOL=SER=SPL002, then SPL002 volume was made online. The spool volumes must not be SMS-managed. You can see below my JCL to initialize the volume. 

    //INITVOL  JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,               
    //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)                                     
    //INITD    EXEC PGM=ICKDSF,PARM='NOREPLYLU,FORCE'                      
    //SYSPRINT DD   SYSOUT=*                                               
    //SYSIN    DD   *                                                      
     INIT UNIT(1021) VTOC(0,1,14) NVFY PRG VOLID(SPL002)

#### Check TGSPACE value

In SPOOLDEF, it is given that TGSPACE could be maximum 114016 and 50075 of TGSPACE is being active now. Since I wanted to use all of my new spool volume (model 27 ≈ 200000 TGS), I needed to change my maximum TGSPACE value sufficiently. Don't forget to update JES2 PARMLIB member!

    $T SPOOLDEF,TGSPACE=(MAX=260000)
    
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24T%20SPOOLDEF%2CTGSPACE.PNG)

Maximum TGSPACE value should be specified as a multiple of 16288, otherwise JES2 will automatically increase your specification to the next highest multiple of 16288.

#### Start New Spool Volume

When you start new volume with command below, spool data set will be allocated (with maximum space to use whole volume) automatically and reserved.

    $S SPL(SPL002),FORMAT,RESERVED=YES,SPACE=(MAX)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/being%20formatted.PNG)

I could take some time while spool data set is being formatted.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/new%20SPOOL%20data%20set%20formatting.PNG)

Then, my new spool data set formatted succesfully.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/volume%20active%20log.PNG)

However, new volume could not be used, because it is RESERVED. It should be RESERVED=NO.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24DSPL%2CLONG%20after.PNG)

You can see that the volume is RESERVED from SP panel of SDSF (ACTIVE-R).

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/status%20ACTIVE-R.PNG)

I made RESERVED=NO and you can see below the Spool Utilization is decreased from 7.7064 to 1.8043.

    $T SPOOL(SPL002),RESERVED=NO

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/RESERVED%3DNO.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/SP%20panel%20after.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24DSPL%2CLONG%20RESERVED%3DNO.PNG)

#### Using New Spool Volume

Now, new TSU, STC jobs, and batch jobs could be allocated on our new spool volume. I submitted dummy jobs and I saw that those allocated on my new spool volume (SPL002).

    $DJQ,SPL=(V=SPL002),STATUS 

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/spl002%20status.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/SP%20panel%20used.PNG)


## Replace Checkpoint Data Sets

#### Display Current Configuration

Before this practise, I just have two model-3 volumes for JES2 checkpoint data sets. Volume serials of the data sets are CKP001 and CKP002. You can see below the information about checkpoint data sets.

My configuration is running in DUAL mode (that is MODE=DUAL) and JES2 uses dual logging so if any of these data sets becomes corrupted or unavailable there is still second one ready to use.

        $D CKPTDEF 
        
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24D%20CKPTDEF.PNG)

        $D CKPTSPACE

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24D%20CKPTSPACE.PNG)

       $D ACTIVATE
       
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24DACTIVATE.PNG)


#### Add New Volumes

It's not recommended to put it on the same DASD that is used by Spool because it's always high usage device. For the best performance, different volumes should be used for Checkpoint and each Spool. Also, if both Checkpoint Data Sets are stored on DASDs they should be places on different devices, mainly to avoid single point of failure.

I added 2 model-9 volumes with CKP003 and CKP004 volume serials, then those volumes were made online. The checkpoint volumes must not be SMS-managed. You can see below my JCL to initialize the volume. 

        //INITVOL  JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,        
        //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)                              
        //INITD    EXEC PGM=ICKDSF,PARM='NOREPLYLU,FORCE'               
        //SYSPRINT DD   SYSOUT=*                                        
        //SYSIN    DD   *                                               
         INIT UNIT(1022) VTOC(0,1,14) NVFY PRG VOLID(CKP003)            
         INIT UNIT(1023) VTOC(0,1,14) NVFY PRG VOLID(CKP004)            
        /*         

#### Specify the NEWCKPTn data sets

Before using reconfiguration dialog to change Checkpoint data sets, I allocated them using JCL below. Old data sets were 300 cylinders and new ones are 4000 cylinders.

        //NEWCKPT  JOB  (),'OZGUR',CLASS=A,MSGCLASS=H,REGION=0M,         
        //   NOTIFY=&SYSUID,MSGLEVEL=(1,1)                               
        //ALLOC EXEC PGM=IEFBR14                                         
        //CKPT1 DD DSN=SYS1.VBT1.HASPCKP3,UNIT=3390,                     
        // VOLUME=SER=CKP003,DISP=(NEW,KEEP),                            
        // SPACE=(CYL,4000),DCB=(DSORG=PSU)                              
        //CKPT2 DD DSN=SYS1.VBT1.HASPCKP4,UNIT=3390,                     
        // VOLUME=SER=CKP004,DISP=(NEW,KEEP),                            
        // SPACE=(CYL,4000),DCB=(DSORG=PSU)                              

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/CKPT%20volumes.PNG)

These data sets are only defined to JES2 at some time after initialization; they are not allocated until they are put into use during the checkpoint reconfiguration dialog.

        $T CKPTDEF,NEWCKPT1=(DSN=SYS1.VBT1.HASPCKP3,VOL=CKP003)
        $T CKPTDEF,NEWCKPT2=(DSN=SYS1.VBT1.HASPCKP4,VOL=CKP004)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24D%20CKPTDEF%20after.PNG)

It is also recommended to exclude Checkpoint data sets from GRS mechanism so access to them is not blocked by it.

#### Start JES2 reconfiguration dialog

To replace currently used Checkpoints we need to use JES2 reconfiguration dialog.

        $T CKPTDEF,RECONFIG=YES

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/JES2%20Checkpoint%20Reconfiguration%20Options.PNG)

Forwarding is the process of replacing the CKPTn data set name and volume serial or structure name (if defined on a Coupling facility) with the specifications of its NEWCKPTn counterpart (NEWCKPT1 for CKPT1 and NEWCKPT2 for CKPT2), and the copying of the in-storage version of the checkpoint data to this newly defined CKPTn data set. The NEWCKPTn data set specification then becomes null.

I respectively forwarded the CKPT1 and CKPT2 to NEWCKPT1 and NEWCKPT2 on the configurations options like below.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/JES2%20CKPT1%20to%20NEWCKPT1.PNG)

After I replied 'CONT' for the two data sets, I got the following message and the JES2 Checkpoint data sets were replaced succesfully. JES2 is now using new Checkpoint data sets.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/checkpoint%20reconfiguration%20completed.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24D%20CKPTDEF%20after%20reconf.PNG)

CKPT1 and CKPT2 capacity increased from 53988 to 719988.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/%24D%20CKPTSPACE%20after%20reconf.PNG)

You can see from the console some messages about NEWCKPT data sets, because NEWCKPT1 and NEWCKPT2 data sets become unavailable after forwarding.

        *17.33.26          *$HASP256 FUTURE AUTOMATIC FORWARDING OF CKPT1 IS       
        * SUSPENDED UNTIL                                                          
        *         NEWCKPT1 IS RESPECIFIED.                                         
        *         ISSUE $T CKPTDEF,NEWCKPT1=(...) TO RESPECIFY                     

        *17.34.26          *$HASP256 FUTURE AUTOMATIC FORWARDING OF CKPT2 IS   
        * SUSPENDED UNTIL                                                      
        *         NEWCKPT2 IS RESPECIFIED.                                     
        *         ISSUE $T CKPTDEF,NEWCKPT2=(...) TO RESPECIFY                 	   

The last and the most important thing is JES2 PARMLIB member modification. If you forget it there may be serious problems during JES2 restart.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/JES2%20Operations/ss/jes2%20parmblib%20member.PNG)

