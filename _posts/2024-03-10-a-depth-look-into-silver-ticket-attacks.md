---
layout: post
tags:
  - activedirectory
---
## what is a silver ticket

it refers to a highly sophisticated and rare form of attack vector which is capable of granting complete, unrestricted access to a network or system without any user interaction. it often leverages undocumented features or bugs within
Active Directory that are not typically exploited due to their rarity and complexity.

a silver ticket does not involve social engineering or brute-forcing credentials, as the attacker is expected to have a high level of domain
privileges already. instead, it allows an attacker to forge Kerberos tickets which can be used to authenticate as any user on the network.

![Silver](https://raw.githubusercontent.com/notluken/notluken.github.io/master/_screenshots/silverticket.png)

## typical kerberos authentication flow

user logs on with username & password.

1. 
	a. password is converted to NTLM hash, a timestamp is encrypted with the hash and sent to the KDC as an authenticator in the authentication ticket (TGT) request (AS-REQ)
	b. the DC (KDC) checks user information (groups, restrictions, etc.) & creates the Ticket-Granting Ticket (TGT).

2. the TGT is encrypted, signed & delivered to the user (AS-REQ). only the Kerberos service (KRBTGT) in the domain can open and read TGT data.
3. the user presents the TGT to the DC when requesting a Ticket-Granting Service (TGS) ticket (TGS-REQ). the DC opens the TGT & validates PAC checksum. if the DC can open the ticket  & the checksum check out, TGT = valid. the data in the TGT is effectively copied to create the TGS ticket.
4. the TGS is encrypted using the target service account's NTLM password hash and sent to the user (TGS-REP)
5. the user connects to the server hosting service on the appropriate port & presents the TGS (AP-REQ). the service opens the TGS ticket using its NTLM password hash.

# silver ticket overview

![](https://raw.githubusercontent.com/notluken/notluken.github.io/master/_screenshots/silver.png)

there is no AS-REQ / AS-REQ and no TGS-REQ / TGS-REP.

# creating Silver Tickets
to fabricate a Silver Ticket, the assailant must first acquire the password data (password hash) pertinent to the target service. If the service operates within the confines of a user account, such as MS SQL, obtaining the Service Account's password hash becomes imperative for Silver Ticket generation.

One plausible approach to unearth the password data associated with the target service is through Kerberoast, which involves cracking Service Account passwords.

Computers also host services, with one of the most prevalent ones being the Windows file share, employing the "cifs" service. Since the computer acts as the host for this service, the password data necessary for crafting a Silver Ticket pertains to the computer account's password hash. Upon joining a computer to Active Directory, a corresponding computer account object is instantiated and linked to the computer. The password and its associated hash are stored on the computer holding the account, while the NTLM password hash is stored within the Active Directory database on the Domain Controllers for the domain.

In the event that an attacker attains administrative privileges over the computer (facilitating debug access) or can execute code as the local System, exploiting Mimikatz enables the extraction of the AD computer account password hash from the system. (It's worth noting that the NTLM password hash is employed in encrypting RC4 Kerberos tickets.)

