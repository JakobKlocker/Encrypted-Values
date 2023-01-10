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
In todays post we will be looking for the gold value and display it in a C consol.<br>
Since the value is encrypted, we can't search for the exact amount of gold we are holding since that value is never stored on the heap.<br>
Therefor we will be using Cheat Engines "Changed Value" & "Unchanged Value" scanning option to find the encrypted gold address.<br><br>

The theorie is to first scan for an "Unknown initial value", then to scan for "Changed Value" when the amout of Gold you are holding has Changed and for "Unchanged Value"
