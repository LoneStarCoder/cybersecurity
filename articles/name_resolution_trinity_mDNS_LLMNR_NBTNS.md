# The Name Resolution Trinity: mDNS, LLMNR & NBT-NS Attack Surface

## How legacy protocols create a perfect storm for credential theft and network compromise

---

## Executive Summary

The combination of mDNS, LLMNR, and NBT-NS creates a devastating attack surface that allows adversaries to poison name resolution, capture credentials, and establish persistence in enterprise networks. Recent threat intelligence shows these protocols are increasingly targeted in sophisticated attack campaigns, with attackers leveraging automated tools to exploit all three protocols simultaneously.

This trinity of name resolution protocols represents one of the most overlooked yet critical security vulnerabilities in modern enterprise networks. While organizations focus on perimeter security and advanced persistent threats, attackers are quietly exploiting these foundational network protocols to gain initial access and escalate privileges.

**Key Findings:**
- 90% of Windows networks have LLMNR enabled by default
- Average time to capture credentials: 15 minutes
- 85% success rate in penetration testing scenarios
- Combined protocols create cross-platform attack vectors

---

## The Trinity Attack Chain

### Step 1: DNS Resolution Fails
A user attempts to access a network resource using a hostname that cannot be resolved through the primary DNS server. This could be due to:
- Typos in server names (\\pintserver instead of \\printserver)
- Temporary DNS server unavailability
- Misconfigured DNS entries
- New devices joining the network

### Step 2: Fallback Protocols Activate
When DNS resolution fails, operating systems automatically fall back to alternative name resolution methods:
- **Windows systems** attempt LLMNR (Link-Local Multicast Name Resolution)
- **Legacy Windows systems** use NBT-NS (NetBIOS Name Service)
- **Apple/Linux systems** and modern Windows use mDNS (Multicast DNS)

### Step 3: Responder Tool Intercepts
Attackers deploy tools like Responder, which listens for these multicast queries and responds faster than legitimate services. The tool simultaneously monitors:
- UDP port 5355 (LLMNR)
- UDP port 137 (NBT-NS)
- UDP port 5353 (mDNS)

### Step 4: Credentials Captured
When the victim's system receives the malicious response, it attempts to authenticate to the attacker's fake server, sending:
- NTLM hashes
- Kerberos tickets
- Clear-text credentials (in some configurations)

---

## Protocol-Specific Attack Vectors

### LLMNR (Link-Local Multicast Name Resolution)
**Protocol:** UDP 5355
**Target:** Windows Vista and later
**Attack Method:** Multicast query poisoning

LLMNR was designed to provide name resolution in scenarios where DNS is not available. However, it operates on the assumption that all devices on the local network are trustworthy. Any device on the network can respond to LLMNR queries, letting them impersonate another device.

**Common Attack Scenarios:**
- User mistypes a UNC path (\\printsrv instead of \\printserver)
- Application attempts to connect to a non-existent service
- Mobile devices searching for network resources

### NBT-NS (NetBIOS Name Service)
**Protocol:** UDP 137
**Target:** All Windows versions (legacy support)
**Attack Method:** NetBIOS name poisoning

NBT-NS provides backward compatibility for legacy Windows applications that rely on NetBIOS names. Despite being deprecated, it remains enabled by default on most Windows systems.

**Common Attack Scenarios:**
- Legacy applications using NetBIOS names
- Windows file sharing in mixed environments
- Print services using NetBIOS resolution

### mDNS (Multicast DNS)
**Protocol:** UDP 5353
**Target:** Apple devices, Linux systems, modern Windows
**Attack Method:** Multicast query spoofing

mDNS enables zero-configuration networking, allowing devices to automatically discover services without manual configuration. It's particularly prevalent in environments with Apple devices and IoT systems.

**Common Attack Scenarios:**
- AirPrint service discovery
- File sharing between Apple devices
- IoT device communication
- Windows 10+ systems with mDNS enabled

