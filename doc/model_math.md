# basisc grammar
## pattern grammar
A pattern is a short chunk of rhythm (that will likely to repeat in the full music). Each beat is represented by an integer. Currently there's in total 3 types:

0 = rest

1 = weak beat (kick drum)

2 = accented beat (kick drum + snare)

e.g. P = [2,1,0,1]

## full music structure
repetition is always assumed unless some other patterns is to be followed (marked by "+"). If there's other patterns to be followed, need to specify how many repetitions will be done for this pattern before the pattern switches.

Note: if not specified the repetition time, will be interpreted as 1 if there's some other pattern following and infinite if no other pattern follows.

e.g. $(P1+P2)$ -- repeat P1+P2 infinitely

e.g. $P1 * 4+P2$ -- repeat pattern P1 for 4 times, then do P2 infinitely.

e.g. $P1 * 4 + (P2+P3) * 2$  -- repeat pattern P1 for 4 times, then do (P2 + P3) and repeat 2 times; when finished, restart from beginning.


# quantify information compression
## code's descriptive length
Coding length = length of the formula + number of beats not agree with the formula (say,there's some randomness).

Length of formula counts for: 1) every pattern, coding length same as pattern length; 2) every repetition number count as 1; 3) every bracket count as 1 (because it is marking the unexpected loop head)

e.g. Length of $[2,1,0,1]*4 + ([2,1,0] + [2,0,1])*2$ = 4+1+3+3+1+1 = 13.

## features (or bugs) of this scheme
- concatenation of patterns are the same length as treating them seperately. E.g. Length([2,0,1]+[2,1,0]) = Length([2,0,1,2,1,0]). Which may be okay since when I listen to this I will group the full 6 beats as one group...

- by default, the last chunk of pattern repeats repetitively unless the repetition number is specified. This prior could be adjusted to fits human intuition better.


## compression
We may define the overall compression as Length(raw code) - DL = Total time - DL. This is to say: longer the music is, shorter and more accurate the formula is, better compression. To avoid this increase with time, we define compression rate as $\delta Compression / \delta T$, which is a constant across time when perfect compression has been achieved.

Compression rate may be equivalent as "compexity" of an input stimuli.

We also care about the compression progress at a given time point. We define progress as the change of compression rate: $d^2 Compression / dt^2$. Thus for perfect compression there will be no progress. 

Intuitively, it is likely that intermediate complexity stimuli will induce more compression pregress. For example, Introduction of new patterns increases complexity as well as "knowledge gap". 

# generative model
## patterns
First, decide pattern length $N$ by sampling from a multinomial distribution. Here we limit the maximum length $N_{max} = 16$. Higher probabilities are assigned to N=3,4,8. *TODO* find support from music literature?

Also, higher probability to keep the same pattern length as multiply of the *previous pattern* (if there exists)

Second, decide pattern fillings. There are two ways: 

### random sampling
Sample from Uniform(3) for each beat

### clock-based
Every pattern have at least one accented beat.

Assume higher probability of keeping the accent in the same beat as the *previous pattern*(if there exists). 

## generate music structure
1. Start with a pattern P0.

2. Decide whether there will be another pattern to be followed: sample from a binomial distribution.

3. If nothing follows, just repeat forever.

4. If something follows: 
	
	4.1 Decide how many repetitions $R(P0)$ P0 will have before P1 kicks in. $R$ samples from a multinomial and puts higher weight on even numbers, highest on 4.  *TODO* find support from music literature?

	4.2 Decide the next pattern P1.

	4.3 Decide whether there will be another pattern to be followed: sample from a binomial distribution.

	4.4 If nothing follows, now need to choose how the to repeat. Sample from binomial / multinomial to decide how many patterns backward to start from, biased towards smaller numbers; then from the multinomial in 4.1 to decide how many repetitions.

	4.5 If something else follows, go to 4 for the same procedure.



# fit the model from a piece of music
## what is the difficulty?
- Dealing with time series data instead of i.i.d, so that inference is based on the exact history.

- If treating each pattern as a "hidden state", the states are not known in priori. You may better not enumerating all possible states...?

## what are the potentially useful tools?
- Structural time series: 
> A structural time series model is defined by two equations. The observation equation relates the observed data  yt  to a vector of latent variables  αt  known as the "state."
$$y_t=Z^T_tα_t+ϵ_t$$.
The transition equation describes how the latent state evolves through time.
$$α_{t+1}=T_tα_t+R_tη_t.$$
(reference)[http://www.unofficialgoogledatascience.com/2017/07/fitting-bayesian-structural-time-series.html]

Difference: in music the observable is not regressed upon anything...it's more like a mapping relation.

