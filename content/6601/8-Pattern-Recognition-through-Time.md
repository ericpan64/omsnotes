## 8 - Pattern Recognition through Time

- Focus on this section: timeseries with well-defined structure (e.g. sign language, handwriting, sound, generally signal processing!)
- Dynamic Time Warping: syncing two signals where one is warped (same signal, shorter/longer time span)
    - Note: using value *deltas* as points (i.e. change in value from previous point)
    - Have X-axis as one signal (longer in example below), Y-axis as other. Points are deltas. Try to align against diagonal line from starts and ends (like in sequence alignment, have threshold for match)
    - Sakoe Chiba Bounds: adding deviation bounds from the horizontal line. Can prevent getting stuck at local minima. In practice, find best by testing diff bounds using cross-validation. Helpful for consistent warps, though difficult if different sections of signal are warped differently
- Hidden Markov Models (HMMs): model that stores hidden states and transitions between states
    - HMM represented as a Markov chain with each node having a state ("output")
        - Alternative view: each node models a section of a signal. Represent as activation signals
    - Represent transition probabilities as `p = 1/(# of time frames we stay in the state)`. The expected # of time frames given transition probability is `f = 1/(1-p)`
- Virterbi Trellis: a way to model HMM. Have each possible state as a row, and each column as timeseries unit. Value of a node is probability of state emitting the given value (modelling emission probabilities as gaussian)
    - Transitions in trellis: all forward paths from start state to end state within given timeframe
    - Viterbi Path: highest likelihood path of states. Calculate by multiplying probabilities of state transistions *and* probability of emission at given state
- HMM Training (Virterbi Alignment)
    - Initialization: from user-defined topology (# of states), split each example into segments (match # of states). From approx average segment size, fix transition probabilities (e.g. for ~4 data points per group, can approx 0.25 transition probability). Then generate gaussians for each state using data from corresponding segments (calculate mean + stdev)
    - Adjustment: move segment "boundaries" (by 1 point at a time) based on which gaussian the point best fits (based on stdev). After boundary adjustments, update transition probabilities, then update gaussian. Repeat until convergence
- Baum Welch: uses Forward-Backwards algorithm to train HMM. Intuition: instead of each data point only contributing info to a single state, have each data point contribute info to all states (weighted by likelihood)
    - Seems infinitely more tedious, though supposedly not that bad for pretty good improvement
- Sign Language using HMMs
    - A mix of HMM theory above with NLP POS involved, dear lord.
    - Stochastic Beam Search example: keep high-probability paths with randomness (similar to genetic algorithms).
- Improving HMMs
    - Brainstorming good features: "feature engineering" component is very important to properly structure knowledge and improve model!
    - How many gaussians to use? It really depends on your data -- as generic guide 2-3 works to model without overfitting
    - What markov topology to use? Again, it depends on the data, requires well-thought-out trial and error on approaches
    - Context Training: accounting for different states of data in practice (e.g. starting points varying depending on previous state).
        - Involves a lot more computation, though with enough data also significant gains. Can expect to reduce error rate by half!
    - Statistical Grammar: use prior probabilities of how often phrase comes-up
        - Can expect to reduce error rate by a fourth. With Context Training, that's 1/8th the original error!
    - State Typing: combining states if models are similar. This can reduce complexity with similar results, though gets trickier with more features...
        - Manually inspect for similar mean and variances during model training
    - Segmentally Boosted HMMs: apply boosting to HMMs. TBH lookup if need the details since I'm not sure
- Using HMMs to Generate Data: generate randomly based on possible emission values
    - Problem with data continuity, though there are ways to overcome this (e.g. in speech synthesis research)
