## Recon
> command: `nmap -sC -sV -O -v -p- -oA ports_scan DC01 DC02 DC03 SRV02 SRV03`

### Domains and hosts found
```markdown-tree
domain sevenkingdoms.local
	DC01 KINGSLANDING
	domain north.sevenkingdoms.local
		DC02 WINTERFELL
		SRV02 CASTELBLACK
		
domain essos.local
	DC03 MEEREEN
	SRV03 BRAAVOS
```

### IP addresses

| Host  | Hostname     | 192.168.86 |
| ----- | ------------ | ---------- |
| DC01  | kingslanding | .10        |
| DC02  | winterfell   | .11        |
| DC03  | meereen      | .12        |
| SRV02 | castelblack  | .22        |
| SRV03 | braavos      | .23        |

### Ports scan results

| Servers          | Ports / Services open                                                                                                                                          |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DC01, DC02, DC03 | TCP: dns 53,HTTP 80, kerberos (88,464,), SMB (139,445),LDAP(S) (389,636,3268,3269),RDP (3389), WinRM(5985,5986,47001), ADWS (9389), rpc (135, 593,49664-49704) |
| SRV02            | TCP: HTTP (80), SMB (139,445), MSSQL (1433), RDP (3389),WinRM(5985,5986,47001),rpc (135, 593,49664-49704)                                                      |
| SRV03            | TCP: HTTP (80), SMB (139,445), MSSQL (1433), RDP (3389),WinRM(5985,5986,47001),rpc (135, 593,49664-49704)                                                      |

### SRV02
1. SMB
```sh
smbclient -N -L //SRV02
```
![[Pasted image 20261109184538.png]]
Found a file `arya.txt` in share `all` with the following content:
```
Subject: Quick Departure

Hey Arya,

I hope this message finds you well. Something urgent has come up, and I have to leave for a while. Don't worry; I'll be back soon.

I left a little surprise for you in your room – the sword You've named "Needle." It felt fitting, given your skills. Take care of it, and it'll take care of you.

I'll explain everything when I return. Until then, stay sharp, sis.

Best,
John   
```
I tried `arya:Needle` with NetExec to check the creds:
![[Pasted image 20261110092628.png]]


2. HTTP
Found an upload page.
![[Pasted image 20261110091407.png]]
I created an white background JPEG file to test and after uploading it, the file is located in `http://castelblack/upload/empty.jpg`
> Can I upload a webshell/reverse shell ?

## Exploitation
### Initial Foothold 
I used this [ASPX reverse shell](https://gist.githubusercontent.com/qtc-de/19dfc9018685fce1ba2092c8e2382a79/raw/6d4df39b991b6fe54c606eee45483b17cdd09c4c/aspx-reverse-shell.aspx) to get an inital foothold.
![[Pasted image 20261110095253.png]]
### Internal enumeration
I uploaded winPEAS via SMB share `all` found in our previous SMB enumeration. The next parts are winPEAS outputs that are different vectors I could/will exploit later.
#### Known vulnerabilities
```
╔══════════╣ Windows Version Vulnerabilities (T1082,T1068)
╚ Product candidates: Windows Server 2019 | Windows Server 2019 (Server Core installation)
╚ Definitions date: 20260430
╚ Installed hotfixes detected: 9
╚ Pre-filter matches: 400, filtered by installed/superseded KBs: 320
Matched 80 known exploited vulnerabilities for this running Windows version.
Matched products: Windows Server 2019 | Windows Server 2019 (Server Core installation)
    CVE-2018-8544 KB4467708 [Critical] Remote Code Execution
    CVE-2020-1472 KB4601345 [Critical] Elevation of Privilege
    CVE-2021-1675 KB5003646 [Critical] Remote Code Execution
    CVE-2021-22947 KB5009557 [Critical] Remote Code Execution
    CVE-2021-24091 KB4601345 [Critical] Remote Code Execution
    CVE-2021-34527 KB5004947 [Critical] Remote Code Execution
    CVE-2025-47981 KB5062557 [Critical] Remote Code Execution
    CVE-2025-59287 KB5070883 [Critical] Remote Code Execution
    CVE-2018-0886 KB4551853 [Important] Remote Code Execution
    CVE-2018-8411 KB4464330 [Important] Elevation of Privilege
    CVE-2018-8423 KB4464330 [Important] Remote Code Execution
    CVE-2018-8453 KB4464330 [Important] Elevation of Privilege
    CVE-2018-8550 KB4467708 [Important] Elevation of Privilege
    CVE-2018-8584 KB4467708 [Important] Elevation of Privilege
    CVE-2019-0543 KB4480116 [Important] Elevation of Privilege
    CVE-2019-0552 KB4480116 [Important] Elevation of Privilege
    CVE-2019-0555 KB4487044 [Important] Elevation of Privilege
    CVE-2019-0570 KB4480116 [Important] Elevation of Privilege
    CVE-2019-0571 KB4480116 [Important] Elevation of Privilege
    CVE-2019-0572 KB4480116 [Important] Elevation of Privilege
╚ Showing 20/80 results.
╚ This check applies version matching with installed/superseded KB filtering.
```
