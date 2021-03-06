# Auto-Keras: An Efficient Neural Architecture Search System

```console
@ARTICLE{2018arXiv180610282J,
       author = {{Jin}, Haifeng and {Song}, Qingquan and {Hu}, Xia},
        title = "{Auto-Keras: An Efficient Neural Architecture Search System}",
      journal = {arXiv e-prints},
     keywords = {Computer Science - Machine Learning, Computer Science - Artificial Intelligence, Statistics - Machine Learning},
         year = 2018,
        month = jun,
          eid = {arXiv:1806.10282},
        pages = {arXiv:1806.10282},
archivePrefix = {arXiv},
       eprint = {1806.10282},
 primaryClass = {cs.LG},
       adsurl = {https://ui.adsabs.harvard.edu/abs/2018arXiv180610282J},
      adsnote = {Provided by the SAO/NASA Astrophysics Data System}
}
```

## Introduction:

The time complexity of NAS is O(nt¯), where n is the
number of neural architectures evaluated during the search, and t¯
is the average time consumption for evaluating each of the n neural networks. Many NAS approaches, such as deep reinforcement
learning, gradient-based methods and evolutionary algorithms, require a large n to reach a good
performance. Also, each of the n neural networks is trained from
scratch which is very slow.

## Development:

Initial efforts have been devoted to making use of network morphism in neural architecture search. It is a technique to
morph the architecture of a neural network but keep its functionality. Therefore, we are able to modify a trained neural network
into a new architecture using the network morphism operations,
e.g., inserting a layer or adding a skip-connection.

`Using network morphism would reduce the average training time t¯in neural architecture search.`

### Drawbacks:
1. They either require a large number of training examples, or 
2. inefficient in exploring the large search space.

### Answer:
Bayesian Optmization

## Bayesian Optimization:

#### Problems:
First, the underlying Gaussian process (GP) is traditionally used for
learning probability distribution of functions in Euclidean space.
To update the Bayesian optimization model with observations, the
underlying GP is to be trained with the searched architectures and
their performances.
`However, the neural network architectures are
not in Euclidean space and hard to parameterize into a fixed-length
vector.`

Second, an acquisition function needs to be optimized for
Bayesian optimization to generate the next architecture to observe.
However, in the context of network morphism, it is not to maximize
a function in Euclidean space, but `finding a node in a tree-structured
search space, where each node represents a neural architecture and
each edge is a morph operation.`

## AIM:

```
In this paper, an efficient neural architecture search with network
morphism is proposed, which utilizes Bayesian optimization to
guide through the search space by selecting the most promising
operations each time.

Besides, a novel acquisition function optimizer, which is capable
of balancing between the exploration and exploitation, is designed
specially for the tree-structure search space to enable Bayesian
optimization to select from the operations.
```

## Bayesian Optimization:
Traditional Bayesian optimization consists of a loop of three steps: update, generation, and observation. In the context of NAS, our proposed Bayesian optimization algorithm iteratively conducts: 
1. Update: train the underlying Gaussian process model with the existing architectures and their performance;
2. Generation: generate the next architecture to observe by optimizing a delicately defined acquisition function; 
3. Observation: obtain the actual performance by training the generated neural architecture.


## System:
### 1. Edit-Distance Neural Network Kernel for Gaussian Process

#### Problem:
The first challenge we need to address is that the NAS space is
not a Euclidean space, which does not satisfy the assumption of
traditional Gaussian process (GP).

The intuition behind the kernel function is the editdistance for morphing one neural architecture to another. More edits needed from one architecture to another means the further distance between them, thus less similar they are.

---
![kernel-function.png](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/kernel-function.PNG)

---

Where, fa and fb are two neural networks;
where function d(·, ·) denotes the edit-distance of two neural networks, whose range is [0, +∞), ρ is a mapping function, which
maps the distance in the original metric space to the corresponding
distance in the new space. The new space is constructed by embedding the original metric space into a new one using Bourgain
Theorem, which ensures the validity of the kernel.

---
![distance.png](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/distance.PNG)

---

where Dl denotes the edit-distance for morphing the layers, i.e., the
minimum edits needed to morph fa to fb
if the skip-connections
are ignored, La = {l
(1)
a
,l
(2)
a
, . . .} and Lb = {l
(1)
b
,l
(2)
b
, . . .} are the
layer sets of neural networks fa and fb
, Ds is the approximated
edit-distance for morphing skip-connections between two neural
networks, Sa = {s
(1)
a
,s
(2)
a
, . . .} and Sb = {s
(1)
b
,s
(2)
b
, . . .} are the skipconnection sets of neural network fa and fb
, and λ is the balancing
factor between the distance of the layers and the skip-connections.

