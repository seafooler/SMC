# MPC/SMC

## Module Intro

This repo contains our implemenation of two protocols in SMC/MPC(Secure Multi-Party Communication).

1. Yao's Garbled Circuit
- Garbled Circuit
    - iLabel: Lable input wires
    - cGarble: Generate garbled circuits
    - gcEval: Evaluate the garbled circuits
- Oblivious Transfer
    - Cyclic Group
    - 1 in 2 OT based on Bellare-Micali protocol
2. GMW
- Secret Sharing
    - n-1 random bits
    - XOR the secret and random bits to get the last bit 
- Oblivious Transfer
    - Expand 1 in 2 OT to 1 in 4 OT
## Usage

Make sure gmpy2 is installed from https://pypi.org/project/gmpy2/

Make sure the module "pycryptodome" is installed.

### Yao's Protocol

Python Usage:

run Alice.py first, then Bob.py separately. A will garble circuits for B to compute.
```
python3 Alice.py
python3 Bob.py
```

### GMW Protocol

Usage:
python3 Alice.py [inputs]

python3 Bob.py [inputs]

python3 David.py

Example:
```
# Terminal 1
python .\Alice.py [0,1,1,0,1,1,1,0]
# Terminal 2
python .\David.py
# Terminal 3
python Bob.py [0,1,1,0,1,1,1,0]
```

## Communication Procedure Intro

1. Yao's Garbled Circuit
- Alice sends Bob the circuit, garbled table, and Alice's inputs
- Bob gets his own inputs through OT
- Bob evaluates the circuit and sends back the result

2. GMW
- Initialization
    - Alice, Bob and Dealer build socket connect with each other
    - Alice and Bob perform Secret Sharing to share their inputs with each other
- Evaluation
    - Dealer evaluates the circuits 
    - Dealer sends requests to Alice and Bob to get necessary data
      - If the gate is NOT
        - If dealer has the data, just do it locally
        - If not, broadcase a requestion about the gate
          - after get the broadcast, Alice and Bob perform caculation locally and store the data lcoally.
      - If the gate is XOR
        - If Dealer has the all the inputs locally, just do it locally.
        - If not, broadcase a requestion about the gate
          - Alice could find both shares of the input wires locally, send the share1 xor share2 to dealer(so dealer can't know the input)
          - Bob does the same thing
          - Dealer could use the data got from previous 2 steps to perform computation
      - If the gate is AND
        - If Dealer has the all the inputs locally, just do it locally.
        - If not, broadcase a requestion about the gate
          - Alice and Bob would check the local inputs, and perform 1 in 4 OT
            - Alice plays the sender
            - Bob plays the sender
          - Alice send mask to the Dealer, Bob send the data he got to the dealer (AKA message xor mask)
          - Dealer got the result and store it locally
      - One thing I did not do / to do
        - If one input is from Alice and the other is stored in dealer, such as the figure in Presentation Page 10
          - Because Alice doesn't want to leak her inputs, this situation is tricky
          - My solution is transfer the model 1 to model 2, they are equal!
          - That's solution needs I write a circuit compiler, due to the limit time, I just assume all the people who provide the circuit is samrt enough and tansfered Model 1 to Model 2
      - So far, dealer could process all the situations!
    - Alice and Bob get the final result from Dealer

## Details

Function used to test the protocols: Equality function

Encryption library: Fernet from Python Cryptography

OT2: [Bellare-Micali protocol](https://crypto.stanford.edu/pbc/notes/crypto/ot.html) & [Implementation](./GMW/OT2.py)

OT4: Combine 3 OT2 & [Implementation](./GMW/OT4.py)

Cyclic Group: [CyclicGroup](./Yao/utils.py)


## Reference

- Lecture 10 slides from Giovanni Di Crescenzo, Ph.D.

- Oblivious Transfer (https://crypto.stanford.edu/pbc/notes/crypto/ot.html)

- Secure Multiparty Computation and Secret Sharing (https://www.cambridge.org/core/books/secure-multiparty-computation-and-secret-sharing/4C2480B202905CE5370B2609F0C2A67A)

- Yao’s Garbled Circuits: Recent Directions and Implementations (https://www.peteresnyder.com/static/papers/Peter_Snyder_-_Garbled_Circuits_WCP_2_column.pdf)

- Secure multi-party computation (http://people.eecs.berkeley.edu/~sanjamg/classes/cs276-fall14/scribe/lec16.pdf)

- GMW Protocol (https://www.cs.purdue.edu/homes/hmaji/teaching/Fall%202017/lectures/39.pdf)

- APragmatic Introduction to Secure Multi-Party Computation (https://www.cs.virginia.edu/~evans/pragmaticmpc/pragmaticmpc.pdf)

- Financial Cryptographyand Data Security (https://link.springer.com/content/pdf/10.1007%2F978-3-642-39884-1.pdf)

- Cyclic Group (https://github.com/ojroques/garbled-circuit)