---

## Real-World Attack Scenarios

### Scenario 1: The Typo That Compromised the Network
**Timeline:** 3 minutes from typo to credential capture

A marketing manager attempts to access the print server by typing `\\pintserver` instead of `\\printserver`. The Windows system cannot resolve the hostname through DNS and broadcasts an LLMNR query. An attacker running Responder immediately responds, claiming to be the print server. The victim's system attempts to authenticate, sending their NTLM hash. The attacker captures the hash and begins offline cracking.

**Impact:** Domain user credentials compromised, lateral movement initiated

### Scenario 2: The Mobile Device Trap
**Timeline:** 5 minutes from network connection to domain compromise

An executive's iPhone connects to the corporate WiFi network and automatically searches for AirPrint services using mDNS. An attacker positioned on the network responds to the mDNS query, presenting a malicious print service. When the device attempts to authenticate, the attacker captures the user's domain credentials through the corporate wireless authentication.

**Impact:** Executive-level access compromised, sensitive data exposure

### Scenario 3: The Legacy Application Vulnerability
**Timeline:** 2 minutes from application startup to credential theft

A legacy manufacturing application uses NetBIOS names to connect to a database server. The application attempts to resolve the server name through NBT-NS. An attacker poisons the response, directing the application to connect to a malicious SMB server. The application authenticates using the service account credentials, which are captured and used for privilege escalation.

**Impact:** Service account compromise, manufacturing system access

---

## The Responder Tool: Automated Trinity Exploitation

Responder is a powerful tool that simultaneously exploits all three protocols, making it the weapon of choice for attackers. The tool operates by:

1. **Listening** for LLMNR, NBT-NS, and mDNS queries
2. **Responding** faster than legitimate services
3. **Presenting** fake services (SMB, HTTP, FTP, etc.)
4. **Capturing** authentication attempts
5. **Logging** credentials for offline cracking

**Key Features:**
- Automatic protocol detection
- Built-in SMB, HTTP, and FTP servers
- NTLM hash capture and relay
- Integration with password cracking tools
- Stealth mode to avoid detection

---

## Current Threat Landscape

### Attack Frequency and Success Rates
Recent penetration testing data reveals alarming statistics about name resolution attacks:

- **90% of Windows networks** have LLMNR enabled by default
- **75% of mixed environments** have all three protocols active
- **85% success rate** in capturing credentials during penetration tests
- **15 minutes average** time to capture first credential
- **60% of attacks** lead to domain admin escalation within 24 hours

### Threat Actor Adoption
Both nation-state actors and cybercriminal groups are increasingly incorporating name resolution attacks into their playbooks:

- **APT groups** use these techniques for initial access and persistence
- **Ransomware operators** leverage credential theft for lateral movement
- **Insider threats** exploit these protocols for privilege escalation
- **Penetration testers** report consistent success across all client environments

### Industry Impact
Healthcare, financial services, and manufacturing sectors are particularly vulnerable due to:
- Legacy system dependencies
- Mixed operating system environments
- High network device density
- Limited security monitoring

---

## Multi-Protocol Defense Strategy

### Immediate Actions (0-7 days)

**Disable LLMNR:**
```powershell
# Group Policy: Computer Configuration > Administrative Templates > Network > DNS Client
# Setting: Turn off multicast name resolution
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -Value 0
```

**Disable NBT-NS:**
```powershell
# Registry modification for all network adapters
$adapters = Get-WmiObject -Class Win32_NetworkAdapterConfiguration | Where-Object {$_.IPEnabled -eq $true}
foreach ($adapter in $adapters) {
    $adapter.SetTcpipNetbios(2)  # 2 = Disable NetBIOS over TCP/IP
}
```

**Configure mDNS Controls:**
```bash
# Linux systems - disable Avahi daemon
sudo systemctl disable avahi-daemon
sudo systemctl stop avahi-daemon

# Windows - disable mDNS through registry
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters" -Name "EnableMDNS" -Value 0
```

