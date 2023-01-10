# Finding &amp; Hooking Encrypted Values
In this post I will explain/guide you how to find encrypted values in games.

# Intro
Most games these days will encrypt important values such as player & enemy information.<br>
This makes it harder to find this information, since we can't just search the memory for an exact value without knowing the encryption.<br>
Since the game needs the actual values to do arithmetic and logic operations on it these values will have to be decrypted, worked with and encrypted again. In this post I will explain how to find the decrypted values, using the gold value as an example.

# Approaches
There are two common approaches, one is to reverse the encryption/decryption function and use that to decrypt the encrypted value which is stored on the heap.<br>
The other one is to hook the place where the game is working with the decrypted values and edit/copy the content there.<br>
We will be doing the second approach today, the first approach will be covered in a later post.

# Tools
We'll be using mainly Cheat Engine for memory scanning & tracing and IDA to get a better understanding of the functions we're analysing.<br>
https://en.wikipedia.org/wiki/Cheat_Engine<br>
https://en.wikipedia.org/wiki/Interactive_Disassembler

# Theory
In today's post we will be looking for the gold value and display it in a C/C++ console.<br>
Since the value is encrypted, we can't search for the exact amount of gold we are holding since that value is never stored on the heap.<br>
Therefore we will be using Cheat Engines "Changed Value" & "Unchanged Value" scanning option to find the encrypted gold address.<br><br>

The theory is to first scan for an "Unknown initial value". Then to scan for "Changed Value" when the amount of Gold you are holding has Changed and for "Unchanged Value" when the Value hasn't changed. Repeat this step till you are down to a few addresses.<br>
After that we will use Cheat Engines "Find out what writes to this address" function to find the function which writes to the memory address where the encrypted value is stored. This should lead us to the encryption function.<br>
By tracing back the stack we can find out which function called the encryption function and fetch the real gold value before it's encrypted. To achieve our goal we will be placing a hook at the place where our value is decrypted, storing it into our own allocated memory & printing it into a C/C++ console.

# Skills requiered
-Basic x86 ASM<br>
-Basic Debugging experience

# Skills acquired
-Basic understanding of memory scanning<br>
-Dynamic Analysis

# Practical - Finding Encrypted gold address
<br>
1. First open Cheat Engine and attach it to the Game.
<img src="https://user-images.githubusercontent.com/108685788/211548714-81dc412d-e006-43bc-a07d-553aa54c5ddb.gif" Width="50%" Height="50%"/>
<br>
2. Search for "Unknown initial value" with value type 4 Bytes. The value type may vary depending on which type is holding the gold, if you can't find any decent result with 4 Bytes try 2 Bytes (short) or 8 Bytes (long). Most games will use 4 Bytes though. This will search the entire memory of the process attached for a specific type.
<img src="https://user-images.githubusercontent.com/108685788/211549848-cab11583-8072-4d57-89e8-7b041a4520dc.gif" Width="50%" Height="50%"/>
3. In game change your gold by dropping or gaining gold and search for "Changed Value". Since we do not know how the encryption works we can't search for "increased" or "decreased value". This will filter out all values which haven't changed. 
<img src="https://user-images.githubusercontent.com/108685788/211550929-5e3ae24f-2563-4349-a5b0-9a7ff6f572a0.gif" Width="50%" Height="50%"/>
4. Use different game functions like attacking, moving without changing your gold. Search for "Unchanged Value". This will filter out all values which have been changed. It is a good idea to use as many in game functions as possible to trigger a change to uninteresting values.
<img src="https://user-images.githubusercontent.com/108685788/211551713-dc4b87f9-28fc-4b43-82c5-27665402d760.gif" Width="50%" Height="50%"/>
Keep repeating steps 3 & 4 until you are left with only a few addresses.
<img width="315" alt="Values_Left" src="https://user-images.githubusercontent.com/108685788/211553392-c9f34a26-7eb4-4f73-971d-e6c5e1e49022.png">
In my case I'm left with four addresses which are all 4 bytes apart. This could lead to the assumption that we found a structure which contains the encrypted gold value and some additional information used to decrypt it.

