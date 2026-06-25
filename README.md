# sapgui-url-protocol

The purpose of this project is to enable hyperlinks, which, when the hyperlink is clicked, starts the SAP GUI and runs a transaction with parameters. The way it works is by registering the URL protocol handler sapgui for the current user in the Windows Registry and passing the commands through a powershell script to the sapshcut.exe program. On Windows, the sapshcut program is by default located at C:\Program Files (x86)\SAP\FrontEnd\SAPgui\sapshcut.exe.


## SAPGUI Protocol Setup Tool - Handler Configuration Menu
This menu is for setting up default values that will be used if the values are not specified in the URL. It is recommended to add these values. This allows your current session to be re-used for SAP servers that have Single Sign On [SSO] enabled. Also, some hyperlink shortcuts may not work without setting this up if the creator of the hyperlink didn't add the server information.

* SAP User Name:
  * Default value for the -user paramenter.
  * Enter your username that you use in SAP.
* Date Format in SAP:
  * Enter the date format that matches the one you use in SAP.
* Server Name/Description:
  * Default value for the -sysname parameter.
  * Enter the system name for the target system. The system name can be found in the SAPLogon window under the Name column.
* Server ID:
  * Default value for the -system parameter.
  * Enter the system ID for the target system. The system ID can be found in the SAPLogon window under the System Description column.
* Client:
  * Default value for the -client parameter.
  * Enter the client number for the target system. It will be a 3 digit number
* Language:
  * Default value for the -language parameter.
  * Enter the language code for the target system. Languages are referred to using a unique two-character language code
* File Path to sapshcut.exe:
  * The file path to the sapshcut.exe program.
  * Default value is C:\Program Files (x86)\SAP\FrontEnd\SAPgui\sapshcut.exe.


## Description of the sapgui:// protocol

For custom URLs, the protocol handler sapgui must comply to the following scheme:

  `sapgui:<sapshcut.exe arguments separated by &>`

Example URL with login and system parameters:
```cmd
sapgui:-SYSTEM=PRD&-CLIENT=100&-LANGUAGE=EN&-COMMAND="""SM37 BTCH2170-JOBNAME=Z*;BTCH2170-USERNAME=HUGO;"""
```

Example URL that uses the user's default values for the login and system parameters:
```cmd
sapgui:-COMMAND="""SM37 BTCH2170-JOBNAME=Z*;BTCH2170-USERNAME=HUGO;"""
```

To escape the & character to prevent the text from being split into separate arguments use && to escape it. For example:
```cmd
sapgui:-SYSNAME="""EH&&S REE [SSO]"""&-COMMAND="""SM37 BTCH2170-JOBNAME=Z*;BTCH2170-USERNAME=HUGO;"""
```

By using such a custom URL you can add a hyperlink to your HTML page that can be used to navigate to the filtered job overview screen in SAP GUI, as indicated by the following snippet.
```html
<html>
    <head>
        <title>Link to SAPGUI transaction (follow custom URL)</title>
    </head>
    <body>
        <a href='sapgui:-SYSTEM=PRD&-CLIENT=100&-USER=HUGO&-LANGUAGE=EN&-REUSE=1&-COMMAND="""SM37 BTCH2170-JOBNAME=Z*;BTCH2170-USERNAME=HUGO;"""'>
            Job Overview
        </a>
    </body>
</html>
```

## SAPshortcut: Program parameters

The ... are placeholders for parameters of the SAP shortcut. All parameters can start with "-" or "/".
The parameter variants ' or ...' specified below (for example, or -u=user ID) are abbreviations of the parameters.

**For official guidance on utilizing the parameters with SAP GUI shortcuts see SAP Note 103019. It contains information about the program parameters of the SAP GUI link (SAP shortcut).**


