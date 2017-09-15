## Analysis of CME bomb lab program in linux using dbg, objdump, and strings  

### Basic Static Analysis  
**First thing I did was to search the binary using strings to see if there was anything interesting that pops out.  I found various strings of interest.  Specifically:**  

That's number 2.  Keep going!  
Halfway there!  
Good work!  On to the next...  
Welcome to my fiendish little bomb. You have 6 phases with  
which to blow yourself up. Have a nice day!  
Phase 1 defused. How about the next one?  
So you got that one.  Try this one.  
When I get angry, Mr. Bigglesworth gets upset.  
Wow! You've defused the secret stage!  
So you think you can stop the bomb with ctrl-c, do you?  
Curses, you've found the secret phase!  
But finding it and solving it are quite different...  
Congratulations! You've defused the bomb!  
Well...  
OK. :-)  
The bomb has blown up.  
DrEvil  
BOOM!!!  

**After looking at these interesting strings, I'm going to make a few guesses at what is going on in this binary "BOMB!!"**  

At the onset of the program you get the string 'Welcome to my fiendish little bomb. You have 6 phases with which to blow yourself up. Have a nice day!' I don't want to run the program/"pull the pin" on the bomb by running it, so this tells me that there are likely 6 stages to the bomb.  

When you fail a phase, and the bomb goes off, you probably get the string 'BOOM!!!' and/or the string 'The bomb has blown up.'  Could this mean alternative endings? Alternative paths?  Could there be a randomization of stages or two planned routes through the bomb?   

After solving stage 1 you likely get the string 'Phase 1 defused. How about the next one?'  

After solving stage 2, you likely get the string 'That's number 2.  Keep going!'  

After solving stage 3 you likely get the string 'Halfway there!'  

It is not clear what may be the output string for solving stage 4 or 5.  They will likely be either 'Good work!  On to the next...' or 'So you got that one.  Try this one.'  

A string that could be the final string outputted when you solve stage 6 is 'Congratulations! You've defused the bomb!'.  

It appears that there may be a secret stage.  Upon entry to that secret stage you likely get the string 'Curses, you've found the secret phase!' and upon beating the stage you get the string 'Wow! You've defused the secret stage!'.  Maybe you get an alternative string for the bomb blowing up if done so via the secret stage?  

So there are some potential strings for solving each of the stages. Specifically:  
When I get angry, Mr. Bigglesworth gets upset.  
So you think you can stop the bomb with ctrl-c, do you?'  
'But finding it and solving it are quite different...'  
Well...  
OK. :-)  
DrEvil  

There are six of them... but some of these could be just added strings outputted upon completion of a stage.  Some of the pass phrases could be integers, or a random set of characters...  if that is the case then the only way to figure things out is through dynamic analysis and disassembling the code.  

**The previous output from the strings program was outputted to stout in order that the strings are found in the binary. I also wanted to see groupings of strings that may have similar prefixes and so I sorted the strings program output and looked for anything interesting in that manner.  I found:**  

initialize_bomb  
initialize_bomb  
initialize_bomb_solve  
node1  
node2  
node3  
node4  
node5  
node6  
phase_1  
phase_1  
Phase 1 defused. How about the next one?  
phase_2  
phase_2  
phase_3  
phase_3  
phase_4  
phase_4  
phase_5  
phase_5  
phase_6  
phase_6  
phase_defused  
phase_defused  
explode_bomb  


**These look like they could pertain to the various phases of the bomb.  Maybe function names or labels?**  

**I also found strings that look like they could be related to attribution:**  
angelshark.ics.cs.cmu.edu  
changeme.edu  
GET /%s/submitr.pl/?userid=%s&lab=%s&result=%s&submit=submit HTTP/1.0  
greatwhite.ics.cs.cmu.edu  
makoshark.ics.cs.cmu.edu  

