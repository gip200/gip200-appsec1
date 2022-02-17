# George Papadopoulos - gip200@nyu.edu

LAB 1, PART 2 SUBMISSION
-------------

## **Task 1 (5pts): Crashing the Program**

In this task, we examine how a remote buffer overflow is achieved. We are able to observe the case where upon passing a simple, short string like "hello" to the server process over the network, it does not seem to cause a problem and we see a clean return and positive message "(\^\_\^)(\^\_\^)  Returned properly (\^\_\^)(\^\_\^)". 

However, when we use the build_string.py script, it generates a rather large 1500 byte message which clearly DOES overflow the buffer. No proper response is returned. As noted, if we  don't see the smiley faces, the format program has probably crashed, but in fact is respawned accordingly. Thus we see how a simple overflow will cause the server process to crash.

![enter image description here](https://github.com/gip200/gip200-appsec1/blob/main/Reports/Artifacts/gip200-lab1part2task1.jpg?raw=true)

## **Task 2 (10pts): Printing Out the Server Program's Memory**



## **Task 3 (5pts): Modifying the Server Program's Memory**



## **Task 4 (10pts): Inject Malicious Code into the Server Program**




## **Task 5 (10pts): Attacking the 64-bit Server Program**






## END OF LAB 1, PART 2 SUBMISSION