## System-independent parameters that can only be entered individually for the SAP shortcut command
| Parameter | Description |
| --- | --- |
| -version | displays the current version of the SAPshortcut. |
| -help, -?	| The short help display for parameterization. |
| -register	 | Registers the SAPshortcut in the Windows registration database. After the registration, Windows supports the following functions:<br><br>* Starting the shortcut by double-clicking the icon<br>* Logon/edit functions in the context menu of the icon<br>* New -> SAP shortcut in the context menu of the desktop<br><br>This is not used because you can simply perform registration by calling sapshcut.exe without parameters. Sapshortcut is also usually registered automatically when the GUI is installed. |
| -edit c:\myshortcut.sap | starts the processing dialog of the SAPshortcut and writes the input data to the file c:\myshortcut.sap after you choose "Enter". You cannot log on using the SAPshortcut. |


## System-dependent parameters that must be entered combined together for the SAP shortcut command

If the specified parameter is incomplete and after calling the SAP shortcut command, the system issues a dialog with an English error message such as:

* Not all data for SAP GUI Shortcut is available:
* System ID is unknown.
* Enter the missing data </div>

In this case, you supplement the missing parameters in the command line as requested.

### User ID
| Parameter | Description |
| --- | --- |
| - user="userID" or -u="userID" | specifies the SAP user ID. |
| -language=[DE\|EN\|...] or -l=[DE\|EN\|...] | specifies the logon language in the SAP system |
| -pw="password" | specifies the password for automatic logon. |
If this parameter is specified and the remaining logon data also exists (target system, client), SAPshortcut starts the transaction immediately in the target system. If this parameter is not specified, the system displays the logon dialog and requests the password. It is only then that the logon is started.

### SAP system ID
| Parameter | Dscription |
| --- | --- |
| -system=SID or -sid=SID | specifies the SAP system ID, SID. |
| -client=001 or -clt=001 | specifies the logon client. |
| -sysname="SID [MyGroup1]" or -desc="SID [MyGroup1]" | specifies the target system using the description in the SAPLogon. |
| -guiparm="sapserver 10" or -gui="sapserver 10" | specifies the target system using start parameters of SAPGUI.EXE. |
| -gui="/R/<sapsystemID>/G/<group>" | specifies a logon using system name and logon group. |
| -gui="[/H/<saprouter1[/S/saprouterservice1][...]/M/<messageserver>/S/<service>/G/<group>" | specifies a load balancing logon by specifying the message server and the logon group. |
| -gui="/R/<sapsystemID>/M/<messageserver>/G/<group>" | specifies a load balancing logon by specifying the system name, the message server and the logon group. |
*** In general, you can insert all of the command parameters of sapgui.exe within the double quotation marks " ".


## Specification of SAP start transactions/reports/commands

### -type=[Transaction|Report|SystemCommand] or -t=[Transaction|...]
    specifies the function type to be started. The following types are supported at present:
| Function Type | Dscription |
| --- | --- |
| Transaction | starts the function as a transaction, default value. |
| Report | starts the function as a report. |
| SystemCommand | starts the function as a system command. |

### -command="se38" or -cmd="se38"
    transfers the transaction code/report name/system command when starting the relevant function.

#### a) Transactions
* Without parameter:
* Starting the ABAP editor:
  * `-command="se38"`
* With Dynpro field parameter:
  * The next command starts the SAP Note transaction with the SAP Note data as a parameter (language and SAP Note number):
  * `-command="*BH33 HWTX3-SPRSL=D; HWHD3-NUMM=49143;"`

* HWTX3-SPRSL and HWHD3-NUMM are the field labels for batch input of the two required fields on the initial screen of transaction BH33, and they can be determined by calling the screen field help:
  | How to get the Dynpro field name for batch input |
