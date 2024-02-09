## Scoping

# Noteworthy issues from the readme 

- The addresses array cannot hold duplicates, but the protocol explains that a user can enter multiple times??


# First glance at codebase
- imports Address from openzeppelin that has note about taking care to prevent Reentrancy after transferring ownership
- High (DoS) `PuppyRaffle::enterRaffle`- nested loop for raffleEnter can be optimised to not read from storage so many times

