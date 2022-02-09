# Qiskit-Hackathon 2022
 
## Introduction

Encryption refers to converting plaintexts into ciphertexts using algorithms so that only those who own encryption keys can see them.
The key should not be known as only the person who owns the key should have access to the data. As cryptanalysis to find a key, there is differential cryptanalysis using a change in an output value according to a change in an input value.  The difference refers to the amount of change in two values and can be calculated through *XOR*.

The input value is X and the value to which the encryption key is applied is called X'. The differential value of the input X, ΔX, can be calculated as follows.

>> X ⊕ X’ = ΔX

Assuming that the result of encrypting the n-bit plaintext X, X' is the ciphertext Y, Y', in the case of an ideal cipher, the probability of a specific Y, Y' difference with respect to the difference between X and X' is 1/(2<sup>n</sup>). However, this probability may be greater if the cipher is designed to be weak.
An attacker can use these properties to recover the secret key (encryption key).

A differential characteristic is a sequence of input and output differences for a round such that the output difference of one round corresponds to the input difference of the next round. The highly probable differential feature allows us to derive bits from the last lower key layer using information coming in the last round of the cipher.

*The steps of differential analysis are as follows.*

1. By analyzing the effect of a specific input difference a′ on the output difference b′, search for the highest probability difference characteristics.
2. Search for pairs of plain and ciphertexts that satisfy the differential characteristic.
3. Make sure the plaintexts found are the correct pair with the candidate key. And Choose the right key for the most pairs.

Here, searching for a pair of plaintext and ciphertext that satisfies the differential characteristic takes a lot of complexity in the differential cryptanalysis.
We use the Grover Search Algorithm to quickly find the plaintext x that satisfies the differential values a', b' in the superposition state plaintext.

---
## Quantum Differential Cryptanalysis

Quantum adversary can quickly find plaintext x that satisfies the differences  *a*′ and *b*′ in the superposition plaintext by using the Grover search algorithm. As cryptanalysis to find a key, there is differential cryptanalysis using a change in an output value according to a change in an input value. The difference refers to the amount of change in two values and can be calculated through XOR.

They assume that the differential characteristics (a′, b′) are given and perform a quantum differential attack.
They prepare superposition state plaintext x using Hadamard gate and prepare the superposition state plaintext x⊕a'(w)) in which the input differential value is a′.
In Grover Oracle, superposition state x(w) , x⊕a′(w) is encrypted and the output differential is calculated.
And when the output differential coincides with the preset output differential value  b', the sign of the amplitude of the plaintext x is inverted.

Grover Diffusion operator amplifies the inverted amplitude of x in Oracle. Through iteration of Oracle and Diffusion operator, the observation probability of the correct differential distinguisher is sufficiently increased and then observed. Searching for the existing differential distinguisher requires 2<sup>n</sup> searches, but using a quantum computer, it can be found in 2<sup>n/2</sup>) times. ( 1/2<sup>n</sup> → Probability of finding a plaintext x that satisfies the differential characteristic)

## Applying Quantum Differential Distinguisher to Simple cipher

The target cipher is the Toy version of PRESENT (“Small Scale Variants Of The Block Cipher PRESENT”, Gregor Leander). The algorithm is shown in the Figure 1.

### Figure 1. The algorithm of Toy version PRESENT
![present_structure](https://user-images.githubusercontent.com/55376144/153237823-b7da6578-9818-4393-b0f2-951440e13eec.png)

The version we used for attack uses 8-bit plaintext, 8-bit key, and runs in 4 rounds. First, we implement the target cipher as a quantum circuit.

```python
def p_sbox_light_r(cir, b):
    cir.cx(b[2], b[1])
    cir.ccx(b[1], b[2], b[3])
    cir.ccx(b[3], b[1], b[2])
    cir.ccx(b[0], b[2], b[1])
    cir.cx(b[3], b[2])
    cir.x(b[3])
    cir.cx(b[1], b[2])
    cir.cx(b[3], b[0])
    cir.cx(b[0], b[1])
    cir.x(b[0])
    cir.ccx(b[1], b[2], b[3])
    
    cir.swap(b[1], b[2])
    cir.swap(b[2], b[3])
    
    
    cir.cx(b[6], b[5])
    cir.ccx(b[5], b[6], b[7])
    cir.ccx(b[7], b[5], b[6])
    cir.ccx(b[4], b[6], b[5])
    cir.cx(b[7], b[6])
    cir.x(b[7])
    cir.cx(b[5], b[6])
    cir.cx(b[7], b[4])
    cir.cx(b[4], b[5])
    cir.x(b[4])
    cir.ccx(b[5], b[6], b[7])
    
    # cir += Permutation(num_qubits = 8,pattern = [0, 2, 3, 1, 4, 6, 7, 5])
    
    cir.swap(b[5], b[6])
    cir.swap(b[6], b[7])
	
	
def permutation(cir, m):
    # cir += Permutation(num_qubits = 8, pattern = [0, 4, 1, 5, 2, 6, 3, 7])
    cir.swap(m[1], m[4])
    cir.swap(m[2], m[4])
    cir.swap(m[3], m[5])
    cir.swap(m[5], m[6])
```

Next, quantum differential cryptography analysis is performed by designing the Oracle that finds plaintext that satisfies the differential characteristics (a’, b’).

We performed cryptanalysis with the differential characteristics pair (a, b = 01100101, 01110011) known.
The process of performing quantum differential cryptanalysis on an 8-bit toy cipher is as follows:
(Toy cipher is a simple cipher)

### Figure 2. Grover Oracle for differential Distinguisher(search plaintext)
<img width="1437" alt="Quantum_circuit" src="https://user-images.githubusercontent.com/55376144/153199913-67ede29b-09fb-4aa1-a65f-14b9a2882e37.png">

1. First, 16-qubits are allocated to find an 8-bit plaintext pair(p, p’) that assigns the differential characteristics(a', b').
2. p is placed in the superposition state using Hadamard gates.
3. Through the CNOT-gate, the state of p' is made the same as that of p.
4. A differential characteristic(a = 01100101) is created for p and p' through the CNOT gate. (CNOT gate for p' and a)
5. In the case where the CNOT gate result of encryption of p and p' is the same as the difference pair (b = 01110011), the sign of the plaintext is reversed.
6. Repeat steps 3 to 5 to amplify the amplitude of the plaintext that satisfies the differential characteristics(a, b = 01100101, 01110011).


We performed quantum differential cryptanalysis on an 8-bit toy cipher with differential characteristics(a, b = 01100101, 01110011). Table 1 shows the quantum resources required for quantum differential cryptanalysis.

Quantum differential cryptanalysis uses a total of 16 qubits, 72 Hadamard gates, 512 Toffoli gates, 768 CNOT gates, 8 7-Controlled gates, and 376 X gates, and the circuit depth is 340.


### Table 1. The quantum resources required for quantum differential cryptanalysis.
|Qubit|Hadamard Gate|Toffoli Gate|CNOT Gate|7-Controlled Gate|X Gate|Depth|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|16|72|512|768|8|376|340|

—
#### References
http://www.secmem.org/blog/2019/04/08/%EC%B0%A8%EB%B6%84-%EA%B3%B5%EA%B2%A9%EC%9D%98-%EC%9D%B4%ED%95%B4/

https://lucvs.xyz/85
