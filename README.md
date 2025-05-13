# ActiveDirectoryLab
<!DOCTYPE html>

  <h1>Active Directory & DHCP Lab: Full Walkthrough</h1>

  <div class="section">
    <h2>Overview</h2>
    <p>This lab demonstrates how to set up a small enterprise network using two virtual machines: a Windows Server 2019 instance configured as a Domain Controller with DHCP, and a Windows 10 client joined to the domain. Key objectives include domain promotion, DHCP scope creation, user account management through Active Directory, and bulk user creation via PowerShell scripting.</p>
    <img src="https://i.imgur.com/4fc6caa2-816b-44cc-8534-e7fb84cc2a5b.png" alt="Network Topology Diagram">
  </div>

  <div class="section">
    <h2>Step 1: Prepare Virtual Machines</h2>
    <ul>
      <li>Deployed <b>Windows Server 2019</b> on the first VM, designated as the Domain Controller (DC).</li>
      <li>Deployed <b>Windows 10 Pro</b> on the second VM to serve as the domain-joined client.</li>
      <li>Configured both VMs on an <b>internal virtual network</b> to facilitate isolated testing.</li>
    </ul>
    <img src="path_to_vm_setup_screenshot.png" alt="VM Setup Screenshot">
  </div>

  <div class="section">
    <h2>Step 2: Install and Configure Active Directory</h2>
    <ul>
      <li>On the Server VM, launched <b>Server Manager</b> &rarr; <b>Add Roles and Features</b>.</li>
      <li>Installed the <b>Active Directory Domain Services (AD DS)</b> role.</li>
    </ul>
    <img src="path_to_post_deployment_screenshot.png" alt="installed active directory">
    <ul>
      <li>Ran the <b>Post-Deployment Configuration Wizard</b> to promote the server to a domain controller in a new forest named <code>mydomain.com</code>.</li>
    </ul>
    <p>During post-deployment, I selected “Add a new forest,” entered <code>mydomain.com</code> as the root domain name, configured DNS options, set the Directory Services Restore Mode (DSRM) password, and completed the wizard. The server automatically restarted to finalize the promotion.</p>
    <img src="path_to_post_deployment_screenshot.png" alt="Post-Deployment Configuration Screenshot">
  </div>

  <div class="section">
    <h2>Step 3: Create a Domain Admin Account</h2>
    <ul>
      <li>Opened <b>Active Directory Users and Computers (ADUC)</b> on the DC.</li>
      <li>Created an Organizational Unit (OU) named <code>_ADMINS</code>.</li>
      <li>Manually created a user account (<code>a-asimmons</code>) within <code>_ADMINS</code>.</li>
      <li>Added <code>a-asimmons</code> to the <b>Domain Admins</b> group.</li>
    </ul>
    <img src="path_to_admin_user_creation_screenshot.png" alt="Admin Account Creation Screenshot">
  </div>

  <div class="section">
    <h2>Step 4: Install and Configure RAS/NAT</h2>
    <ul>
      <li>On the Domain Controller VM, opened <b>Server Manager</b> &rarr; <b>Add Roles and Features</b>.</li>
      <li>Installed the <b>Remote Access</b> role and the <b>DirectAccess and VPN (RAS)</b> features.</li>
    </ul>
    <img src="path_to_ras_nat_configuration_screenshot.png" alt="Ras Nat Installation Screenshot">
    <ul>
      <li>Launched the <b>Routing and Remote Access</b> console to enable RAS and configure NAT services.</li>
      <li>Configured the internal network interface for NAT, enabling client VMs to access external network resources through the server.</li>
    </ul>
    <img src="path_to_ras_nat_configuration_screenshot.png" alt="RAS NAT Configuration Screenshot">
  </div>

  <div class="section">
    <h2>Step 5: Configure DHCP</h2>
    <ul>
      <li>Installed the <b>DHCP Server</b> role via Server Manager on the DC.</li>
    </ul>
    <img src="path_to_dhcp_configuration_screenshot.png" alt="DHCP Server Install Screenshot">
    <ul>
      <li>Created a DHCP scope (range: <code>192.168.0.100</code> to <code>192.168.0.200</code>).</li>
    </ul>
    <img src="path_to_dhcp_configuration_screenshot.png" alt="DHCP Server Scope Config Screenshot">
    <ul>
      <li>Configured scope options: default gateway, DNS servers, and lease duration.</li>
    </ul>
    <img src="path_to_dhcp_configuration_screenshot.png" alt="DHCP Scope Configuration Screenshot">
    <ul>
      <li>Activated the scope and authorized the DHCP server in AD.</li>
    </ul>
    <img src="path_to_dhcp_configuration_screenshot.png" alt="DHCP Scope Configuration Screenshot">
  </div>

  <div class="section">
    <h2>Step 6: Bulk User Creation via PowerShell</h2>
    <p>This script reads a list of first and last names from <code>names.txt</code>, sets a secure default password, and creates an Organizational Unit (<code>_USERS</code>) to house the new accounts. For each name, it splits the string into first and last names, generates a username by combining the first initial with the lowercase last name, and then creates an AD user with properties like DisplayName and EmployeeID. PasswordNeverExpires is enabled, and all accounts are placed under <code>ou=_USERS</code>.</p>
    <pre><code>
