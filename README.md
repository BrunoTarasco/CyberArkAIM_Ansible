# CyberArkAIM_Ansible
This project was conceived based on the constant need of some companies to use CyberArk AIM to provide credentials for Ansible tasks.
Please keep in mind that there's no official support for this.

## Getting Started

### Prerequisites
You'll need:

- CyberArk CorePAS (already installed and configured, obviously)
- CyberArk AIM License
- RHEL/CentOS (5, 6 or 7) Servers (1 for Credential Provider + x to be used as targets)
- CyberArk AIM Provider RPM folder (usually containing something like CARKaim-<version>.rpm)
- A fully functional brain :D
- A configured Safe to use with AIM
- A configured Application on PVWA
- A CyberArk Administrative user


## Initial Setup

### Step 1 - Install Ansible

Configure CentOS EPEL
```
$ sudo yum install epel-release
```

Install Ansible
```
$ sudo yum install ansible
```

### Step 2 - Configure Ansible

Edit /etc/ansible/hosts file
```
$ sudo vim /etc/ansible/hosts
```

Add the hosts you want to manage using Ansible by adding this to your file
```
[hosts_group_name]
server1_ip
server2_ip
```

### Step 4 - Prepare CyberArk AIM
- Use WinSCP to copy the AIM folder file to your Ansible server

- Install redhat-lsb
```
$ sudo yum install redhat-lsb
```
- Create the credential file (used to authenticate AIM to the Vault)
```
$ sudo chmod +x ./CreateCredFile
$ ./CreateCredFile <filename.cred>
```
It will ask you for the admin user and password. You can ignore all other options, just spam the enter key :D

- Copy the configuration files to /var/tmp
```
$ sudo cp <filename.cred> aimparms.sample Vault.ini /var/tmp
```

- Copy aimparms so you can edit it
```
$ sudo cd /var/tmp
$ sudo cp aimparms.sample airmparms
```

- Edit aimparms
```
$ sudo vim aimparms
```
**it should look like this:**
```
[Main]
AcceptCyberArkEULA=yes
#CreateVaultEnvironment=yes
LicensedProducts=AIM
CredFilePath=/var/tmp/<filename.cred>
VaultFilePath=/var/tmp/Vault.ini
OverrideExistingConfFile=yes
AppProviderUserLocation=Applications
#HardenFSPermissions=no

### In Repair/Upgrade the following parameters will be read from the existing "/etc/opt/CARKaim/conf/basic_appprovider.conf"
### file unless they are uncommented and contain different values
AppProviderConfSafe=<AppProvider Name>
MainAppProviderConfFile=main_appprovider.conf.linux.10.05
MainOPMConfFile=main_opm.conf.linux.10.05
AppProviderUser=<TestProvider>
#OPMUser=OPM_<host_name>
#PIMConfigurationSafe=PVWAConfig
#PIMConfigurationFolder=Root
#PIMPVConfigurationFileName=PVConfiguration.xml
#PIMPoliciesConfigurationFileName=Policies.xml
###

[AIM]
CacheLevel=persistent
CacheRefreshInterval=3

[PIMSu]
PIMSuCacheLevel=persistent
PIMSuCacheRefreshInterval=3
```

- Edit Vault.ini
```
$ sudo vim Vault.ini
```
**sample:**
```
VAULT = "Demo Vault"
ADDRESS=<VaultServer_ip>
PORT=1858

#-----------------------------------
# Additional parameters (optional)
#-----------------------------------

TIMEOUT=8                       - Seconds to wait for a Vault to respond to a request

#VAULTDN=                                  - Vault's Distinguished Name (PKI Authentication)

#Proxy server connection settings  - cannot be used together with BEHINDFIREWALL
#--------------------------------
#PROXYTYPE=HTTP                    - Possible values - HTTP, HTTPS, SOCKS4, SOCKS5
#PROXYADDRESS=<proxy_address>         - Proxy server IP address (mandatory when using proxy server)
#PROXYPORT=8081                    - Proxy server IP Port
#PROXYUSER=xxx                     - User for Proxy server if NTLM authentication is required
#PROXYPASSWORD=yyy                 - Password for Proxy server if NTLM authentication is required
#PROXYAUTHDOMAIN=NT_DOMAIN_NAME    - Domain for Proxy server if NTLM authentication is required

#BEHINDFIREWALL=NO                 - Accessing the Cyber-Ark vault via a Firewall.

#USEONLYHTTP1=NO                   - Use only HTTP 1.0 protocol. Valid either with proxy settings Or with BEHINDFIREWALL

#NUMOFRECORDSPERSEND=15            - Number of file records that require an acknowledgement from the Vault server
#NUMOFRECORDSPERCHUNK=15           - Number of file records to transfer together in a single TCP /IP send/receive operation
#ENHANCEDSSL=NO                    - Enhanced SSL based connection (port 443) is required

#PREAUTHSECUREDSESSION=NO              - Enable pre authentication secured session
#TRUSTSSC=NO                               - Trust self-sign certificates in pre authentication secured session
#ALLOWSSCFOR3PARTYAUTH=NO              - Are self-sign certificates allowed for 3rd party authentication (like RADIUS)

#PROXYCREDENTIALS=                         - Instead of specifying proxy user and clear text proxy password, they can be given in the file pointed by this parameter
```

## Install CyberArk AIM Provider
Go to the rpm folder location and install the rpm file:
```
$ sudo rpm -ivh CArkAIM-<version>.rpm
```

### Just one more thing... 
Allow the AppProviderUser defined on **aimparms** on your Safe

## How to test
Use the **ping.yml** playbook with -vvv
```
$ sudo ansible-playbook -vvv ping.yml
```

**You rock, kid ;)**