# Practical - Finding Encryption Function
The next step is to find out where the encrypted value is written to the address we found. To do this, right click the memory address and "find out what writes to this address". By doing this a window will pop up which keeps track at which location the content of our address is modified. If you now change your gold value it should display which assembly instruction, including the address, is changing the value.<br>
<img width="212" alt="writesToAddr" src="https://user-images.githubusercontent.com/108685788/211561466-5af8af2c-cad8-4f0c-bd40-1ce37b8a9b57.png"><br>
We now found the function which encrypts our value. Most encryption functions have xor, shl/shr assembly instructions in them which is a good indicator to know you're at the correct place.<br>
I decided to look at that function in IDA to get an overview of how many functions call that encryption function and to get an idea of how the encryption looks like.<br>
<img width="442" alt="callsToEncryption" src="https://user-images.githubusercontent.com/108685788/211563033-1ca423c0-353b-4291-a3fa-aa02c0839d77.png"><br>
The encryption is called at 1123 places, meaning it won't just be used to encrypt our gold value. This tells us it won't be as easy as setting a breakpoint and tracing back the stack when changing the gold value, since the encryption will most likely be called by another function before we are able to change our gold value.<br>
<img width="290" alt="encryptionFunction" src="https://user-images.githubusercontent.com/108685788/211564083-f42159f9-7d0e-4d7a-af61-cccb113de52c.png"><br>
Looking at the encryption function we see that it creates a random value, stores it at the address of the second argument. It uses the random value to perform some arithmetic on the first argument(which will most likely be our decrypted value) which is stored at the second arguments address + 4(most likely a structure). The function returns some kind of checksum, which is created with some arithmetic including the encrypted value, the random value and 0xBAADF00D. The random value will most likely be stored on the heap to decrypt the value later. Since we're hooking the gold value before it gets encrypted, we won't bother any further with looking at the encryption. I'd still suggest naming the values & functions inside of IDA once you know what they are doing.<br>
<img width="423" alt="encrpytionFucntionNamed" src="https://user-images.githubusercontent.com/108685788/211568552-a426edb6-9c2b-4aee-a91a-c8784b9e9e4b.png"><br>

# Practical - Finding Decrypted gold value
Moving back to Cheat Engine, we still have the window open which tells us what instruction writes to our encrypted gold value. If you double click the instruction a window will pop up, at the bottom it will show what the registers held at the time the instruction was executed.<br>
<img src="https://user-images.githubusercontent.com/108685788/211572675-b3785c00-1e60-492c-b9cb-3994c2dc311a.gif" Width="50%" Height="50%"/><br>
As explained above the encryption function is called at 1122 other places, meaning we can't simply set a breakpoint. What we'll have to do is set a conditional breakpoint. Since we know that the instruction which writes our address is "mov [edi], esi" we know that edi must hold our memory address we found. Looking at the registers, we see that EDI equals 559A9131, which is our memory address(It's our memory address + 1, Cheat Engine displays "The value of the pointer needed to find this address is probably 559A9131").<br>
We want to trace back which function called the encryption function for the gold, there are multiple methods for that. One way is to use Break and trace, a Cheat Engine function which is quite powerful. For the sake of simplicity we will use a normal conditional breakpoint, step through the code till we hit a return and safe that address. To set a breakpoint, go to the memory address which wrote our encrypted gold value, right click it and click "set breakpoint(Hardware Breakpoint)".<br>
<img src="https://user-images.githubusercontent.com/108685788/211582740-0436aa60-1ffb-4b5f-a32b-70084a8184f8.gif" Width="50%" Height="50%"/><br>
After setting a breakpoint we have to add a condition to it. Right click the address again, select "set/change Breakpoint condition". A window will pop up where you'll enter the condition.<br>
<img src="https://user-images.githubusercontent.com/108685788/211584065-8c66cf29-be06-438a-b11f-89044267127b.gif" Width="50%" Height="50%"/><br>
In my case the breakpoint should hit if EDI equals 0x559A9131. It is possible that the breakpoint was hit before you were able to set the condition, in that case press "Run"(f9) to continue the game. Now drop/gain some gold, the breakpoint should get hit. Step Over (f8) the instructions until you reach a return. Once you step over the return, you'll reach the function which called the encryption. Above the instructions you're currently at you'll see a call (the call to the encryption). Set a breakpoint at that call and you should see the encrypted gold value inside a register or the stack.<br>
<img src="https://user-images.githubusercontent.com/108685788/211588878-11ab4a15-df0f-4cad-bce6-90d9ea4589bb.gif" Width="50%" Height="50%"/><br>
In my case the register EAX and ECX both hold my current gold value in hex. The location your gold is stored may differ depending on the game. <br>
<img width="202" alt="gold" src="https://user-images.githubusercontent.com/108685788/211589934-423d47c5-25d8-4e68-902b-c77ac8d4648c.png">

# Detour
Since there are lots of tutorials about detouring online I will not cover that topic today. Simply write a detour at the address where the gold value is stored, mov it into your own allocated memory and print it. Inline Assembly can be used for this.<br>
<img src="https://user-images.githubusercontent.com/108685788/211629149-e41d9fb1-e1bf-4bd2-b9a5-c1e15a89d299.gif" Width="50%" Height="50%"/><br>
