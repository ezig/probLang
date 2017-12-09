---
layout: chapter
title: English Reciprocals in RSA
---

#### Shannon Bryant and Ezra Zigmond

Recall that we’re assuming that there are two salient groups: in our running example, a group of boys and a group of girls. Those groups both have some size, but really what we care about is how many pairs are within each group as well as how many pairs exist across the groups. We can compute the number of pairs combinatorially – the math is folded, but is simple for small cases. As shown in the figure below, if each group has 3 individuals, then there will be three pairs within each group and 9 pairs across groups:

<img src="assets/img/groups.png" style="width: 600px;"/>

The model is flexible enough that we could have groups of different sizes, but for examples, we’re going to stick to small groups of the same size because that’s where we have the strongest intuitions.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});
///

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;
~~~~

Recall that there’s ambiguity about whether we have a within-group or across-group interpretation. We assume a priori that both of these interpretations are equally accessible.

~~~~
// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}
~~~~

In a particular context where you hear a sentence like "the boys and the girls taught each other", you have some sort of prior beliefe about the likelihood that the predicate "taught" would hold over a pair of boys, a pair of girls, or a pair containing a boy and a girl. Thus, prior knowledge depends on the particular context and predicate in a sentence.

Within each context prior entry, `within_group_1_prob` is the prior probability that the predicate would hold for a pair within group 1 (and likewise for `within_group_2_prob` and group 2) and `across_prob` is the probability that the predicate would hold for a pair across the two groups. As an example of what this means, looking at "classroom.taught", we see that our prior belief is that it's much more likely that a boy would teach a boy or a girl would teach a girl than that a boy would teach a girl.

~~~~
var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}
~~~~

The state we are interested in sampling from these priors is a count of how many pairs the predicate holds for within each group and across the groups. Sampling this state is a bit more involved than some RSA models. We know exactly how many within group and across group pairs exist and the prior probability that the predicate would hold for each pair. Therefore, to sample a state, we can flip an (appropriately biased) coin for each state and count the number of total successes for each sort of pair. Summarizing:

<img src="assets/img/sampling.png" style="width: 600px;"/>

The resulting state will look something like:

<img src="assets/img/sampled.png" style="width: 600px;"/>

Note that the state does not encode the configuration of the pairs, so there are many valid ways to translate the state shown on the left into a diagram such as the one on the right.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}
///

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Run the code box to sample a state 
// Changing the argument will change the context
statePrior("classroom.taught")
~~~~

We add uncertainty about the topic of conversation (QUD). This recognizes that "The boys and girls taught each other" might be addressing many different possible questions. In particular, we allow for the questions "Did all the boys and girls teach other?", "Did most of the boys and the girls teach other?",  "Did any of the boys and girls teach other?", and "How many of the boys and girls teach other?". Note that these QUDs are still ambiguous: it is unclear what group interpretation the QUDs are asking about. Therefore, unlike many RSA models with QUDs where the value of the QUD can be computed purely based on the state, our model also uses the group interpretation to compute the QUD.


We assume that each QUD is equally likely, but this prior could be shifted by conversational context (for example, if someone asked you one of the questions directly).

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}
///

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

// True just if predicate holds for every pair within group or every pair across
// (depending on group interpretation)
var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

// True just if predicate holds for any pair within group or every pair across
// (depending on group interpretation)
var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

// True just if predicate holds for most pairs within group or every pair across
// (depending on group interpretation)
var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

// Returns how many pairs the predicate held for within each group or across groups
// (depending on group interpretation)
var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}


// Package all the functions up
var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};


// Example of getting a QUD value
var state = {
  group1: 2,
  group2: 2,
  across: 1
};
var most = qudFns["most"]
most(state, "within");

~~~~

We allow a speaker two possible utterances: saying an ambiguous utterance involving "each other" (like "The boys and girls taught each other") or staying silent. The literal meaning of the each other utterance sticks to the strong reciprocity semantic within the particular group interpretation (that is, either strong reciprocity within each group or across groups).

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}

var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};
///

// Speaker can say ambiguous sentence involving "each other" or say nothing
var utterances = ["null", "eachother"];

var utterancePrior = function() {
  return uniformDraw(utterances);
}

// Literal meaning of utterance given the utterance, state, group_interp
var meaning = function(utterance, state, group_interp) {
  if (utterance === "eachother") {
    return isAll(state, group_interp);
  } else if (utterance === "null") {
    return true;
  }
}

// Example where truth value of utterance depends on group interpretation
var state = {
  group1: 0,
  group1: 0,
  across: 9
}
display(meaning("eachother", state, "across"))
display(meaning("eachother", state, "within"))
~~~~

