# Finding &amp; Hooking Encrypted Values
In this post I will explain/guide you how to find and hook encrypted values in games.

# Intro
Most games these days will encrypt important values such as player & enemy information to make it harder to find the addresses and edit/work with it's content.<br>
This makes it harder for us to find the encrypted value, since we can't just search the memory for an exact value without knowing the encryption.<br>
Since the game needs the actual values to do arithmetic and logic operations on it these values will have to be decrypted, worked with and encrypted again.

# Approaches
There are two common approaches, one is to reverse the encryption/decryption function and apply it onto the encrypted value.<br>
The other one is to hook the place where the game is working with the decrypted values and edit/copy the content there.<br>
We will be doing the second approach today, the first approach will be covered in a later post.

# Tools
We'll be using mainly Cheat Engine, for memory scanning & tracing, and IDA to get a better understanding of the function.<br>
https://en.wikipedia.org/wiki/Cheat_Engine<br>
https://en.wikipedia.org/wiki/Interactive_Disassembler

# Theorie
In todays post we will be looking for the gold value and display it in a C/C++ consol.<br>
Since the value is encrypted, we can't search for the exact amount of gold we are holding since that value is never stored on the heap.<br>
Therefor we will be using Cheat Engines "Changed Value" & "Unchanged Value" scanning option to find the encrypted gold address.<br><br>

The theorie is to first scan for an "Unknown initial value". Then to scan for "Changed Value" when the amout of Gold you are holding has Changed and for "Unchanged Value" when the Value hasn't changed. Repeat this step till you are down to a few addresses.<br>
After that we will use Cheat Engines "Find out what writes to this address" function to find the function which writes to the memory address where the encrypted value is stored. This should lead us to the encryption function.<br>
By placing a breakpoint at the decryption function, we can than step through the code until we find a Register holding our decrypted gold value. To achive our goal we will be placing a hook at the place where our value is decrypted, storing it into our own allocated memory & printing it into a C/C++ console.

# Skills acquired
-Basic understanding of memory scanning<br>
-Dynamic Analysis

# Pratical - Finding Encrypted gold value
1. first open Cheat Engine and attach it to the Game.
<img src="https://user-images.githubusercontent.com/108685788/211548714-81dc412d-e006-43bc-a07d-553aa54c5ddb.gif" Width="50%" Height="50%"/>
<br>
2. Search for "Unknown initial value".
<img src="https://user-images.githubusercontent.com/108685788/211549848-cab11583-8072-4d57-89e8-7b041a4520dc.gif" Width="50%" Height="50%"/>
3. Ingame change your gold by dropping or gaining gold and search for "Changed Value".
<img src="https://user-images.githubusercontent.com/108685788/211550929-5e3ae24f-2563-4349-a5b0-9a7ff6f572a0.gif" Width="50%" Height="50%"/>
4. Trigger different game functions like attacking, moving without changing your gold. Search for "Unchanged Value".
<img src="https://user-images.githubusercontent.com/108685788/211551713-dc4b87f9-28fc-4b43-82c5-27665402d760.gif" Width="50%" Height="50%"/>
<br>
Keep repeating steps 3 & 4 until you are left with only a few addresses.
