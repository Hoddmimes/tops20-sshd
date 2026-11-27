# tops20-sshd
SSH daemon for TOPS-20

This projects contains the code	for building and SSH service for the TOPS-20 operating system.

### How	to build (on TOPS-20)

The script BUILD.CMD will compile and build the SSHD and the SSHRUN executables
```
@TAKE BUILD.CMD
```


- All code i.e.	*.MAC should be	located	in one directory
-  The following commands will compile and build the SSHD.EXE 

```
@COMPILE SSHDEF
@LOAD /COMPILE SSHD,SSHDAT,SSHKEX,SSHPKT,SSHCON,SSHLOG,SSHCFG,SSHKEY,SSHUTL,SSHSCP,CHACHA,POLY,FE255,X25519,SC255,ED255,EDCON,RSA,SHA2,SHAK
@SAVE SSHD
```

- The SSHD run in priviliaged mode, in order to safely launch the SSHD without allowing created SSH forks to gain priviliges, the SSH is launched via a main program SSHRUN.EXE
- The SSHRUN.EXE ios compiled and built with the following commands

```
@LOAD SSHRUN
@SAVE SSHRUN
```







### How configure and run the the SSHD

- Create a directory e.g PS:\<SSHD>
- Must have wheel capabilities
- Copy the executables SSHD.EXE and SSHRUN.EXE to the created directory
- Copy the file SSHD.CFG to the created directory, edit the file after your desire

```
; SSHD.CFG - TOPS-20 SSH server configuration (PS:<SSHD>SSHD.CFG)
;
; TCP port to listen on (ports below 256 need WHEEL, which SSHD has)
PORT 22
;
; File in each user's login directory with their SSH public keys
AUTHORIZED-KEYS SSH-AUTH-KEYS.TXT
;
; Password logins: YES (default) or NO = only authorized SSH keys
;PASSWORD-AUTHENTICATION NO
;
; Users who may not log in over SSH (one line per user, up to 50)
;DISABLE-USER OPERATOR
;
; Concurrent sessions (1-16, default 8)
;MAX-SESSIONS 8
;
; Seconds a client may take to log in (0 = no limit, default 120)
;LOGIN-TIMEOUT 120
;
; Controlling the running server: create one of these files (the server
; deletes it within a second and acts), e.g.  COPY NUL: PS:<SSHD>SSHD.STATUS
;   PS:<SSHD>SSHD.STATUS   state of the server and its sessions -> SSHD.LOG
;   PS:<SSHD>SSHD.STOP     stop accepting connections, exit when sessions end
;   PS:<SSHD>SSHD.RESTART  start a new server (newest SSHD.EXE) at once; the
;                          old one exits when its sessions have ended
```


Most likely you would to to start the SSHD as part of the system startup.
This is best done by adding the execution of the SSHD to the SYSJOB. Edit 
the SYSTEM:SYSJB1.RUN and add the following lnes to end of file. Where <n> is
the job number that the SSHD will have in the SYSJOB RUN file.

```
JOB <n> "LOGIN OPERATOR
ENABLE
RUN PS:<SSHD>SSHRUN
"
```


### File transfer with scp
The SSHD supports the classic scp protocol (enabled with FTSCP==1 in SSHDEF.MAC,
which is the default). SFTP is not supported, so OpenSSH 9.0 and later must use
the -O option:

```
scp -O myfile.txt user@tops20host:
scp -O user@tops20host:LOGIN.CMD .
scp -O 'user@tops20host:*.MAC' .
```

- Plain names and "." are relative to the user's PS:<user> directory; names are upper-cased
- Text (7-bit) files are converted CRLF <-> LF; 8-bit files are copied as is
- 36-bit files (.EXE, .REL ...) and directories (scp -r) are refused
- Wildcards work for downloads
- File access is checked as the logged-in user

### User Authorized SSH Keys
The setting _AUTHORIZED-KEYS_ in the SSHD.CFG will point out the file where the SSHD server will try to locate the SSH keys being authorized for the user. The file must be located the the user top directory.

```
; SSH-AUTH-KEYS.TXT - SSH public keys allowed to log in as GAZONK
; One key per line, as in OpenSSH's authorized_keys:
;   ssh-ed25519 <base64 key> <comment>
;   ssh-rsa <base64 key> <comment>      (RSA 1024-4096 bits)
;   Other key formats currently not suported.
;
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1DC9mXzYJG51234567890TVgVpd7ggggggggggg4871yhOJQZUXyj3GIywsRoEGSPM4nsF7sXfk4PsxxxxxxxxxxK30Rp2GAV+n
9DNPmcQ2j4SPtVwrvUKMXiU4jpHrr7cd4EeOEyyyyyyyyyyV4Hozy+l4XHfHU8y2TgdWFghRhg32dmj21gDE8o72P1rTGRg2T0Rwtb3nFTCmj+ahAk2SpoEvg4tjuQphyvEIuNdYze
WriiUM4pLs0Q8ZF8Mi2QuOAh+1akXATx/o6RFSUAYTubksFy8UEeuhECj0GwHmjCetJPqfEdkwea1Gm0nSziX/CkBhYuMUwrhv gazonk@karma
```
