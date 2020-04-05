# ShorsQuantumAlgorithm
An implementation of Shor's Quantum Algorithm with sequential QFT. (2n + 3 qubits)


The original code is part of Qiskit and it can be found [here](https://qiskit.org/documentation/_modules/qiskit/aqua/algorithms/single_sample/shor/shor.html#Shor).
It adapts code from a [project](https://github.com/ttlion/ShorAlgQiskit) that it was accomplished by two students at TU Delft University. 

The code in this repo has the potentials to run on an actual Quantum Computer as it surpasses a critical limitation. And that is the measurements inside the circuit. If you try to run such a circuit you will get the error (code 7006): *Qubit measurement is followed by instructions*. However, due to physical limitations it can’t run on any quantum device provided by IBM. The output is: 
*Circuit runtime is greater than the device repetition rate [8020].*

Circuits of that form have more than 70,000 gates and that makes the error look legitimate. 
Hopefully, there are simulators.

This implementation is based on a [paper](https://arxiv.org/abs/quant-ph/0001066) and more specifically it makes the circuit below:
![sequential QFT](https://github.com/nikoSchoinas/ShorsQuantumAlgorithm/blob/master/images/sequential_QFT.png)
As you can see the Fourier Transform is applied sequentially and this can be achieved with just one qubit. 
Ua gates were created according to another [paper](https://arxiv.org/abs/quant-ph/0205095v3).

Note: *n is the number of binary digits from the Number that we want to factorise. e.g. if N=15 then n=4 as bin(15)=1111. In the figures below there is the symbol L which practically equals 2n. Thus, L=2n.*

This version saves 2n - 1 qubits and the circuit's output is a binary number that is used in continuing fractions in order to provide the wanted period.
On the contrary, the original code uses 4n + 2 qubits in total and that makes it impossible to run on a physical quantum device. It implements the circuit below:
![QFT](https://github.com/nikoSchoinas/ShorsQuantumAlgorithm/blob/master/images/QFT.png)
IBM’s back end (free to use) with the greatest number of qubits is ibmq_16_melbourne with 15 qubits. 
Even if we want to factorise the number 21 it would required 18 qubits. On the other hand, in case that we could overcome the runtime limitation, for the same factorisation we would need just 11 qubits.

So, basically the code creates the first Ua gate, it adds the following gates (H, R'j) and it measures the qubit on the upper register. In the next iteration, it creates the first two Ua gates (with all the within H, R’j gates) and it measures the qubit again. This procedure continues L (or 2n) times as in every iteration it adds one more Ua gate. The circuit needs L (or 2n) Ua gates. 

Before the iteration starts, a list with all the 2n binary-digit-numbers is created. After every iteration we know how many times the state 1 and 0 was observed (note: circuit runs 1024 times). In case that one state is observed with significantly higher probability than the other, we can remove from the list all those binary numbers that are not useful anymore. 
After every iteration a more significant bit is returned.

### Example: 
Let's say that after the third iteration the state 1 is predominant. So, we go to the list and remove all the binary numbers that have the 0 bit at the third position (indexing starts from 0). In a 8-binary-digit-number list, we would remove the numbers in the form of xxxxx0xx. 
In case that both 1 and 0 states are equally observed, we do not remove any binary number. However, we need to keep a state (either 1 or 0) for calculating the R’j gates. 
To be more accurate, we should run a circuit for 1 state and another circuit for 0 state since R’j gates would be different. It is possible, though, that the states will be equally observed again. But that time we need to examine 4 different circuits and so on. You can imagine it as a tree that every time we equally observed the states,more branches are created (so as circuits). 
![tree](https://github.com/nikoSchoinas/ShorsQuantumAlgorithm/blob/master/images/tree.png)

Generally, this is a recursive procedure and that means that the algorithm can me implemented with a recursive function. Nevertheless, since every circuit needs more than 70,000 gates, it is not a good idea to construct many of those and wait to be executed by a simulator. That’s why we keep the list with the binary numbers. Experiments showed than even if a different circuit was created, it is highly possible that its output exists in the list of binary numbers. 

Unfortunately, the continuing fractions procedure is subjected to limitations, since the numbers that it needs to manipulate are great enough to cause an overflow error. In the code there is an if-statement to prevent that error and the computation does not provide a solution. Thus, the algorithm can’t provide a solution for numbers greater than 21.
The error would be: *OverflowError: (34, 'Numerical result out of range').* 
A stackoverflow user (meowgoesthedog) gives a pretty good explanation of this.*The built-in pow function returns a float if either of the arguments are also floats. Internally these are double-precision floating point numbers which can store up to ~10^308.*