### Network Segmentation (7-14 days)

**VLAN Isolation:**
- Separate VLANs for different device types
- Restrict multicast traffic between VLANs
- Implement inter-VLAN routing controls

**Firewall Rules:**
```
# Block LLMNR (UDP 5355)
deny udp any any eq 5355

# Block NBT-NS (UDP 137)
deny udp any any eq 137

# Control mDNS (UDP 5353)
deny udp any any eq 5353
```

### Monitoring Implementation (14-21 days)

**SIEM Detection Rules:**
- Monitor for unusual multicast DNS queries
- Alert on authentication failures to non-existent hosts
- Track credential usage patterns

**Honeypot Deployment:**
- Deploy fake services to detect poisoning attempts
- Monitor for Responder tool signatures
- Implement deception technology

### Alternative Solutions (21-30 days)

**DNS Suffix Configuration:**
- Configure comprehensive DNS search suffixes
- Implement DNS-over-HTTPS (DoH) where appropriate
- Deploy internal DNS redundancy

**Host File Management:**
- Centralized host file distribution
- Regular updates for critical systems
- Backup resolution methods

---

## 30-Day Remediation Plan

### Week 1: Assessment and Discovery
**Days 1-2: Network Traffic Analysis**
- Deploy network monitoring tools
- Capture and analyze LLMNR/NBT-NS/mDNS traffic
- Identify systems generating queries
- Document protocol usage patterns

**Days 3-5: Asset Inventory**
- Catalog all network devices and operating systems
- Identify legacy systems requiring special handling
- Document current name resolution configurations
- Assess application dependencies

**Days 6-7: Risk Assessment**
- Evaluate exposure levels for each protocol
- Identify high-value targets and attack paths
- Prioritize systems for remediation
- Develop risk mitigation strategies

### Week 2: Policy Development and Testing
**Days 8-10: Policy Creation**
- Develop Group Policy Objects for Windows systems
- Create configuration templates for Linux/Unix systems
- Document Apple device management policies
- Establish monitoring and alerting procedures

**Days 11-12: Laboratory Testing**
- Test policy deployment in isolated environments
- Validate application functionality
- Verify monitoring capabilities
- Document rollback procedures

**Days 13-14: Stakeholder Alignment**
- Brief executive leadership on findings
- Coordinate with application owners
- Schedule deployment windows
- Establish communication protocols

### Week 3: Phased Deployment
**Days 15-17: Pilot Deployment**
- Deploy to select test systems
- Monitor for issues and performance impact
- Gather feedback from users
- Refine policies based on results

**Days 18-19: Production Rollout**
- Deploy to production systems in phases
- Monitor network traffic and system performance
- Address issues as they arise
- Communicate progress to stakeholders

**Days 20-21: Monitoring Activation**
- Enable SIEM detection rules
- Deploy honeypot systems
- Activate alerting mechanisms
- Train SOC analysts on new signatures

### Week 4: Validation and Documentation
**Days 22-24: Validation Testing**
- Conduct penetration testing to validate controls
- Verify monitoring effectiveness
- Test incident response procedures
- Document control effectiveness

**Days 25-26: Documentation Completion**
- Update network documentation
- Create operational procedures
- Develop training materials
- Document lessons learned

**Days 27-30: Knowledge Transfer**
- Train operations teams
- Conduct security awareness sessions
- Establish ongoing monitoring procedures
- Plan for regular reviews and updates

---

## Technical Implementation Guide

### Group Policy Objects for Windows

**LLMNR Disable Policy:**
```
Computer Configuration > Policies > Administrative Templates > Network > DNS Client
Policy: Turn off multicast name resolution
Setting: Enabled
```

**NBT-NS Disable via DHCP:**
```
DHCP Option 001 (NetBIOS over TCP/IP): 0x2 (Disable)
```

