# Setting up a  Windows Event Collector

## __ENVIRONMENT:__

#### MACHINES:
HOSTNAME | MACHINE IP | OS | Description 
--- | --- | --- | --- 
MSEDGEWIN10 | 192.168.150.128 | Windows 10 Enterprise Evaluation | Source Machine
WIN-BO2CT95INDP | 192.168.150.133 | Windows Server 2016 | Collector Machine

__NOTE(S)__:
- The FQDN for WIN-BO2CT95INDP is __win-bo2ct95indp.bossmanben.local__

---

## __ASSUMPTIONS:__
### i.  The Source Machine (MSEDGEWIN10) is part of a Domain Controller (WIN-BO2CT95INDP).
### ii. This guide uses __*Security Logs*__ as an example.
---

## __PROCEDURE:__

__NOTE(S)__:
- The steps below will create a subscription that collects __*Security logs*__ from the __Source Machine__ (MSEDGEWIN10)

### __i. Start the WinRM service__

1. Open __PowerShell__ on the Source Machine (MSEDGEWIN10):
   ```ps1
   winrm quickconfig
   ```
   __NOTE(S)__:
   - Add the Collector Machine to the Source Machine's trustedhosts:
     ```ps1
     Set-Item wsman:localhost/client/trustedhosts 192.168.150.133
     ```
   - Restart the service for changes to take effect:
     ```ps1
     Restart-Service WinRM
     ```

2. Check if the service is running:
   ```ps1
   winrm get winrm/config
   ```
   ```
   ...omitted...
           AllowRemoteAccess = true
       Winrs
           AllowRemoteShellAccess = true
   ...omitted...
   ```
   __NOTE(S)__:
   - `AllowRemoteAccess = true` signifies that the service is running.

3. Test if the Collector Machine (BOSSMANBEN) is reachable using WinRM:
   ```ps1
   Test-WSMan WIN-BO2CT95INDP
   ```
   ```
   wsmid           : http://schemas.dmtf.org/wbem/wsman/identity/1/wsmanidentity.xsd
   ProtocolVersion : http://schemas.dmtf.org/wbem/wsman/1/wsman.xsd
   ProductVendor   : Microsoft Corporation
   ProductVersion  : OS: 0.0.0 SP: 0.0 Stack: 3.0
   ```
   __NOTE(S)__:
   - WinRM is __enabled by default__ on Windows Server 2012 and up.
   - This is just a measure to check if the Collector Machine is indeed reachable.

### __ii. Add the Collector Machine to the Event Log Readers groups__

- __In the Source Machine (MSEDGEWIN10):__

  1. Open the __Local Users and Groups__:
    
     - Press `Win` + `R` then enter `lusrmgr.msc`

  2. Navigate to `Local Users and Groups (Local)` __>__ `Groups`:
      
     1. Right-click `Event Log Readers` and select `Properties`
     2. Select `Add...`

  3. Select `Object Types...` then check the box, `Computers`

  4. `Enter the object names to select` -- "__*WIN-BO2CT95INDP*__"

     __NOTE(S)__:
     - Select `Check Names` for good measure.

  5. Select `OK` when done.

### __iii. Create Subscriptions using Event Viewer__