From the level of the literal listener $$L_0$$ up through the $$S_2$$ speaker, the model resembles the structure of Scontras and Tessler's [model of scope ambiguity](https://gscontras.github.io/probLang/chapters/04-ambiguity.html) wherein there is ambiguity about a scope parameter that affects the literal meaning as well as ambiguity about the QUD. Further, at the $$L_1$$ level, there will be an interaction between inference about the interpretation and inference about the QUD. The main difference is that there is a `context` value passed down through the model. This is necessary so that the state prior can be sampled appropriately based on context. This resembles the [Lassiter and Goodman (2013)](http://journals.linguisticsociety.org/proceedings/index.php/SALT/article/view/2658) gradable adjective model where an "item" value is threaded through the model since priors depend on what item is under discussion.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}

var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};

// Speaker can say sentence involving "each other" or nothing
var utterances = ["null", "eachother"];

var utterancePrior = function() {
  return uniformDraw(utterances);
}

// Literal meaning of utterance given the utterance, state, group_interp
var meaning = function(utterance, state, group_interp) {
  if (utterance === "eachother") {
    return isAll(state, group_interp);
  } else if (utterance === "null") {
    return true;
  }
}
///

// Literal listener samples a state and conditions on truth value
// Returns qudValue
var literalListener = cache(function(utterance, group_interp, qud, context) {
  Infer({method: "enumerate"}, function() {
    var state = statePrior(context);
    condition(meaning(utterance, state, group_interp));
    
    var qudFn = qudFns[qud]
    return qudFn(state, group_interp);
  });
});
~~~~

The pragmatic speaker $$S_1$$ chooses an utterance based on how well it informs the listener about the intended QUD.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}

var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};

// Speaker can say sentence involving "each other" or nothing
var utterances = ["null", "eachother"];

var utterancePrior = function() {
  return uniformDraw(utterances);
}

// Literal meaning of utterance given the utterance, state, group_interp
var meaning = function(utterance, state, group_interp) {
  if (utterance === "eachother") {
    return isAll(state, group_interp);
  } else if (utterance === "null") {
    return true;
  }
}

// Literal listener samples a state and conditions on truth value
// Returns qudValue
var literalListener = cache(function(utterance, group_interp, qud, context) {
  Infer({method: "enumerate"}, function() {
    var state = statePrior(context);
    condition(meaning(utterance, state, group_interp));
    
    var qudFn = qudFns[qud]
    return qudFn(state, group_interp);
  });
});
///

// Pragmatic speaker samples an utterance and factors based on listener informativity
var alpha1 = 1;
var pragmaticSpeaker = cache(function(state, group_interp, qud, context) {
  return Infer({method: 'enumerate', model: function() {
    var qudFn = qudFns[qud];
    var qudValue = qudFn(state, group_interp);

    var utterance = utterancePrior();
    factor(alpha1 * literalListener(utterance, group_interp, qud, context)
           .score(qudValue));
    return utterance;
  }});
});
~~~~

The pragmatic listener $$L_1$$ must jointly reason about what the group interpretation, QUD, and true state are.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}

var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};

// Speaker can say sentence involving "each other" or nothing
var utterances = ["null", "eachother"];

var utterancePrior = function() {
  return uniformDraw(utterances);
}

// Literal meaning of utterance given the utterance, state, group_interp
var meaning = function(utterance, state, group_interp) {
  if (utterance === "eachother") {
    return isAll(state, group_interp);
  } else if (utterance === "null") {
    return true;
  }
}

// Literal listener samples a state and conditions on truth value
// Returns qudValue
var literalListener = cache(function(utterance, group_interp, qud, context) {
  Infer({method: "enumerate"}, function() {
    var state = statePrior(context);
    condition(meaning(utterance, state, group_interp));
    
    var qudFn = qudFns[qud]
    return qudFn(state, group_interp);
  });
});

// Pragmatic speaker samples an utterance and factors based on listener informativity
var alpha1 = 1;
var pragmaticSpeaker = cache(function(state, group_interp, qud, context) {
  return Infer({method: 'enumerate', model: function() {
    var qudFn = qudFns[qud];
    var qudValue = qudFn(state, group_interp);

    var utterance = utterancePrior();
    factor(alpha1 * literalListener(utterance, group_interp, qud, context)
           .score(qudValue));
    return utterance;
  }});
});
///

// Pragmatic listener jointly infers the state, two thresholds, and which
// interpretation to use, and factors based on pragmatic speaker
var pragmaticListener = cache(function(utterance, context) {
  return Infer({method: 'enumerate', model: function() {
    /// priors ///
    var state = statePrior(context);
    var qud = qudPrior();
    var group_interp = groupInterpPrior();
    //////////////
    
    observe(pragmaticSpeaker(state, group_interp, qud, context), utterance);
    
    return state;   
  }});
});

