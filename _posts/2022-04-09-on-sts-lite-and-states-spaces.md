---
layout: post
title: On Slay the Spire Lite and State Spaces
date: 2022-04-09
---

As my final project for my class on planning, I am comparing traditional [Q-Learning](https://en.wikipedia.org/wiki/Q-learning) with Deep Q-Learning. I am applying them to a reduced version of the rogue-like deck-building game [Slay the Spire](https://store.steampowered.com/app/646570/Slay_the_Spire/), a game that I have fun sucking at. How much did I reduce it by? A lot. Rather than describing Slay the Spire here and then describing how I reduced it, I shall just describe the reduced version, which I have dubbed Slay the Spire Lite.

Slay the Spire Lite is a turn-based game where you fight monsters. Each turn, you draw 5 cards and play three cards from your hand. There are four cards: Strike, Block, Finesse, and Dash. Strike deals 6 damage to the monster. Block increases your block by 5. Finesse draws 2 cards. Dash deals 3 damage to the monster and increases your block by 3. After you play a card, it is sent to the discard pile. After you've played your three cards, your entire hand is sent to the discard pile and the monster attacks you for 10 damage. If you have block, the amount of block you have is subtracted from the damage the monster deals to your health. Your starting health is 50. The monster has 30 health points. When the draw pile is empty, the discard pile is shuffled and becomes the new draw pile.

With this reduced version, there are a lot of states that the game could be in. I have calculated the upper limit of states that this game set-up could visit. This game could visit about $$10^{13}$$ states. This is the number of every combination of player health, monster health, block amount, energy left, and draw/hand/discard pile configurations.  

Here's my work:

- $$D$$ - the deck size (12)
- $$C$$ - the number of card types (4) 
- $$H_p$$ - the max health of the player (50)
- $$H_m$$ - the max health of the monster (30)
- $$E$$ - energy, or how many cards can be played in a turn (3)
- $$B$$ - the maximum amount of block (15) 

To calculate how many different ways a deck of $$D$$ cards with $$C$$ card types can be in 3 different, I came up with a formula. I realized that the question can be re-framed as how many different ways can $$N$$ integers $$\in [0, S]$$ add up to $$S$$, where $$N = 3C$$ and $$S = D$$. The formula below answers that question. 

$$
f(N,S) = 
\left\{ 
  \begin{array}{ll} 
    1, && \text{if} \space N = 1 \\ 
    \sum_{s=1}^{S+1}f(N - 1,s), && \text{otherwise} 
  \end{array} 
\right.
$$

The final equation for the size of the state space is:

$$f(3*C,D) * H_p * H_m * E * B$$

This state space is far too big to represent in a table for traditional Q-Learning. I need to figure out a way to reduce the number of entries in my table to actually make it possible. I can get rid of the discard pile from the state, fold similar states into each other, reduce the game even further (which I would like to avoid), and probably a hundred other things. My next post will likely detail solutions to this problem.  