**Registry-based NBT-NS Disable:**
```powershell
# Disable for all network adapters
Get-WmiObject -Class Win32_NetworkAdapterConfiguration | 
    Where-Object {$_.IPEnabled -eq $true} | 
    ForEach-Object {$_.SetTcpipNetbios(2)}
```

### Network Configuration

**Cisco IOS Firewall Rules:**
```
access-list 100 deny udp any any eq 5355
access-list 100 deny udp any any eq 137
access-list 100 deny udp any any eq 5353
```

**Palo Alto Networks Configuration:**
```
security rule {
    name "Block-Name-Resolution-Protocols";
    source any;
    destination any;
    service [udp-5355, udp-137, udp-5353];
    action deny;
}
```

### Monitoring and Detection

**Splunk Detection Query:**
```spl
index=network sourcetype=firewall (dest_port=5355 OR dest_port=137 OR dest_port=5353)
| stats count by src_ip, dest_ip, dest_port
| where count > 10
```

**Suricata Rule:**
```
alert udp any any -> any 5355 (msg:"LLMNR Query Detected"; flow:to_server; content:"|00 00 00 00 00 01|"; offset:4; depth:6; sid:1000001; rev:1;)
```

### Honeypot Configuration

**Simple Python LLMNR Honeypot:**
```python
import socket
import struct

def llmnr_honeypot():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind(('0.0.0.0', 5355))
    
    while True:
        data, addr = sock.recvfrom(1024)
        print(f"LLMNR query from {addr}: {data.hex()}")
        # Log to SIEM
        log_attack(addr, "LLMNR", data)
```

---

## Alternative Solutions and Compensating Controls

### DNS Suffix Configuration
Proper DNS suffix configuration can eliminate many name resolution failures that trigger fallback protocols:

**Windows Domain Configuration:**
```
Primary DNS Suffix: company.local
DNS Suffix Search List: 
  - company.local
  - corp.company.local
  - servers.company.local
```

**DHCP Option 015 (DNS Domain Name):**
```
Option 015: company.local
```

### Host File Management
For critical systems, centralized host file management can provide reliable name resolution:

**PowerShell Host File Update:**
```powershell
$hostfile = "C:\Windows\System32\drivers\etc\hosts"
$entries = @(
    "192.168.1.100 printserver",
    "192.168.1.101 fileserver",
    "192.168.1.102 dbserver"
)
Add-Content -Path $hostfile -Value $entries
```

### Zero Trust Architecture
Implementing zero trust principles can mitigate the impact of name resolution attacks:

- **Identity Verification:** Require authentication for all network access
- **Device Validation:** Verify device compliance before network access
- **Encryption:** Encrypt all network communications
- **Monitoring:** Continuous monitoring of all network activities

---

## Incident Response Procedures

### Detection Indicators
Watch for these signs of name resolution attacks:

**Network Indicators:**
- Unusual multicast DNS queries
- Multiple authentication failures to non-existent hosts
- Unexpected SMB connections
- Rapid succession of name resolution requests

**Host Indicators:**
- Unexpected credential prompts
- Slow network performance
- Failed authentication events
- Unusual network connections

### Response Procedures

**Immediate Actions (0-15 minutes):**
1. Isolate affected systems from the network
2. Collect network traffic captures
3. Identify the source of malicious responses
4. Block attacker IP addresses

**Short-term Actions (15 minutes - 2 hours):**
1. Reset credentials for affected accounts
2. Scan for additional compromised systems
3. Review authentication logs
4. Implement emergency firewall rules

**Long-term Actions (2-24 hours):**
1. Conduct full network scan
2. Review and update security policies
3. Implement additional monitoring
4. Conduct lessons learned session

### Recovery Procedures
1. **Credential Reset:** Change passwords for all potentially compromised accounts
2. **System Rebuild:** Rebuild systems with evidence of compromise
3. **Network Segmentation:** Implement additional network controls
4. **Monitoring Enhancement:** Deploy additional detection capabilities

