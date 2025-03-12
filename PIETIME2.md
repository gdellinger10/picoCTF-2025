***

**Title**: Binary Exploitation - PIE TIME 2

**CTF Title**: PIE TIME

**Category**: Binary Exploitation

**Difficulty**: HARD

**Description**: Given a binary file and a C source file and told to find the flag.

***

**Approach**: With solving PIE TIME, I decided to use Ghidra to find the offsets between the functions, then use a format string vulnerability to get the memory addresses from the server side program.

***

**Solution**:
Step 1: Run the Program

The program had the same interface that PIE TIME had, but this time didn't output the memory address of main for you. Instead having the following output:
```
Enter your name:
Enter the address to jump to, ex => 0x12345:
```

If a wrong address was given the following error would appear:
``Segfault Occurred, incorrect address.``

Step 2: Examine the C Source Code

I downloaded and open vuln.c into sublime.
```
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void segfault_handler() {
  printf("Segfault Occurred, incorrect address.\n");
  exit(0);
}

void call_functions() {
  char buffer[64];
  printf("Enter your name:");
  fgets(buffer, 64, stdin);
  printf(buffer);

  unsigned long val;
  printf(" enter the address to jump to, ex => 0x12345: ");
  scanf("%lx", &val);

  void (*foo)(void) = (void (*)())val;
  foo();
}

int win() {
  FILE *fptr;
  char c;

  printf("You won!\n");
  // Open file
  fptr = fopen("flag.txt", "r");
  if (fptr == NULL)
  {
      printf("Cannot open file.\n");
      exit(0);
  }

  // Read contents from file
  c = fgetc(fptr);
  while (c != EOF)
  {
      printf ("%c", c);
      c = fgetc(fptr);
  }

  printf("\n");
  fclose(fptr);
}

int main() {
  signal(SIGSEGV, segfault_handler);
  setvbuf(stdout, NULL, _IONBF, 0); // _IONBF = Unbuffered

  call_functions();
  return 0;
}
```

After examining, it had a similar structure to the PIE TIME so I knew I would just have to find the offset between the main and win functions to be able to manipulate the server main address into running the win function.

I also spotted that the call_function had a buffer which read the input from the "Enter your name" statement. I kept this in mind as I thought it might be useful later.

Step 3: Finding the Offset

Using Ghidra, I loaded the vuln binary and gad Ghidra analyze it. Once that was complete, I went to the functions folder and went to both the main and win function to grab their starting memory addresses.

```
win = 0010136a
main = 00101400

00101400 - 0010136a = 96
```

With this complete I knew I had an offset of 96 bytes.

Step 4: Finding the Main Function Memory Address

Now knowing the offset, I needed to find which memory address was for the function main. To do this, I asked chatgpt "What vulnerability can be exploited to leak the address?" as it was a hint given on the CTF. Chatgpt informed me that a format string vulnerability could be used. Not knowing how to perform the attack, I asked Chatgpt to explain how to perform it and had it analyze the source code to see where it could be performed. Chatgpt told me to use %p (more information on why below) and that because there is no input validation for the buffer, that I should be able to use it within the buffer.

The %p works, as C the printf function has different format string options one of those being %p. Which when specified is used to print the memory address of a variable or object.

Step 5: Using the Format String Vulnerability

Using this new technique, I started off with only using %p to see if I would get any type of output from the program. Which I ended up getting part of a memory address as shown below:
![image](https://github.com/user-attachments/assets/e3e0c15d-e63d-465b-9fb4-d68bdba55f44)

Seeing this, I made the assumption that I had to put more %p into the buffer to get more output. So, I kept adding %p until I eventually ran into an error which prevented me from adding more to the input. Now having the memory addresses, I ended up brute forcing the memory addresses into the program till I eventually found two different addresses that wouldn't produce a `Segfault Occurred, incorrect address` error. I noted down which positions they were in as the memory addresses would randomize because of Address Space Layout Randomization (ASLR). 

![image](https://github.com/user-attachments/assets/4ca404d4-8aa3-4743-91cd-347f460f9738)


With these two addresses, I went and subtracted the offset from each and attempted to put in the far right address in first, which ended up giving no error but at the same time gave no output. So, I went back through and attempted the left address. Which ended up running the win function and producing the flag.

![image](https://github.com/user-attachments/assets/0ee1b1d3-fcb8-43e2-b6c1-1aa348110a4f)

***

Conclussion:
By using static analysis with Ghidra and Sublime, I was able to figure out what the C program was doing as well figure out the offset between the main and win functions in order to perform binary exploitation. As well, was able to use a format string vulnerability to read the memory addresses from the server side memory, allowing me to find the main function's memory address and subtract it from the offset found in static analysis. Which allowed me to solve the CTF by inputting the memory address of the win function to print out the flag.
