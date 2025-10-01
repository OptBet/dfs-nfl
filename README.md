# dfs-nfl

## Data / Attributes

Given the following data:

- $K$ is the number of players on a lineup
- $N$ is the number of lineups we are selecting
    - For DFS NFL we usually play where $N$ is 150.
- $B$ is the amount of salary each team can take up
    - Standard for DFS NFL is $50,000.
- $T$ is the set of all NFL Teams
    - $T_{LAR}$ refers to the team formally known as the Saint Louis Rams
- $P$ is the set of all Players
    - $P_i$  refers to Player $i$
- $\text{Pos}$ is the set of all Positions
- $\mathbb{T}$ is the set of Weeks
    - $\mathbb{T}_i$ refers to Week $i$
    - $\mathbb{T}_{i, j}$ refers to Week $i$ of Year $j$
- $G$ is the set of player groups that have been predefined:
    - $G_i$ refers to Player Group $i$
    - For example, $G_i: \{P_j, P_k, \cdots, P_q\}$
    - A specific Player Group, $G_i$ can not contain duplicate players
        - A combination of players
    - Groups of other limits on them:
        1. $|G_i| \le K$ for all groups
        2. Every subgroup of a Group must be defined in $G$
        3. If two groups of size $K$ have some intersection, then $G$ must also contain all subgroups of size $K$ of the union of those two subgroups.

## Variables / Decisions

The abstract values that we need to decide. These values might not be encoded in every model  in the same way, but they usually are function equivalents

- $\mathbb{L}$ is a binary matrix with $|P|$ rows and $|N|$ columns
    - If $\mathbb{L}_{i,j}$ is $1$ then $P_i$ has been drafted on lineup $j$
- $\mathbb{G}$ is a linear matrix with $|G|$ rows and $|N|$ columns
    - If $\mathbb{G}_{i, j}$ is 1, then the following statements are true:
        - All players in $G_i$ have been drafted on lineup $j$
        - All groups that contains one or more of the players in $G_i$ have their corresponding decision variable. $\mathbb{G}_{u,j}$ set to 0.

## Functions / Maps

### Player Functions

We have a set of functions that encode information about a players data. These functions are often time varying functions. Therefore, they have one required parameter, the player , and one optional parameter, the date, $t$.

Forms:

- $\text{F(player)}: P \rightarrow \text{ReturnType<F>}$
    - Examples:
        - $\text{team}(P_i) = T_j$
        - $\text{team}(\text{Tom Brady}) = \text{TB}$
- $\text{F(player, date)}: P \times \mathbb{T} \rightarrow \text{ReturnType<F>}$
    - Examples:
        - $\text{team}(P_i, \mathbb{T}_{j, k}) = T_p$
        - $\text{team}(\text{Tom Brady}, \text{Week 5 2019}) = \text{NE}$

Functions:

- $\text{team}$ → Returns the team a player is on
- $\text{projectedScore}$ → Returns the projected score for a player
- $\text{salary}$ → Returns the salary/drafting cost for a player
- $\text{position}$ → Returns the position for a player
- …

### Group Functions:

