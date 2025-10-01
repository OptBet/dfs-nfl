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
