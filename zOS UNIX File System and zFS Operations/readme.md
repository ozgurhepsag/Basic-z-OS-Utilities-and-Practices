## Create and Format a zFS Aggregate

First, use IDCAMS (or IEFBR14) to create a VSAM linear data set. A zFS file system is created in a zFS aggregate (which is a VSAM linear data set). 

		//CREZFS   JOB (VBT),'OZGUR',MSGCLASS=X,NOTIFY=&SYSUID                  
		//*                                                                     
		//********************************************************************  
		//* ALLOCATE A ZFS AGGREGATE (ACTUALLY A VSAM LINEAR DATASET WITH 8     
		//* KB BLOCKS).                                                         
		//* NOTES: - THERE ARE 6 8 KB PER TRACK -> 90 8 KB PER CYL.             
		//*        - WHEN THE TOTAL SIZE OF A FILE IS GREATER THAN 4 GB         
		//*          (EXTENDED FORMAT) THE DFSMS DATACLASS MUST PROVIDE         
		//*          EXTENDED ADDRESSABILITY.                                   
		//********************************************************************  
		//DEFINE   EXEC PGM=IDCAMS                                              
		//SYSPRINT DD SYSOUT=*                                                  
		//SYSIN    DD *,SYMBOLS=CNVTSYS                                         
		 SET MAXCC=0                                                            
		 DEFINE CLUSTER(NAME(&SYSUID..FIRST.ZFS) -                              
				LINEAR SHAREOPTIONS(3) VOLUMES(USR001) CYL(1))                  

Then, VSAM linear data set need to be formatted as a compatibility mode aggregate and create a file system in the aggregate using ioeagfmt or ioefsutl format utility. When using compatibility mode aggregates, the aggregate and the file system are created at the same time. For simplicity, we refer to a file system in a compatibility mode aggregate as a compatibility mode file system. You can also create a compatibility mode aggregate by using the zfsadm define and zfsadm format commands. 

		//ZFSFORMT JOB (VBT),'OZGUR',MSGCLASS=X,NOTIFY=&SYSUID         
		//FORMAT  EXEC PGM=IOEAGFMT,                                   
		//             PARM=('-aggregate OZGUR.FIRST.ZFS -compat')     
		//SYSPRINT DD SYSOUT=*                                         
		//STDOUT   DD SYSOUT=*                                         
		//STDERR   DD SYSOUT=*                                         
		//SYSUDUMP DD DUMMY                                            
		//CEEDUMP  DD DUMMY                                            

or 

		//ZFSFORMT JOB (VBT),'OZGUR',MSGCLASS=X,NOTIFY=&SYSUID    
		//FORMAT  EXEC PGM=IOEFSUTL,REGION=0M,                     
		// PARM=('format -aggregate OZGUR.FIRST.ZFS')             
		//SYSPRINT DD SYSOUT=H                                     

## Mount zFS File System

You need to mount file systems to mount points that you prepared before to use file system. You can create a mount point using the mkdir shell command or MKDIR TSO/E command. 

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/zOS%20UNIX%20File%20System%20and%20zFS%20Operations/ss/mountpoint.png)

or

		MKDIR '/u/ozgur/myzfs' MODE(7,5,5)

Then, you can mount a zFS to a mount point with several ways.

1- MOUNT TSO/E command

MOUNT specifies a file system that z/OS UNIX is to logically mount onto the root file system or another file system.

		MOUNT FILESYSTEM('OZGUR.FIRST.ZFS') MOUNTPOINT('/u/ozgur/myzfs') MODE(RDWR) TYPE(ZFS)
		UNMOUNT FILESYSTEM('OZGUR.FIRST.ZFS')

2- mount shell command

The mount shell command in /usr/sbin is used to mount a file system or list all mounts over a file system.

		/usr/sbin/mount -t ZFS -f OZGUR.FIRST.ZFS /u/ozgur/myzfs
		/usr/sbin/unmount /u/ozgur/myzfs