**Dunno, lets just get a static printout of the disassembled code and see what comes out.  First, interesting sections/function names:**  
start  
main  
phase_1  
phase_2  
phase_3  
phase_4  
func4          ???  
phase_5  
phase_6  
fun7           ???  
secret_phase    !!!  
sig_handler  
invalid_phase  
string_length  
strings_not_equal  
initialize_bomb  
initialize_bomb_solve  
blank_line  
skip  
explode_bomb  
read_six_numbers  
read_line  
phase_defused  

**After looking at the static Main() code, I've got a reasonable understanding of the gross control flow through this program now lets do a more dynamic analysis with GDB.**  

**Stepping through the code with the GDB debugger I can say plenty about the various functions called in this program:**  
**string_length() -** This function first checks to see that the passed character pointer in %rdi is not null terminated.  If so, put zero in %eax and return.  If not null terminated then preserve the originally passed pointer argument by copying it to %rdx. Increment %rdx by 1 to point to the next character byte and move to %eax.  Subtract original pointer from %eax and get the running total of the string. Check to see if the incremented character pointer is not null terminated. If so, pass the counter back to the calling function else continue the incrementing loop through string pointer until it hits null termination.  

**strings_not_equal() -** This function implements the test of equality between the user inputed string and the pass-phrase for phase_1 of the bomb challenge.  It is passed the inputed user phrase and the pass-phrase and then checks that the two strings are the same length.  If not then the detonation flag that was initialized to 1 is not set to low and will eventually trigger the detonate function.  If the two string are of the same length, then it looks to see that the first inputed character is a non-zero (anything but a zero).  If the first character in the input string is anything but a zero then the detonation flag is set to low and passed out the function.  There are a ton of dead ends that you can follow in this code that all land on detonation.  Plenty of work for people who enjoy rabbit holes...  Ultimately to pass this test all you need to do is input any string of 46 characters in length that does not start with a zero.  

**phase_defused()** - So this function implements stack protection by adding, checking, and removing a canary.  You just pass through the function and it does nothing.  There is an accessed memory area that serves as a counter.  I believe this function also acts as the gateway to the secret phase.  

**read_six_numbers() -** Checks that the user inputed at least 6 numbers and if less than 6 numbers then detonate the bomb.  There was a bunch of manipulation of stack space but there was nothing in the stack at that location and so it is likely a bunch of leg work.  I'm getting a feeling that the author wants you to really have to work to get through some of these functions.  Regardless, I'm not falling for it this time.  Simple function made to look like a mess.  

**func4() -** This function was rather difficult for me to get through logically and so I ultimately had to take it as somewhat as a black box.  It is called recursively and in the end you need it to spit out the number 11.  

**phase_1() -**  I'm first going to start stepping through the program starting at main.  I found the memory position for the beginning of phase_1 and placed a break point there.  I then continue to run the program until I am prompted for a phrase to input.  I inputed the word 'blah' and continued to run the program.  GDB then stopped at the break before entering into the phase_1 function call.  I start stepping by single instructions until I get to the point where I am about to hit the function strings_not_equal.  I'm guessing that this function will likely compare the string that I inputed to some string stored in memory somewhere.  I know that due to x86-64 calling conventions on programs compiled with GCC that %rdi and %rsi may contain pointers to the words to compare.  I dereference the string pointed to by %rdi using x/s $rdi and see that the string pointed to is 'blah'.  I then did the same for the possible second pointer arguement which would be in %rsi with x/s $rsi and get 'When I get angry, Mr. Bigglesworth gets upset.'.  Thus I'm pretty confident that this will be the pass phrase for the first phase.  I then restart the program and see if that got me through phase 1.  Score!!! I see the output 'Phase 1 defused. How about the next one?'  

