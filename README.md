## Reverse engineering a CLI bomb  
### Background x64 bomb_lab:  
The original bomb lab was created by Randal Bryant and David O'Harraron at Carnegie Mellon for use as a training exercise for students learning how to reverse engineer a binary and immersion into assembly language.  The program is a powerful way for someone to learn this technique and begin to recognize disassembly code patterns that result from regularly used algorithms such as for-loops, array access, switch statements, recursion, simple encoding/decoding, loop nesting, etc.  I believe that there are three version of the bomb lab.  
 
 The essence of the bomb lab is that when you run the executable a "bomb" is activated that has 6 phases you need to pass in order for it to now "detonate."  By detonate, it means that you get a line of text that lets you know that the bomb went off and you failed the challange.  Each phase requires the student to enter a string that will disarm that phase and allow you to move to the next phase.  I believe that in the original bomb lab used at Carnegie Mellon University, that the bomb would communicate to a server and keep track at the number of attempts made to disarm the bomb.  More than 3 and you failed...!!!  I tried to keep this in mind when I went through this.  Thus, I did not run the bomb executable, aside from during dynamic analysis, until I was absolutely certain that I knew the pass code for moving between all the phases.  I had a ton of fun going through this exercise and would highly recommend anyone that loves a challenge and is training themselves at reverse engineering to both go through the Linux version to gain a real understanding on how to work with GDB as well as to go through the Windows version to gain experience with IDA.  

There are various versions of this bomb lab available.  For sure, the x86 version compiled for Windows and the x86-64 version compiled for Linux are different, in that the solutions and the implementation as different.  I am how the x86 version compiled for Linux compares to these other two variants or how any of these compare to the original version released by the original authors.  


###SPOILER ALERT!!!  
What I have included here is an outline of the work I put in towards working through the x86-64 version compiled for Linux.  You can get the binary from the OpenSecurityTraining - Intro to x86-64 course web page.  The link is down in the references section.  


### Tools:  
1. strings  
2. GDB  
3. hexdump  
4. IDA  

### Files:
1. Analysis.md - Essentially a walk-through of my going through the bomb lab.  I included descriptions of the various functions used in implementing the program.  
2. bomb-x64.dump - this is a objdump file with many annotations that I used while going through the code in GDB.  I found that having this file open while doing dynamic analysis was very helpful in keeping track of where I was and what what going on.  
3. String.txt - This is just a copy of the strings pulled from the binary.  I also used a version of this file that was sorted but did not include that here.  I found that sorting strings as well as seeing the strings in order that they are found within the binary useful in targeting strings that could be interesting in pulling the binary apart.  

### Still to do:  
1.  There is a "secret" phase that I still mean to go through.  I have not in this analysis but will post some analysis when I do.  

### References:  
Finally, I want to comment on the [OpenSecurityTraining.info](http://www.opensecuritytraining.info/Welcome.html) website in general.  I believe that this is one of the most valuable open-source paths into computer security available.  Special thanks to Xeno Kovah and his team for putting this together!!!  Really an amazine job and clearly a lot of effort went into this project.  

1. [OpenSecurityTraining - Intro to x86 course](http://www.opensecuritytraining.info/IntroX86.html) - get the x86 bomb lab for Linux  
2.  [OpenSecurityTraining - Intro to x86-64 course](http://www.opensecuritytraining.info/IntroX86-64.html) - get the x86-64 bomb lab for Linux  
3.  [OpenSecurityTraining - Intro to RE](http://www.opensecuritytraining.info/IntroductionToReverseEngineering.html) - get the x86 bomb lab for Windows  
4. Randal E. Bryant and David R. O'Hallaron, Carnegie Mellon University - [bomb lab](http://csapp.cs.cmu.edu/3e/labs.html) - original citation  
