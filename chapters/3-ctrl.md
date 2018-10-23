---
layout: chapter
title: Introducing the <code>ctrl</code> model
description: "Extending the baseline RSA model to built environments."
---


There main difference between baseline RSA and `ctrl` is the addition of a **discourse update** function, in all other ways, `ctrl` is identical. In short this allows multiple utterances to be interpreted simultaneously, like "green square" or "green circle", rather than just "green" or "circle". However, it is worthwhile exploring how the model processes inputs, since there are subtle differences. The `ctrl` model begins with $$L_0$$, the **literal listener**, receiving an utterance $$u$$ as input. Because $$u$$ can consist of more than one word, we denote it as, $$u_{1:N} = u_1,...,u_N$$. This is the most notable divergence between the two models. In order to process more than one utterance at a time, we replace the **interpretation function** $$I(u)$$ in RSA, with the **Iterated interpretation function** $$I^{\ast}$$, which uses Kleene star notation to denote that it processes one or more utterances.[^ambient]



$$
P_{L_0}(\text{device} \mid \text{utt}) \propto 
I^{\ast}(U) P(\text{device}),
$$

where

$$I^{\ast}(U) = \bigwedge_{i=1} I(u_i) = I(u_1) \wedge I(u_2) ... I(u_N).$$

$$P(\text{device})$$ is a uniform Bayesian prior on all referents in $$C$$. Multiplying the prior by the iterated semantic update function enumerates the truth conditions of every symbol in the input sequence, and yields a discrete uniform distribution over referents.[^ordering]


In WebPPL code, this function can be represented in the following way:

<!---
At a high level, our model attempts to infer the optimal control scheme for running services within a building. From a candidate set of control schemes, the optimal choice is the one that maximises a utility function $U$. The utility function can capture various aspects of occupant utility. For instance, this could be maximizing the physical comfort of occupants, or environmental goals that chooses devices and actuations that minimizing energy consumption. This could also be a combination of the two. This allows us to consider alternative models of interaction, including whimsical system behavior, where the building can use hyperbole, sarcasm, oportunism, rudeness, politeness, Assertiveness (say, if a person is not the owner)
--->

~~~
// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects);
  return obj
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  var y = filter(function(x) { return _.includes( utterance, x ) }, Object.values(obj));
  var size = Object.keys(utterance).length == Object.keys(y).length;
  return size
}
~~~

The complete literal listener is implemented in the following code block

~~~
///fold:
var objects = [ { color: "blue", 
                  shape: "square", 
                  string: "blue square"
                },
                { color: "blue", 
                  shape: "circle", 
                  string: "blue circle"
                },
                { color: "green", 
                  shape: "square", 
                  string: "green square"
                } ]

// set of utterances
var utterances = [["blue"], ["green"], ["square"], ["circle"],["blue","square"],["blue","circle"]]

// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects);
  return obj
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  var y = filter(function(x) { return _.includes( utterance, x ) }, Object.values(obj));
  var size = Object.keys(utterance).length == Object.keys(y).length;
  return size
}
///
// literal listener
var literalListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior();
    var uttTruthVal = meaning(utterance, obj);
    condition(uttTruthVal == true)
    return obj
  }})
}

var utterance = ["blue","square"]

viz.table(literalListener(utterance))

~~~

> Play around with different utterance inputs using `var utterance = [" ", " "]`, including more than one utterance at a time. Don't forget to wrap strings in brackets `[ ]` and utterances in quotes `" "`.


Next in the pipeline is the **dialogic speaker** $$S_1$$. Recall that at the beginning of each game, an idealized speaker is assigned a referent $$r \in C$$ and is assumed to have produced the input dialog. Since we do not know *a priori* which referent they were assigned, $$L_1$$ uses the $$S_1$$ function to simulate the generative process of the speaker. The likelihood function, $$P(x_{1:N})$$ is used to recursively enumerate all semantically well formed utterances that refer to the candidate set of referents.

$$
P_{S_1}(\text{utt} \mid \text{device}) \propto 
P_{L_1}(\text{device} \mid \text{utt})^\alpha \cdot P(\text{utt}).
$$

~~~
///fold:
var objects = [ 
                { 
                  color: "blue", 
                  shape: "square", 
                  string: "blue square"
                },
                { 
                  color: "blue", 
                  shape: "circle", 
                  string: "blue circle"
                },
                { 
                  color: "green", 
                  shape: "square", 
                  string: "green square"
                } 
              ]

// set of utterances
//var utts = ["blue", "green", "square", "circle"]
//var utterances = powerset(utts).slice(1, powerset(utts).length)
var utterances = [["blue"], ["green"], ["square"], ["circle"],
                  ["blue","square"],["blue","circle"],["green","square"]]


// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects);
  return obj
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  var y = filter(function(x) { return _.includes( utterance, x ) }, Object.values(obj));
  var size = Object.keys(utterance).length == Object.keys(y).length;
  return size
}

// literal listener
var literalListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior();
    var uttTruthVal = meaning(utterance, obj);
    condition(uttTruthVal == true)
    return obj
  }})
}

///
// set speaker optimality
var alpha = 1

// pragmatic speaker
var speaker = function(obj){
  Infer({model: function(){
    var utterance = uniformDraw(utterances)
    factor(alpha * literalListener(utterance).score(obj))
    return utterance
  }})
}


