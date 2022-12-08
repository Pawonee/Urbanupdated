# Urban Economics Replication #2
## The Price of Prejudice
#### Authors: Morten Størling Hedegaard and Jean-Robert Tyran
#### Replication by: Pawonee Khadka, University of Alabama

The paper is based on a field experiment that goes on to investigate ethnic prejudice in the workplace. The authors want to see how potential discriminators respond to changes in the cost of discrimination. The paper finds that ethnic discrimination is common but highly responsive to the
“price of prejudice,” i.e., to the opportunity cost of choosing a less productive worker on ethnic grounds. Discriminators/Employers are willing to forego, on average,  8% of their earnings to avoid a coworker of other ethnic type. It was published in the American Economic Journal: Applied Economics in 2018. Here is a link to it:

https://doi.org/10.1257/app.20150241
All analysis for the original paper was done using STATA and all data and necessary code instructions was made publicly available.

With my replication work, I didn't have much to do with the raw data, but jumped into replication right away. Following is the chuck for replication of 
##### Table 2: Team Production Function.
In this table:
    the Dependent variable is the log of the number of envelopes stuffed in round 2 by worker i
    prod1i is the number of envelopes stuffed in round 1 by worker i
    prod1j is the number of envelopes stuffed by i’s coworker in round 2
    Alone is a dummy set to 1 if worker i works alone in round 2
    Male is worker i’s gender 
    Decision maker indicates if worker i makes a choice of coworker 
    The remaining dummies characterize team composition in round 2. 