- __In the Collector Machine (WIN-BO2CT95INDP):__

  1. Open the __Event Viewer__:

     - Press `Win` + `R` then enter gpedit `eventvwr.msc`

  2. On the left panel, right-click on `Subscriptions` then select `Create Subscription...`

     1. `Subscription Name` -- "__*Remote Security Logs*__"

     2. `Description` -- "__*Security Logs from the Domain Computer, MSEDGEWIN10*__"
      
     3. `Destination log` -- "__*Forwarded Events*__"
        
        __NOTE(S)__:
        - Custom logs could be created but `Forwarded Events` is selected by default.
        - Click [here](./Creating%20Custom%20Logs.md) to create custom logs.

     4. Select `Subscription type and source computers`:

        - If you choose `Collector initiated` then select `Select Computers...`:

          1. Select `Add Domain Computers...`
          2. `Enter the object name to select` -- "__*MSEDGEWIN10*__"
          3. Select `Check Names` for good measure.
          4. Select `OK`
          5. Select `Test` for good measure.
          6. Select `OK`

        - If you choose `Source initiated` then select `Select Computer Groups...`:

          1. Select `Add Domain Computers...`
          2. `Enter the object name to select` -- "__*MSEDGEWIN10*__"
          3. Select `Check Names` for good measure.
          4. Select `OK`
          5. Select `Test` for good measure.
          6. Select `OK`

          - On the Source Machine (MSEDGEWIN10):
            
            1. Press `Win` + `R` then enter `gpedit.msc`
               
               1. Navigate to `Computer Management` __>__ `Administrative Templates` __>__ `Windows Components` __>__ `Event Forwarding`
               2. Right-click on `Configure target Subscription Manager` then select `Edit`
               3. Choose `Enabled`
               4. Under `Options`, beside `SubscriptionManagers`, press `Show...`
               5. Enter `Server=http://win-bo2ct95indp.bossmanben.local:5985/wsman/SubscriptionManager/WEC,Refresh=30`
               6. Press `OK`
               7. Press `OK`

            2. Open __PowerShell__ or __cmd__ the run `gpupdate /force`

          - On the Collector Machine (WIN-BO2CT95INDP):

            1. Open __PowerShell__ or __cmd__ then run `wecutil quick-config`         

     5. Select `Select Events...`:

        1. `Logged` -- "__*Any time*__"
        2. `Event level` -- __*Critical*__, __*Error*__, __*Information*__, __*Warning*__
        3. Choose `By log` -- __*Windows*__ -> __*Security*__
        4. Filter __Event IDs__ -- 4624,4657,4688,4698,4720,4722,4724,4732,4738,4769
        5. Select `OK`
   
     6. Select `Advanced...`:
         
        1. `User Account` -- Choose `Machine Account`
        2. `Event Delivery Optimization` -- Choose `Minimize Latency`
        3. Select `OK`

        __NOTE(S)__:
        - There are three `Event Delivery Optimization` options:

          OPTION | DESCRIPTION | INTERVAL
          --- | --- | ---
          Normal | Does not conserve bandwidth | 15 minutes via pull delivery
          Minimize Bandwidth | Bandwidth for delivery is controlled | 6 hours via push delivery
          Minimize Latency | Delivery with minimal delay | 30 seconds via push delivery 

     7. Select `OK`

  3. Right-click on the newly created subscription then select `Runtime Status`:
     ```
     [MSEDGEWIN10.bossmanben.local] - Error - Last retry time: 7/17/2019 8:27:52 PM. 
     Code (0x138C): <f:ProviderFault provider="Event Forwarding Plugin" path="C:\Windows\system32\wevtfwd.dll" 
     ```

- __In the Source Machine (WIN-BO2CT95INDP):__

  1. Run `wevtutil`:
     ```ps1
     wevtutil get-log Security
     ```
     ```
     name: Security
     enabled: true
     type: Admin
     owningPublisher:
     isolation: Custom
     channelAccess: O:BAG:SYD:(A;;0xf0005;;;SY)(A;;0x5;;;BA)(A;;0x1;;;S-1-5-32-573)
     logging:
       logFileName: %SystemRoot%\System32\Winevt\Logs\Security.evtx
       retention: false
       autoBackup: false
       maxSize: 20971520
     publishing:
       fileMax: 1
     ```

  2. Add the __Network Service Account__ (S-1-5-20) to the `channelAccess` field:
     ```ps1
     wevtutil set-log Security /ca:"O:BAG:SYD:(A;;0xf0005;;;SY)(A;;0x5;;;BA)(A;;0x1;;;S-1-5-32-573)(A;;0x1;;;S-1-5-20)"
     ```
     __NOTE(S)__:
     - WinRM runs under the __*Network Service Account*__ which had no access to the __Security Logs__

- __Going back to the Collector Machine (WIN-BO2CT95INDP):__

  1. Go to the __Event Viewer__:

     - Press `Win` + `R` then enter gpedit `eventvwr.msc`

  2. On the left panel, go to `Subscriptions` then select the recently created subscription

  3. On the right panel, under the __*subscription name*__, select `Retry`

  4. Right-click on the recently created subscription then select `Runtime Status`:
     ```
     [MSEDGEWIN10.bossmanben.local] - Active - : No additional status.
     ```
     __NOTE(S)__:
     - An Event with __ID 100 (Name="SubscribeSuccess")__ will appear on __*Microsoft-Windows-Event-ForwardPlugin/Operational*__ in the Source Machine (MSEDGEWIN10)

### __iv. Wait for logs to be sent to the Forwarded Events logs__

__NOTE(S)__:
- TImestamps are preserved
- Log contents are preserved

---

## __REFERENCES:__
- https://www.vkernel.ro/blog/how-to-configure-windows-event-log-forwarding?fbclid=IwAR1bQ9VpgL--PWaqvEWcJBduR3xJ2UnBBhZmO7UGef-NXcKN9PCINZ3gmQ0
- https://www.itprotoday.com/strategy/q-what-are-some-simple-tips-testing-and-troubleshooting-windows-event-forwarding-and?fbclid=IwAR3ceGoJU-jgkD2U_rVo2FmQee5M0spvE85lZRVw0FHv4YFTphLaX-5JJe8
- https://rockyprogress.wordpress.com/2011/12/04/security-event-log-collection-from-a-domain-controller/?fbclid=IwAR01Puy9Wvr4eCQeV828raqfLesYJwVTw_8EAmDgvJIKYBVWoaT3giv24PA
- https://blogs.technet.microsoft.com/supportingwindows/2016/07/18/setting-up-a-source-initiated-subscription-on-an-event-collector-computer/?fbclid=IwAR2JagIePrComWaIcZknK_92Igakb4_jvnrmJJnGpZlFGnms_2PM7z6trJc
