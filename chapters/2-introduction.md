---
layout: chapter
title: Introducing the Rational Speech Act framework
description: "An introduction to language understanding as Bayesian inference"
---

## Modelling Language understanding as Bayesian inference


Success in our day to day lives depends on our ability to explain the behavior of other agent's behavior in terms of their likely mental states. If I wink, will they know I was being ironic? or, if I take the last cookie, will they think I'm greedy?  Philosophers and psychologists call this process of mentalizing **theory of mind**. On this view, the mind is a tool of undertaking mental simulations of the thoughts, motivations, and actions of others. This process can also be inverted, allowing us to to reason backwards from observations of others' actions, using this as evidence for reconstructing the likely thoughts and motivations that led to their behavior. This allows us to see others as rational actors, where actions are not merely motions, but rather, a consequence of larger plans that satisfy desires. On this view, humans are (approximately) rational planners, who use goals as the basis for carrying out plans. 


<img src="../assets/img/ToM.png" alt="Fig. 1: Graphical representation of the Bayesian RSA model." style="width: 800px;"/>
<center>Fig. 1: Bayesian RSA schema.</center>

Mentalizing also extends to settings involving multiple persons reasoning about one another's decision making, a process known as **social cognition**. Central to this idea is a recursive process of two agents reasoning about the possible mental states of each other --- an agent reasoning about how their partner is reasoning about how they are reasoning, and vice versa. To clarify how such a process might work with respect to communication, we now consider a simple example of a **reference game**, depicted in figure 1. Reference games formalize goal oriented language use by embedding it within a precisely defined context between an idealized speaker $$S$$, and a listener $$L$$. Games begin with the speaker being privately assigned a referent (_blue-square_, _blue-circle_, _green-square_) from the figure above. For our purposes, we will assume that $$S$$ has been assigned _blue-square_. Their task is to choose a message from an intentionally limited vocabulary (blue, green, square, circle) that conveys the shape to the listener with minimal ambiguity. 

A naive strategy for $$S$$ would be to indiscriminately send messages that are literally true of the referent, in this case "blue" and "square", without any consideration for referential ambiguity. In the parlance of reference games, we call an agent with this strategy a **literal speaker**. A **pragmatic speaker** on the other hand, attempts to communicate with the listener with _minimal_ ambiguity. If the speaker wanted to communicate _blue-square_ to the listener with minimal ambiguity, they would need to choose "blue", since "square" is ambiguous between both the left and right squares, but distinct from the circle which also happens to be blue. But since _green-circle_ can be referred to unambiguously using "circle", a cooperative listener will consider the possibility that the speaker was sensitive to this alternative, and chose "blue" on this basis.

However, missing from this picture is a precise quantitative explanation as to how $$S$$ would actually go about choosing an utterance that minimizes ambiguity, and how an idealized listener $$L$$, would reason about the process that $$S$$ used to infer their communicative intent. The **rational speech acts** (RSA) model offers a Bayesian formalization of pragmatic reasoning in terms of a recursive, three stage interpretation process. Sitting at the top of the recursion is the pragmatic listener $$L_1$$. Upon observing an utterance $$u$$, $$L_1$$ uses Bayesian inference to simulate how a pragmatic speaker, $$S_1$$, would choose an utterance. One level down, the pragmatic speaker considers how a naive, literal listener $$L_0$$ would interpret a given utterance. This involves $$S_1$$ choosing an utterance that maximizes the probability that $$L_0$$ will correctly infer the correct shape. At this point the recursion terminates, and $$L_1$$ has derived a pragmatically enriched interpretation of $$u$$. In short These theories posit that much of the ambiguity of language use can be resolved through a process of enumerating what _could_ have been said in a context, and filtering candidate possibilities based on their ability to satisfy a mutually shared goal.


## Implementing the RSA Model as a Probabilistic Program

The process begins with the **literal listener** function, denoted $$L_0$$, which is used to determine the **semantic values** (literal truth values) of the input utterance $$u$$. For instance, the semantic truth values of "blue" are the referents $$r_1$$ and $$r_2$$.


$$P_{L_0} (s \mid u) \propto I(u) \cdot P(s)$$