// Observe the different pragmatic listener predictions for a case
// where the context strongly biases a within interpretation 
// compare to where a within interpration is more weakly biased
viz.marginals(pragmaticListener("eachother", "classroom.taught"));
viz.marginals(pragmaticListener("eachother", "classroom.presentto"));
~~~~

Last, $$S_2$$ models a truth value judgment. The speaker is presented with a state and the context and must decide whether or not to endorse the ambiguous utterance.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}

var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};

// Speaker can say sentence involving "each other" or nothing
var utterances = ["null", "eachother"];

var utterancePrior = function() {
  return uniformDraw(utterances);
}

// Literal meaning of utterance given the utterance, state, group_interp
var meaning = function(utterance, state, group_interp) {
  if (utterance === "eachother") {
    return isAll(state, group_interp);
  } else if (utterance === "null") {
    return true;
  }
}

// Literal listener samples a state and conditions on truth value
// Returns qudValue
var literalListener = cache(function(utterance, group_interp, qud, context) {
  Infer({method: "enumerate"}, function() {
    var state = statePrior(context);
    condition(meaning(utterance, state, group_interp));
    
    var qudFn = qudFns[qud]
    return qudFn(state, group_interp);
  });
});

// Pragmatic speaker samples an utterance and factors based on listener informativity
var alpha1 = 1;
var pragmaticSpeaker = cache(function(state, group_interp, qud, context) {
  return Infer({method: 'enumerate', model: function() {
    var qudFn = qudFns[qud];
    var qudValue = qudFn(state, group_interp);

    var utterance = utterancePrior();
    factor(alpha1 * literalListener(utterance, group_interp, qud, context)
           .score(qudValue));
    return utterance;
  }});
});

// Pragmatic listener jointly infers the state, two thresholds, and which
// interpretation to use, and factors based on pragmatic speaker
var pragmaticListener = cache(function(utterance, context) {
  return Infer({method: 'enumerate', model: function() {
    /// priors ///
    var state = statePrior(context);
    var qud = qudPrior();
    var group_interp = groupInterpPrior();
    //////////////
    
    observe(pragmaticSpeaker(state, group_interp, qud, context), utterance);
    
    return state;   
  }});
});
///

// S2 is given a state, samples an utterance, and factors based on informativity
// to the pragmatic listener (models speaker endorsement in a truth-value judgement)
var alpha2 = 1;
var s2 = cache(function(state, context) {
  return Infer({method: 'enumerate', model: function() {
    var utterance = utterancePrior();
    factor(alpha2 * pragmaticListener(utterance, context).score(state))
    return utterance;
  }});
});

// S2 is much more likely to endorse when strong reciprocity holds
var state1 = {
  group1: 2,
  group2: 2,
  across: 0
}

var state2 = {
  group1: 3,
  group2: 3,
  across: 0
}

viz(s2(state1, "classroom.taught"));
viz(s2(state2, "classroom.taught"));
~~~~

To address the intuition that a speaker may only care about conveying information about a particular group interpretation, we introduce $$S_2'$$ which takes a group interpretation as an argument and marginalizes out parts of the listener's inference that are irrelevant to the particular interpretation.

~~~~
///fold:
// Helper functions for computing number of pairs
var factorial = cache(function(n) {
  product(mapIndexed(function(i, v) { i + 1 },
                     repeat(n, function() { return 0 })));
});

var nchoosek = cache(function(n, k) {
  return (factorial(n) / factorial(k) / factorial(n - k));
});

// Number of individuals in each of the two salient groups
var group_1_size = 3;
var group_2_size = 3;

// Number of pairs within and across groups
var group_1_pairs = nchoosek(group_1_size, 2);
var group_2_pairs = nchoosek(group_2_size, 2);
var across_pairs = group_1_size * group_2_size;

// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}

var contextPriors = {
  "classroom.taught" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "classroom.dontknow" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "classroom.hate" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "classroom.presentto" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
  "sports.raced" : {
    within_group_1_prob: 0.8,
    within_group_2_prob: 0.8,
    across_prob: 0.2
  },
  "sports.cheeredon" : {
    within_group_1_prob: 0.2,
    within_group_2_prob: 0.2,
    across_prob: 0.8
  },
  "sports.congratulated" : {
    within_group_1_prob: 0.4,
    within_group_2_prob: 0.4,
    across_prob: 0.6
  },
  "sports.warmedup" : {
    within_group_1_prob: 0.6,
    within_group_2_prob: 0.6,
    across_prob: 0.4
  },
}

// Given either group 1 or group 2, sample from the prior for how many
// pairs the predicate holds on within that group
var within_group_prior = function(group_num, context) {
  // Get the prior information for the context
  var cp = contextPriors[context];
  
  if (group_num == 1) {
    // Prior distribution is <number of pairs> independent coin flips
    // where each flip has fixed probability determined from context prior
    return sample(Binomial({p: cp.within_group_1_prob, n: group_1_pairs}));
  } else {
    return sample(Binomial({p: cp.within_group_2_prob, n: group_2_pairs}));
  }
}

