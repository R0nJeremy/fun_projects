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
Here, $A$, $B$, $C$ etc. are numbers stored in the corresponding byte, and as mentioned above,
the value of these numbers should be in range from $0$ to $63$.

A collection of $N$ bytes, can store unsigned integers from $0$ up to $2^{6N}-1$.

A Word in the MIX machine, is a combination of $5$ Bytes and a sign $\pm$.
$$
\text{Word}\quad=\quad\pm\quad W1\quad W2\quad W3\quad W4\quad W5
$$
hence it can contain integer numbers from $-1\,073\,741\,823$ to $+1\,073\,741\,823$.

### Registers and Memory
There are $9$ registers in the MIX machine. 
* Register $A$, or **accumulator** consists of $5$ bytes and a sign
* Register $X$, or **accumulator extension** consists of $5$ bytes and a sign
* Registers $I$, or **index registers**, are $I_1$, $I_2$, $I_3$, $I_4$, $I_5$ and $I_6$. They consist of $2$ bytes and a sign
* Register $J$, or **jump register**, consists of $2$ bytes and has always a positive sign $+$

Usually, the letter $r$ is added in front of the names of register, in order to point out 
that we are talking about *register*. Hence, a writing $rA$, $rX$, $rI_4$ means the 
corresponding register.

Apart from registers, the MIX machine has
* An **overflow trigger**, a single bit of information, which can be ***True*** or ***False***
* A **comparison flag**, which can have values of *LESS*, *EQUAL* or *GREATER*
* A **memory**, which is a structure of $4000$ machine Words ($\pm$ sign and $5$ bytes)
* *Input-output devices* 

### Detailed structure of the machine Word
The machine Word contains a sign and $5$ bytes, as already discussed above. So, there are
$6$ fields. The sign is at a position $0$, and the other bytes are ordered at positions from 
$1$ to $5$. We can specify the fields within the Word by $(L:R)$ structure. Here, both 
$L$ and $R$ are integers from $0$ to $5$, and obviously $L\leqslant R$. Certain examples are
* $(0:0)$ - only the sign
* $(0:2)$ - the sign and the first two Bytes
* $(0:5)$ - the entire Word (the most common specification)
* $(1:5)$ - all bytes except the sign
* $(4:4)$ - only the fourth Byte
* $(4:5)$ - the last two significant Bytes

Inside the machine, the structure $(L:R)$ is stored as a number $8L+R$, which can fit in a 
single Byte (the largest number corresponds to $(5:5)=8\cdot 5+5=45<64$).

### Command Formats
All commands that are given to the machine, can be represented as a single Word. The specific
fields of this Command Word have the next format
$$
\begin{equation}
    \pm AA\quad I\quad F\quad C.    
\end{equation}
$$
The sign and the first two Bytes, or $(0:2)$ of the Word, are the Address specifications. 
We can see that the sign is also a part of the Address. The field $I$, or $(3:3)$ after 
the Address is an index specification. It is used to modify the Address. If the field $I=0$, 
then the Address field is used without any modifications. Otherwise, the index field $I$ can 
have values $i$ from $1$ to $6$, which indicates one of the index registers $Ii$. In that 
case, the value of the specified index register is algebraically added to the Address field
and modifies it. We denote the resulting address of this indexation process as $M$.
Note, that the indexation process is done for **all commands**. It is expected, that the 
result of the indexation can fit in Two Bytes. If the value of $M$ cannot fit in Two Bytes,
the value of $M$ will be *undefined*.

For the majority of commands, the value of $M$ represent the field in a Memory. Since our 
MIX machine has only $4000$ Memory fields, the value of $M$ has to be in range
$0\leqslant M\leqslant 3999$. The notation $\text{CONTENTS}(M)$ means the value stored in 
the Memory by an Address of $M$. Note, that the value of $M$ can be negative for certain 
commands.

The last Byte of the command $C$, or $(5:5)$ field, is the Code of a command. For example
$C=8$ means the $\text{LDA}$ command, or `load the register A` command. 

The fourth Byte of the command $F$, or $(4:4)$ field, is the **modification** of the command 
given at $C$. Usually, it is the field modification $F=8L+R$. For example, if the command
is given with $C=8$ and $F=11=8\cdot 1+3$, the machine understands it as 
`load the register A by field (1:3)`. Or, to be more specific, such command tells the machine,
to load the register $A$, by the fields $F$ of the Word in Memory at an address $M$. 

### Notation specifications
The form given in $(1)$ is the actual command representation for the machine reading. However,
for human readable form, we use a different notation
$$
\begin{equation}
    \text{OP}\qquad\text{ADDRESS, I(F)}.
