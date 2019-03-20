# basisc grammar
## pattern grammar
A pattern is a short chunk of rhythm (that will likely to repeat in the full music). Each beat is represented by an integer. Currently there's in total 3 types:

0 = rest

1 = weak beat (kick drum)

2 = accented beat (kick drum + snare)

e.g. P = [2,1,0,1]

## full music structure
repetition is always assumed unless some other patterns is to be followed (marked by "+"). If there's other patterns to be followed, need to specify how many repetitions will be done for this pattern before the pattern switches.

e.g. P1*4 + (P2+P3)*2  -- repeat pattern P1 for 4 times, then do (P2 + P3) and repeat 2 times; when finished, restart from beginning.

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