
I have a monoplex that has no CICS, DB2, IMS, MQ or any ISV products. Here you can find some basic z/OS UNIX and zFS operations.

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

You need to mount file systems to mount points that you prepared before. You can mount a zFS to a mount point with several ways.

1- MOUNT TSO/E command

MOUNT specifies a file system that z/OS UNIX is to logically mount onto the root file system or another file system.

2- mount shell command

The mount shell command in /usr/sbin is used to mount a file system or list all mounts over a file system.

3- OSHELL command from Batch job (IKJEFT01 utility)

Mounting and unmounting z/OS UNIX file systems are privileged operations. A user must have UID(0) or have read access to the SUPERUSER.FILESYS.MOUNT resource in the UNIXPRIV class before file systems can be mounted or unmounted.


## Back-up and Restore zFS

## Copy zFS File System to Larger Data Set

## References

https://www.ibm.com/docs/en/zos/2.4.0?topic=aggregates-creating-compatibility-mode-aggregate </br>
https://www.ibm.com/docs/en/zos/2.4.0?topic=descriptions-mount-logically-mount-file-system </br>
https://www.ibm.com/docs/en/zos/2.2.0?topic=commands-mount-logically-mount-file-system </br>