\end{equation}
$$
Here, the $\text{OP}$ is the codename for the command code $(C)$. For example, for $C=8$, we
have $\text{OP}=\text{LDA}$. The field $\text{ADDRESS}$ is the $(0:2)$ field of the command,
or $\pm AA$. The rest are the same $I$ and $F$ fields.

Several rules for better readability are
* If the index field is empty $I=0$, then the notation $I$ is dropped
* If the modification field $F$ is the default, then the notation $(F)$ is dropped

For most of the commands, the default value of the $F$ field is $(0:5)$, unless otherwise
specified. Here are some examples
$$
\begin{split}
    &\text{LDA}\qquad 2000,2(0:3)\qquad\qquad +2000\quad 2\quad 3\quad 8\\
    &\text{LDA}\qquad 2000,2(1:3)\qquad\qquad +2000\quad 2\quad 11\quad 8\\
    &\text{LDA}\qquad 2000(1:3)\qquad\qquad\quad +2000\quad 0\quad 11\quad 8\\
    &\text{LDA}\qquad 2000\qquad\qquad\qquad\quad\;\; +2000\quad 0\quad 5\quad 8\\
    &\text{LDA}\qquad -2000,4\qquad\qquad\quad\; -2000\quad 4\quad 5\quad 8
\end{split}
$$

### Description of the Commands
All the commands can be specified by the $M$, $F$ and $C$ values. 
#### Loading commands
* $\text{LDA}$ command: `C=8, F=field`. Loads the $\text{CONTENTS}(M)$ into the
  register $A$, with the specified $F$ field. 

  In all operations, in the field $F$ is not full, the sign is loaded only if it is the part
  of the field. Otherwise, $+$ sign is loaded. The rest of word is shifted towards *right*.

  **Examples** If $F$ is the default $(0:5)$, then the whole Word at an address $M$ is loaded
   into $rA$. If $F$ is $(1:5)$, then the Word is loaded with sign $+$. If at $M$ there is a 
   command, and the field $F$ is $(0:2)$, then $\pm AA$ at $M$ is loaded as 
   $$
    \pm\quad 0\quad 0\quad 0\quad AA
   $$

   Let us say, the Word by an Address $2000$ is 
   $$
    -\quad 80\quad 3\quad 5\quad 4 
   $$
   Then, the value of $rA$ after several variations of the $\text{LDA}$ command, are
   $$
   \begin{split}
    &\text{LDA}\qquad 2000\qquad\qquad\qquad\;\; -\quad 80\quad 3\quad 5\quad 4\\
    &\text{LDA}\qquad 2000(1:5)\qquad\qquad +\quad 80\quad 3\quad 5\quad 4\\
    &\text{LDA}\qquad 2000(3:5)\qquad\qquad +\quad 0\quad 0\quad 3\quad 5\quad 4\\
    &\text{LDA}\qquad 2000(0:3)\qquad\qquad -\quad 0\quad 0\quad 80\quad 3\\
    &\text{LDA}\qquad 2000(4:4)\qquad\qquad +\quad 0\quad 0\quad 0\quad 0\quad 5\\
    &\text{LDA}\qquad 2000(0:0)\qquad\qquad -\quad 0\quad 0\quad 0\quad 0\quad 0\\
    &\text{LDA}\qquad 2000(1:1)\qquad\qquad +\quad 0\quad 0\quad 0\quad 0\quad ?\\
   \end{split}
   $$
    In the last examples, the unknown sign appears if the Byte size is undefined. 

* $\text{LDX}$ command: `C=15, F=field`. Identical to $\text{LDA}$ command, but loads the $rX$
  
* $\text{LD}i$ command: `C=8+i, F=field`. Identical to $\text{LDA}$ command, but the 
 index register $rI_i$ is loaded. Since index registers are only two Bytes, it is assumed
 that the Bytes by numbers $1,2,3$ are zero. If those Bytes are not set to zero, the result
 of $\text{LD}i$ is undefined. Obviously, $1\leqslant i\leqslant 6$, and the actual notations 
 of the commands are $\text{LD}2, \text{LD}2, \text{LD}3, \text{LD}4, \text{LD}5, \text{LD}6$.

* $\text{LDAN}$ command: `C=16, F=field`. Identical to $\text{LDA}$, but loads $rA$ with inverse sign.

* $\text{LDXN}$ command: `C=23, F=field`. Identical to $\text{LDXN}$, but load $rX$.
* $\text{LD}i\text{N}$ command: `C=16+i, F=field`. Identical to $\text{LD}i$, but load the index
  register with inverse sign.

#### Storing commands





