// Sample from the prior on how many across-group pairs the predicate holds on
var across_group_prior = function(context) {
  var cp = contextPriors[context];
  
  // Flip coins again
  return sample(Binomial({p: cp.across_prob, n: across_pairs}));
}

// Put all of the different state priors together for convenience
var statePrior = function(context) {
  return {
    group1: within_group_prior(1, context),
    group2: within_group_prior(2, context),
    across: across_group_prior(context),
  }
}

// Possible QUDs
var quds = ["all", "any", "most", "howMany"];

var qudPrior = function() {
  return categorical([1, 1, 1, 1], quds)
}

var isAll = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 === group_1_pairs
            && state.group2 === group_2_pairs);
  } else if (group_interp === "across") {
    return state.across === across_pairs;
  }
}

var isAny = function(state, group_interp) {
    if (group_interp === "within") {
      return (state.group1 > 0 && state.group2 > 0);
    } else if (group_interp === "across") {
      return (state.across > 0);
    }
}

var isMost = function(state, group_interp) {
  if (group_interp === "within") {
    return (state.group1 >= (group_1_pairs / 2)
            && state.group2 >= (group_2_pairs / 2));
  } else if (group_interp === "across") {
    return state.across >= (across_pairs / 2);
  }
}

var howMany = function(state, group_interp) {
  if (group_interp === "within") {
    return {
      group1: state.group1,
      group2: state.group2
    }
  } else {
    return {
      across: state.across
    }
  }
}

var qudFns = {
  any : isAny,
  most : isMost,
  all : isAll,
  howMany: howMany
};

// Speaker can say sentence involving "each other" or nothing
var utterances = ["null", "eachother"];

var utterancePrior = function() {
  return uniformDraw(utterances);
}

// Literal meaning of utterance given the utterance, state, group_interp
var meaning = function(utterance, state, group_interp) {
  if (utterance === "eachother") {
    return isAll(state, group_interp);
  } else if (utterance === "null") {
    return true;
  }
}

// Literal listener samples a state and conditions on truth value
// Returns qudValue
var literalListener = cache(function(utterance, group_interp, qud, context) {
  Infer({method: "enumerate"}, function() {
    var state = statePrior(context);
    condition(meaning(utterance, state, group_interp));
    
    var qudFn = qudFns[qud]
    return qudFn(state, group_interp);
  });
});

// Pragmatic speaker samples an utterance and factors based on listener informativity
var alpha1 = 1;
var pragmaticSpeaker = cache(function(state, group_interp, qud, context) {
  return Infer({method: 'enumerate', model: function() {
    var qudFn = qudFns[qud];
    var qudValue = qudFn(state, group_interp);

    var utterance = utterancePrior();
    factor(alpha1 * literalListener(utterance, group_interp, qud, context)
           .score(qudValue));
    return utterance;
  }});
});

// Pragmatic listener jointly infers the state, two thresholds, and which
// interpretation to use, and factors based on pragmatic speaker
var pragmaticListener = cache(function(utterance, context) {
  return Infer({method: 'enumerate', model: function() {
    /// priors ///
    var state = statePrior(context);
    var qud = qudPrior();
    var group_interp = groupInterpPrior();
    //////////////
    
    observe(pragmaticSpeaker(state, group_interp, qud, context), utterance);
    
    return state;   
  }});
});

// S2 is given a state, samples an utterance, and factors based on informativity
// to the pragmatic listener (models speaker endorsement in a truth-value judgement)
var alpha2 = 1;
var s2 = cache(function(state, context) {
  return Infer({method: 'enumerate', model: function() {
    var utterance = utterancePrior();
    factor(alpha2 * pragmaticListener(utterance, context).score(state))
    return utterance;
  }});
});
///

var s2prime = cache(function(state, context, group_interp) {
  return Infer({method: 'enumerate', model: function() {
    var utterance = utterancePrior();
    var listenerPost = pragmaticListener(utterance, context);

    var listenerMarg = (group_interp === "within") ?
      marginalize(listenerPost, function(s) {
        return {
          group1: s.group1,
          group2: s.group2
        };
      })
      :
      marginalize(listenerPost, function(s) {
        return {
          across: s.across
        };
      });

    factor(alpha2 * listenerMarg.score(state));
    return utterance
  }});
});

// S2' is more willing to endorse exclusive "most" readings than S2
var state = {
  group1: 2,
  group2: 2,
  across: 0
}

var stateprime = {
  group1: 2,
  group2: 2,
}

viz(s2(state, "classroom.presentto"));
viz(s2prime(stateprime, "classroom.presentto", "within"));
~~~~