#### Calculating Dl:
We assume |La | < |Lb|, the edit-distance for
morphing the layers of two neural architectures fa and fb
is calculated by minimizing the follow equation:

---
![distance-layer-wise.png](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/calculating-distance-layer-wise.PNG)

---
where φl
: La → Lb
is an injective matching function of layers
satisfying: ∀i < j, φl
(l
(i)
a
) ≺ φl
(l
(j)
a
) if layers in La and Lb are
all sorted in topological order. dl
(·, ·) denotes the edit-distance of
widening a layer into another defined in,

![edit-distance](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/distance-edit-distance.PNG)

To morph fa to fb with the given
matching, we need to first widen the three nodes in fa to the same
width as their matched nodes in fb
, and then insert a new node of
width 20 after the first node in fa. Based on this morphing scheme,
the edit-distance of the layers is defined as Dl.

Since there are many ways to morph fa to fb
, to find the best
matching between the nodes that minimizes Dl
, we propose a
dynamic programming approach by defining a matrix A|La |× |Lb |
,
which is recursively calculated as follows:

![distance-aij](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/distance-aij.PNG)

where Ai,j
is the minimum value of Dl
(L
(i)
a
, L
(j)
b
), where L
(i)
a =
{l
(1)
a
,l
(2)
a
, . . . ,l
(i)
a
} and L
(j)
b
= {l
(1)
b
,l
(2)
b
, . . . ,l
(j)
b
}.


Example Change:

![example-change](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/example-change.PNG)

#### Notes:

Well, I will be skipping the postulates for skipped connections as of now, interesting as they may seem. I do not use them very often neither have I explored them much

### 2. Optimization for Tree Structured Space

Upper-confidence bound (UCB) is selected as our acquisition
function, which is defined as:

![UCB](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/UCB.PNG)

where yf = Cost(f ,D), β is the balancing factor, µ(yf
) and σ(yf
)
are the posterior mean and standard deviation of variable yf
. It
has two important properties, which fit our problem. 

First, it has
an explicit balance factor β for exploration and exploitation. 

Second, α(f ) is directly comparable with the cost function value c
(i)
in search history H = {(f
(i)
, θ
(i)
,c
(i)
)}. It estimates the lowest
possible cost given the neural network f .
ˆ
f = arдminf α(f ) is the
generated neural architecture for next observation.

The tree-structured space is defined as follows. During the optimization of α(f ),
ˆ
f should be obtained from f
(i)
and O, where
f
(i)
is an observed architecture in the search history H, O is a
sequence of operations to morph the architecture into a new one.
Morph f to ˆ
f with O is denoted as ˆ
f ← M(f ,O), where M(·, ·) is
the function to morph f with the operations in O. Therefore, the
search can be viewed as a tree-structured search, where each node
is a neural architecture, whose children are morphed from it by
network morphism operations.

![Optmise-acquisition-function](https://github.com/AmanPriyanshu/literate-train/blob/master/images/Auto_Keras_An_Efficient_Neural_Architecture_Search_System/Optimize-Acquisition-Function.PNG)

As shown in Algorithm 1, the algorithm takes minimum temperature Tlow , temperature decreasing rate r for simulated annealing, and search history H described in Section 2 as the input. It outputs a neural architecture f ∈ H and a sequence of operations O to morph f into the new architecture. From line 2 to 6, the searched architectures are pushed into the priority queue, which sorts the elements according to the cost function value or the acquisition
function value. 

From line 7 to 18, it is the loop optimizing the acquisition function.
Following the setting in A* search, in each iteration, the architecture with the lowest acquisition function value is popped out to be
expanded on line 8 to 10, where Ω(f ) is all the possible operations
to morph the architecture f , M(f , o) is the function to morph the
architecture f with the operation sequence o. 

However, not all the
children are pushed into the priority queue for exploration purpose.
The decision of whether it is pushed into the queue is made by
simulated annealing on line 11, where 

e*( cmin−α(f′) )/T 

is a typical acceptance function in simulated annealing. cmin and fmin are updated
from line 14 to 16, which record the minimum acquisition function
value and the corresponding architecture.

### 3. Graph-Level Network Morphism

Follow the four network morphism operations on a neural network f ∈ F defined in, which can all be reflected in the change
of the computational graph G.

1. inserting a layer to f to make it deeper denoted as deep(G,u), where u is
the node marking the place to insert the layer. 
2. widening a node in f denoted as wide(G,u), where u is the node
representing the intermediate output tensor to be widened. 
3. s adding an additive connection from node u to
node v denoted as add(G,u,v). 
4. adding an concatenative connection from node u to node v denoted as concat(G,u,v).

