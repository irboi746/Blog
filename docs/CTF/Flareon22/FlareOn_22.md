# Flareon-22
## Flaredle
### Challenge
![[Pasted image 20221004110658.png]]
### Process
#### Understanding Game / Application Logic
- Understand what Yellow, Green and Grey means in wordle
   - Green: The letter is right and in the correct position in the word.
   - Yellow: The letter is correct but is in the wrong position.
   - Grey: The letter is wrong and isn’t in the word at all.
- We notice that in the source code file of the game there is a "words.js" which contains a list of valid words. These words are all small letters and a-z with no numbers or characters.

![[Pasted image 20221112200006.png]]
![[Pasted image 20221112200332.png]]

#### Python file to count occurence of all Characters 
We then wrote a python file to count the occurences of characters from A-Z in the wordlist.
```python
import string

def loopWordCount():
    testtest = ''
    for char in string.ascii_lowercase:
        testtest += str(word.count(char)) + ','
    return testtest

def create_counted_wordlist():
    r_word_file = open(r'C:\Users\flare\Desktop\Script\wordlist.txt','r')
    w_word_file = open (r'C:\Users\flare\Desktop\Script\counted.txt','w')
    wordlist = r_word_file.readlines()
    for word in wordlist :
        #   print(word +
        #    loopWordCount() + '\n')    
        w_word_file.writelines(loopWordCount())
        w_word_file.writelines(word)

```

Comma-separated-value from the occurences of A-Zs.
![[Pasted image 20221112202038.png]]

#### Parse CSV into Excel / LibreOffice
- We will now parse the csv file into excel so that we can filter with each try to finally get the correct word.

![[Pasted image 20221112203826.png]]

- Using this word "AEROBACTERIOLOGICALLY" we know that there are 1 `e`, 1 `i`, 2 `o`, 1 `c`.
- By Filtering down, we are left with these 2 words, we can filter i.e '0 `g`' and we will get `flareonisallaboutcats` 

![[Pasted image 20221112203733.png]]

### Flag
There we have it the flag.
![[Pasted image 20221112204049.png]]
```
flareonisallaboutcats@flare-on.com
```

## Pixel Poker
### Challenge
![[Pasted image 20221004110751.png]]
### Process
#### PE Profiling with DiE
- To check if application is Packed and the high level language used.

![[Pasted image 20221112204635.png]]

- It seems like C++ is used and executable is likely not packed.

#### Playing Around the Application
- We find the fail condition has some strings like "Womp womp... :(", "Please play again!"

![[Pasted image 20221112204701.png]]

#### Strings
- Using strings (`shift` + `f12`), we find the strings that make up the fail condition
![[Pasted image 20221112210706.png]]

- Then we use the x-ref function to see location it has been referenced
![[Pasted image 20221112210803.png]]

- Then we sift out only for the function that calls it.
![[Pasted image 20221004165756.png]]

#### Tricking the Program to think that we found the pixel
##### Unlimited Tries
- Then we trace back to before it is called. We see that the function does a `cmp 0Ah` where `0A` is `10`. This seems like the function to check if 10 tries has exceeded. 
![[Pasted image 20221004170025.png]] 
- To get unlimited tries we can use a very convenient function in Cutter the "Edit Instruction" . 

![[Pasted image 20221112213752.png]]

- We change the instuction from `jnz`/`jne` to `jmp`, after 10 tries, it will jump to the exit function, but go the next function.

![[Pasted image 20221112213858.png]]

##### Function that checks for correct pixel click
- Right after the 10 tries check, there are 2 `jnz` instructions. As we are searching for pixels, I assume they are the X-axis and Y-axis.

![[Pasted image 20221112213210.png]]

- Again, we are back at Cutter, we use the "reverse jump" function to switch both of the `jnz`/`jne` instructions to `jz`/`je` instructions.

![[Pasted image 20221112214416.png]]

### Flag
- We open the application, and after one click, and a lag, we get the picture below.
![[Pasted image 20221004165117.png]]

