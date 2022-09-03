# Primel Solver

## What is Primel?
Primel is one of those wordle variants, playable at https://converged.yt/primel/.

The game has a 5 digit prime number in mind daily which you are required to guess, lets call this number X. 
 
As you guess prime numbers, the game will give you hints in the form of each digit in your guess turning green, yellow, or grey. 

Green digits means you got the same digit in the same spot as the digit in X. 
A yellow digit means that digit is in X, but not in the same spot. 
Finally, a grey digit means that digit is not anywhere in X.


For example, if X = 10007 and we guessed 10079, we would get the result

#### [GREEN, GREEN, GREEN, YELLOW, GREY]

## Setup for guess #1
The goal with this solver is to get a first guess that limits our options left to guess the most on average.
What this means is that the solver wants to find a prime that gives us the most information on average out of any other prime, also known as the most *entropy* in information theory. 

### How do you calculate information?
Information is measured in bits, and the formula we will use is giving us how many bits of information gained. 
Every bit of information splits the data in half once, so the more bits (information) gained, the less possibilities there are left. 
If we get 6 bits of information, we split the possible primes left in half 6 times.

The formula for information gained is given by 

$$I(x)=\log_2\left(\frac{1}{p(x)}\right)$$

In our problem, $x$ is a possible outcome (a combination of greens, yellows, and greys), while $p(x)$ is the likelihood of getting that outcome.

So we can get the information gained from an outcome as long as we know how often that outcome occurs in the list of primes we're looking at.


### How do you calculate entropy?

There are $3^5$ possible outcomes that can result from a guess, 
so to get the entropy of a guess, we would compare the prime that we guessed to every other five digit prime available to see the outcomes produced.

Earlier, we guessed 10079 and saw that the outcome if we were to compare it to 10007 was [GREEN, GREEN, GREEN, YELLOW, GREY], 
but to get the information gained for this outcome, we'd need to find every instance where this happens. 
This means we would compare 10079 to every prime in the list, then see where this particular combination of greens, yellows, and greys occurs and count them up.
To get the entropy, we would simultaneously count up all other outcomes that happen and get the information gained from those.

Once we count up the amount of times each outcome happens for that guess, we get the probability of each outcome, get the information from each outcome,
then average those informations.

To average the information gained for every outcome, since each depends on a probability density function, we get the formula for entropy as:

$$Entropy = \sum p(x) * I(x)$$

### So how can we see which number gives the most information on average?

There are 8363 primes in our list, and we just described how to get the entropy for a single prime. To get the prime with the best entropy, we would repeat this process for every prime in the list, using that prime as a guess and seeing the frequency of each outcome to eventually get the entropy. 

Once we've iterated through all 8363 primes, we can look at the prime that gave the most entropy to see which prime is best to guess first.


For instance, we can see that we won't get a lot of entropy from guessing 44449 because it doesn't limit the amount of primes available for most of the outcomes. The majority of primes in the list will match the outcome [GREY, GREY, GREY, GREY, GREY] when guessing 44449, which makes it less likely to limit our possible primes left unless we're very lucky and dont get this outcome.

###### In the Primel algorithm, I first create a list of the 8363 possible 5 digit primes 

###### I then take a guess and compare it to each prime in the list with the get_guess_comparisons function, which replaces each of the 8363 primes with its digits having a correspondence of 2 for green, 1 for yellow, and 0 for grey.

###### Finally, I count the amount of times each outcome happens through the function get_count_bucket, get the probability of each outcome thorugh get_probability_bucket, and get the entropy of that guess with the function get_entropy.

Iterating through all primes and treating them each as a guess and doing this process will give entropies for each prime, to which we can get the prime with the best entropy.


## Results for guess #1
The entropies for each prime as a first guess compared to the rest of the other primes gives the dataframe in the repo called "entropy_list_one.csv"

When only factoring information gained from a first outcome, we get that the best prime is 
#### 12539 with entropy 6.70
while the worst prime to guess is
#### 88883 with entropy 3.87
AKA, 12539 will split the 8363 primes in half 6.7 times on average, leaving us with only 80 primes left to guess from.

Meanwhile, 88883 will split the 8363 primes in half 3.87 times, leaving us with a whole 571 primes left to guess from on average.
## Setup for guess #2 
While the results from only factoring outcomes from a first guess is sufficient, we can do better by looking at the resulting prime list from an outcome and getting information from that sublist.

The goal for this is to look at every possible prime as before, but now assume an outcome for that prime and see the remaining primes available if that outcome were to happen. Then, we will do this for all other remaining outcomes for that prime and get every possible sublist of primes that can appear from guessing a first guess.

Once we do this, we can get the entropy of a second guess as before with these sublists of primes, then average these entropies to get the second entropy of a prime guess. 

Thus, using information about a second guess can lead to a different result, as it is possible you can get more information from the second guess even if you didn't get that much from the first, leading to that prime being better in the long run.

To get the total entropy, we would simply add the first and second entropies, and once we get this for every possible prime, we will see which prime is better if you were to factor in a second guess.

In the code, I create a massive dataframe consisting of $8363*3^5$ datapoints, or 2 million. 
This is looking at every possible outcome for each possible prime, then moving forward to get an entropy for a second guess for that outcome.

In the function ````get_primes_in_each_outcome````, we get the actual primes in each outcome for a guess once we compare it to a prime list.

In the function ````get_info_for_outcome````, we look at each outcome from the prime and see what particular information would be gained specfically given that we get that outcome.

The biggest function, ````get_second_entropy_dataframe```` combines every function that has been written to get the prime with the best second entropy for each first outcome, 
resulting in a dataframe with first_prime, first_outcome, corresponding_second_prime, first_info, first_entropy, second_entropy, total_entropy, and first_outcome_probability.





## Results for guess #2
After analyzing the dataframe from by looking at the average of each of the second entropies and adding up the first and average second entropy for each first prime, we get that the prime to give you the most information on average ignoring the outcome of a first guess is
### 28571 with total entropy  11.627 out of 13.03 after two guesses
And the worst number remains to be:
### 88883 with total entropy 9.30 after two guesses
If we continue to ignore the outcome of the first guess, the numbers that gave the most second entropy for these primes were 40933 and 14957 respectively.

Thus, we will get 11.627 entropy on average if we guess 28571 followed by 40933 all the time. 

The big dataframe does exist to give the best second guess given a first outcome, which would usually result in better entropies if you cared about what you got after the first guess.

## Extra Info
This primel algorithm can be replicated and used for other wordle variants upon changing the function
````get_guess_comparison```` to fit whatever variant you are dealing with. Every other function can remain the same, as the math used is standard and dependent on the above function. 

If you do end up changing this function, keep greens as 2, yellows as 1, and greys as 0 and return a list of lists similar to this function.


