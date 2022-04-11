---
layout: post
title: On Slay the Spire Lite and State Spaces (cont'd)
date: 2022-04-10
---

In my [previous post](/2022/04/09/on-sts-lite-and-states-spaces/), I laid out the state space of Slay the Spire Lite 
and pointed out that it was far too big to perform traditional Q-Learning on. I have come up with a solution. It has 
two components: 1) I'll ignore the draw and discard piles for the traditional Q-Learner and 2) I'll do a more thorough 
analysis of this newly reduced state space.

By discarding the draw and discard piles, I now can just consider the cards in the hand. I also realized that I don't 
need to keep track of the number of a card I have in my hand. I just need to know whether a card is in my hand. That 
means that the number of configurations I need to represent my hand is $$2^C$$, where $$C$$ is the number of types of 
cards there are. 

The previous equation comes from the fact the I either have a certain card in my hand, which can be represented as a 
$$1$$, or I don't have it in my hand, which can be represented as a $$0$$. Using this, every hand can be represented 
as a sequence of binary digits. The number of unique binary numbers of a particular length is $$2^N$$, where $$N$$ is 
the maximum number of digits. This can be generalized to any base: the amount of numbers representable in base $$B$$ 
with a maximum length of $$N$$ digits is $$B^N$$.  

In my case, there are $$4$$ cards or binary digits, so there are $$2^4 = 16$$ hand configurations that will be 
represented in the state space for my Q-Learner. That is a lot less than the $$154,969,620$$ configurations, which I 
used in my previous calculation. It is, in fact, $$0.0000103\%$$ of my previous configuration count.

$$0.0000103\%$$ of my previous state space count is still too much though. I think I can find a closer maximum 
threshold for the amount of states that the Q-Learner will encounter. We just need to take into account three of 
the four cards: Strike, Defend, and Leap. Finesse doesn't matter; it only draws $$2$$ cards, and with our new hand 
representation, the number of cards in our hand doesn't matter anymore. Let's review what the three cards that matter 
do: Strike deals $$6$$ damage, Block gives us $$5$$ block, and Leap deals $$3$$ damage and gives us $$3$$ block.

How many block values we can reach? We can play three cards each turn. We have a card that gives us $$5$$ block and a 
card that gives $$3$$ block. So how much block can we have at any point in our turn? We start at $$0$$ block, so 
that's one block state. If we play one block card, we can have either $$5$$ or $$3$$ block. That's two more block 
values, bringing us to three block values. If we play another, we can have $$10$$, $$8$$, or $$6$$ block. That's three
more blocks values, bringing us to six values. If our last card is also a block card, we can have $$9$$, $$11$$, 
$$13$$, or $$15$$ block. That's four more block values, but we can ignore those. Our Q-Learners won't see the state
right before the monster attacks. So we have $$6$$ potential block values that the Q-Learner will see.  


How many values for the health of the monster can we reach? Well, the monster's health starts at $$30$$. Why? Because 
I said so. It also happens to work out very well for this. Strike deals $$6$$ damage and Leap deals $$3$$ damage to 
the monster. Since $$6$$ happens to be a multiple of $$3$$, for this math we can just consider the damage done by Leap.
If we use Leap once, the monster will be at $$27$$ health, so with $$30$$, that makes two values. Well, we can speed 
this up. $$30$$ is $$3 \cdot 10$$. There are $$10$$ states of monster health. Why am I excluding $$0$$? The learners
will never see a monster with $$0$$ health. Once a monster dies, a new one immediately takes it place. For our final
equation, $$H_m = 10$$. 


What about our player health? We actually cannot get less than $$51$$. Since the monster is always attacking for 
$$10$$ and we can block for $$9$$, we can actually visit every value within the interval $$[0, 50]$$. There are $$51$$
numbers in that interval. For our final equation, $$H_p = 51$$.

I'm going to revisit block now that energy is the only thing left to worry about. Our amount of block and our amount 
of energy are actually related. If we have $$3$$ energy, we haven't played any cards, so we have $$0$$ block. If we 
have $$2$$ energy, we have $$0$$, $$3$$, or $$5$$ block. If we have $$1$$ energy, we've played two cards and 
have $$0$$, $$3$$, $$5$$, $$6$$, or $$10$$ block. By combining energy and block like this, we have $$9$$ potential
configurations to consider and not $$27$$, which is $$B \cdot E$$, or $$9 \cdot 3$$. We'll represent this value with
$$B_E = 9$$, since it's primarily block states $$B$$, but we're considering energy $$E$$, too. 

So our final state space size is:


$$ 
S = B_E \cdot H_m \cdot H_p \cdot 2^C = 9 \cdot 10 \cdot 51 \cdot 16 = 73,340
$$

Phew. I can actually work with that now! 