3- OSHELL command from Batch job (I submit this job with IBMUSER)

		//ZFSMOUNT JOB (VBT1),'OZGUR',CLASS=A,NOTIFY=&SYSUID,                  
		//             MSGCLASS=X,MSGLEVEL=(1,1)                               
		//PAX1     EXEC PGM=IKJEFT01,REGION=0M                                 
		//SYSTSPRT DD  SYSOUT=*                                                
		//SYSEXEC  DD  DSN=SYS1.SBPXEXEC,DISP=SHR                              
		//SYSTSIN  DD  *                                                       
		 oshell /usr/sbin/mount -t zfs -f OZGUR.FIRST.ZFS                    + 
		  /u/ozgur/myzfs                                                   ;   
		/*                                                                     

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/zOS%20UNIX%20File%20System%20and%20zFS%20Operations/ss/mounted-zfs.png)

Note: Mounting and unmounting z/OS UNIX file systems are privileged operations. A user must have UID(0) or have read access to the SUPERUSER.FILESYS.MOUNT resource in the UNIXPRIV class before file systems can be mounted or unmounted.

		===> TSO LU OZGUR OMVS

		OMVS INFORMATION    
		----------------    
		UID= 0000000067     
		HOME= /u/ozgur      
		PROGRAM= /bin/sh    
		CPUTIMEMAX= NONE    
		ASSIZEMAX= NONE      
		FILEPROCMAX= NONE    
		PROCUSERMAX= NONE    
		THREADSMAX= NONE     
		MMAPAREAMAX= NONE    

		***

		===> RL FACILITY BPX.SUPERUSER AU 

		CLASS      NAME                                                   
		-----      ----                                                   
		FACILITY   BPX.SUPERUSER                                          

		LEVEL  OWNER      UNIVERSAL ACCESS  YOUR ACCESS  WARNING          
		-----  --------   ----------------  -----------  -------          
		 00    IBMUSER         NONE               READ    NO            

		USER      ACCESS   ACCESS COUNT                                                
		----      ------   ------ -----                                                
		IBMUSER   READ        000000                                                   
		CIMGP     READ        000000                                                   
		OZGUR     READ        000000 
		
		***

 For a permanent mount (applies also at next IPL), add the zFS data set name and USS directory path name to the MOUNT section of your PARMLIB library BPXPRMxx member.
 
 ![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/zOS%20UNIX%20File%20System%20and%20zFS%20Operations/ss/BPXPRMxx.png)

## Back-up and Restore zFS

You can back up a zFS aggregate using a DFSMSdss logical dump. DFSMSdss automatically performs a quiesce of the mounted zFS aggregate before dumping the data set and an unquiesce when the dump ends.

		//ZFSBKUP  JOB (VBT1),'OZGUR',CLASS=A,NOTIFY=&SYSUID,                
		//             MSGCLASS=X,MSGLEVEL=(1,1)                             
		//*----------------------------------------------------------------- 
		//* THIS JOB QUIESCES A ZFS AGGREGATE, DUMPS IT, THEN UNQUIESCES IT. 
		//*----------------------------------------------------------------- 
		//DUMP    EXEC PGM=ADRDSSU,REGION=4096K                              
		//SYSPRINT DD  SYSOUT=*                                              
		//SYSABEND DD  SYSOUT=*                                              
		//OUT      DD  DSN=OZGUR.ZFS.BACKUP,                                 
		//             DISP=(NEW,CATLG,DELETE),SPACE=(CYL,(5,1),RLSE)        
		//SYSIN    DD  *                                                     
		 DUMP DATASET(INCLUDE(OZGUR.FIRST.ZFS)) -                            
		 RESET -                                                             
		 OUTDD(OUT)                                                          
		/*                                                                   

Note: Do not specify TOL(ENQF) when backing up zFS aggregates because it can cause corruption of the file system.

Use DFSMSdss logical restore to restore a zFS aggregate. The aggregate can be restored into a new aggregate (in the example, OZGUR.THIRD.ZFS). 

		//ZFSREST  JOB (VBT1),'OZGUR',CLASS=A,NOTIFY=&SYSUID,                 
		// MSGCLASS=X,MSGLEVEL=(1,1)                                          
		//*-----------------------------------------------------------------  
		//* THIS JOB RESTORES A ZFS AGGREGATE.                                
		//*-----------------------------------------------------------------  
		//ZFSREST EXEC PGM=ADRDSSU,REGION=0M                                  
		//SYSPRINT DD SYSOUT=*                                                
		//SYSABEND DD SYSOUT=*                                                
		//INDS DD DISP=SHR,DSN=OZGUR.ZFS.BACKUP                               
		//SYSIN DD *                                                          
		 RESTORE DATASET(INCLUDE(**)) -                                       
		 CATALOG -                                                            
		 RENAMEU(OZGUR.FIRST.ZFS,OZGUR.THIRD.ZFS)) -                          
		 WRITECHECK -                                                         
		 INDD(INDS)                                                           
		/*                                                                    
		//                                                                    

## Copy zFS File System to Larger Data Set



## Grow a zFS



https://www.ibm.com/docs/en/zos/2.4.0?topic=zag-creating-managing-zfs-file-systems-using-compatibility-mode-aggregates </br>
https://www.ibm.com/docs/en/zos/2.4.0?topic=aggregates-creating-compatibility-mode-aggregate </br>
https://www.ibm.com/docs/en/zos/2.4.0?topic=descriptions-mount-logically-mount-file-system </br>
https://www.ibm.com/docs/en/zos/2.2.0?topic=commands-mount-logically-mount-file-system </br>
