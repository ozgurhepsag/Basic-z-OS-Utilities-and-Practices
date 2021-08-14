I have a small z/OS system that has no CICS, DB2, IMS, MQ or any ISV products and this is a basic practice to understand how to use RACF variables. The goal in this practice is that some non-special users that I will specify are able to submit jobs in behalf of some special users sometimes using RACF variables. To do that, we will define a RACFVARS profile that will be used in SURROGAT profile (SURROGAT class will not be covered in this practice in detail).

## RACF Variables

A RACF variable could be used in a profile name to define one general resource profile to protect many resources with dissimilar names and to minimize administrative effort for some cases. RACF variables can be used for general resource profiles only, not for data set profile names. A profile that contains a RACF variable in its name is considered a generic profile.

#### Activate RACFVARS Class

If you want to use RACF Variables in your environment, RACFVARS general resource class must be activated. Because the RACFVARS class requires high performance, you must RACLIST the profiles. To activate the RACFVARS class and RACLIST the profiles:

    SETROPTS CLASSACT(RACFVARS) RACLIST(RACFVARS)

You can check your RACF options before or after issue the command above.

    SETROPTS LIST (or =R.5.1)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/activate-racfvars.png)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/raclist-racvars.png)

#### Define RACFVARS profile

RACF variable names must begin with an ampersand (&), can be up to eight characters long, and cannot contain any periods (.) or generic characters. Do not define variable names that start with &RAC; they are reserved for RACF use. </br>
**Note:** The variable values &RACUID and &RACGPID are not RACF variables and are not members of the RACFVARS class. These variables have specific uses for Global Access Table (GLOBAL class).

A RACF variable defined named '&SUSERS' that contains some special users that I specified for that some non-special users are be able to submit jobs in behalf of them when needed.

    RDEFINE RACFVARS &SUSERS ADDMEM(OZGUR CETIN)

Any time a change is made to a RACFVARS profile, the in-storage profiles for the RACFVARS class must be refreshed for the changes to take effect. To refresh the in-storage profiles for the RACFVARS class:

    SETROPTS RACLIST(RACFVARS) REFRESH

You can see the new definition with the command below: (or from RACF panels)

    RLIST RACFVARS &SUSERS

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/rlist-racfvars-susers.png)

#### Use a RACF Variable in SURRAGAT profile

When RACF authorization checking matches a resource name with the name of a general resource profile that contains a RACF variable, it locates the RACFVARS profile that matches the RACF variable. When a match is found, RACF substitutes the member name for the RACF variable in the name of the general resource profile and then searches for a matching general resource profile to check the access authorization. Therefore, some changes needs to be made on SURROGAT class to complete our practice.

I have a non-special user EMRE and some special users OZGUR and CETIN. Before make the necessary changes on SURROGAT class, EMRE has no permisson to submit a job in behalf of OZGUR or CETIN. I have an example below for this:

When EMRE submit the job below, a security violation occurs.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/surrogat-job.png)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/sec-error.png)

and OZGUR got the message below:

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/sec-viol.png)

So, a SURROGAT profile needs to be created like below for that EMRE has the permission to submit jobs in behalf of OZGUR and CETIN.

    RDEFINE SURROGAT &SUSERS.SUBMIT UACC(NONE)
    PERMIT  &SUSERS.SUBMIT CLASS(SURROGAT) ID(EMRE) ACCESS(READ)
    SETROPTS RACLIST(SURROGAT) REFRESH

You can see the access list of the SURROGAT profile.

    CLASS      NAME                                               
    -----      ----                                               
    SURROGAT   &SUSERS.SUBMIT (G)                                 
    
    LEVEL  OWNER      UNIVERSAL ACCESS  YOUR ACCESS  WARNING      
    -----  --------   ----------------  -----------  -------      
     00    IBMUSER         NONE              ALTER    NO          

    USER      ACCESS          
    ----      ------          
    IBMUSER   ALTER           
    EMRE      READ  

Finally, EMRE has a permision to submit a job in behalf of OZGUR or CETIN. When I submit the same job above again, the job finished succesfully and EMRE.SURROGAT.OZGUR data set created.

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/surrogat-success.png)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/dataset-created.png)

## References

https://www.ibm.com/docs/en/zos/2.1.0?topic=pgr-using-racf-variables-in-profile-names-racfvars-class </br>
https://www.ibm.com/docs/en/zos/2.4.0?topic=cujn-allowing-tso-user-cancel-all-jobs-originating-from-local-nodes </br>
https://www.rshconsulting.com/RSHpres/RSH_Consulting__RACFVARS__Oct_2013.pdf </br> 

