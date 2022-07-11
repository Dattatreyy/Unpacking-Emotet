# Unpacking Malware with x32dbg

But how do we know if the sample is really packed or not ??

Here are some parameters, check if 2 or 3 from below conditions is fulfilled or not.

1. Check Entropy of the sample (usually above 7 for packed, but don't rely on this parameter)
2. Sample will have few imported DLLs and Functions
3. Non Standard section names
4. Monitor Process after execution, usually Packed malware create its own child process with same name. 
Basically Malware is Unpacking itself in memory. Btw this is called Process Injection
5. There will be many obfuscated Strings.
6. Non Common executable binary sections (only .text / .code section should be executables)
7. Good difference between Raw size and Virtual Size of sections
8. Entrypoint pointing to other section than .text / .code section.

So Lets Unpack…..

MD5: f3f48c57c38bff2ddd220f20569e1ee6

![image](https://user-images.githubusercontent.com/107531426/178321702-98f675c5-dff5-4351-8ff7-3e50fc0dfb3b.png)

If you execute this Sample, you can see in Process Monitor that sample is creating another child process with same name. This 
means Malware is performing Process Injection.

This sample is Emotet. 

Load the sample into x32dbg and Run the Sample (F9)

<img width="953" alt="emo1" src="https://user-images.githubusercontent.com/107531426/178323047-15b6e09e-ede3-43e9-a833-0b350ff04bba.PNG">

We’re now at Entrypoint of our Sample. Hit the BreakPoint (F2), from here we’ll start our analysis.

<img width="960" alt="emo2" src="https://user-images.githubusercontent.com/107531426/178323293-6eddf88f-5f48-40c9-b1fb-41e8706d2d98.PNG">

We know that Malware is Unpacking itself in memory so we’ve to focus on those WindowsAPIs which create new 
processes and allocate memory within that process. (VirtualAlloc, VirtualAllocEx, VirtualProtect)

Check this from PEStudio from Import Library for Memory APIs. We are looking here for VirtualAlloc by searching via 
Ctrl+G.

<img width="959" alt="emo3" src="https://user-images.githubusercontent.com/107531426/178323633-58a0b038-b5ad-4f9d-a793-2baa39dbbad8.PNG">

We are now at the place where VirtualAlloc starts, We need to set breakpoint at the end of the function. So hit 
the next JMP.

<img width="960" alt="emo4" src="https://user-images.githubusercontent.com/107531426/178324901-67564b38-ee68-4237-bacd-9f26786f83b8.PNG">

<img width="960" alt="emo5" src="https://user-images.githubusercontent.com/107531426/178324389-4e76ffd7-735f-482a-8d4d-304a244d0193.PNG">

The instruction “ret10” is the end the VirtualAlloc function. So set the breakpoint over there. (F2)

<img width="684" alt="emo6" src="https://user-images.githubusercontent.com/107531426/178325023-f1c39b9f-0877-41e5-863a-d9eb372d76c8.PNG">

So our breakpoint is at Entrypoint of the sample and End of VirtualAlloc Function

<img width="794" alt="emo7" src="https://user-images.githubusercontent.com/107531426/178327510-c4a13c02-9656-486d-b93e-be9bd44e7bbe.PNG">

Now go to the entrypoint of sample (Shift+8) and Run the Malware (F9). You’ll be at return value of 
Memory where malware is Unpacking itself. 

<img width="942" alt="emo8" src="https://user-images.githubusercontent.com/107531426/178327657-46f34fb7-6557-4236-bcb4-4f8eb13ab15c.PNG">

Now do step over (F7). It will take you to the value where allocated memory is being used to move 
the unpacked malware into. (esp+28)

Above this instruction (CALL EDI) and right side (EDI). You’ll see memory is being stored in EDI 
register.

<img width="956" alt="emo9" src="https://user-images.githubusercontent.com/107531426/178328066-2bed4926-a8a2-4293-9762-119d5a34ae32.PNG">

Right click on the Value (esp+28) and do follow in dump and choose select address.

We need to find Header of the Sample in Dump. The Header of Windows Executables starts with MZ 
(4D 5A). Here we can see MZ is not present in the dump.

<img width="956" alt="emo10" src="https://user-images.githubusercontent.com/107531426/178328272-10a77520-f546-457d-ab3b-2fc47fd6765b.PNG">

