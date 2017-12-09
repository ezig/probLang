---
layout: chapter
title: English Reciprocals in RSA
---

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

~~~~
// Two possible interpretations of domain of reciprocity
var group_interp = ["within", "across"];

var groupInterpPrior = function() {
  return categorical([1, 1], group_interp);
}
~~~~

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
~~~~

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
~~~~

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
~~~~

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
~~~~

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
~~~~