We have a set of functions that encode information about a group of players data. These functions are similar to [Player Functions](https://www.notion.so/Player-Functions-031b1c02bdfa41d5b898e6675135a381?pvs=21) as they can also very with time and  have one required parameter, the group , and one optional parameter, the date, $t$.

Forms:

- $\text{F(group)}: G \rightarrow \text{ReturnType<F>}$
    - Examples:
        - $\text{salary}(G_j) = \sum\limits_{i \in G_j}{\text{salary}(P_i)}$
        - $\text{salary}(\text{\{Tom Brady, JJ Watt\}}) = \text{5,000} + \text{4,500}$
- $\text{F(group, date)}: G \times \mathbb{T} \rightarrow \text{ReturnType<F>}$
    - Examples:
        - $\text{salary}(G_i, \mathbb{T}_{j, k}) = \sum\limits_{i \in G_j}{\text{salary}(P_i, \mathbb{T}_{j, k})}$
        - $\text{team}(\text{\{Tom Brady, JJ Watt\}}, \text{Week 5 2019}) = \text{5,400} + \text{4,600}$

### Projected Score of Group:

S

QB → WR or TE or some RB highly positive

RB → Opposing Team Anything negative

WR → RB Opposite

## Player Constraints:

Constraints that are a directly computed from players

### Team Cost

The total cost of a selected team must be less than a specified amount:

$$
\sum_{i \in P}{\text{salary}(P_i)\mathbb{L}_{i,j}} \le B
$$

## Precomputed Group Constraints

There are certain limits to optimizing teams as combinations of players. For example, higher order interactions between specific players are not easily supported

### Group Relationships

1. Players on the same teams have extra relationships when compared to players on different teams. For example, 
    1. A QuarterBack and Wide Receiver on the same team have highly correlated projected points
    2. A QuarterBack and Wide Receiver on the same team have highly correlated ownerships.
    3. Running backs and wide receivers have negative correlation - if one is scoring then the other does not 
    4. @Kevin Can you write down any and all ideas you can think of here
2. Players on opposing teams have extra relations when compared to players on non-opposing teams
    1. @Kevin can you write down any and all ideas you can think of here?

### Group Definition

A group, $G_i$, is a combination of players that have a special relationship.

### Conservation of Player Groups and Players

We need to ensure that we can compute $\mathbb{G}$ from a linear set of constraints on $\mathbb{L}$

Example given $N$ is 1 so $\mathbb{L}$ needs only 1 index $i$ for $\mathbb{L_{i, 1}}$ → $\mathbb{L_i}$ :

### Example of 2 Person Groups

Suppose $G_1$ contains players $P_i$ and $P_j$ 

If $P_i$ has been selected on $\mathbb{L}$ but $P_j$ has not then we want $\mathbb{G_i}$ to be 0:

- $\mathbb{G_1} \le \mathbb{L_i}$
- $\mathbb{G_1} \le \mathbb{L_j}$

If $P_i$ and $P_j$ have been selected then  $\mathbb{L_i}$ and $\mathbb{L_j}$ are $1$. Then we might to add the following constraint but it is not valid. However, since our optimization value includes $\mathbb{G}_1$ in a positive way the optimization logic will try to maximize it!

- [BAD!] $\mathbb{G_1} \ge \frac{1}{2} (\mathbb{L_i} + \mathbb{L_j})$

However, if $G_2$ contains players $P_i$, $P_j$, and $P_k$ and they have been selected on the lineup then $\mathbb{L_i}$, $\mathbb{L_j}$, and $\mathbb{L}_k$ are $1$ and $\mathbb{G_1}$ should be $0$

- $\mathbb{G_1} ≤ 1- \mathbb{L}_k$

### Example of 3 Person Groups

Suppose $G_1$ contains only player $P_1$

Suppose $G_2$ contains players $P_1$, and $P_2$

Suppose $G_3$ contains players $P_1$, $P_2$, and $P_3$

Suppose $G_4$ contains players $P_1$, $P_2$, $P_3$, and $P_4$

List out constraints:

- $\mathbb{G_1} \le \mathbb{L}_1$
- $\mathbb{G_1} \le 1 - \mathbb{L}_2$
- $\mathbb{G_1} \le 1 - \mathbb{L}_3$
- $\mathbb{G_1} \le 1 - \mathbb{L}_4$
- $\mathbb{G}_2 \le \mathbb{L}_1, \mathbb{L}_2$
- $\mathbb{G}_2 \le 1 - \mathbb{L}_3$
- $\mathbb{G}_2 \le 1 - \mathbb{L}_4$
- $\mathbb{G}_3 \le \mathbb{L}_1, \mathbb{L}_2, \mathbb{L}_3$
- $\mathbb{G}_3 \le 1 - \mathbb{L}_4$

### Example of N Person Groups

Suppose $G_i$ contains players $\{P_j | j \in 1, \cdots, i\}$

Then our constraints for $\mathbb{G}_i$ in terms of $\mathbb{L}_1, \mathbb{L}_2, \cdots, \mathbb{L}_N$  are as follows:

- $\mathbb{G_i} \le \mathbb{L}_j$ for $j$ in $1$ to $i$
- $\mathbb{G_i} \le 1 - \mathbb{L_j}$ for $j$ in $i$ to $N$

# Objective

Maximize expected dollars of the submitted lineups

$$
\max_{\mathbb{L}, \mathbb{G}} \sum_{j=0}^{N} D(\mathbb{L_j}, \mathbb{G_j})
$$

Where $D$ is function that takes the lineups and groups and gives expected earnings for that team.

$D$ has to convert from projected points and historical data to dollars.

## Marginal Points and Variance

Each additional point that a team scores is worth more in dollar terms than the previous point. In addition, the rate at which this value increase from point to point itself is increasing. At bare minimum the value of each additional point (or the derivative dollars with regards to points is at least quadratic and more like exponential).

### Example 1:

Compare these two cases highly simplified cases. 

Case 1

We pick two lineups, A and B, where both A and B are projected 100 points

Case 2:

We pick two lineups, C and D, where C is projected 105 points and D is project 95 points

In both cases the total number of points projected is 200 but Case 2 has a higher expected outcome.

### Example 2:

More often, the tradeoff is not between points on each lineup, it has to do with the variance of the lineup too. Therefore, comapre these two cases:

Case 1:

We pick lineup A that has projected points of 100 with variance 10

Case 2:

We pick lineup B that has projected points of 90 with variance 30

Which team is better? To answer this we can compute the expected payout for both teams

Lets approximate the payout as:

$$
\text{Dollars} = e^{\text{\# of points}}
$$

Modeling both teams as Normal variables we can compute $\mathbb{E}[\text{Dollars}(\mathbb{N}(100, \sqrt{10}))]$ for lineup A vs $\mathbb{E}[\text{Dollars}(\mathbb{N}(90, \sqrt{32}))]$ for lineup B.

Using mathematica, we can calculate this:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/234aca85-29eb-4c25-82c3-dc1b0c90f011/Untitled.png)

For lineup A we have $e^{105}$ for lineup B we have $e^{106}$

This is more formally how we quantify the value of a “variance boosting”

## First Team Objective Function

Simple.

Let $D(\mu, \sigma)$ be a modelable function (e.g. $e^x$ using [gurobi’s exponential constraint](https://www.gurobi.com/documentation/9.1/refman/py_model_agc_exp.html) or a piece wise linear function) that maps a teams mean and variance to dollars.

Since the sum of n uncorrelated normal distributions is very simple (https://en.wikipedia.org/wiki/Sum_of_normally_distributed_random_variables):

$$
N_{sum} = N_1 + N_2 + ... + N_n
$$

$$
N_{sum} \sim N(\mu_1+\,u_2+...+\mu_n, \sigma_1 + \sigma_2 + ... + \sigma_n) 
$$

Then we can calculate our objective function as such since we picked our groups $G$ such that only uncorrelated can be selected at the same time.

$$
D(\sum_{i \in G} \mu_{G_i},\sum_{i \in G} \sigma_{G_i})  
$$

## Team Overlaps

Lets say we are analyzing the selection of two teams that have significant overlap

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/664782f5-3949-4439-8354-1c854ebe2e75/Untitled.png)

(https://drive.google.com/file/d/1CpqLcA33AZruISCEdvygz1Ijxe80I988/view)

Individuals the first team’s value is:

$$
 ⁍ 
$$

and the second teams value is 

$$
 D(\mu_b + \mu_c + \mu_d + \mu_e, \sigma_b + \sigma_c + \sigma_d + \sigma_e ) 
$$

If $a$ and $e$ are positively correlated with each other, then these two teams may be too close to each other

However, optimizing these teams together the collective value is $2*D(b, c, d, H(a, e))$

Where $H$ somehow encodes the fact that we have hedged player $a$’s outcome with players $e’s$ outcome. 


## Comments

This work was started while the authors were students in the MIT Master of Business Analytics program under the advisorship of Prof. Dimitris Bertismas. The work has been expanded in the following years.

