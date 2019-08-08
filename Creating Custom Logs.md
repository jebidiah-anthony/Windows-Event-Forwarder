# CREATING __CUSTOM LOGS__ FOR __WINDOWS EVENT FORWARDER__

## ENVIRONMENT

WINDOWS | OS | BUILD No. | REMARKS
--- | --- | --- | ---
WIN-BO2CT95INDP | Windows Server 2016 Standard | 10.0.14393 Build 14393 | Collector Machine

---

## APPLICATIONS/TOOLS USED

1. Windows SDK (Development Kit):
   - ecmangen.exe
   - mc.exe (Message Compiler)
   - rc.exe (Resource Compiler)
2. csc.exe (C# Compiler)
3. wevtutil

__NOTE(S)__:
- __ecmangen.exe__ was removed from the Windows 10 SDK starting from 10.0.16299.15
  - Parallel installations of Windows SDK are allowed.
  - In this case, [Windows 10 SDK (10.0.14393.795)](https://go.microsoft.com/fwlink/p/?LinkId=838916) was installed alongside the latest SDK version.
- __csc.exe__ is included in the __*Microsoft .NET Framework*__.
  - __.NET Framework__ is native to Windows Operating Systems.
  - The Framework's version depends on the OS installed.
- __wevtutil__ is a command native to the __*Command Prompt*__

---

## PROCEDURE

1. Create a manifest file:
   1. Open `ecmangen.exe`:

      ```
      C:\Program Files (x86)\Windows Kits\10\bin\x64\ecmangen.exe
      ```

   2. Create a new __*provider*__:
      
      1. On the left panel, right click on `Events Section` then select __New__ -> __Provider__.
      2. Fill up the following fields:

         FIELD | VALUE
         --- | ---
         Name | `WEF-Events`
         Symbol | `WEF_Events`
         GUID | Press `New` beside the input field
         Resources | `C:\Windows\System32\WEF-Events.dll`
         Messages | `C:\Windows\System32\WEF-Events.dll`

      3. On the right panel, click on `Save`.

   3. Create a new __*template*__:

      1. On the left panel, under `WEF-Events`, select `Templates`.
      2. On the right panel, select `New Template`. 
      3. Fill up the following fields:
 
         FIELD | VALUE
         --- | ---
         Name | `WEF-Template`
      
      4. Add `Field Attributes`:

         Name | InType | OutType | Count | Length
         --- | --- | --- | --- | ---
         Unicode | win:UnicodeString | xs:string | default | default
         UInt32 | win:UInt32 | xs:unsignedInt | default | -

      5. On the right panel, click `Save`.

   4. Create __*channels*__ (maximum of 8):
      
      1. On the left panel, under `WEF-Events`, select `Channels`.
      2. On the right panel, select `New Channel`.
      3. Fill up the following fields:

         NAME | SYMBOL | TYPE | ENABLE | DESCRIPTION | CHANNEL SECURITY
         --- | --- | --- | --- | --- | ---
         WEF-Security | WEF_Security | Operational | Yes | DC Security Logs | Default
         WEF-System | WEF_System | Operational | Yes | DC System Logs | Default
         WEF-PowerShell | WEF_PowerShell | Operational | Yes | DC PowerShell Logs | Default
         WEF-Sysmon | WEF_Sysmon | Operational | Yes | DC Sysmon Logs | Default

      4. On the right panel, click `Save`.

   5. Create a new __*event*__:
    
      1. On the left panel, under `WEF-Events`, select `Events`.
      2. On the right panel, select `New Event`.
      3. Fill up the following fields:
         
         FIELD | VALUE
         --- | ---
         Symbol | `WEF_Event`
         Event ID | `6969`
         Message | `$(string.WEF-Events.event.6969.message)`
         Channel | `WEF-Security`
         Template | `WEF-Template`
         Keywoards | `win:AuditSuccess`, `win:AuditFailure`
    
      4. On the right panel, click on `Save`.

   6. Save the manifest file as "WEF_Events.man"

      __NOTE(S)__:
      - Avoid using the character, '`-`', in the filename.
        - The generated C# file during compiling will face an error.

      - Resulting manifest file (XML formatted):

        ```xml
        <?xml version="1.0"?>
        <instrumentationManifest xsi:schemaLocation="http://schemas.microsoft.com/win/2004/08/events eventman.xsd" xmlns="http://schemas.microsoft.com/win/2004/08/events" xmlns:win="http://manifests.microsoft.com/win/2004/08/windows/events" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:trace="http://schemas.microsoft.com/win/2004/08/events/trace">
        	<instrumentation>
        		<events>
        			<provider name="WEF-Events" guid="{CB3EB4AA-FEDD-41C4-A7BB-E173045E4DC7}" symbol="WEF_Events" resourceFileName="C:\Windows\System32\WEF_Events.dll" messageFileName="C:\Windows\System32\WEF_Events.dll">
        				<events>
        					<event symbol="WEF_Event" value="6969" version="0" channel="WEF-Security" template="WEF-Template" keywords="win:AuditSuccess win:AuditFailure " message="$(string.WEF-Events.event.6969.message)"></event>
        				</events>
        				<channels>
        					<channel name="WEF-Security" chid="WEF-Security" symbol="WEF_Security" type="Operational" enabled="true" message="$(string.WEF-Events.channel.WEF_Security.message)"></channel>
        					<channel name="WEF-System" chid="WEF-System" symbol="WEF_System" type="Operational" enabled="true" message="$(string.WEF-Events.channel.WEF_System.message)"></channel>
        					<channel name="WEF-PowerShell" chid="WEF-PowerShell" symbol="WEF_PowerShell" type="Operational" enabled="true" message="$(string.WEF-Events.channel.WEF_PowerShell.message)"></channel>
        					<channel name="WEF-Sysmon" chid="WEF-Sysmon" symbol="WEF_Sysmon" type="Operational" enabled="true" message="$(string.WEF-Events.channel.WEF_Sysmon.message)"></channel>
        				</channels>
        				<keywords></keywords>
        				<templates>
        					<template tid="WEF-Template">
        						<data name="Unicode" inType="win:UnicodeString" outType="xs:string"></data>
        						<data name="UInt32" inType="win:UInt32" outType="xs:unsignedInt"></data>
        					</template>
        				</templates>
        			</provider>
        		</events>
        	</instrumentation>
        	<localization>
        		<resources culture="en-US">
        			<stringTable>
        				<string id="keyword.AuditSuccess" value="Audit Success"></string>
        				<string id="keyword.AuditFailure" value="Audit Failure"></string>
        				<string id="WEF-Events.event.6969.message" value="$(string.WEF-Events.event.6969.message)"></string>
        				<string id="WEF-Events.channel.WEF_System.message" value="DC System Logs"></string>
        				<string id="WEF-Events.channel.WEF_Sysmon.message" value="DC Sysmon Logs"></string>
        				<string id="WEF-Events.channel.WEF_Security.message" value="DC Security Logs"></string>
        				<string id="WEF-Events.channel.WEF_PowerShell.message" value="DC PowerShell Logs"></string>
        			</stringTable>
        		</resources>
        	</localization>
        </instrumentationManifest>
        ```

2. Compile the manifest file and generate relevant files (e.g. WEF-Events.dll)

   1. Press `Win` + `R` then enter `cmd`.
   2. Navigate to where `WEF_Events.man` was saved.
   3. Enter the following commands:

      1. __mc.exe__ (Message Compiler)
        
         ```cmd
         "C:\Program Files (x86)\Windows Kits\10\bin\x64\mc.exe" WEF_Events.man
         "C:\Program Files (x86)\Windows Kits\10\bin\x64\mc.exe" -css WEF_Events.DummyEvent WEF_Events.man
         ```
         __NOTE(S)__:
         - The compiler generates the message resource files to which your application links.
         - Switches used:
  
           OPTION | DESCRIPTION
           --- | ---
           -css &lt;namespace> | <ul><li>Generates a __static__ C# class</li><li>It includes the methods that you would call to log the events defined in your manifest</li></ul>

         - File(s) generated after execution:

           - MSG00001.bin
           - WEF_Events.cs
           - WEF_Events.h
           - WEF_Events.rc
           - WEF_EventsTEMP.bin

      2. __rc.exe__ (Resource Compiler)

         ```cmd
         "C:\Program Files (x86)\Windows Kits\10\bin\x64\rc.exe" WEF_Events.rc
         
         # Microsoft (R) Windows (R) Resource Compiler Version 10.0.10011.16384
         # Copyright (C) Microsoft Corporation.  All rights reserved.
         ```
         __NOTE(S)__:
         - `rc.exe` compiles an application's resources and could be used to build Windows-based applications.
         - File(s) generated after execution:

           - WEF_Events.res
        
      3. __csc.exe__ (C# Compiler)
         ```cmd
         "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe" /win32res:WEF_Events.res /unsafe /target:library /out:WEF_Events.dll WEF_Events.cs
         ```
         __NOTE(S)__:
         - No output means the command was successfully executed 
         - Switches used:
             
           OPTION | DESCRIPTION
           --- | ---
           /win32res:&lt;file> | Specify a Win32 resource file (.res) 
           /unsafe | Allow 'unsafe' code
           /target:library | Build a library (Short form: /t:library)
           /out:&lt;file> | Specify output file name (default: base name of file with main class or first file)

         - File(s) generated after execution:

           - WEF_Events.dll

3. Install the __manifest file__ with the matching __dll file__:

   1. Move both files to the `C:\Windows\System32` directory:

      ```cmd
      copy .\WEF_Events.man C:\Windows\System32\WEF_Events.man
      copy .\WEF_Events.dll C:\Windows\System32\WEF_Events.dll
      ```

   2. Install the manifest file using `wevtutil`:
     
      ```cmd
      wevtutil im C:\Windows\System32\WEF_Events.man
      ```
4. The created logs should appear under `Applications and Services Logs` inside __Event Viewer__

   __NOTE(S)__:
   - The logs generated could be used for created subscriptions.
   - Additional columns could be added/removed from the logs (e.g. `Log`, `Computer`) 

---

## REFERENCES

- https://blogs.technet.microsoft.com/russellt/2016/05/18/creating-custom-windows-event-forwarding-logs/
- https://stackoverflow.com/questions/53028775/cannot-locate-ecmangen
- https://developer.microsoft.com/en-us/windows/downloads/sdk-archive
- https://blogs.msdn.microsoft.com/astebner/2007/03/14/mailbag-what-version-of-the-net-framework-is-included-in-what-version-of-the-os/
- https://docs.microsoft.com/en-us/windows/win32/wes/message-compiler--mc-exe-         
- https://docs.microsoft.com/en-us/windows/win32/menurc/using-rc-the-rc-command-line-