Formally, this is achieved using a **semantic interpretation function**. Intuitively, this function maps $$1$$ to utterances that truthfully describe a feature of an object, and $$0$$ otherwise. Applying this function iteratively (to all utterance-object pairs) yields a truth table containing all semantic values that can be generated by the model. In the parlance of Bayesian inference, this is the hypothesis space $$H$$. At this point the recursion terminates, and $$L_1$$ has derived a pragmatically enriched interpretation of $$u$$. 

~~~
// set of states (here: objects of reference)
// we represent objects as JavaScript objects to demarcate them from utterances
// internally we treat objects as strings nonetheless
var objects = [{color: "blue", shape: "square", string: "blue square"},
               {color: "blue", shape: "circle", string: "blue circle"},
               {color: "green", shape: "square", string: "green square"}]

// set of utterances
var utterances = ["blue", "green", "square", "circle"]

// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects)
  return obj.string 
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  _.includes(obj, utterance)
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

viz.table(literalListener("blue"))
~~~

> **Exercises:** 
> 1. Check what happens with the other utterances.
> 2. In the model above, `worldPrior()` returns a sample from a `uniformDraw` over the possible world states. What happens when the listener's beliefs are not uniform over world states? (Hint, use a `categorical` distribution by calling `categorical({ps: [list_of_probabilities], vs: [list_of_states]})`).


Fantastic! We now have a way of integrating a listener's prior beliefs about the world with the truth functional meaning of an utterance.


## Pragmatic Speaker

Next in the pipeline is the **pragmatic speaker** function, $$S_1$$, that simulates a speaker tasked with unambiguously referring to a shape in $$C$$. $$S_1$$ is described as pragmatic because it is designed to simulate a cooperative speaker who is capable of considering the ways in which certain utterances in the vocabulary lead to ambiguities. It then uses this knowledge as the basis for choosing an utterance that minimizes the interpretative ambiguity that a literal listener must face when encountering the intended meaning of an utterance.


  $$P_{S_1}(u \mid s)  \propto P(u) P_{L_0}(s \mid u)^\alpha$$


Formally, the right hand side of $$S_1$$ uses the **softmax choice rule**. The purpose of the softmax function is to assign more probability mass to utterances that refer unambiguously to an object, and suppress the mass of utterances that do not. Indeed, this is how the model determines that the utterance "blue" is more likely to refer to $$r_2$$ than $$r_1$$ (cf. the probabilities specified in subfigure (1c)).

~~~norun
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
~~~


## The Pragmatic Listener

The process ends with the **pragmatic listener**, who is faced with inferring which hypothesis $$h$$ characterizes the true state of the world, among the collection of all possible world states that might exist called the **hypothesis space**, denoted  $$H = (h_1,...,h_N)$$. In the case of a reference game, $$h$$ refers to the listeners belief about which referent the speaker is referring to in the context $$C$$, and $$H$$ refers to all possible mappings between utterances in $$V$$ and referents in $$C$$. The complete hypothesis space for a simple game is given in subfigure (1b) above. Upon encountering an utterance, our learner, the **pragmatic listener** $$L_1$$, updates it's beliefs in a three step process via Bayes' rule, given as the equation below

$$P_{L_1} (s \mid u) \propto P_{S_1}(u \mid s)P(s).$$

> Unfold the code to see the complete model from start to finish.

~~~~
///fold:

// set of states (here: objects of reference)
// we represent objects as JavaScript objects to demarcate them from utterances
// internally we treat objects as strings nonetheless
var objects = [{color: "blue", shape: "square", string: "blue square"},
               {color: "blue", shape: "circle", string: "blue circle"},
               {color: "green", shape: "square", string: "green square"}]

// set of utterances
var utterances = ["blue", "green", "square", "circle"]

// prior over world states
var objectPrior = function() {
  var obj = uniformDraw(objects)
  return obj.string 
}

// meaning function to interpret the utterances
var meaning = function(utterance, obj){
  _.includes(obj, utterance)
}

// literal listener
var literalListener = function(utterance){
  Infer({model: function(){
    var obj = objectPrior();
    condition(meaning(utterance, obj))
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

viz.table(pragmaticListener("blue"))
~~~~