$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last  = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName       $first `
               -Surname         $last `
               -DisplayName     $username `
               -Name            $username `
               -EmployeeID      $username `
               -PasswordNeverExpires $true `
               -Path            "ou=_USERS,$(([ADSI]"""").distinguishedName)" `
               -Enabled         $true
}
</code></pre>
    <img src="path_to_powershell_script_execution_screenshot.png" alt="PowerShell Script Execution Screenshot">
  </div>

  <div class="section">
    <h2>Step 7: Join the Client to the Domain</h2>
    <ul>
      <li>Verified Windows 10 client received a DHCP lease.</li>
    </ul>
    <img src="path_to_client_domain_join_screenshot.png" alt="Client Domain Join Screenshot">
    <ul>
      <li>Renamed the client to <code>Client01</code> for clarity in AD.</li>
      <li>Joined the client to the <code>mydomain.com</code> domain using <code>a-asimmons</code> credentials.</li>
    </ul>
    <img src="path_to_client_domain_join_screenshot.png" alt="Client Domain Join Screenshot">
    <ul>
      <li>Restarted the client and logged in with domain account.</li>
      <li>Opened <b>Active Directory Users and Computers</b> on the DC to confirm <code>Client01</code> appears under the Computers container.</li>
    </ul>
    <img src="path_to_client_domain_join_screenshot.png" alt="Client Domain Join Screenshot">
    <ul>
      <li>Checked the <b>DHCP</b> console to verify an active lease for <code>Client01</code> within the configured scope.</li>
    </ul>
    <img src="path_to_client_domain_join_screenshot.png" alt="Client Domain Join Screenshot">
  </div>

  <div class="section">
    <h2>Summary</h2>
    <p>In this lab, I configured a Windows Server 2019 host as a Domain Controller with AD DS, DNS, and DHCP, then joined a Windows 10 client to the domain. I streamlined administrative access by creating an OU for domain admins and provisioning accounts via the ADUC console. I enabled RAS/NAT to facilitate secure client connectivity beyond the isolated network. The DHCP role was deployed and scoped to automatically assign IP settings. To demonstrate automation, I scripted bulk user creation—generating unique credentials and organizing them under a dedicated OU. Finally, I validated the environment by verifying client leases in the DHCP console and confirming object visibility in Active Directory Users and Computers. This end-to-end exercise showcases key skills in Windows server deployment, service configuration, PowerShell automation, and network resource management.</p>
  </div>

</body>
</html>
