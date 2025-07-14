# Windows Server Network Protocol Standard

## Document Information
- **Document Title:** Windows Server Network Protocol Standard
- **Version:** 1.0
- **Effective Date:** [Date]
- **Review Date:** [Date + 1 Year]
- **Document Owner:** [Name/Role]
- **Approved By:** [Name/Role]

## 1. Purpose and Scope

### 1.1 Purpose
This standard defines the required configuration of network protocols on Windows Server systems to mitigate security risks associated with legacy name resolution protocols and insecure network communications.

### 1.2 Scope
This standard applies to all Windows Server systems (2012 R2 and later) in the organization, including:
- Domain Controllers
- Member Servers
- Standalone Servers
- Virtual Machines

### 1.3 Exclusions
[List any specific systems or scenarios excluded from this standard]

## 2. Standards Requirements

### 2.1 Multicast DNS (mDNS)
**Requirement:** mDNS must be disabled on all Windows Server systems.

**Configuration:**
- Registry Key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters`
- Value Name: `EnableMDNS`
- Value Type: `REG_DWORD`
- Value Data: `0` (Disabled)

**Group Policy Path:** Computer Configuration > Administrative Templates > Network > DNS Client > Turn off Multicast Name Resolution

### 2.2 Link-Local Multicast Name Resolution (LLMNR)
**Requirement:** LLMNR must be disabled on all Windows Server systems.

**Configuration:**
- Registry Key: `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient`
- Value Name: `EnableMulticast`
- Value Type: `REG_DWORD`
- Value Data: `0` (Disabled)

**Group Policy Path:** Computer Configuration > Administrative Templates > Network > DNS Client > Turn off Multicast Name Resolution

### 2.3 NetBIOS over TCP/IP (NBT-NS)
**Requirement:** NetBIOS over TCP/IP must be disabled on all network adapters.

**Configuration:**
- Network Adapter Properties > Internet Protocol Version 4 (TCP/IPv4) > Properties > Advanced > WINS Tab
- Set NetBIOS setting to: "Disable NetBIOS over TCP/IP"

**Alternative Registry Configuration:**
- Registry Key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetBT\Parameters\Interfaces\[Interface GUID]`
- Value Name: `NetbiosOptions`
- Value Type: `REG_DWORD`
- Value Data: `2` (Disable NetBIOS over TCP/IP)

### 2.4 SMBv1 Protocol
**Requirement:** SMBv1 must be disabled on all Windows Server systems.

**Configuration Methods:**
1. **PowerShell Command:**
   ```powershell
   Disable-WindowsOptionalFeature -Online -FeatureName smb1protocol
   ```

2. **Registry Configuration:**
   - Registry Key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters`
   - Value Name: `SMB1`
   - Value Type: `REG_DWORD`
   - Value Data: `0` (Disabled)

### 2.5 SMB Signing
**Requirement:** SMB signing must be required for both client and server communications.

**Server Configuration:**
- Registry Key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters`
- Value Name: `RequireSecuritySignature`
- Value Type: `REG_DWORD`
- Value Data: `1` (Required)

**Client Configuration:**
- Registry Key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters`
- Value Name: `RequireSecuritySignature`
- Value Type: `REG_DWORD`
- Value Data: `1` (Required)

**Group Policy Path:** Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options
- Microsoft network client: Digitally sign communications (always) - Enabled
- Microsoft network server: Digitally sign communications (always) - Enabled

### 2.6 IPv6 Configuration
**Requirement:** If IPv6 is not used in the environment, it should be disabled or configured securely.

**Disable IPv6 (if not used):**
- Registry Key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters`
- Value Name: `DisabledComponents`
- Value Type: `REG_DWORD`
- Value Data: `0xffffffff` (Disable all IPv6 components)

**Note:** Only disable IPv6 if it's not required for your environment.

## 3. Implementation Procedures

### 3.1 Pre-Implementation
1. Document current network protocol configurations
2. Test changes in a non-production environment
3. Coordinate with network and application teams
4. Schedule maintenance windows for production changes

### 3.2 Implementation Steps
1. Apply configurations via Group Policy where possible
2. For standalone servers, apply registry changes directly
3. Restart services or reboot as required
4. Verify configurations have taken effect
5. Document any issues or exceptions

### 3.3 Verification
**Registry Verification Commands:**
```cmd
reg query "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters" /v EnableMDNS
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" /v EnableMulticast
reg query "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" /v SMB1
```

**PowerShell Verification:**
```powershell
Get-WindowsOptionalFeature -Online -FeatureName smb1protocol
Get-SmbServerConfiguration | Select RequireSecuritySignature
```

## 4. Monitoring and Compliance

### 4.1 Monitoring Requirements
- Regular compliance scans using vulnerability scanners
- Periodic manual verification of key settings
- Event log monitoring for protocol-related security events

### 4.2 Compliance Reporting
- Monthly compliance reports to management
- Document any deviations with business justification
- Track remediation of non-compliant systems

## 5. Exceptions and Waivers

### 5.1 Exception Process
1. Business justification must be documented
2. Risk assessment must be completed
3. Compensating controls must be implemented
4. Management approval required
5. Regular review of exceptions

### 5.2 Approved Exceptions
[Document any approved exceptions here]

## 6. Roles and Responsibilities

### 6.1 IT Security Team
- Develop and maintain this standard
- Monitor compliance
- Approve exceptions

### 6.2 System Administrators
- Implement standard configurations
- Report compliance issues
- Maintain documentation

### 6.3 Network Team
- Coordinate network-level changes
- Validate network connectivity post-implementation

## 7. Related Documents
- Windows Server Base Hardening Standard
- Windows Server Firewall Standard
- Network Security Policy
- Change Management Procedures

## 8. Revision History
| Version | Date | Changes | Approved By |
|---------|------|---------|-------------|
| 1.0 | [Date] | Initial version | [Name] |

## 9. Appendices

### Appendix A: Security Rationale
**mDNS/LLMNR/NBT-NS Risks:**
- Credential harvesting attacks (Responder)
- Network reconnaissance
- Man-in-the-middle attacks
- Information disclosure

**SMBv1 Risks:**
- Known vulnerabilities (WannaCry, NotPetya)
- Lack of security features
- Deprecated protocol

### Appendix B: Troubleshooting
**Common Issues:**
- Legacy applications requiring NetBIOS
- Network browsing functionality loss
- Application connectivity issues

**Resolution Steps:**
1. Identify affected applications
2. Update application configurations
3. Implement DNS-based name resolution
4. Test connectivity thoroughly