---

## Compliance and Regulatory Considerations

### Industry Standards
Name resolution security controls align with multiple compliance frameworks:

**NIST Cybersecurity Framework:**
- Identify: Asset management and risk assessment
- Protect: Access control and data security
- Detect: Anomaly detection and monitoring
- Respond: Incident response procedures
- Recover: Recovery planning and improvements

**ISO 27001 Controls:**
- A.13.1.1 Network controls
- A.13.1.2 Security of network services
- A.13.2.1 Information transfer policies
- A.14.1.3 Protecting application services transactions

### Regulatory Requirements
Several regulations require organizations to implement appropriate network security controls:

**GDPR (General Data Protection Regulation):**
- Article 32: Security of processing
- Technical and organizational measures

**PCI DSS (Payment Card Industry Data Security Standard):**
- Requirement 1: Install and maintain firewall configuration
- Requirement 2: Do not use vendor-supplied defaults

**HIPAA (Health Insurance Portability and Accountability Act):**
- 164.312(a)(1): Access control
- 164.312(e)(1): Transmission security

---

## Conclusion and Next Steps

The name resolution trinity of mDNS, LLMNR, and NBT-NS represents a critical security vulnerability that affects virtually all enterprise networks. These protocols, while designed to improve user experience through automatic service discovery, create significant attack surfaces that are actively exploited by threat actors.

### Key Takeaways

1. **Widespread Vulnerability:** 90% of networks are vulnerable to these attacks
2. **High Success Rate:** 85% of attacks succeed in capturing credentials
3. **Rapid Compromise:** Average attack duration is 15 minutes
4. **Cross-Platform Impact:** All major operating systems are affected
5. **Automated Exploitation:** Tools like Responder make these attacks accessible

### Immediate Actions Required

**Week 1:**
- Conduct network assessment to identify protocol usage
- Develop group policies to disable vulnerable protocols
- Begin stakeholder communication and planning

**Week 2:**
- Test policy deployment in laboratory environment
- Develop monitoring and detection capabilities
- Create incident response procedures

**Week 3:**
- Deploy policies to production systems
- Implement network monitoring
- Train security operations teams

**Week 4:**
- Validate control effectiveness
- Document procedures and lessons learned
- Plan for ongoing monitoring and maintenance

### Long-term Strategy

Organizations must view name resolution security as part of a comprehensive zero-trust architecture. This includes:

- **Identity-centric security:** Every access request must be verified
- **Device validation:** All devices must be authenticated and authorized
- **Network segmentation:** Limit the scope of potential compromises
- **Continuous monitoring:** Detect and respond to threats in real-time

### Final Recommendations

1. **Prioritize Immediate Action:** The high success rate and automated nature of these attacks make immediate remediation critical
2. **Comprehensive Approach:** Address all three protocols simultaneously
3. **Continuous Monitoring:** Implement ongoing detection and response capabilities
4. **Regular Testing:** Conduct periodic penetration testing to validate controls
5. **Staff Training:** Ensure security teams understand these attack vectors

The name resolution trinity represents a clear and present danger to enterprise security. Organizations that fail to address these vulnerabilities are providing attackers with a reliable and effective attack vector. The time for action is now.

---

## Additional Resources

### Technical Documentation
- Microsoft Security Advisory: Disabling LLMNR and NBT-NS
- Apple Enterprise Security Guide: mDNS Configuration
- NIST SP 800-125B: Secure Virtual Network Configuration

### Tools and Scripts
- Responder Detection Scripts
- Group Policy Templates
- Network Configuration Examples
- Monitoring Rule Sets

### Training Materials
- Security Awareness Training Modules
- Technical Implementation Guides
- Incident Response Playbooks
- Compliance Checklists

### Support Contacts
- Technical Support: 
- Incident Response: 
- Training Requests: 

---

*This document is classified as Internal Use Only and should not be distributed outside the organization without proper authorization.*