viz.hist(speaker(objects[0]))
~~~


> The pragmatic speaker does not take utterances as inputs, but rather states of the world. It then generates all utterances that are compatible with the particular state. Test that this works by trying the different states using `viz.hist(speaker(objects[0]))` where `objects[2]` is the green square.


The $$S_1$$ function uses a softmax function that is identical to the one found in the baseline RSA model (cf. chapter 2). This has the effect of suppressing dialog strings that deviate from a one to one mapping of the compatible referents generated by $$L_0$$.

Finally, we have the **dialogic listener** function, $$L_1$$, given in the equation below. The dialogic listener multiplies a uniform prior on referents by the distribution over utterances generated by $$S_1$$, conditioning the probability of a compatible referent on new evidence generated by $$S_1$$.

$$
  P_{L_1}(\text{device} \mid \text{utterance}) \propto P_{S_1}(\text{utterance} \mid \text{device}) P(\text{device})
$$

~~~
///fold:
// set of states (here: objects of reference)
// we represent objects as JavaScript objects to demarcate them from utterances
// internally we treat objects as strings nonetheless
var objects = [ 
                { 
                  color: "blue", 
                  shape: "square", 
                  string: "blue square"
                },
                { 
                  color: "blue", 
                  shape: "circle", 
                  string: "blue circle"
                },
                { 
                  color: "green", 
                  shape: "square", 
                  string: "green square"
                } 
              ]

// set of utterances
//var utts = ["blue", "green", "square", "circle"]
//var utterances = powerset(utts).slice(1, powerset(utts).length)
var utterances = [["blue"], ["green"], ["square"], ["circle"],
                  ["blue","square"],["blue","circle"],["green","square"]]


// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects);
  return obj
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  var y = filter(function(x) { return _.includes( utterance, x ) }, Object.values(obj));
  var size = Object.keys(utterance).length == Object.keys(y).length;
  return size
}

// literal listener
var literalListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior();
    var uttTruthVal = meaning(utterance, obj);
    condition(uttTruthVal == true)
    return obj
  }})
}

// set speaker optimality
var alpha = 1

// pragmatic speaker
var speaker = function(obj){
  Infer({model: function(){
    var utterance = uniformDraw(utterances)
    factor(alpha * literalListener(utterance).score(obj))
    return utterance
  }})
}

///
// pragmatic listener
var pragmaticListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior()
    observe(speaker(obj),utterance)
    return obj
  }})
}

var utterance = ["blue","square"]

viz.table(pragmaticListener(utterance))
~~~


## The complete `ctrl` model

~~~
///fold:
// set of states (here: objects of reference)
// we represent objects as JavaScript objects to demarcate them from utterances
// internally we treat objects as strings nonetheless
var objects = [ 
                { 
                  color: "blue", 
                  shape: "square", 
                  string: "blue square"
                },
                { 
                  color: "blue", 
                  shape: "circle", 
                  string: "blue circle"
                },
                { 
                  color: "green", 
                  shape: "square", 
                  string: "green square"
                } 
              ]

// set of utterances
//var utts = ["blue", "green", "square", "circle"]
//var utterances = powerset(utts).slice(1, powerset(utts).length)
var utterances = [["blue"], ["green"], ["square"], ["circle"],
                  ["blue","square"],["blue","circle"],["green","square"]]


// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects);
  return obj
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  var y = filter(function(x) { return _.includes( utterance, x ) }, Object.values(obj));
  var size = Object.keys(utterance).length == Object.keys(y).length;
  return size
}

///
// literal listener
var literalListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior();
    var uttTruthVal = meaning(utterance, obj);
    condition(uttTruthVal == true)
    return obj
  }})
}

// set speaker optimality
var alpha = 1

// pragmatic speaker
var speaker = function(obj){
  Infer({model: function(){
    var utterance = uniformDraw(utterances)
    factor(alpha * literalListener(utterance).score(obj))
    return utterance
  }})
}

// pragmatic listener
var pragmaticListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior()
    observe(speaker(obj),utterance)
    return obj
  }})
}

var utterance = ["blue","square"]

viz.table(literalListener(utterance))
viz.hist(speaker(objects[0]))
viz.table(pragmaticListener(utterance))
~~~

## Footnotes

[^ambient]: If we wanted to include contextual information, we could do so in the following way: 1. let $$t$$ denote a value related to time of day, and let $$\theta$$ be a contextully determined free parameter representing the ambient light levels detected by a sensor, or ambient temperature. More concretely, in the case of a temperature request like "it's cold in here" we are interested in computing the following posterior distribution: $$P_{L_1}(r \mid \mathbf{u}, t, \theta) \propto P_{S_1}(\mathbf{u} \mid r, t, \theta) P(r).$$


[^ordering]: At a high level, $$L_0$$ recursively conditions the truth value of the $$i^{th}$$ utterance in $$u_{1:N}$$ on all past utterances. It is worthwhile noting that the model will produce identical results regardless of whether each element in the input string is processed independently, or simultaneously, since $$P(x_1)\cdot P(x_2\mid x_1)\cdot...\cdot P(x_n\mid x_{1:x_n-1}) = P(x_{1:N})$$, which is equal to $$\prod^{N}_{i=1} P(x_i \mid x_{1:i-1})$$. 
