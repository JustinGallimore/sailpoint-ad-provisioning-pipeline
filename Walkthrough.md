# SailPoint ISC — Active Directory Provisioning Pipeline
## Visual Walkthrough | Phase 2 of 3

This document walks through every step of the Phase 2 lab build. Each screenshot is paired with an explanation of what is happening, why it matters, and how it connects to real enterprise IAM work. This is not just a configuration guide. It is a record of how a production-grade identity provisioning pipeline is built, tested, and troubleshot from scratch.

---

## Table of Contents

1. [Infrastructure Build — Windows Server 2022 VM](#section-1)
2. [Windows Server 2022 Installation](#section-2)
3. [Active Directory Installation and Domain Promotion](#section-3)
4. [Domain Controller Verification](#section-4)
5. [SailPoint AD Source Configuration](#section-5)
6. [Virtual Appliance Migration](#section-6)
7. [AD Aggregation and OU Structure](#section-7)
8. [Access Profile and Lifecycle State Configuration](#section-8)
9. [IQService Installation and TLS Configuration](#section-9)
10. [Joiner Workflow Execution and Results](#section-10)

---

<a name="section-1"></a>
## Section 1 — Infrastructure Build: Windows Server 2022 VM

Before SailPoint can provision accounts into Active Directory, there needs to be an Active Directory to provision into. This section covers building the Windows Server 2022 domain controller from scratch inside VMware Workstation 16 Pro.

---

### Step 1 — New VM Wizard
**Screenshot: 01_vmware_new_vm_wizard.png**

The New Virtual Machine Wizard is opened in VMware Workstation 16 Pro. This is where the DC01 build begins. A custom configuration is selected rather than typical to allow full control over hardware specifications. In enterprise environments domain controllers are sized carefully because they handle authentication for every user and device on the network.

---

### Step 2 — Custom Hardware Wizard
**Screenshot: 02_vmware_custom_hardware_wizard.png**

The custom hardware wizard opens allowing manual specification of every hardware parameter. This level of control is important in a lab because resources are shared across multiple VMs running simultaneously including the SailPoint Virtual Appliance.

---

### Step 3 — ISO Selection
**Screenshot: 03_vmware_iso_selection.png**

The Windows Server 2022 Standard Evaluation ISO is selected as the installation media. This ISO was downloaded directly from Microsoft at 940Mbps. Using evaluation media is standard practice for lab environments and provides full functionality for 180 days.

---

### Step 4 — OS Type Selection
**Screenshot: 04_vmware_os_type_selected.png**

VMware is configured to recognize this VM as a Windows Server 2022 machine. This ensures VMware applies the correct hardware compatibility settings and driver optimizations for the operating system.

---

### Step 5 — VM Name and Location
**Screenshot: 05_vmware_vm_name_location.png**

The VM is named Windows Server 2022 DC01 and the storage location is set on the G drive under the HomeLab project folder structure. Consistent naming conventions matter in enterprise environments where dozens of VMs may exist across multiple hosts.

---

### Step 6 — Processor Configuration
**Screenshot: 06_vmware_processor_config.png**

4 processor cores are allocated to DC01. A domain controller handling authentication, DNS, and Group Policy for a small lab environment runs efficiently on 4 cores. Overallocating CPU to one VM starves other VMs running simultaneously.

---

### Step 7 — Firmware Type
**Screenshot: 07_vmware_firmware_type.png**

UEFI firmware is selected for DC01. Modern Windows Server installations perform better on UEFI compared to legacy BIOS. This also enables Secure Boot which is standard in production server deployments.

---

### Step 8 — RAM Configuration
**Screenshot: 08_vmware_ram_config.png**

4GB of RAM is allocated to DC01. This is the minimum recommended for a Windows Server 2022 domain controller and is sufficient for a lab environment handling a small number of identities and authentication requests.

---

### Step 9 — Network Configuration: Bridged
**Screenshot: 11_vmware_network_bridged.png**

The network adapter is set to Bridged mode pointing to the Intel I226-V physical adapter. This is a critical configuration decision. Bridged networking places the VM directly on the physical network, giving DC01 its own IP address on the 192.168.50.0/24 subnet. This is required for the SailPoint Virtual Appliance to communicate with DC01 over the network. NAT networking would have isolated DC01 behind the host machine and broken connectivity.

---

### Step 10 — IO Controller
**Screenshot: 12_vmware_io_controller.png**

The LSI Logic SAS IO controller is selected for optimal disk performance with Windows Server 2022. Controller selection affects how the VM communicates with its virtual disk and impacts overall read/write performance.

---

### Step 11 — Disk Type
**Screenshot: 13_vmware_disk_type.png**

NVMe is selected as the disk type. NVMe provides significantly faster disk IO compared to SCSI or SATA virtual disk types. For a domain controller handling Active Directory database operations, faster disk access improves authentication response times.

---

### Step 12 — Select Disk
**Screenshot: 14_vmware_select_disk.png**

A new virtual disk is created for DC01. Creating a fresh disk ensures a clean installation with no residual data from previous VMs.

---

### Step 13 — Disk Capacity
**Screenshot: 15_vmware_disk_capacity.png**

60GB is allocated for the DC01 virtual disk. This provides more than enough space for Windows Server 2022, the Active Directory database, system logs, and any additional tooling installed on the domain controller.

---

### Step 14 — Disk File
**Screenshot: 16_vmware_disk_file.png**

The virtual disk file location is confirmed on the G drive within the project folder. Keeping all VM files organized under the project folder makes it easy to back up, move, or archive the entire lab.

---

### Step 15 — CDVD ISO Connected
**Screenshot: 17_vmware_cdvd_iso_connected.png**

The Windows Server 2022 ISO is connected to the virtual CDVD drive. This is the installation media the VM will boot from on first startup.

---

### Step 16 — VM Created
**Screenshot: 18_vmware_vm_created.png**

The VM creation wizard completes and DC01 appears in the VMware library. The VM is ready to boot and begin the Windows Server 2022 installation process.

---

<a name="section-2"></a>
## Section 2 — Windows Server 2022 Installation

With the VM built, Windows Server 2022 is installed and the server is prepared for the Active Directory role.

---

### Step 17 — Setup Boot
**Screenshot: 19_windows_server_setup_boot.png**

Windows Server 2022 setup boots from the ISO. The installation wizard initializes and prepares to configure language, time, and keyboard settings.

---

### Step 18 — Edition Selection
**Screenshot: 20_windows_server_edition_selection.png**

Windows Server 2022 Standard with Desktop Experience is selected. Desktop Experience provides the full GUI interface which is appropriate for a lab environment where visibility into the server configuration is important for learning and documentation.

---

### Step 19 — Install Type
**Screenshot: 21_windows_server_install_type.png**

A custom installation is selected to perform a clean install on the new virtual disk. This ensures no residual operating system data exists on the drive.

---

### Step 20 — Drive Selection
**Screenshot: 22_windows_server_drive_selection.png**

The 60GB virtual disk created in VMware is selected as the installation target. Windows Server 2022 setup will partition and format this disk automatically.

---

### Step 21 — Installing
**Screenshot: 23_windows_server_installing.png**

Windows Server 2022 installation is in progress. The setup process copies files, installs features, and prepares the operating system on the virtual disk.

---

### Step 22 — Admin Password
**Screenshot: 24_windows_server_admin_password.png**

The local Administrator password is set during initial setup. This password is used for all subsequent logins and service configurations including IQService authentication later in the project.

---

### Step 23 — Desktop First Login
**Screenshot: 25_windows_server_desktop_first_login.png**

Windows Server 2022 Desktop Experience loads for the first time after installation. Server Manager opens automatically which is the central management console for adding roles and features to the server.

---

### Step 24 — VMware Tools Install
**Screenshot: 26_vmware_tools_install.png**

VMware Tools is installed on DC01. This is a required step that installs drivers and utilities enabling full resolution display, clipboard sharing, and improved VM performance. After installing VMware Tools DC01 renders at full 4K resolution on the Samsung S90C OLED display.

---

### Step 25 — Desktop VMware Tools
**Screenshot: 27_windows_server_desktop_vmware_tools.png**

DC01 desktop is now running at full resolution with VMware Tools installed. The server is ready for the Active Directory Domain Services role installation.

---

### Step 26 — Network Editor
**Screenshot: 32_vmware_network_editor.png**

The VMware Virtual Network Editor is opened as Administrator to resolve a bridged adapter issue. The VMnet0 bridged adapter was not pointing to the correct physical network adapter. It was reassigned to the Intel I226-V adapter which is the active network card in the host machine. This is a common VMware configuration issue that causes VMs to fail to obtain an IP address or communicate with other devices on the network.

---

### Step 27 — Rename to DC01
**Screenshot: 33_windows_server_dc01.png**

The server is renamed from the default Windows-generated name to DC01. Proper server naming is important in enterprise environments because Active Directory, DNS records, and service configurations all reference the server by its hostname. Renaming after AD promotion causes significant issues, so this is done before any roles are installed.

---

### Step 28 — DC01 Desktop
**Screenshot: 33_windows_server_dc01_desktop.png**

DC01 desktop confirms the server has been successfully renamed. Server Manager shows the new hostname in the top right. The server is now ready for the Active Directory Domain Services role.

---

<a name="section-3"></a>
## Section 3 — Active Directory Installation and Domain Promotion

This section covers installing the Active Directory Domain Services role and promoting DC01 to a domain controller, creating the lab.local forest and domain from scratch.

---

### Step 29 — AD Roles Wizard
**Screenshot: 34_ad_roles_wizard.png**

The Add Roles and Features Wizard is opened from Server Manager. This wizard guides the installation of the Active Directory Domain Services role including all required dependencies and management tools.

---

### Step 30 — Installation Type
**Screenshot: 35_ad_installation_type.png**

Role-based installation is selected. This installs the AD DS role on the local server DC01 rather than deploying to a remote server or virtual desktop infrastructure.

---

### Step 31 — Server Selection
**Screenshot: 36_ad_server_selection.png**

DC01 is selected as the target server. The server pool shows DC01 with its current IP address confirming it is reachable and ready for role installation.

---

### Step 32 — Role Selected
**Screenshot: 37_ad_role_selected.png**

Active Directory Domain Services is selected from the roles list. This role includes the AD DS binaries, Group Policy Management Console, Remote Server Administration Tools, and the Active Directory PowerShell module. All of these tools are needed for managing the lab.local domain.

---

### Step 33 — Features Selection
**Screenshot: 38_ad_features_selection.png**

Additional features required by AD DS are confirmed. The wizard automatically selects dependencies including .NET Framework components and remote management tools.

---

### Step 34 — AD DS Info Screen
**Screenshot: 39_ad_ds_info_screen.png**

The AD DS information screen explains what the role does and what is required to complete the deployment. Notably it explains that after installation the server must be promoted to a domain controller through a separate configuration wizard.

---

### Step 35 — Confirmation Screen
**Screenshot: 40_ad_confirmation_screen.png**

The installation summary is reviewed before proceeding. All selected roles and features are listed. Installation is confirmed and the wizard begins copying and installing the AD DS binaries.

---

### Step 36 — Install Complete
**Screenshot: 41_ad_install_complete.png**

Active Directory Domain Services installation completes successfully. A notification in Server Manager prompts to promote this server to a domain controller. This is the next required step to actually create the domain.

---

### Step 37 — New Forest lab.local
**Screenshot: 42_ad_new_forest_lab_local.png**

The Active Directory Domain Services Configuration Wizard opens. A new forest is created with the root domain name lab.local. Creating a new forest means this DC01 will be the first and authoritative domain controller for this environment. In enterprise environments forests define the outermost security boundary of Active Directory.

---

### Step 38 — DC Options
**Screenshot: 43_ad_dc_options.png**

Domain controller options are configured. Forest Functional Level and Domain Functional Level are both set to Windows Server 2016 for maximum compatibility. The Directory Services Restore Mode password is set which is used for AD recovery operations. DNS Server is checked which installs the DNS role on DC01 so it can resolve names within the lab.local domain.

---

### Step 39 — DNS Options
**Screenshot: 44_ad_dns_options.png**

DNS delegation options are reviewed. Since this is a new forest with no parent DNS zone, the delegation warning is expected and can be safely dismissed in a lab environment.

---

### Step 40 — Additional Options
**Screenshot: 45_ad_additional_options.png**

The NetBIOS domain name is confirmed as LAB. NetBIOS names are the legacy short names used for backwards compatibility with older Windows systems. In a lab environment this is rarely used directly but is still required during domain creation.

---

### Step 41 — AD Paths
**Screenshot: 46_ad_paths.png**

The default paths for the AD DS database, log files, and SYSVOL folder are accepted. In production environments these are often placed on separate volumes for performance and recovery purposes. For this lab the defaults are sufficient.

---

### Step 42 — Review Options
**Screenshot: 47_ad_review_options.png**

The full configuration summary is reviewed before promotion begins. Forest name, domain name, functional levels, DNS installation, and NetBIOS name are all confirmed correct.

---

### Step 43 — Prerequisites Passed
**Screenshot: 48_ad_prerequisites_passed.png**

All prerequisite checks pass successfully. The wizard confirms DC01 meets all requirements to be promoted to a domain controller. Any warnings shown here are informational and expected in a lab environment. The Promote button is now active and promotion can begin.

---

<a name="section-4"></a>
## Section 4 — Domain Controller Verification

After promotion DC01 is verified to confirm Active Directory is running correctly and the domain structure is intact.

---

### Step 44 — Domain Controller Desktop
**Screenshot: 49_dc01_domain_controller_desktop.png**

DC01 reboots after promotion and logs back in as LAB\Administrator. The domain prefix on the login confirms the server has successfully joined and is now serving the lab.local domain. Server Manager shows AD DS and DNS in the left navigation confirming both roles are active.

---

### Step 45 — Active Directory Users and Computers
**Screenshot: 50_active_directory_users_computers.png**

Active Directory Users and Computers is opened showing the lab.local domain structure. The default containers are visible including Builtin, Computers, Domain Controllers, ForeignSecurityPrincipals, and Users. DC01 appears under Domain Controllers confirming it is the authoritative domain controller for the lab.local forest.

---

### Step 46 — LabUsers OU Expanded
**Screenshot: 51_ad_lab_local_expanded.png**

The OU structure created for the lab is visible in Active Directory Users and Computers. The LabUsers OU contains five sub-OUs representing the departments in the CSV HR source: Engineering, Finance, HR, IT, and Marketing. This mirrors real enterprise AD designs where accounts are organized by department or business unit to simplify Group Policy application and access management.

---

### Step 47 — Test User Created
**Screenshot: 52_ad_test_user_created.png**

A test user account is manually created in Active Directory to verify that account creation works correctly before connecting SailPoint. This is a validation step that confirms AD is healthy and accepting new accounts before introducing automation into the process.

---

### Step 48 — DC01 Network Verified
**Screenshot: 53_dc01_network_verified.png**

Network connectivity from DC01 is verified. The static IP address 192.168.50.10 is confirmed and connectivity to other devices on the 192.168.50.0/24 network is tested. A static IP is required for DC01 because SailPoint ISC and IQService both reference the domain controller by its IP address. A DHCP address could change and break the connection.

---

### Step 49 — Pinging SailPoint VA
**Screenshot: 54_dc01_pinging_sailpoint_va.png**

DC01 successfully pings the SailPoint Virtual Appliance at 192.168.50.211 confirming bidirectional network connectivity between the two systems. This is a critical validation step. If DC01 and the VA cannot reach each other over the network, SailPoint will never be able to provision accounts regardless of how everything else is configured.

---

<a name="section-5"></a>
## Section 5 — SailPoint AD Source Configuration

With DC01 verified and Active Directory running, the next step is connecting SailPoint ISC to the lab.local domain by creating and configuring the AD source.

---

### Step 50 — SailPoint Sources Page
**Screenshot: 55_sailpoint_sources_page.png**

The Sources page in SailPoint ISC Admin shows the existing HR_Source_CSV_Lab from Phase 1. A new source will be created here for Active Directory. Sources in SailPoint represent the systems that SailPoint reads from and provisions into. Each source has its own connector, schema, and provisioning configuration.

---

### Step 51 — AD Connector Search
**Screenshot: 56_sailpoint_ad_connector_search.png**

The source creation wizard is opened and Active Directory is searched in the connector library. SailPoint has native connectors for Active Directory, Azure AD, and hundreds of other enterprise systems. The Active Directory connector uses LDAP to read from AD and IQService to write to it.

---

### Step 52 — AD Source Create
**Screenshot: 57_sailpoint_ad_source_create.png**

The Active Directory source is selected and the source creation begins. The source is named AD_Lab_DC01 to clearly identify which domain controller it connects to. Descriptive naming is important in environments with multiple AD sources representing different domains or forests.

---

### Step 53 — Forest Settings
**Screenshot: 58_sailpoint_ad_forest_settings.png**

Forest Settings are configured with the forest name lab.local and the domain controller IP 192.168.50.10 on port 3268. Port 3268 is the Global Catalog port used for forest-wide searches in Active Directory. The Global Catalog contains a partial replica of all objects in the forest and is the correct port for SailPoint to use when discovering accounts across the entire forest.

---

### Step 54 — Forest Settings Saved
**Screenshot: 59_sailpoint_ad_forest_settings_saved.png**

Forest settings are saved successfully. SailPoint now knows the entry point into the lab.local forest.

---

### Step 55 — Domain Settings
**Screenshot: 60_sailpoint_ad_domain_settings.png**

Domain Settings are configured with the base DN DC=lab,DC=local on port 389. Port 389 is the standard LDAP port used for domain-level operations. The base DN tells SailPoint where to start searching for accounts within the domain. DC=lab,DC=local is the root of the lab.local domain.

---

### Step 56 — Domain Settings Saved
**Screenshot: 61_sailpoint_ad_domain_settings_saved.png**

Domain settings are saved. SailPoint now has both the forest entry point and the domain search base configured. These two settings together define the scope of what SailPoint can see and manage in Active Directory.

---

### Step 57 — Review Summary
**Screenshot: 62_sailpoint_ad_review_summary.png**

The source configuration summary is reviewed showing all settings before saving. Source name, virtual appliance cluster, forest settings, and domain settings are all confirmed correct.

---

### Step 58 — Account Group Settings
**Screenshot: 63_sailpoint_ad_account_group_settings.png**

Account and Group Settings are configured. This tells SailPoint which object classes in Active Directory represent user accounts and groups. The standard values for user accounts are the person and organizationalPerson object classes. These settings define what SailPoint treats as an identity versus a group entitlement.

---

### Step 59 — Servers Fixed
**Screenshot: 64_sailpoint_ad_servers_fixed.png**

The domain controller server list is confirmed with DC01 at 192.168.50.10. In environments with multiple domain controllers this list would contain all DCs for redundancy. SailPoint will use this list to connect to Active Directory for aggregation and provisioning operations.

---

### Step 60 — Test Connection Success
**Screenshot: 65_sailpoint_ad_test_connection_success.png**

The test connection from SailPoint ISC to Active Directory returns successfully. This confirms that the SailPoint Virtual Appliance can reach DC01 on port 389, authenticate with the provided credentials, and perform an LDAP bind to the lab.local domain. A successful test connection here means the read path from SailPoint to AD is fully operational.

---

### Step 61 — Account Schema
**Screenshot: 66_sailpoint_ad_account_schema.png**

The account schema is configured with 88 attributes mapped from Active Directory. The schema defines every attribute SailPoint can read from and write to AD accounts. Key attributes include sAMAccountName, distinguishedName, displayName, mail, userPrincipalName, and memberOf. Having a complete schema is important for accurate identity correlation and provisioning.

---

### Step 62 — Correlation Saved
**Screenshot: 68_sailpoint_ad_correlation_saved.png**

Account correlation is configured with the rule Work Email equals mail. This tells SailPoint how to match an AD account to a SailPoint identity. When SailPoint reads an AD account it compares the mail attribute to the Work Email on each identity. If they match the account is correlated to that identity. Without correlation SailPoint treats every AD account as unowned and cannot associate it with an identity for governance purposes.

---

### Step 63 — Account Aggregation
**Screenshot: 69_sailpoint_ad_account_aggregation.png**

The first account aggregation is run on AD_Lab_DC01. This is the process where SailPoint connects to Active Directory, reads all accounts under the configured base DN, and imports them into the ISC tenant. The aggregation runs through the Virtual Appliance which proxies the LDAP connection to DC01.

---

### Step 64 — Aggregation Success
**Screenshot: 70_sailpoint_ad_aggregation_success.png**

Account aggregation completes successfully. SailPoint has imported all existing accounts from Active Directory including Administrator, Guest, krbtgt, and the test user created earlier. This confirms the full read path from SailPoint ISC through the Virtual Appliance to Active Directory is working correctly.

---

### Step 65 — Accounts Discovered
**Screenshot: 71_sailpoint_ad_accounts_discovered.png**

The accounts page for AD_Lab_DC01 shows all 4 discovered accounts. Each account shows its correlation status. The default system accounts like Administrator, Guest, and krbtgt show as Uncorrelated because there are no matching SailPoint identities for them. The test user also shows Uncorrelated because it was created manually without a corresponding CSV identity. This is expected behavior at this stage.

---

<a name="section-6"></a>
## Section 6 — Virtual Appliance Migration

Before provisioning could be configured, a critical infrastructure issue was discovered and resolved. The SailPoint Virtual Appliance running in VirtualBox could not communicate with DC01 running in VMware despite both using bridged networking on the same physical adapter.

This is one of the most important troubleshooting sections of the project because it reflects a real enterprise scenario where two systems appear to be on the same network but cannot reach each other due to virtual networking isolation.

---

### Step 66 — Entitlement Aggregation
**Screenshot: 72_ad_lab_users_created.png**

After resolving the VA migration, entitlement aggregation is run on AD_Lab_DC01 pulling in 48 entitlements from Active Directory including all groups, OUs, and security principals. Entitlements in SailPoint represent the permissions and group memberships that can be assigned to identities through access profiles and roles.

---

### Step 67 — Account Schema Confirmed
**Screenshot: 66_sailpoint_ad_account_schema.png**

The account schema is reviewed after migration to confirm all 88 attributes are still mapped correctly. The VA migration did not affect the source configuration in SailPoint ISC because the configuration lives in the cloud tenant, not on the VA itself.

---

<a name="section-7"></a>
## Section 7 — AD Aggregation and OU Structure

With the VA migrated and connectivity confirmed, the OU structure is built and the provisioning target is established.

---

### Step 68 — OU Structure Complete
**Screenshot: 73_ad_ou_structure_complete.png**

The OU structure in Active Directory is confirmed complete. LabUsers contains five department sub-OUs matching the departments in the CSV HR source. Engineering, Finance, HR, IT, and Marketing are all present. This OU structure is where SailPoint will create new accounts when the Joiner workflow fires. The distinguishedName in the provisioning policy points to OU=LabUsers,DC=lab,DC=local as the target container.

---

<a name="section-8"></a>
## Section 8 — Access Profile and Lifecycle State Configuration

The access profile connects the AD entitlement to the identity lifecycle, ensuring that every active employee automatically receives an AD account.

---

### Step 69 — Create Account Policy
**Screenshot: 74_sailpoint_create_account_policy.png**

The Create Account provisioning policy is configured on the AD_Lab_DC01 source. This policy defines exactly what SailPoint will write to Active Directory when creating a new account. It maps SailPoint identity attributes to AD attributes including distinguishedName, sAMAccountName, displayName, and mail.

---

### Step 70 — Static Value for sAMAccountName
**Screenshot: 75_sailpoint_dn_static_value.png**

The distinguishedName is configured as a Static value pointing to OU=LabUsers,DC=lab,DC=local. This tells SailPoint exactly which OU to create new accounts in. In a production environment this would typically be a dynamic rule that routes accounts to the correct department OU based on the user's department attribute. For this lab a static OU keeps the configuration straightforward.

---

### Step 71 — Create Account Complete
**Screenshot: 76_sailpoint_create_account_complete.png**

The provisioning policy configuration is saved. SailPoint now knows what to write to Active Directory when a new account needs to be created. The policy includes all required attributes and the target OU location.

---

### Step 72 — AD Lab DC01 Test Connection Success
**Screenshot: 77_AD_Lab_DC01_Test_Connection_Success.png**

A final test connection is run confirming SailPoint ISC can reach DC01 through the VA and authenticate to Active Directory. This green status confirms the read path is healthy before enabling automated provisioning.

---

### Step 73 — Entitlement Aggregation
**Screenshot: 78_AD_Lab_DC01_Entitlement_Aggregation.png**

Entitlement aggregation is run to pull all available AD entitlements into SailPoint. This is required before an access profile can be created because access profiles reference specific entitlements by name. The aggregation discovers all groups and OUs that exist in Active Directory.

---

### Step 74 — Access Profile Enabled
**Screenshot: 79_AD_LabUsers_Access_Profile_Enabled.png**

The AD_LabUsers_Access access profile is created with AD_Lab_DC01 as the source and Domain Users as the entitlement. This access profile is what SailPoint grants to identities when they need an Active Directory account. Attaching Domain Users as an entitlement ensures every provisioned account is automatically added to the Domain Users group in AD, which is the baseline group membership every AD user needs.

---

### Step 75 — HR CSV Identity Profile Action
**Screenshot: 80_HR_CSV_Identity_Profile_Action.png**

The AD_LabUsers_Access access profile is attached to the Active lifecycle state in the HR_CSV_Lab_Identity_Profile. This is the critical configuration that connects identity lifecycle to AD provisioning. When any identity has an Active lifecycle state, SailPoint will automatically ensure they have an AD account through this access profile. This is how birthright provisioning works in SailPoint. The moment a new hire appears in the HR source with an active status, SailPoint grants the access profile and triggers account creation.

---

<a name="section-9"></a>
## Section 9 — IQService Installation and TLS Configuration

IQService is the SailPoint provisioning agent that must be installed on the domain controller to enable write operations to Active Directory. SailPoint can read from AD over LDAP without IQService, but creating, modifying, or disabling accounts requires IQService to be running on or near the DC.

---

### Step 76 — IQService Settings Configuration
**Screenshot: 84_sailpoint_ad_iqservice_settings.png**

IQService Settings are configured in SailPoint ISC on the AD_Lab_DC01 source. The host is set to 192.168.50.10 which is DC01. The port is 5050 which is the default TLS port for IQService Build 842. The user is Administrator and TLS is enabled. IQService Build 842 enforces TLS on all connections and cannot be configured to run without it. In production environments the IQService certificate would be issued by the internal certificate authority that the VA already trusts, eliminating the need for manual certificate import.

---

<a name="section-10"></a>
## Section 10 — Joiner Workflow Execution and Results

With the full pipeline configured, Emily Johnson is added to the HR source and the end to end automation is tested.

---

### Step 77 — Emily Johnson Identity Details
**Screenshot: 81_sailpoint_emily_identity_details.png**

Emily Johnson's identity is visible in SailPoint ISC with employee ID JG9006. Her lifecycle state shows Active and her identity state shows ACTIVE confirming the identity was successfully created from the CSV HR source. All 8 attributes are populated correctly including department HR, title HR Specialist, email emily.johnson@lab.local, first name Emily, last name Johnson, username JG9006, employee ID JG9006, and start date 2026-05-01. The identity profile is correctly set to HR_CSV_Lab_Identity_Profile. This confirms the Joiner workflow ingested her data correctly and the identity profile mapped all attributes as configured.

---

### Step 78 — Emily Johnson Accounts Tab
**Screenshot: 82_sailpoint_emily_accounts_tab.png**

The Accounts tab on Emily's identity shows 2 results. Her IdentityNow account and her HR_Source_CSV_Lab account are both present and enabled. The absence of an AD_Lab_DC01 account reflects the current state of the lab where the provisioning pipeline reached IQService but the TLS certificate trust issue between the VA and IQService prevented the final account write to Active Directory. In a production environment with a properly trusted IQService certificate a third account would appear here showing ejohnson in the AD_Lab_DC01 source as Enabled.

---

### Step 79 — Emily Johnson Events Tab
**Screenshot: 83_sailpoint_emily_events_tab.png**

The Events tab shows the complete automation history for Emily's identity with 8 events recorded. Reading from the bottom up in chronological order: the Joiner workflow fired and attempted to Create Account in AD_Lab_DC01, Add Entitlement for Domain Users was attempted, the Change Identity State to Active passed via WebService, two Add Role events passed at the system level, and subsequent provisioning retry attempts also show Create Account Failed and Add Entitlement Failed against AD_Lab_DC01. Every event that SailPoint controls directly passed successfully. The only failures are the AD provisioning events which trace back to the IQService TLS certificate trust issue. This event log is exactly the kind of audit trail that compliance teams look for. Every action taken on an identity is recorded with a timestamp, actor, source, and result.

---

### Step 80 — Create Account Provisioning Policy
**Screenshot: 85_sailpoint_ad_create_account_policy.png**

The sAMAccountName attribute in the Create Account provisioning policy is shown set to Static with the value ejohnson. This configuration was updated from the original Create Unique LDAP Attribute generator which was causing provisioning timeouts. The generator was making repeated LDAP queries to Active Directory to check whether Emily.Johnson, Emily.Johnson1, Emily.Johnson2 and so on already existed, and those queries were timing out before receiving a response. Setting a static value bypasses the uniqueness check entirely and provides a clean username format of first initial plus last name. In production this would be handled by a well-tuned uniqueness rule or a SeaGate rule that accounts for common name collisions.

---

### Step 81 — AD Accounts Discovered
**Screenshot: 86_sailpoint_ad_accounts_discovered.png**

The AD_Lab_DC01 source accounts page shows 4 accounts discovered from Active Directory. Administrator, Guest, krbtgt, and test user are all visible. Administrator and test user show Enabled status while Guest and krbtgt show Disabled which matches the default Active Directory account states. All four accounts show as Uncorrelated because none of them match a SailPoint identity by the Work Email equals mail correlation rule. This is expected and correct behavior for default AD system accounts.

---

## Summary

This walkthrough documents the complete build of an enterprise Active Directory provisioning pipeline using SailPoint ISC. The project covered standing up a Windows Server 2022 domain controller from scratch, creating a full Active Directory domain with a structured OU hierarchy, configuring the SailPoint AD source with proper LDAP settings and account correlation, migrating the SailPoint Virtual Appliance from VirtualBox to VMware to resolve a virtual network isolation issue, installing and configuring IQService with TLS certificate binding, building the provisioning policy and access profile, and executing the end to end Joiner workflow.

The pipeline successfully demonstrated every component of enterprise AD provisioning architecture. The Joiner workflow fired automatically, the identity was created with correct attributes, SailPoint reached IQService on DC01, and provisioning was attempted against Active Directory. The current limitation is a TLS certificate trust issue between the VA and IQService that requires a CA-issued certificate in production to resolve properly.

This project represents the kind of real-world IAM engineering work that distinguishes experienced practitioners from candidates who have only worked with documentation and simulated environments.

---

*Phase 1: [SailPoint JML Pipeline](https://github.com/JustinGallimore/sailpoint-jml-pipeline)*
*Phase 3: SailPoint Governance Program — Coming Soon*
