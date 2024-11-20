# ideas

## idea:1
LLM workflow to help create investigation steps.  LLM doesn't have to be involved in the actual steps.

### Example:1
Alert: Name

Alert Definition:

Alert Source:

#### LLM Steps
Step:1
- Input
  - alert name and description
- Output
  - List of steps in order that should be taken.
  - List includes any immediate remediation actions and investigative steps.
  - Steps should cover at leat 3 objectives
    - Is there a critical or immediate risk warranting containment.  This should take into account the user and any involved assets.
    - Information and analysis steps needed to confirm false positive
    - Information and analysis steps needed to confirm true positive

Step:2 - For each investigation
- Input - Investigation step
  - List of data sources we have with descriptions
- Output
  - Investigation step with relevant data sources

Step:3 - For each data source and investigation step
- Input
  - Investigation step and data source
- Output
  - SQL query template to gather the data

Assumptions/Omitted:
- Add steps to evaluate output to ensure intent is met and output makes sense.
- Human approval steps as needed


### Example Result

#### Alert: Suspicious Logon Activity

**Alert Definition:**  
An alert is triggered when a user account logs in from two geographically distant locations within a short time frame, indicating potential account compromise.

**Alert Source:**  
SIEM (Security Information and Event Management) platform with log data from Active Directory and VPN systems.

---

### Investigation Steps  

#### Step 1: Assess Critical or Immediate Risk  
**Objective:** Determine if immediate containment is necessary.  

**Query Template:**  
1. Check user activity in the past 24 hours.  
   - **SQL Query:**  
     ```sql
     SELECT Timestamp, UserName, SourceIP, DestinationIP, ActivityType 
     FROM LogonEvents 
     WHERE UserName = '<USERNAME>'
       AND Timestamp >= DATEADD(day, -1, GETDATE())
     ORDER BY Timestamp DESC;
     ```

2. Identify if any high-privileged accounts were used.  
   - **SQL Query:**  
     ```sql
     SELECT UserName, PrivilegeLevel 
     FROM ActiveDirectory 
     WHERE UserName = '<USERNAME>';
     ```

3. Review involved devices for critical systems.  
   - **SQL Query:**  
     ```sql
     SELECT DeviceName, IsCritical 
     FROM AssetInventory 
     WHERE DeviceName IN (SELECT DISTINCT DestinationIP FROM LogonEvents WHERE UserName = '<USERNAME>');
     ```

---

#### Step 2: Confirm False Positive  

**Objective:** Analyze data to rule out legitimate causes for the activity.  

**Query Template:**  
1. Check VPN log data for session continuity.  
   - **SQL Query:**  
     ```sql
     SELECT Timestamp, UserName, SourceIP, VPNStatus 
     FROM VPNLogs 
     WHERE UserName = '<USERNAME>' 
       AND Timestamp BETWEEN DATEADD(hour, -6, GETDATE()) AND GETDATE();
     ```

2. Correlate login timestamps with known travel or schedule data.  
   - **SQL Query:**  
     ```sql
     SELECT TravelSchedule, Timestamp 
     FROM EmployeeActivity 
     WHERE UserName = '<USERNAME>';
     ```

3. Check recent changes in user location or IP address reassignment.  
   - **SQL Query:**  
     ```sql
     SELECT UserName, SourceIP, GeoLocation, Timestamp 
     FROM GeoLocationData 
     WHERE UserName = '<USERNAME>' 
       AND Timestamp BETWEEN DATEADD(day, -7, GETDATE()) AND GETDATE();
     ```

---

#### Step 3: Confirm True Positive  

**Objective:** Identify potential compromise and additional indicators.  

**Query Template:**  
1. Identify related alerts or anomalies involving the same user or IP.  
   - **SQL Query:**  
     ```sql
     SELECT AlertName, AlertTimestamp, UserName, SourceIP 
     FROM Alerts 
     WHERE UserName = '<USERNAME>' OR SourceIP = '<SOURCEIP>';
     ```

2. Review recent file access or sensitive data interaction.  
   - **SQL Query:**  
     ```sql
     SELECT FileName, AccessTime, AccessedBy 
     FROM FileAccessLogs 
     WHERE AccessedBy = '<USERNAME>' 
       AND AccessTime >= DATEADD(day, -1, GETDATE());
     ```

3. Evaluate potential lateral movement from involved devices.  
   - **SQL Query:**  
     ```sql
     SELECT SourceDevice, TargetDevice, ActivityTime 
     FROM NetworkTraffic 
     WHERE SourceDevice IN (SELECT DISTINCT DeviceName 
                            FROM AssetInventory 
                            WHERE DeviceName IN 
                               (SELECT DISTINCT DestinationIP 
                                FROM LogonEvents 
                                WHERE UserName = '<USERNAME>'))
       AND ActivityTime >= DATEADD(day, -1, GETDATE());
     ```

---

### Containment Steps  

1. Disable user account if compromise is confirmed.  
   - **Command:**  
     ```shell
     Disable-ADAccount -Identity '<USERNAME>'
     ```

2. Block involved IP addresses at the firewall.  
   - **Command:**  
     ```shell
     New-NetFirewallRule -DisplayName "Block Suspect IP" -RemoteAddress <SOURCEIP> -Action Block
     ```

3. Isolate affected devices from the network.  
   - **Command:**  
     ```shell
     Invoke-Command -ComputerName '<COMPUTERNAME>' -ScriptBlock { Disable-NetAdapter -Name "<AdapterName>" }
     ```