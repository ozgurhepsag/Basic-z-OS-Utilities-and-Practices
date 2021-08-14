This is a basic practice example to understand how to use RACF variables. 

## RACF Variables

A RACF variable could be used in a profile name to define one general resource profile to protect many resources with dissimilar names and to minimize administrative effort for some cases. RACF variables can be used for general resource profiles only, not for data set profile names. A profile that contains a RACF variable in its name is considered a generic profile.

#### Activate RACFVARS Class

If you want to use RACF Variables in your environment, RACFVARS general resource class must be activated. Because the RACFVARS class requires high performance, you must RACLIST the profiles. To activate the RACFVARS class and RACLIST the profiles:

    SETROPTS CLASSACT(RACFVARS) RACLIST(RACFVARS)

You can check your RACF options before or after issue the command above.

    SETROPTS LIST (or =R.5.1)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/activate-racfvars.png)

![Screenshot](https://github.com/ozgurhepsag/Basic-z-OS-Utilities-and-Practices/blob/main/RACF%20Variables/ss/raclist-racvars.png)

#### Define RACF Variable

RACF variable names must begin with an ampersand (&), can be up to eight characters long, and cannot contain any periods (.) or generic characters. Do not define variable names that start with &RAC; they are reserved for RACF use. </br>
**Note:** The variable values &RACUID and &RACGPID are not RACF variables and are not members of the RACFVARS class. These variables have specific uses for Global Access Table (GLOBAL class).

## References

https://www.ibm.com/docs/en/zos/2.1.0?topic=pgr-using-racf-variables-in-profile-names-racfvars-class </br>
https://www.ibm.com/docs/en/zos/2.4.0?topic=cujn-allowing-tso-user-cancel-all-jobs-originating-from-local-nodes </br>
https://www.rshconsulting.com/RSHpres/RSH_Consulting__RACFVARS__Oct_2013.pdf </br> 

