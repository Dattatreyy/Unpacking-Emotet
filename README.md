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



