
I have a monoplex that has no CICS, DB2, IMS, MQ or any ISV products. I decided to begin my practice to add one more spool volume for JES2 and then replace my check point data sets both.

## Adding SPOOL Volume

#### Display Current Configuration

Before this practise, I just have one model-9 volume for JES2 spool dataset. Volume serial mask of the spool is 'SPL0' in my system (as you can see below VOLUME=SPL0) and my volume serial is SPL001.

    $D SPL,LONG

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/dspl%2Clong.PNG)

You can see below my spool definition. Track Group is simply an allocation unit for spool (You can see the definitions of the SPOOLDEF parameters from [IBM Knowledge Center](https://www.ibm.com/docs/en/zos/2.4.0?topic=definition-parameter-description-spooldef)). The more TG space, the more space for the spool.

    $D SPOOLDEF

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/%24dspooldef.PNG)

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

    $T SPOOLDEF,TGSPACE
    
![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/%24T%20SPOOLDEF%2CTGSPACE.PNG)

Maximum TGSPACE value should be specified as a multiple of 16288, otherwise JES2 will automatically increase your specification to the next highest multiple of 16288.

#### Start New Spool Volume

When you start new volume with command below, spool data set will be allocated (with maximum space) automatically and reserved.

    $S SPL(SPL002),FORMAT,RESERVED=YES,SPACE=(MAX)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/being%20formatted.PNG)

I could take some time while spool data set is being formatted.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/new%20SPOOL%20data%20set%20formatting.PNG)

Then, my new spool data set formatted succesfully.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/volume%20active%20log.PNG)

However, new volume could not be used, because it is RESERVED. It should be RESERVED=NO.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/%24DSPL%2CLONG%20after.PNG)

You can see that the volume is RESERVED from SP panel of SDSF (ACTIVE-R).

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/status%20ACTIVE-R.PNG)

I made RESERVED=NO and you can see below the Spool Utilization is decreased from 7.7064 to 1.8043.

    $T SPOOL(SPL002),RESERVED=NO

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/RESERVED%3DNO.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/SP%20panel%20after.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/%24DSPL%2CLONG%20RESERVED%3DNO.PNG)

#### Using New Spool Volume

Now, new TSU, STC jobs, and batch jobs could be allocated on our new spool volume. I submitted dummy jobs and I saw that those allocated on my new spool volume (SPL002).

    $DJQ,SPL=(V=SPL002),STATUS 

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/spl002%20status.PNG)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities/blob/main/JES2%20Operations/ss/SP%20panel%20used.PNG)


## Replace Checkpoint Data Sets

will be added.
