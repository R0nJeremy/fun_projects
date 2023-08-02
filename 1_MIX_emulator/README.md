# MIX machine emulator

In this project we build an emulator of the MIX machine, described in Donald Knuth's book.
Here is an outline of the machine architecture

### Byte and Word
The classic byte consists of $8$ binary bits. However, the MIX machine is agnostic of the 
underlying numeric system. Following the standards accepted in the books, we will assume that
the machine is decimal, and a single byte can store $64=2^6$ integer numbers, from $0$ 
to $63$. Combination of two bytes, can store values from $0$ to $4095=63\cdot 2^6+63$.
Let us say, we have two bytes $BA$. If the byte $A=28$ and byte $B=12$, this gives us the value
$$
    BA=12\cdot 2^6+28=796.
$$
So, the general formula for the value of several byte word, is
$$
    \begin{split}
        BA&=B\cdot 2^6+A,\\
        CBA&=C\cdot 2^{12}+B\cdot 2^6+A,\\
        DCBA&=D\cdot 2^{18}+C\cdot 2^{12}+B\cdot 2^6+A,\\
        EDCBA&=E\cdot 2^{24}+D\cdot 2^{18}+C\cdot 2^{12}+B\cdot 2^6+A.\\
    \end{split}
$$
A collection of $N$ bytes, can store unsigned integers from $0$ up to $2^{6N}-1$.

A Word in the MIX machine, is a combination of $5$ Bytes and a sign $\pm$.
$$
\text{Word}\quad=\quad\pm\quad W1\quad W2\quad W3\quad W4\quad W5
$$
hence it can contain integer numbers from $-1\,073\,741\,823$ to $+1\,073\,741\,823$.

### Registers

