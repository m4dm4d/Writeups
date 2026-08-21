pwnable.kr — random Writeup

Challenge: random
Category: Exploitation / Binary
Difficulty: Beginner
Points: 1

Challenge Description

Daddy, teach me how to use random value in programming!

The challenge provides a binary named random and its source code random.c. The program asks for an integer and checks whether it satisfies a specific XOR condition.

Source Code
#include <stdio.h>


int main(){
    unsigned int random;
    random = rand(); // random value!


    unsigned int key=0;
    scanf("%d", &key);


    if( (key ^ random) == 0xdeadbeef ){
        printf("Good!\n");
        system("/bin/cat flag");
        return 0;
    }


    printf("Wrong, maybe you should try 2^32 cases.\n");
    return 0;
}

The important part is:

random = rand();

followed by:

if ((key ^ random) == 0xdeadbeef)

So we need to find a key such that:

key XOR random = 0xdeadbeef
1. The Problem With rand()

At first glance, rand() appears to make the challenge difficult because we don't know the generated value.

However, rand() generates pseudo-random numbers. Its output depends on an internal seed.

Normally, a program can initialize the PRNG with something such as:

srand(time(NULL));

which gives different starting points.

This program never calls srand().

According to the C library behavior, when srand() has not been called, the generator starts from a fixed default seed. Therefore, the first call to rand() produces the same value on this environment.

The first value is:

1804289383

which is:

0x6b8b4567
2. Verify the Value

We can confirm the value locally with a small C program:

#include <stdio.h>
#include <stdlib.h>


int main() {
    printf("%d\n", rand());
    return 0;
}

Compile and run:

gcc rand.c -o rand
./rand

Output:

1804289383

Running the program repeatedly produces the same first value because the PRNG is not seeded.

3. Solving the XOR Condition

The program requires:

key XOR random = 0xdeadbeef

We know:

random = 1804289383

Therefore:

key XOR 1804289383 = 0xdeadbeef

XOR has an important property:

A XOR B = C


therefore


A = B XOR C

So:

key = random XOR 0xdeadbeef

Using Python:

python3 -c 'print(1804289383 ^ 0xdeadbeef)'

Output:

3039230856

Therefore, the required input is:

3039230856
4. Exploitation

Run the challenge:

./random

Enter:

3039230856

The condition becomes:

3039230856 XOR 1804289383
= 0xdeadbeef

which satisfies the check and reaches:

system("/bin/cat flag");

The challenge then prints the flag.

Why This Works

The vulnerability isn't a memory corruption bug or a buffer overflow.

The mistake is relying on rand() as though it were unpredictable while never initializing its seed.

The program effectively does:

fixed seed
    ↓
rand()
    ↓
predictable value
    ↓
XOR with user input
    ↓
0xdeadbeef

Because we can predict random, we can calculate the exact key.

Final Exploit
python3 -c 'print(1804289383 ^ 0xdeadbeef)'

Result:

3039230856

Then:

./random

Input:

3039230856