
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

## Back-up and Restore zFS

## Copy zFS File System to Larger Data Set

## References

https://www.ibm.com/docs/en/zos/2.4.0?topic=aggregates-creating-compatibility-mode-aggregate </br>
