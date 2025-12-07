# Smart-Cooperative-Parking-Decision-System

 **Full project (all text, figures, animations):**  
 **https://annkarpe.github.io/Smart-Cooperative-Parking-Decision-System/**

## What this is about

Two cars, one shared parking space, no human drivers to communicate who goes first.  
This project is a small world where:

- each car has a *distance* to the spot,  
- a sense of *urgency*,  
- possible *obstacles* in the lane,  
- and a little *memory* of how often it already got to go first.

A fuzzy-logic controller looks at all of that and decides:

- how assertive each car should be (go / wait / somewhere in between),  
- how dangerous the situation feels (conflict level),  
- and from that, how fast each car should move.

The HTML report linked above  with all code output, plots and animations.  
This README is the short tour of the ideas behind it.

---

## The core idea: fuzzy values instead of strict thresholds

Classical rules look like this:

> “If distance < 3 m, then the car is close; otherwise it is not close.”

That is a hard boundary. In practice, perception is more gradual:

> At 2 m the car feels very close, at 4 m it is somewhat close, and at 9 m it is clearly not close.

Fuzzy logic formalises this by allowing intermediate truth values between 0 and 1. For example:

- distance = 2 m → membership in “close” ≈ 0.9  
- distance = 4 m → membership in “close” ≈ 0.5  
- distance = 9 m → membership in “close” ≈ 0.0  

In this project, several intuitive terms are modelled as fuzzy sets, such as:

- distance → `close`, `medium`, `far`  
- urgency → `low`, `medium`, `high`  
- obstacle distance → `near`, `mid`, `clear`  
- history (“how often this car has gone first”) → `rarely_first`, `balanced`, `often_first`  

Each label is defined by a membership function: a curve that specifies, for any numeric value, to what degree that value belongs to the corresponding concept.


## How the controller thinks (Mamdani in plain language)

The controller is a **Mamdani-type fuzzy system**.  
Under the hood it follows a simple rhythm:

1. **Fuzzify**  
   Take the raw numbers (distances, urgencies, obstacle distances, history) and look up how much they belong to each label: how “close”, how “urgent”, how “often_first”, etc.

2. **Fire rules**  
   Use human-style rules like:

   - *If A is close and more urgent than B → A should be more assertive, B should yield.*  
   - *If there’s an obstacle near A but B’s lane is clear → be cautious with A and let B move.*  
   - *If A has been first many times and B rarely → give B a little extra assertiveness.*

   Each rule doesn’t just say yes/no; it fires with a strength between 0 and 1, depending on how well its conditions match the current situation.

3. **Blend everything together**  
   For each output (decision for A, decision for B, conflict level) all rules paint overlapping fuzzy shapes. These are merged into one fuzzy picture per output.

4. **Defuzzify**  
   That fuzzy picture is then collapsed into a single number using the **centroid** (center of gravity) method.  
   The result: a numeric decision level for each car and a numeric conflict level.

Finally, a separate risk-aware mapping turns “decision” + “conflict” + distance/obstacle info into actual speeds for the cars.

## What’s inside the HTML report

The HTML report walks through three main groups of experiments:

- **Static scenarios**  
  Several fixed situations (narrow vs. wide space, symmetric vs. asymmetric positions, different urgencies).  
  For each one you can see how the fuzzy controller sets decisions, conflict level and speeds.

- **Dynamic obstacle scenario**  
  An obstacle appears on one lane while both cars are approaching.  
  The controller is re-evaluated at every time step and the animation shows how positions, decisions and speeds change over time.

- **Fairness and history analysis**  
  A large number of random encounters are simulated.  
  After each run, the “history” of who went first is updated, so you can observe how often each car gets priority and how the fairness variables evolve.