**phase_2() -** This phase is about typing in a code.  The code must be at least six numbers long or else the bomb detonates.  This count is checked by the function read six numbers which also takes the user input string and formats them into integers that are then dumped onto the stack.  Next there is pattern that must be applied to the first 6 numbers.  First, the numbers must be positive.  Second, each progressive number in the code series entered by the user must be 1 larger than the next.  Thus, the second number in the series must be 1 greater than the first number, the third number in the series must be 2 larger than the second number, etc.  So, possible codes would be 1, 2, 4, 7, 11, 16 or 21, 22, 24, 27, 11, 16.  Any numbers entered after the first 6 can be anything.  

**phase_3() -** In this phase you are required to type in another code of at least 2 numbers.  Less than two and the bomb detonates.  More than 2 is fine but the code is only dependent on the first two numbers.  The first number must be between 0 and 7.  The second number is simply linked to the first number: 0 must be followed by 704, 1 by 848, 2 by 736, 3 by 346, 4 by 607, 5 by 147, 6 by 832, and 7 by 536.  Nothing special other than the first number acting like a selector of jump paths to a linked second number.  

**phase_4() -** In this phase you are dealing with a recursively called function.  First you must enter two integers and the bomb will detonate if you enter more or less than that.  There is also a test that the first user inputed number is less than or equal to 14.  There are two hard coded variables that are then initialized and they, as well as the first user inputed value, are passed to func4.  There are many things going on with shuffling of variables between registers, some bit shifting, and either a subtraction or an addition being applied to some of the hard coded constants.  I will likely take another shot at figureing out exactly how to come up with the solution by following the implemented logic but I eventually brute forced it, which took a whole 30 seconds to figure out.  Regardless, the first user inputed value had to be less than or equal to 14 and had to spit out an 11 after its computation.  The answer is that the first input had to be 1.  The second input had to be a 11, because the the phase_4 code did a simple compare, nothing special.  

**phase_5() -** This function requires you to go backwards through an array of numbers to crack the code.  In memory there is a 16 element array of the numbers 0-15.  Based on the first user inputed number, you enter into that indexed element of the array, which then gives you the index of the next element in the array, etc.  You continue to bounce through the array.  The key is that each time you enter into the next element in the array there is a counter that increments.  The two stipulations that you must satisfy to move to the last portion of this phase is that you have incremented the counter to 15 and that the final value when you leave the loop is 0xf (decimal 15).  So, I mapped out the array from element 0 to 15 and then worked backwards through it to find the element I needed to start with.  Given you ultimately needed to have the element containing 0xf to exit after 15 iterations, I saw that f was at array element index 6.  Thus on the 14th iteration if I needed a 6, I would need to be in the 14th index of the array on the 13th iteration, then on index 2 of the 12th iteration.  Going back all the way to the first iteration you needed to enter into the array at the 5th index, which is the first interger needed for the user input.  After satisfying this first requirement of phase_5 there is a comparison of the second user input to what turns out to be the sum of the numbers in the array you accessed.  This number was 115.  

**phase_6() -** This function does a few initial checks on the numbers inputed by the user.  It first checks that you have inputed 6 numbers, then that they are within the range of 1 through 6, and finally that they are all unique numbers, in that no number is repeated.  These numbers act as indices within a six element array in memory, each element of which contains a number.  Each element in the array has an empty element directly adjacent to it.  The function then takes the address of the memory location within the array indexed by the second user input and places it in the empty adjacent element designated by the first user input.  Next it takes the address of the memory location within the array indexed by the third user input and places in the empty adjacent element designated by the second user input.  This continuous through all the user inputed indices and finally places the value zero in the last remaining empty element in the array.  Thus the memory array contains an element that holds an integer followed by an element that holds a memory location from within the same array to one of the integers, followed by another integer, and then another memory location from within the array, etc, until the end of the array.  The key is to place the correct memory locations, as indexed by the user inputs, so as that the integer pointed to by the address is always greater than the preceding adjacent integer.  In order to do this you must look at the various integers within the array and then place them in ascending order by the index of those integer containing elements.  The user input is then, 4 5 1 6 2 3. 