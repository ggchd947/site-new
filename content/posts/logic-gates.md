+++
title = 'Logic Gates'
date = 2024-04-17T18:49:18-07:00
tags = ['nand2tetris', 'computer science']
+++

## Gate Logic
- A technique for implementing Boolean functions using logic gates
- Logic gates:
    - Elementary (Nand,And,Or,Not,...)
    - Composite (Mux,Adder, ...)

## Elementary logic gates: NAND
![nand gate](/img/logic-gates/NAND-gate.png)
> A NAND gate
#### Functional Specification:
{{<highlight py>}}
if (a == 1 and b == 1):
    out=0
else
    out=1
{{</highlight>}}

#### Truth Table:
|a|b|out|
|:-:|:-:|:-:|
|0|0|1|
|0|1|1|
|1|0|1|
|1|1|0|


## Elementary logic gates: AND
![nand gate](/img/logic-gates/and-gate.png)
> An AND gate
#### Functional Specification:
{{<highlight py>}}
if (a == 1 and b == 1):
    out=1 
else 
    out=0
{{</highlight>}}

#### Truth Table:
|a|b|out|
|:-:|:-:|:-:|
|0|0|0|
|0|1|0|
|1|0|0|
|1|1|1|


## Elementary logic gates: OR
![nand gate](/img/logic-gates/or-gate.png)
> An OR gate
#### Functional Specification:
{{<highlight py>}}
if (a == 1 or b == 1):
    out=1 
else 
    out=0
{{</highlight>}}

#### Truth Table:
|a|b|out|
|:-:|:-:|:-:|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|


## Elementary logic gates: NOT
![nand gate](/img/logic-gates/not-gate.png)
> A NOT gate
#### Functional Specification:
{{<highlight py>}}
if (in==0):
    out=1 
else 
    out=0
{{</highlight>}}

#### Truth Table:
|in|out|
|:-:|:-:|
|0|1|
|1|0|


## Circuit Implementations: AND
![and circuit](/img/logic-gates/and-circuit.png)
> An AND circuit

#### Truth Table:
|a|b|out|
|:-:|:-:|:-:|
|0|0|0|
|0|1|0|
|1|0|0|
|1|1|1|


## Circuit Implementations: OR
![or circuit](/img/logic-gates/or-circuit.png)
> An OR circuit

#### Truth Table:
|a|b|out|
|:-:|:-:|:-:|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|


## Circuit Implementations: Three-way AND 
![three way and circuit](/img/logic-gates/3way-and-gate.png)
> 3 way AND circuit

#### Functional Specification
{{<highlight py>}}
if (a==1 and b==1 and c==1):
    out=1 
else 
    out=0
{{</highlight>}}