| --- |
| <ins>Preferred Method</ins><br> * Place the cursor in the field, for example, "Number".<br> * Choose F1 and call the technical information.<br> * Choose the field description for batch input, screen fields HWTX3-SPRSL.<br><br><ins>Alternate Method to use when “no documentation available”</ins><br> * With the selection screen open, type **ctrl + s** to enter the variant screen.<br> * Then press the **Technical Name** button to toggle the display of the technical name for all of the selection screen fields. |

  The asterisk character <ins>*****</ins> before transaction code BH33 means that the initial screen is skipped when the transaction is started, provided that all required fields of the initial screen are filled with the parameter.

  Check the validity of the command string within " " by entering the following data in the OK code field of an SAP GUI window /n that is followed by this string as below:
    `/n*BH33 HWTX3-SPRSL=D; HWHD3-NUMM=49143;`
  Then choose ENTER.
  If this does not work using the direct OK code call, it cannot work in the case of shortcuts either.

*** For a parameter transaction such as F-01 that calls a native transaction (FBM1) with default values for one or more screen fields, or for a variant transaction such as YVA03 that is created using transaction SHD0 for the native transaction such as VA03 with a transaction variant, the default values **CANNOT** be written to the corresponding field for the transaction call with parameterusing OK-CODE as:
    `/n*F-01 BKPF-BUKRS=1000;    or    /n*YVA03 VBAK-VBELN=1234;`
  You must enter the following OK-CODE instead:
    `/n*F-01 BKPF-BUKRS=1000;    or    /n*YVA03 VBAK-VBELN=1234;`
  Therefore, parameter transactions and variant transactions of this type cannot be populated correctly with default values by Sapshortcut. You should always enter the native transaction such as
    `-command="*FBM1 BKPF-BUKRS=1000;"`
      instead of
    `-command="*F-01 BKPF-BUKRS=1000;"`
    
To determine whether the transaction is a parameter transaction such as F-01 or a variant transaction:
  * call transaction SM31,
  * specify the table name TSTC,
  * and choose "Display".
  * Then specify the transaction code and choose the glasses icon to display the data.


#### b) Reports
`-command="grbusg_3" -type=Report`

  The variant of the report can be specified as follows:
`-command="report variant"`
  as in, for example
`-command="RSPARAM TEST"`
  if the ABAP report RSPARAM has the variant TEST.
  
  ***To be able to carry out the report, the SAPShortcut user must be authorized to call transaction SUB%.


#### c) System commands
`-command="/H" -type=SystemCommand`
  activates the debugger
`-command="?STAT" -type=SystemCommand`
  displays the status.

### -title="Any Title for the function" or -tit="My Test"
  specifies the title of the function to be started (this appears in the logon dialog for example).

### To diagnose the Sapshortcut program:
`-trace=3 or -trc=3`

### To set the working directory:
`-workdir="C:temp" or -wd="C:temp"`
 * In SAP GUI 620 patch lower than 56 and SAP GUI 640 patch lower than 12:
   * This setting determines the trace directory where you want to write the SAPShortcut and other SAP GUI trace files.
 * For a later SAP GUI version:
   * This setting no longer has any effect. The trace files are always written to SAPWorkdir. You can change the standard SAPWorkdir with the SAP GUI Configuration program.

### To set the window size of the Sapgui:
`-maxgui or -max`
  sets the registry entry 'Maximize' under:
    _HKEY_CURRENT_USER\Software\SAP\SAPGUI Front\SAP Frontend Server\Window_
  to 1 so that the GUI is started in a maximum window after logging on (through Sapshortcut or Saplogon).
  
  This parameter can be used alone or together with other system-dependent parameters.

### To set whether or not an existing connection should be reused
 | Parameter | Description |
 | --- | --- |
 |-reuse=1 | reuse, default setting |
 |-reuse=0 | do not reuse |
  Reuse only occurs when the system ID specified (parameter -system=SID), client(parameter -client=###), user (parameter -user=...), and logon language (parameter -language=??) matches the ID of an existing connection. If a session from the existing connection has the initial screen, that is, transaction SESSION_MANAGER ("SAP Easy Access") or S000, the session is reused. Otherwise, a new session is started for the existing connection.
