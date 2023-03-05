# Deadlock

## The Dining Philosphers Problem
1. The problem:
    - There are n philosphers and n chopsticks around a table.
    - Each philosopher needs two chopsticks to eat. After finish eating, the philosopher puts the chopstick down.
    - Philosophers do not grab a chopstick out of the hands of another philosopher (only one person can be in possession of a chopstick at a time, each chopsitck may be represented by a binary semaphore)
    - Deadlock occurs when each philosopher grabs the chopstick on the lefft.