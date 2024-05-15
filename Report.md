# Computational Logic Coursework 2024
# Fahd Abdelazim

## Introduction
The aim of this coursework was to extend the logical functionality of the prolexa project. We implemented negation and default rules. This report discusses the implementations of these functionalities. 


## Negation
Negation was implemented to handle the following statement:
```
Every teacher is happy. Donald is not happy. Therefore, Donald is not a teacher.
```
### Changes to Grammar
In order for Prolexa to understand negation it was first necessary to implement the "not" operator, which was defined in the prolexa_grammar.pl file as follows:
```
:- op(900, fy, not).
```
In order to test the example provided we added the definitions of the terms:
```
pred(teacher, 1,[n/teacher]).
pred(happy,   1,[a/happy]).
proper_noun(s,donald) --> [donald].
```
We then defined the negated verb phrase and its different forms:
```
verb_phrase(s,not(M)) --> [is],[not],property(s,M).
verb_phrase(p,not(M)) --> [are],[not],property(p,M).
```
In order for Prolexa to parse sentences with negated terms, we added a sentence defintion. The sentence handles cases where a noun has a negative property ```Donald is not a teacher```.
```
sentence1([(not(L):-true)]) --> proper_noun(N,X),verb_phrase(N,not(X=>L)).
```
To allow for questions to be asked about sentences with negative terms we added two questions. The first question handled queries to identyfy nouns that have a negative property ```who is not happy```. While the second question handles queries about whether a noun has a negative property
```is donald not happy```
```
question1(not(Q)) --> [who],verb_phrase(s,(not(_X=>Q))).
question1(not(Q)) --> [is],proper_noun(N,X),[not],property(N,X=>Q).
```
### Changes to Engine
The prolexa_engine.pl file was updated to extend the ```prove_rb``` meta-interpreter to implement proof by contrapositive to handle negation. This implies that proving that ```A:-B``` is the same as proving ```not(B):-not(A)```.  
```
prove_rb(not(B),Rulebase,P0,P):-
    find_clause((A:-B),Rule,Rulebase),
	prove_rb(not(A),Rulebase,[p(not(B),Rule)|P0],P).
```

### Testing

```
1 ?- prolexa_cli.
prolexa> "tell me everything you know".
*** utterance(tell me everything you know)
*** goal(all_rules(_57700))
*** answer(every human is mortal. peter is human. every teacher is happy. donald is not happy)
every human is mortal. peter is human. every teacher is happy. donald is not happy

prolexa> "who is not happy".
*** utterance(who is not happy)
*** query(not(happy(_60626)))
*** answer(donald is not happy)
donald is not happy

prolexa> "is donald not happy".
*** utterance(is donald not happy)
*** query(not(happy(donald)))
*** answer(donald is not happy)
donald is not happy

prolexa> "explain why donald is not a teacher". 
*** utterance(explain why donald is not a teacher)
*** goal(explain_question(not(teacher(donald)),_62576,_62302))
*** answer(donald is not happy; every teacher is happy; therefore donald is not a teacher)
donald is not happy; every teacher is happy; therefore donald is not a teacher
```

## Default Rules
Default rules were implemented to handle the following statement:
```
Most birds fly except penguins. Tweety is a bird. Therefore, assuming Tweety is not a penguin, Tweety flies.
```
### Changes to Grammar
The term definitions needed to test the example where already defined:
```
pred(bird,    1,[n/bird]).
pred(fly,     1,[v/fly]).
pred(penguin, 1,[n/penguin]).
proper_noun(s,tweety) --> [tweety].
```
We then defined the negated verb phrase and its different forms:
```
verb_phrase(s,not(M)) --> [does], [not],iverb(p,M).
verb_phrase(p,not(M)) --> [do], [not],iverb(p,M).
```
In order for Prolexa to parse sentences with the term except, we added a sentence defintion. The definition allow for the exclusion of certain nouns from a rule.
```
sentence1(C) --> determiner(N,M1,M2,C),noun(N,M1),verb_phrase(N,M2), [except], noun(N,M3), {C=[(C1:-C2,not C3)], M1=(_X2=>C2), M2=(_X1=>C1), M3=(_X3=>C3)}.
```
To allow for questions to be asked about sentences including exclusions from default rules we added a question. The question handles queries about whether a noun is an exception to the rule
```does tweety not fly```
```
question1(not(Q)) --> [does],proper_noun(_,X),[not],verb_phrase(_,X=>Q).
```
### Changes to Engine
The changes extend the implementation of negation in the previous section.The prolexa_engine.pl file was updated to extend the ```prove_rb``` meta-interpreter to implement negation as failure.  
```
prove_rb(not(A),Rulebase,P0,P):-
    prove_rb(A,Rulebase,P0,P), !, fail.
prove_rb(not(_A),_Rulebase,P,P):-!.
```
### Testing
```
prolexa> 'all birds fly except penguins'.
*** utterance(all birds fly except penguins)
*** rule([(fly(_59766):-bird(_59766),not(penguin(_59892)))])
*** answer(I will remember that all birds fly except penguins)
I will remember that all birds fly except penguins

prolexa> 'Tweety is a bird'.
*** utterance(Tweety is a bird)
*** rule([(bird(tweety):-true)])
*** answer(I will remember that Tweety is a bird)
I will remember that Tweety is a bird

prolexa> 'who flies'.
*** utterance(who flies)
*** query(fly(_60370))
*** answer(tweety flies)
tweety flies

prolexa> 'explain why tweety flies'.
*** utterance(explain why tweety flies)
*** goal(explain_question(fly(tweety),_60914,_60734))
*** answer(tweety is a bird; every bird flies except penguin; therefore tweety flies)
tweety is a bird; every bird flies except penguin; therefore tweety flies

prolexa> 'Tweety is a penguin'. 
*** utterance(Tweety is a penguin)
*** rule([(penguin(tweety):-true)])
*** answer(I will remember that Tweety is a penguin)
I will remember that Tweety is a penguin

prolexa> 'does tweety fly'.
*** utterance(does tweety fly)
*** query(fly(tweety))
*** answer(Sorry, I don't think this is the case)
Sorry, I don't think this is the case

prolexa> 'does tweety not fly'. 
*** utterance(does tweety not fly)
*** query(not(fly(tweety)))
*** answer(tweety does not fly)
tweety does not fly
```

## Conclusion
This report demonstrated the implementation of functions to extend the Prolexa project. We highlighted the changes made to implement negation and default rules. While these implementation allow for basic use cases they fall short when more complex logic is presented, for example combining default rules and negation ```some birds do not fly```. Investigating a way to combine them together will help extend the range of questions Prolexa can handle.