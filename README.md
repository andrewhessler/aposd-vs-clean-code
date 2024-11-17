## Introductions

**JOHN:**

Hi (Uncle) Bob! You and I have each written books on software design.
We agree on some things, but there are some pretty big differences of
opinion between my recent book *A Philosophy of Software Design*
(herafter "APOSD") and your classic book *Clean Code*. Thanks for
agreeing to discuss those differences here.

**UB:**

My pleasure John.  Before we begin let me say that I've carefully read through your book and I found it very enjoyable, and full of valuable insights.  There are some things I disagree with you on, such as TDD, and Abstraction-First incrementalism, but overall I enjoyed it a lot.

**JOHN:**

I'd like to discuss three topics with you: method length, comments,
and test-driven development. But before getting into these,
let's start by comparing overall philosophies. When you hear about a
new idea related to software design, how do you decide whether or not
to endorse that idea?

I'll go first. For me, the fundamental goal of software design is
to make it easy to understand and modify the system. I use the term
"complexity" to refer to things that make it hard to understand and
modify a system. The most important contributors
to complexity relate to information:

* How much information must a developer have in their head in order to carry out a task?
* How accessible and obvious is the information that the developer needs?

The more information a developer needs to have, the harder it will be
for them to work on the system. Things get even worse if the required
information isn't obvious. The worst case is when there is a crucial
piece of information hidden in some far-away piece of code
that the developer has never heard of.

When I'm evaluating an idea related to software design, I ask whether
it will reduce complexity. This usually means either reducing the amount
of information a developer has to know, or making the required information
more obvious.

Now over to you: are there general principles that you use when deciding
which ideas to endorse?

**UB:**

I agree with your approach. A discipline or technique should make the job of programmers easier. I would add that the programmer we want to help most is not the author.  The programmer whose job we want to make easier is the programmer who must read and understand the code written by others (or by themself a week later).  Programmers spend far more hours reading code than writing code, so the activity we want to ease is that of reading.

**JOHN:**
I agree: ease of reading is much more important for code than ease of writing.

## Method Length

**JOHN:**

Our first area of disagreement is method length.
On page 34 of *Clean Code* you say "The first rule of functions is that
they should be small. The second rule of functions is that
*they should be smaller than that*." Later on, you say "Functions
should hardly ever be 20 lines long" and suggest that functions
should be "just two, three, or four lines long". On page 35, you
say "Blocks within `if` statements, `else` statements, `while` statements,
and so on should be one line long. Probably that line should be a function
call." I couldn't find anything in *Clean Code* to suggest that a function
could ever be too short.

I agree that dividing up code into relatively small units ("modular design")
is one of the most important ways to reduce the amount of information a
programmer has to keep in their mind at once. The idea, of course, is to take a
complex chunk of functionality and encapsulate it in a separate method
with a simple interface. Developers can then harness the methodality
of the method (or read code that invokes the method) without learning
the details of how the method is implemented; they only need to learn its
interface. The best methods are those that provide a lot of functionality
but have a very simple interface: they replace a large cognitive load
(reading the detailed implementation) with a much smaller
cognitive load (learning the interface). I call these methods "deep".

However, like most ideas in software design, decomposition can be taken too far.
As methods get smaller and smaller there is less and less
benefit to further subdivision.
The amount of functionality hidden behind each interface
drops, while the interfaces often become more complex.
I call these interfaces "shallow": they don't help much in terms of
reducing what the programmer needs to know. Eventually, the point is
reached where someone using the method needs
to understand every aspect of its implementation. Such methods
are usually pointless.

Another problem with decomposing too far is that it tends to
result in *entanglement*. Two methods
are entangled (or "conjoined" in APOSD terminology) if, in order to
understand how one of them works internally, you also need to read the
code of the other. If you've ever found yourself flipping back and forth
between the implementations of two methods as you read code, that's a
red flag that the methods might be entangled. Entangled methods
are hard to read because the information you need to have in your head
at once isn't all in the same place. Entangled methods can usually
be improved by combining them so that all the code is in one place.

The advice in *Clean Code* on method length is so extreme that it encourages
programmers to create teeny-tiny methods that suffer from both shallow
interfaces and entanglement.  Setting arbitrary numerical limits such
as 2-4 lines in a method and a single line in the body of an
`if` or `while` statement exacerbates this problem.

**UB:**

While I do strongly recommend very short functions, I don't think it's fair to say that the book sets arbitrary numerical limits. The 2-4 line functions that you referred to on page 34 were part of the _Sparkle_ applet that Kent Beck and I wrote together in 1999 as an exercise for learning TDD. I thought it was remarkable that most of the functions in that applet were 2-4 lines long because it was a Swing program; and Swing programs tend to have very long methods.

As for setting limits, on page 13 I make clear that although the recommendations in the book have worked well for me and the other authors, they might not work for everyone.  I claimed no final authority, nor even any absolute "rightness". They are offered for consideration.

**JOHN:**

I think these problems will be easiest to understand if we look at
specific code examples. But before we do that, let me ask you, Bob:
do you believe that it's possible for code to be over-decomposed, or
is smaller always better? And, if you believe that over-decomposition
is possible, how do you recognize when it has occurred?

**UB:**

It is certainly possible to over-decompose code.  Here's an example:

	void doSomething() {doTheThing()} // over-decomposed.

In general I like your narrow/deep, wide/shallow dichotomy.  However, I don't think extracting small methods inevitably drives a module towards the wide/shallow end.  In fact, I think the opposite.

In your book you define a module as "deep" if it has a powerful functionality and a narrow interface. To say this differently, you want the aspect ratio (Interface/Functionality) to be small.  

This implies that deepness is relative.  A module that does very little, but that has a trivial interface, still has a small aspect ratio, and is therefore still deep.

Consider a large and deep method that contains three sections.  If I extract those three sections into separate methods the original still has a powerful functionality and a narrow interface.  It is still deep.  It's aspect ratio is unchanged even though it's line count has decreased dramatically.

Now let's consider the three sections prior to their extraction.  Section 1 sets up some local variables for section 2, and some for section 3, and some for both.  Those local variables become an undeclared and implicit interface to sections 2 and 3. 

At first the reader must consider all the local variables when reading sections 2 and 3.  Thus the implicit interface to those sections is wider than necessary.  It is only when the reader had understood a section that he can exclude the superfluous local variables for that section and narrow the interfaces in his mind. 

Extracting those sections causes the interfaces to be explicit and declared.  The reader does not have to understand the functions to understand the interfaces.

The extracted sections are shallower than the original function; because they do less.  They may also have wider interfaces because section 1 split some data elements from it's input. But those interfaces are no wider than the implicit and undeclared interfaces within the unextracted module; and are likely quite a bit narrower.  Thus the aspect ratio of the extracted functions will be no larger than the unextracted sections, but could well be considerably smaller.

If we repeat this process by breaking up each extracted method we create a hiearchy of extracted functions.  Those functions will become shallower and shallower as we descend; but their interfaces will become narrower too as we break the data of the problem up into smaller and smaller parts.  

Therefore, decomposing larger methods into smaller methods does not ipso-facto lead to increasing aspect ratios and a loss of depth.  So long as the programmer is careful to manage the functionality and interface of each extracted method, he can keep the aspect ratios very small.
  
Thus the strategy that I use for deciding how far to take extraction is the old rule that a method should do "*One Thing*".  If I can *meaningfully* extract one method from another, then the original method did more than one thing.  "Meaningfully" means that the extracted functionality can be given a descriptive name; and that it does less than the original method.

**JOHN:**

Unfortunately the One Thing approach will lead to over-decompositon:

 1. The term "one thing" is vague and easy to abuse. For example, if a method has two lines of code, isn't it doing two things?

 2. You haven't provided any useful guardrails to prevent over-decomposition. The example you gave is too extreme to be useful, and the "can it be named" qualification doesn't help: anything can be named.

 3. The One Thing approach is simply wrong in many cases. If two things are closely related, it might well make sense to implement them in a single method. For example, any thread-safe method will first have to acquire a lock, then carry out its function. These are two "things", but they belong in the same method.

**UB:**

I don't consider ease of abuse to be a concern. `If` statements are easy to abuse.  `Switch` statements are easy to abuse.  Assignment statements are easy to abuse.  The fact that something is easy to abuse does not mean that it should be avoided or suppressed.  It simply means people should take appropriate care. There will always be this thing called: _judgment_.

So when faced with this snippet of code in a larger method:

	...
	amountOwed=0;
	totalPoints=0;
	...

It would be poor judgement to extract them as follows, because the extraction is not meaningful.  The implementation is not more deeply detailed than the interface.

	void clearAmountOwed() {
	  amountOwed=0;
	}

	void clearTotalPoints() {
	  totalPoints=0;
	}

However it may be good judgement to extract them as follows because the interface is abstract, and the implemention has deeper detail.

	void clearTotals() {
		amountOwed=0;
		totalPoints=0;
	}

The latter has a nice descriptive name that is abstract enough to be meaningful without being redundant.  And the two lines together are strongly related so as to qualify for doing _one thing_: initialization.

**JOHN:**

Of course anything can be abused. But the best approaches to design
encourage people to do things the right way and discourage abuse.
Unfortunately, the One Thing Rule encourages abuse for the reasons I
gave above.

And of course software designers will need to use judgment: it isn't
possible to provide precise recipes for software design.
But good judgment requires principles and guidance. The
*Clean Code* arguments about decomposition, including the One Thing
Rule, are one-sided. They give strong, concrete, quantitative
advice about when to chop things up, with virtually no guidance for
how to tell you've gone too far. All I could find is a 2-sentence
example on page 36 about Listing 3-3 (which is pretty trivial),
buried in the middle of exhortations to "chop, chop, chop".

One of the reasons I use the deep/shallow characterization is that it
captures both sides of the tradeoff; it will tell you when a decomposition
is good and also when decomposition makes things worse.

**UB:**

You make a good point that I don't talk much, in the book, about how to make the judgement call.  Back in 2008 my concern was breaking the habit of the very large functions that were common in those early days of the web.  I'll have to consider being a bit more balanced in the 2d ed.

Still, if I must err, I'd rather err on the side of extraction.  Extractions can always be inlined if they we judge them to be too decomposed.

**JOHN:**

Coming back to your `clearTotals` example:

* I don't recall seeing the guidance "the implementation must be more deeply
  detailed than the interface" in *Clean Code*; is this something new?
	* **UB:** No, you'll find that page 36: _The Stepdown Rule_.
* The `clearTotals` method seems to contradict the One Thing Rule: the
  variables `amountOwed` and `totalPoints` don't seem particularly related, so
  initializing them both is doing two things, no? You say that both
  statements are performing initialization, which makes it just one thing
  (initialization). Does that mean it would also be okay to have a single
  method that initializes two completely independent objects with nothing in
  common? I suspect not. It feels like you are struggling to create a clean
  framework for applying the One Thing Rule; that makes me think it isn't
  a good rule.
* Without seeing more context I'm skeptical that the `clearTotals`
  method makes sense.

**UB:**

I hope you agree that between these two examples, the former is a bit better.

	public String makeStatement() {
	  clearTotals();
	  return makeHeader() + makeRentalDetails() + makeFooter();
	}

---

	public String makeStatement() {
      amountOwed=0;
      totalPoints=0;
      return makeHeader() + makeRentalDetails() + makeFooter();
	}

**JOHN:**

Well, actually, no. The second example is completely clear and obvious:
I don't see anything to be gained by splitting it up.

I think it will be easier to clarify our differences if we consider
a specific code example. Let's look at the `PrimeGenerator` class from
*Clean Code*, which is Listing 10-8 on pages 145-146. This Java class
generates the first N prime numbers:

	package literatePrimes;

	import java.util.ArrayList;

	public class PrimeGenerator {
	  private static int[] primes;
	  private static ArrayList<Integer> multiplesOfPrimeFactors;

	  protected static int[] generate(int n) {
	    primes = new int[n];
	    multiplesOfPrimeFactors = new ArrayList<Integer>();
	    set2AsFirstPrime();
	    checkOddNumbersForSubsequentPrimes();
	    return primes;
	  }

	  private static void set2AsFirstPrime() {
	    primes[0] = 2;
	    multiplesOfPrimeFactors.add(2);
	  }

	  private static void checkOddNumbersForSubsequentPrimes() {
	    int primeIndex = 1;
	    for (int candidate = 3;
	         primeIndex < primes.length;
	         candidate += 2) {
	      if (isPrime(candidate))
	        primes[primeIndex++] = candidate;
	    }
	  }

	  private static boolean isPrime(int candidate) {
	    if (isLeastRelevantMultipleOfLargerPrimeFactor(candidate)) {
	      multiplesOfPrimeFactors.add(candidate);
	      return false;
	    }
	    return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
	  }

	  private static boolean
	  isLeastRelevantMultipleOfLargerPrimeFactor(int candidate) {
	    int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
	    int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
	    return candidate == leastRelevantMultiple;
	  }

	  private static boolean
	  isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
	    for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
	      if (isMultipleOfNthPrimeFactor(candidate, n))
	        return false;
	    }
	    return true;
	  }

	  private static boolean
	  isMultipleOfNthPrimeFactor(int candidate, int n) {
	    return candidate ==
	      smallestOddNthMultipleNotLessThanCandidate(candidate, n);
	  }

	  private static int
	  smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
	    int multiple = multiplesOfPrimeFactors.get(n);
	    while (multiple < candidate)
	      multiple += 2 * primes[n];
	    multiplesOfPrimeFactors.set(n, multiple);
	    return multiple;
	  }
	}

Before we dive into this code, I'd encourage everyone reading
this article to take time to read over the code and draw your own conclusions
about it. Did you find the code easy to understand? If so, why? If not, what
makes it complex?

Also, Bob, can you confirm that you stand by this code (i.e. the code
properly exemplifies the design philosophy of *Clean Code* and this
is the way you believe the code should appear if it were used in
production)?

**UB:**

Ah, yes.  The `PrimeGenerator`.  This code comes from the 1982 paper on [*Literate Programming*](https://www.cs.tufts.edu/~nr/cs257/archive/literate-programming/01-knuth-lp.pdf) written by Donald Knuth.  The program was originally written in Pascal, and was automatically generated by Knuth's WEB system.  I translated it to Java.

Of course this code was never meant for production because it is a pedagogical exercise.  It appears in a chapter named *Classes*.  I used it as a demonstration for how to partition a large and messy legacy function into a set of smaller classes and methods.

The lesson of the chapter is that a large function can contain many different sections of code that are better extracted into independent classes.  My recommendation was that my readers should attempt such extractions when they see large functions.

In the chapter I extracted three classes from that function: `PrimePrinter`, `RowColumnPagePrinter` and `PrimeGenerator`.

Once the classes were extracted, the `PrimeGenerator` class had the following code (which I did not publish in the book.)

	public class PrimeGenerator {
	  protected static int[] generate(int n) {
	    int[] p = new int[n];
	    ArrayList<Integer> mult = new ArrayList<Integer>();
	    p[0] = 2;
	    mult.add(2);
	    int k = 1;
	    for (int j = 3; k < p.length; j += 2) {
	      boolean jprime = false;
	      int ord = mult.size();
	      int square = p[ord] * p[ord];
	      if (j == square) {
	        mult.add(j);
	      } else {
	        jprime=true;
	        for (int mi = 1; mi < ord; mi++) {
	          int m = mult.get(mi);
	          while (m < j)
	            m += 2 * p[mi];
	          mult.set(mi, m);
	          if (j == m) {
	            jprime = false;
	            break;
	          }
	        }
	      }
	      if (jprime)
	        p[k++] = j;
	    }
	    return p;
	  }
	}

To be fair to Knuth, this is not the program as he wrote it but rather as it was output by his WEB tool.  The variable names, however, were Knuth's.  I used this code because it made a great starting place for breaking up a big function into many smaller functions and classes.

My goal was not to describe how to generate prime numbers.  I wanted my readers to see how large methods, that violate the Single Responsibility Principle, can be broken down into a few smaller well-named classes containing a few smaller well-named methods.

It was _not_ my intent, in that chapter, to teach my readers how to write the most optimal prime number generator.  That was the last thing on my mind, and the last thing I wanted on theirs.

**JOHN:**

Agreed; for this discussion I assume we will take the algorithm as a given,
and focus on the cleanest possible way to implement that algorithm.

I think there are many design problems with `PrimeGenerator`, but for now I'll
focus on method length. The code is chopped up so much (8 teeny-tiny methods)
that it's difficult to read. For starters, consider the
`isNotMultipleOfAnyPreviousPrimeFactor` method. This method invokes
`isMultipleOfNthPrimeFactor`, which invokes
`smallestOddNthMultipleNotLessThanCandidate`. These methods are shallow
and entangled:
in order to understand
`isNot...` you have to read the other two
methods and load all of that code into your mind at once. For example,
`isNot...` has side effects (it modifies `multiplesOfPrimeFactors`) but
you can't see that unless you read all three methods.

**UB:**

I agree.  Though I would not have agreed eighteen years ago when I was in the throes of refactoring this.  At the time the names and structure made perfect sense to me.  They make sense to me now, too -- but that's because I once again understand the algorithm.  When I returned to the algorithm a few days ago, I also struggled with the names and structure.  Once you understand the algorithm the names make perfect sense, and that understanding breaks the entanglement.

So, a good critique of those names is that they depend, to some extent, upon gaining some understanding of the algorithm.

**JOHN:**

Yes, if code no longer makes sense to its writer when the writer returns to the
code later, that's a problem.

**UB:**

Would that we had such a crystal ball that we could see how our future selves would react to our current creations.  ;-)

**JOHN:**

And, those names are highly problematic even if you understand the code;
we'll talk about that a bit later, when discussing comments.

**JOHN:**

Going back to my introductory remarks about complexity, splitting up
`isNot...` into three methods doesn't reduce the amount of information
you have to keep in your mind. It just spreads it out, so it isn't as
obvious that you need to read all three methods together. And, it's harder
to see the overall structure of the code because it's split up: readers have
to flip back and forth between the methods, effectively reconstructing a
monolithic version in their minds. Because the pieces are all related,
this code will be easiest to understand if it's all together in one place.

**UB:**

I disagree.  Here is `isNotMultipleOfAnyPreviousPrimeFactor`.

	  private static boolean
	  isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
	    for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
	      if (isMultipleOfNthPrimeFactor(candidate, n))
	        return false;
	    }
	    return true;
	  }

If you trust the `isMultipleOfNthPrimeFactor` method, then this method stands alone quite nicely.  I mean we loop through all n previous primes and see if the candidate is a multiple.  That's pretty straight forward.

Now it would be fair to ask the question how we determine whether the candidate is a multiple, and in that case you'd want to inspect the `isMultiple...` method.

**JOHN:**

This code does appear to be simple and obvious.
Unfortunately, this appearance is deceiving.
If a reader trusts the name `isMultipleOfNthPrimeFactor` (which suggests
a predicate with no side effects) and doesn't bother to read its code, they
will not realize that it has side effects, and that the side effects
create a constraint on the `candidate` argument to `isNot...`
(it must be monotonically non-decreasing from invocation
to invocation). To understand these behaviors, you have to
read both `isMultiple...` and `smallestOdd...`. The current decomposition
hides this important information from the reader.

If there is one thing more likely to result in bugs than not understanding code,
it's thinking you understand it when you don't.

**UB:**

That's a valid concern.  However, it is tempered by the fact that the functions are presented in the order they are called.  Thus we can expect that the reader has already seen the main loop and understands that candidate increases by two each iteration.

The side effect buried down in `smallestOddNth...` is a bit more problematic. Now that you've pointed it out I don't like it much.  Still, that side effect should not confound the basic understanding of `isNot...`.

In general, if you trust the names of the methods being called then understanding the caller does not require understanding the callee.  For example:

	for (Employee e : employees)
	  if (e.shouldPayToday())
		  e.pay();

This would not be made more understandable if we replaced those two method calls with the their implementations.  Such a replacement would simply obscure the intent.

**JOHN:**

This example works because the called methods are relatively independent of
the parent. Unfortunately that is not the case for `isNot...`.

In fact, `isNot...` is not only entangled with the methods it calls, it's also
entangled with its callers. `isNot...` only works if it is invoked in
a loop where `candidate` increases monotonically. To convince yourself
that it works, you have to find the code that invokes `isNot...` and
make sure that `candidate` never decreases from one call to the next.
Separating `isNot...` from the loop that invokes it makes it harder
for readers to convince themselves that it works.

**UB:**

Which, as I said before, is why the methods are ordered the way they are.  I expect that by the time you get to `isNot...` you've already read `checkOddNumbersForSubsequentPrimes` and know that `candidate` increases by twos.

**JOHN:**

Let's discuss this briefly, because it's another area where I
disagree with *Clean Code*. If methods are entangled, there is no
clever ordering of the method definitions that will fix the problem.

In this particular situation two other methods intervene between the
loop in `checkOdd...` and `isNot...`, so readers will have forgotten
the loop context before they get to `isNot...`. Furthermore, the actual
code that creates a dependency on the loop isn't in `isNot...`: it's in
`smallestOdd...`, which is even farther away from `checkOdd...`.

**UB:**

I sincerely doubt anyone is going to forget that `candidate` is being increased by twos.  It's a pretty obvious way to avoid waste.

**JOHN:**

In my opening remarks I talked about how it's important to reduce the
amount of information people have to keep in their minds at once.
In this situation, readers have to remember that loop while they read
four intervening methods that are mostly unrelated. You apparently think
this will be easy and natural (I disagree). But it's even worse than
that. There is no indication which parts of `checkOdd...` will be important
later on, so the only safe approach is to remember *everything*, from *every*
method, until you have encountered every other method that could possibly
descend from it. And, to make the connection between the pieces, readers
must also reconstruct the call graph to notice that, even through
4 layers of method call, the code in `smallestOdd...` places constraints
on the loop in `checkOdd...`. This is an unreasonable cognitive burden to
place on readers.

If two pieces of code are tightly related, the solution is to bring
them together. Separating the pieces, even in physically adjacent methods,
makes the code harder to understand.

To me, all of the methods in `PrimeGenerator` are entangled: in order to
understand the class I had to load all of them into my mind
at once. I was constantly flipping back and forth between the methods
as I read the code. This is a red flag indicating
that the code has been over-decomposed.

Bob, can you help me understand why you divided the code into such
tiny methods?
Is there some benefit to having so many methods that I have missed?

**UB:**

I think you and I are just going to disagree on this.  In general I believe in the principle of small well-named methods and the separation of concerns. Generally speaking if you can break a large method into several well-named smaller methods with different concerns, and by doing so expose their interfaces, and the high level functional decomposition, then that's a good thing.

* Looping over the odd numbers is one concern.
* Determining primality is another.
* Marking off the multiples of primes is yet another.

It seems to me that separating and naming those concerns helps to expose the way the algorithm works.

In your solution, below, you break the algorithm up in a similar way.  However, instead of separating the concerns into functions, you separate them into sections with comments above them.

You mentioned that in my solution readers will have to keep the loop context in mind while reading the other functions.  I suggest that in your solution, readers will have to keep the loop context in mind while reading your explanatory comments.  They may have to "flip back and forth" between the sections in order to establish their understanding.

Now perhaps you are concerned that in my solution the "flipping" is a longer distance (in lines) than in yours.  I'm not sure that's a significant point since they all fit on the same screen (at least they do on my screen) and the landmarks are pretty obvious.

**JOHN:**

It sounds like it's time to wrap up this section. Is this a reasonable
summary of where we agree and disagree?

* We agree that modular design is a good thing.

* We agree that it is possible to over-decompose, and that *Clean Code*
  doesn't provide much guidance on how to recognize over-decomposition.

* We disagree on how far to decompose: you recommend decomposing
  code into much smaller units than I do.

* You believe that the One Thing Rule, applied with judgment, will
  lead to appropriate decompositions. I believe it lacks guardrails
  and will lead to over-decomposition.

* You believe that `PrimeGenerator` provides a good example of how to
  decompose code and that it is easy to read.
  I believe that `PrimeGenerator` is a really bad decomposition
  (sorry to be blunt) and that the problems stem from following the
  advice of *Clean Code*: it is way over-decomposed and as a result is
  difficult to read.
>`PrimeGenerator` is part of a good example of how to break a large function into classes -- which was the point of that chapter.  `PrimeGenerator` itself was not meant to be an exemplar of function decomposition.  

* You believe that `PrimeGenerator` separates concerns. I believe that
  the code appears separated on the surface, but in fact it is entangled.

* You believe that when someone reads a class, it's reasonable for them
  to load all of the code of the class into their head as they go, so
  that in each method they remember what they've read in previous methods.
  I believe that this creates an unreasonable cognitive load: it should
  be possible to read each method relatively independently, without having
  to remember the implementations of other methods.
>No. I believe that is true of a method, but not of a class.  As you read a method, whether the elements are extracted or not, you have to keep the state of the execution in mind.  When you read a class, you should not have to keep the state of the execution of each method in mind. 

* You believe that ordering the methods in a class is an effective way
  to manage dependencies between them. I believe that ordering can
  sometimes improve readability a bit, but if methods are entangled then
  ordering won't fix the problem.
>I agree with both.  Ordering can be effective; but it is not a cure-all.

* You believe that it's OK for (private?) methods in class to be entangled,
  where one method cannot be fully understood without considering code
  in other methods. I believe that entanglement is a red flag for bad
  design; if two pieces of code are dependent, the code will be more
  readable if the pieces are right next to each other in a single method.
> Yes, the extent to which classes have fields suggests that the methods within them are somewhat entangled. 

## Comments

**JOHN:**

Let's move on to the second area of disagreement: comments. In my opinion,
the *Clean Code* approach to commenting results in code with
inadequate documentation, which increases the cost of software development.
I'm sure you disagree, so let's discuss.

Here is what *Clean Code* says about comments (page 54):

> The proper use of comments is to compensate for our failure to express
  ourselves in code. Note that I use the word failure. I meant it.
  Comments are always failures. We must have them because we cannot always
  figure out how to express ourselves without them, but their use is not
  a cause for celebration... Every time you write a comment, you should
  grimace and feel the failure of your ability of expression.

I have to be honest: I was horrified when I first read this text, and it
still makes me cringe. This stigmatizes writing comments. Junior developers
will think "if I write comments, people may think I've failed, so the
safest thing is to write no comments."

**UB:**

That chapter begins with these words:
>*Nothing can be quite so helpful as a well placed comment.*

It goes on to say that comments are a *necessary* evil.

The only way a reader could infer that they should write no comments is if they hadn't actually read the chapter.  The chapter walks through a series of comments, some bad, some good.

**JOHN:**

*Clean Code* focuses a lot more on the "evil" aspects of comments than the
"necessary" aspects. The sentence you quoted above is followed by two
sentences criticizing comments. Chapter 4 spends 4 pages talking about good
comments, followed by 15 pages talking about bad comments. There are snubs
like "the only truly good comment is the comment you found a way
not to write". And "Comments are always failures" is so catchy
that it's the one thing readers are most likely to remember from the
chapter.

**UB:**

The difference in page count is because there are just a few ways to write good comments, and so many more ways to write bad ones.

**JOHN:**

I disagree; this illustrates your bias against comments. If you look at
Chapter 13 of APOSD, it finds a lot more
constructive ways to use comments than *Clean Code*. And if you compare
the tone of Chapter 13 of APOSD with Chapter 4 of *Clean Code*, the hostility
of *Clean Code* towards comments becomes pretty clear.

**UB:**
I'll leave you to balance that last comment with the initial statement, and the final example, in the _Comments_ chapter. They do not communicate "hostility".

I'm not hostile to comments in general.  I _am_ very hostile to gratuitous comments.

You and I likely both survived through a time when comments were absolutely necessary.  In the '70s and '80s I was an assembly language programmer.  I also wrote a bit of FORTRAN. Programs in those languages that had no comments were impenetrable.

As a result it became conventional wisdom to write comments by default.  And, indeed, computer science students were taught to write comments uncritically.  Comments became _pure good_.

In the book I decided to fight that mindset.  Comments can be _really bad_ as well as good.

**JOHN:**

I don't agree that comments are less necessary today than they were
40 years ago.

Comments are crucially important and add enormous value to software.
The problem is that there is a lot of important information that simply
cannot be expressed in code. By adding comments to fill in this missing
information, developers make code dramatically easier to read.
This is not a "failure of their ability to express themselves", as you
put it.

**UB:**

It's very true that there is important information that is not, or cannot be, expresssed in code.  That's a failure.  A failure of our languages, or of our ability to use them to express ourselves.  In every case a comment is a failure of our ability to use our languages to express our intent.

And we fail at that very frequently, and so comments are a necessary evil -- or, if you prefer, _an unfortunate necessity_.  If we had the perfect programming language (TM) we would never write another comment.

**JOHN:**

I don't agree that a perfect programming language would
eliminate the need for comments. Comments and code serve very different
purposes, so it's not obvious to me that we should use the same
language for both. In my experience, English works quite well
as a language for comments.
Why do you feel that information about a program should
be expressed entirely in code, rather than using a combination of code
and English?

**UB:**

I bemoan the fact that we must sometimes use a human language instead of a programming language.  Human languages are imprecise and full of ambiguities. Using a human language to describe something as precise as a program is very hard, and fraught with many opportunities for error and inadvertent misinformation.

**JOHN:**

I agree that English isn't always as precise as code, but it can still be
used in precise ways and comments typically don't need the same
degree of precision as code.
Comments often contain qualitative information such
as *why* something is being done, or the overall idea of something.
English works better for these than code because it is a more
expressive language.

Are you concerned that comments will be incorrect or
misleading and that this will slow down software development?
I often hear people complain about stale comments (usually as an excuse
for writing no comments at all) but
I have not found them be a significant problem
over my career. Incorrect comments do happen, but I don't encounter them
very often and when I do, they rarely cost me much time. In contrast, I waste
*enormous* amounts of time because of inadequate documentation; it's not
unusual for me to spend 50-80% of my development time wading through
code to figure out things that would be obvious if the code was properly
commented.

**UB:**

You and I have had some very different experiences.

I have certainly been helped by well placed comments.  I have also, just as certainly, (and within this very document) been distracted and confused by a comment that was incorrect, misplaced, gratuitous, or otherwise just plain bad.

**JOHN:**

I invite everyone reading this article to ask yourself the following questions:

* How much does your software development speed suffer because of
  incorrect comments?
* How much does your software development speed suffer because of
  missing comments?

For me the cost of missing comments is easily 10-100x the cost of incorrect
comments. That is why I cringe when I see things in *Clean Code* that
discourage people from writing comments.

Let's consider the `PrimeGenerator` class. There is not a single comment
in that code; does this seem appropriate to you?

**UB:**

I think it was appropriate for the purpose for which I wrote it. It was created to make the point that large methods can be broken down into smaller classes containing smaller methods. Adding lots of explanatory comments would have detracted from that point.

I agree, however, that the commenting style in Listing 4-8 is generally more appropriate.

**JOHN:**

I disagree that adding comments would have distracted from your point,
but let's not argue that. Instead, let's discuss what comments this code
*should* have if it were used in production. I will make some suggestions
and you can agree or disagree.

For starters, let's discuss your use of megasyllabic names like
`isLeastRelevantMultipleOfLargerPrimeFactor`.  My understanding is that
you advocate using names like this instead of using shorter names
augmented with descriptive comments: you're effectively moving the
comments into code. To me, this approach is problematic:

* Long names are awkward. Developers effectively have to retype
  the documentation for a method every time they invoke it, and the long
  names waste horizontal space and trigger line wraps in the code. The names are
  also awkward to read: my mind wants to parse every syllable every time
  I read it, which slows me down. Notice that both you and I resorted to
  abbreviating names in this discussion: that's an indication that
  the long names are awkward and unhepful.
* The names are hard to parse and don't convey information as effectively
  as a comment.
  When students read `PrimeGenerator` one of the first things they
  complain about is the long names (students can't make sense of them).
  For example, the name above is
  vague and cryptic: what does "least relevant" mean, and what is a
  "larger prime factor"? Even with a complete understanding of the code in
  the method, it's hard for me to make sense of the name.  If this name
  is going to eliminate the need for a comment, it needs to be even longer.

In my opinion, the traditional approach of using shorter names with
descriptive comments is more convenient and conveys the required information
more effectively. What advantage is there in the approach you advocate?

**UB:**

"_Megasyllabic_": Great word!

I like my method names to be sentence fragments that fit nicely with keywords and assignment statements.  It makes the code a bit more natural to read.

	if (isTooHot)
	  cooler.turnOn();

I also follow a simple rule about the length of names.  The larger the scope of a function, the shorter its name should be.  The private functions I extracted in this case live in very small scopes, and so have longish names.  Functions like this are typically called from only one place, so there is no burden on the programmer to remember a long name for another call.

**JOHN:**

Names like `isTooHot` are totally fine by me.
My concerns are about names like `isLeastRelevantMultipleOfLargerPrimeFactor`.

It's interesting that as functions get smaller and narrower, you recommend
longer names.
What this says to me is that the interfaces for those functions are
more complex, so it takes more words to describe them. Earlier in
this discussion I said that when systems are decomposed into tiny functions
the interfaces tend to get more complex. You disagreed at the time,
but your recommendation here seems to support my view.

**UB:**
It's not the functions that get smaller, it's the scope that gets smaller.  A private function has a smaller scope than the public function that calls it.  A function called by that private function has an even smaller scope.  As we descend in scope, we also descend in situational detail.  Describing such detail often requires a long name, or a long comment.  I prefer to use a name.

As for long names being hard to parse, that's a matter of practice.  Code is full things that take practice to get used to.

**JOHN:**

I don't accept this. Code may be full of things that take practice to get used
to, but that's a problem, not an excuse.
Approaches that require more practice are worse than
those that require less.
If it's going to take a lot of work to get comfortable with the long names
then there better be some compensating benefit; so far I'm not seeing that.
It's not even clear to me that practice will make the names easier to
digest.

**UB:**

As for the meaning of "leastRelevant", that's a much larger problem that you and I will encounter shortly.  It has to do with the intimacy that the author has with the solution, and the reader's lack of that intimacy.

**JOHN:**

You still haven't answererd my question: why is it better to use super-long names
rather than shorter names augmented with descriptive comments?

**UB:**

It's a matter of preference for me.  I prefer long names to comments.  I don't trust comments to be maintained, nor do I trust that they will be read.  Have you ever noticed that many IDEs paint comments in light grey so that they can be easily ignored?  It's harder to ignore a name than a comment.  

(BTW, I have my IDE paint comments in bright fire-engine red)   

**JOHN:**

Now that we've discussed the specific issue of comments vs. long method
names, let's talk about comments in general. I think are two major reasons
why comments are needed. The first reason for comments is abstraction.
Simply put, without comments there is no way to have abstraction or modularity.

Abstraction is one of the most important components of good software design.
I define an abstraction as "a simplified way of thinking about something
that omits unimportant details." The most obvious example of an abstraction
is a method. It should be possible to use a method without reading its code.
The way we achieve this is by writing a header comment that describes
the method's *interface* (all the information someone needs in order
to invoke the method). If the method is well designed, the interface will be
much simpler than the code of the method (it omits implementation details),
so the comments reduce the amount of information people must have in
their heads.

**UB:**

Long ago, in a 1995 book, I defined abstraction as:
>*The amplification of the essential and the elimination of the irrelevant.*

I certainly agree that abstraction is of importance to good software design.  I also agree that well placed comments can enhance the ability of readers to understand the abstractions we are attempting to employ.  I disagree that comments are the _only_, or even the _best_, way to understand those abstractions.  But sometimes they are the only option.

But consider:

	addSongToLibrary(String title, String[] authors, int durationInSeconds);

This seems like a very nice abstraction to me, and I cannot imagine how a comment might improve it.

**JOHN:**

Our definitions of abstraction are very similar; that's good to see.
However, the `addSongToLibrary` declaration is not (yet) a good abstraction
because it omits information
that is essential. In order to use `addSongToLibrary`, developers
will need answers to the following questions:

* Is there any expected format for an author string, such as "LastName, FirstName"?
* Are the authors expected to be in alphabetical order? If not, is the order
  significant in some other way?
* What happens if there is already a song in the library with the given title
  but different authors? Is it replaced with the new one, or will the library
  keep multiple songs with the same title?
* How is the library stored (e.g. is it entirely in memory? saved on disk?)?
  If this information is documented somewhere else, such as the
  overall class documentation, then it need not be repeated here.

Sometimes the signature of a method (names and types of the method, its
arguments, and its return value) contains all the information
needed to use it, but this is pretty rare. Just skim through the documentation
for your favorite library package: in how many cases could you understand how
to use a method with only its signature?

**UB:**

Yes, there are times when the signature of a method is an incomplete abstraction and a comment is required.  There are other times when the signature tells you everything you want to know.  It seems to me that we should try to create more of the latter kind and avoid the former where possible.

**JOHN:**

Let's consider an example from `PrimeGenerator`: the `isMultipleOfNthPrimeFactor`
method. When someone reading the code encounters the call to `isMultiple...`
in `isNot...` they need to understand enough about how `isMultiple...` works
in order to see how it fits into the code of `isNot...`.
The method name does not fully document the interface, so if there
is no header comment then readers will have to read the code of `isMultiple`.
This will force readers to load more information into their
heads, which makes it harder to work in the code.

Here is my first attempt at a header comment for `isMultiple`:

```
    /**
     * Returns true if candidate is a multiple of primes[n], false otherwise.
     * May modify multiplesOfPrimeFactors[n].
     * @param candidate
     *      Number being tested for primality; must be at least as
     *      large as any value passed to this method in the past.
     * @param n
     *      Selects a prime number to test against; must be
     *      <= multiplesOfPrimeFactors.size().
     */
```

What do you think of this?

**UB:**

I think it's accurate.  I wouldn't delete it if I encountered it.  I don't think it should be a javadoc.

The first sentence is redundant with the name `isMultipleOfNthPrimeFactor` and so could be deleted.  The warning of the side effect is useful.

**JOHN:**

I agree that the first sentence is largely redundant with the name,
and I debated with myself about whether to keep it. I decided to keep it
because I think it is a bit more precise than the name; it's also easier
to read. You propose to eliminate the redundancy between the comment and
the method name by dropping the comment; I would eliminate the redundancy by
shortening the method name.

By the way, you complained earlier about comments being less precise than
code, but in this case the comment is *more* precise (the method
name can't include text like `primes[n]`).

**UB:**

The name 'candidate' is synonymous with "Number being tested for primality".

The constraints on the arugments could be written as executable preconditions following Bertrand Meyer's idea of _Design by Contract_ used in the Eiffel language.

	require
		candidate >= last.candidate and
		n <- multipleOfPrimeFactors.size()

In Eiffel these could be turned off during production.

Eiffel is an example of a language that diminishes the need for comments like that.

**JOHN:**

Using the Eiffel language isn't an option for this example; are you suggesting
that the precondition should be written in Eiffel anyway? To me, the Eiffel
description is harder to read and less precise than a comment. For example,
it's unclear what "last" means in the Eiffel precondition. This is
another example where you made information less precise by moving it from a
comment to code.

**UB:**

In the end, however, all those words are just going to have to sit in my brain 
until I understand why they are there.  I'm also going to have to worry if 
they are accurate.  So I'm going to have to read the code to understand and 
validate the comment.

**JOHN:**

There is a fundamental issue that we need to address here: can comments
generally be trusted? If you are unwilling to trust comments, and
insist on reading the code to validate each comment, then
comments are pointless. If this is your view, please say so. In that case,
this entire
discussion of comments will reduce to a couple of paragraphs where you
state this opinion, I disagree, and we are done. Note that this
would have the following implications:
* Those long method names you recommend are no more trustworthy than
  comments, so they can't be trusted either.
* In order to understand a method, the reader will have to read
  the complete text of every method it invokes, recursively. In order to
  read the main program of an application I must read all of the code of
  the entire application. I think readers will find this approach absurd.

If you agree that comments can generally be trusted, then can you delete
your comment above, plus this comment and any others from you that are based
on distrust of comments?

**UB:**

My trust in comments is not a binary thing.  I read them if they are there; but
I don't implicitly trust then.  The more gratuitous I feel the author was, or the less adept at english the author is, the less I trust the comments.

As I said above, our IDEs tend to paint comments in an ignorable color.  I have my IDE paint comments in bright fire engine red because when I write a comment I intend for it to be read.

By the same token I use long names as a subsitute for comments because I intend for those long names to be read; and it is very hard for a programmer to ignore names.

**JOHN:**

I mentioned earlier that there are two general reasons why comments are
needed. So far we've been discussing the first reason (abstraction).
The second general reason for comments is for important information
that is not obvious from the code. The algorithm in `PrimeGenerator`
is very non-obvious, so quite a few comments are needed to help readers
understand what is going on and why. Most of the algorithm's complexity
arises because it is designed to compute primes efficiently:

* The algorithm goes out of its way to avoid divisions, which were quite
  expensive when Knuth wrote his original version (they aren't that expensive
  nowadays).

* The first multiple for each new prime number is computed by squaring the
  prime, rather than multiplying it by 3. This is mysterious: why is it safe
  to skip the intervening odd multiples? Furthermore, it might seem that this
  optimization only has a small impact on performance, but in fact it makes an
  *enormous* difference (orders of magnitude). Using the square has the
  side-effect that when
  testing a candidate, only primes up to the square root of the
  candidate are tested. If 3x were used as the initial multiple, primes
  within a factor of 3 of the candidate would be tested; that's a *lot*
  more tests.
  This implication of using the square is so non-obvious that I only realized
  it while preparing material for this discussion; it never occurred to me in
  the many times I have discussed the code with students.

Neither of these issues is obvious from the code; without
comments, readers are left to figure them out on their own. The students
in my class are generally unable to figure out either of them in the
30 minutes I give them, but I think that comments would have
allowed them to understand in a few minutes. Going back to my
introductory remarks, this is an example where information is important,
so it needs to be made available.

Do you agree that there should be comments to explain each of these
two issues?

**UB:**

I agree that the algorithm is subtle.  Setting the first prime multiple as the square of the prime was deeply mysterious at first.  I had to go on an hour long bike ride to understand it.

Would a comment help?  Perhaps.  However, my guess is that no one who has been reading our conversation has been helped by it, because you and I are now too intimate with the solution.  You and I can talk about that solution using words that fit into that intimacy; but our readers likely do not yet enjoy that fit.

One solution is to paint a picture -- being worth a thousand words.  Here's my attempt.

	                                                                X
	                                                    1111111111111111111111111
	       1111122222333334444455555666667777788888999990000011111222223333344444
	   35791357913579135791357913579135791357913579135791357913579135791357913579
	   !!! !! !! !  !!  ! !! !  !  !!  ! !!  ! !  !   ! !! !! !
	 3 |||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-||-
	 5 |||||||||||-||||-||||-||||-||||-||||-||||-||||-||||-||||-||||-
	 7 |||||||||||||||||||||||-||||||-||||||-||||||-||||||-||||||-||||||-
	11 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||-||||||||||-
	13 ||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
	...
	113||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||

I expect that our readers will have to stare at this for some time, and also look at the code.  But then there will be a _click_ in their brains and they'll say "Ohhh!  Yes!  I see it now!"

**JOHN:**

I found this diagram extremely hard to understand.
It begs for supplemental English text to explain the basic ideas being
presented. Even the syntax is non-obvious: what does
`1111111111111111111111111` mean?

Maybe we have a fundamental difference of philosophy here. I get the sense
that you are happy giving readers a few clues and leaving it to them to put
the clues together. Perhaps you don't mind if people have to stare at something
for a while to figure it out? I don't agree with this approach: it results
in wasted time, misunderstandings, and bugs.
I think software should be totally *obvious*, where readers don't need to
be clever or "stare at this for some time" to figure things out.
Suffering followed by catharsis is great for Greek tragedies, but not
for reading code. Every question
a reader might have should be naturally answered, either in the code or
in comments. Key ideas and important conclusions should be stated explicitly,
not left for the reader to deduce. Ideally, even if a reader is in a hurry
and doesn't read the code very carefully, their first guesses about how
things work (and why) should be correct. To me, that's clean code.

**UB:**
I don't disagree with your sentiment.  Good clean code should be as easy as possible to understand.  I want to give my readers as many clues as possible so that the code is intuitive to read.  

That's the goal.  As we are about to see, that can be a tough goal to achieve.

## John's Rewrite of PrimeGenerator

**JOHN:**

I mentioned that I ask the students in my software design class to rewrite
`PrimeGenerator` to fix all of its design problems. Here is my rewrite:

	package literatePrimes;

	import java.util.ArrayList;

	public class PrimeGenerator2 {

	    /**
	     * Computes the first prime numbers; the return value contains the
	     * computed primes, in increasing order of size.
	     * @param n
	     *      How many prime numbers to compute.
	     */
	    public static int[] generate(int n) {
	        int[] primes = new int[n];

	        // Used to test efficiently (without division) whether a candidate
	        // is a multiple of a previously-encountered prime number. Each entry
	        // here contains an odd multiple of the corresponding entry in
	        // primes. Entries increase monotonically.
	        int[] multiples = new int[n];

	        // Index of the last value in multiples that we need to consider
	        // when testing candidates (all elements after this are greater
	        // than our current candidate, so they don't need to be considered).
	        int lastMultiple = 0;

	        // Number of valid entries in primes.
	        int primesFound = 1;

	        primes[0] = 2;
	        multiples[0] = 4;

	        // Each iteration through this loop considers one candidate; skip
	        // the even numbers, since they can't be prime.
	        candidates: for (int candidate = 3; primesFound < n; candidate += 2) {
	            if (candidate >= multiples[lastMultiple]) {
	                lastMultiple++;
	            }

	            // Each iteration of this loop tests the candidate against one
	            // potential prime factor. Skip the first factor (2) since we
	            // only consider odd candidates.
	            for (int i = 1; i <= lastMultiple; i++) {
	                while (multiples[i] < candidate) {
	                    multiples[i] += 2*primes[i];
	                }
	                if (multiples[i] == candidate) {
	                    continue candidates;
	                }
	            }
	            primes[primesFound] = candidate;

	            // Start with the prime's square here, rather than 3x the prime.
	            // This saves time and is safe because all of the intervening
	            // multiples will be detected by smaller prime numbers. As an
	            // example, consider the prime 7: the value in multiples will
	            // start at 49; 21 will be ruled out as a multiple of 3, and
	            // 35 will be ruled out as a multiple of 5, so 49 is the first
	            // multiple that won't be ruled out by a smaller prime.
	            multiples[primesFound] = candidate*candidate;
	            primesFound++;
	        }
	        return primes;
	    }
	}


Everyone can read this and decide for themselves whether they think
it is easier to understand than the original. I'd like to mention a
couple of overall things:

* There is only one method. I didn't subdivide it because I felt the method already divides naturally into pieces that are distinct and understandable. It didn't seem to me that pulling out methods would improve readability significantly. When students rewrite the code, they typically have 2 or 3 methods, and those are usually OK too.
* There are a *lot* of comments. It's extremely rare for me to write code with this density of comments. Most methods I write have no comments in the body, just a header comment describing the interface. But this code is subtle and tricky, so it needs a lot of comments to make the subtleties clear to readers. Even with all this additional explanatory material my version is a bit shorter than the original (65 lines vs. 70).

**UB:**

I presume this is a complete rewrite.  My guess is that you worked to understand the algorithm from Clean Code and then wrote this from scratch.  If that's so, then fair enough.

In _Clean Code_ I *refactored* Knuth's algorithm in order to give it a little structure.  That's not the same as a complete rewrite.

Having said that, your version is much better than either Knuth's or mine.  I could not have used it in the chapter I was writing *because* it is still in a single method, and I needed to show a more granular partioning.

I wrote that chapter 18 years ago, so it's been a long time since I saw and understood this algorithm.  When I first saw your challenge I thought: "Oh, I can figure out my own code!"  But, no.  I could see all the moving parts, but I could not figure out why those moving parts generated a list of prime numbers.

So then I looked at your code.  I had the same problem.  I could see all the moving parts, all with comments, but I still could not figure out why those moving parts generated a list of prime numbers.

Figuring that out required a lot of staring at the ceiling, closing my eyes, visualizing, and riding my bike.

Among the problems I had were the comments you wrote.  Let's take them one at a time.

	    /**
	     * Computes the first prime numbers; the return value contains the
	     * computed primes, in increasing order of size.
	     * @param n
	     *      How many prime numbers to compute.
	     */
	    public static int[] generate(int n) {

It seems to me that this would be better as:

	public static int[] generateNPrimeNumbers(int n) {

or if you must:

	//Return the first n prime numbers
	public static int[] generate(int n) {

I'm not opposed to Javadocs as a rule; but I write them only when absolutely necessary. I also have an aversion for descriptions and `@param` statements that are perfectly obvious from the method signature.

The next comment cost me a good 20 minutes of puzzling things out.

	        // Used to test efficiently (without division) whether a candidate
	        // is a multiple of a previously-encountered prime number. Each entry
	        // here contains an odd multiple of the corresponding entry in
	        // primes. Entries increase monotonically.

First of all I'm not sure why the "division" statement is necessary.  I'm old school so I expect that everyone knows to avoid division in inner loops if it can be avoided.  But maybe I'm wrong about that...

Also, the *Sieve of Eratosthenes* does not do division, and is a lot easier to understand *and explain* than this algorithm.  So why this particular algorithm?  I think Knuth was trying to save memory -- and in 1982 saving memory was important.  This algorithm uses a lot less memory than the sieve.

Then came the phrase: `Each entry here contains an odd multiple...`.  I looked at that, and then at the code, and I saw: `multiples[0] = 4;`.

"That's not odd" I said to myself.  "So maybe he meant even."

So then I looked down and saw: `multiples[i] += 2*primes[i];`

"That's adding an even number!" I said to myself.  "I'm pretty sure he meant to say 'even' instead of 'odd'."

I hadn't yet worked out what the `multiples` array was.  So I thought it was perfectly reasonable that it would have even numbers in it, and that your comment was simply an understandable word transposition.  After all, there's no compiler for comments so they suffer from the kinds of mistakes that humans often make with words.

It was only when I got to `multiples[primesFound] = candidate*candidate;` that I started to question things.  If the `candidate` is prime, shouldn't `prime*prime` be odd in every case beyond 2?  I had to do the math in my head to prove that.  (2n+1)(2n+1) = 4n^2+4n+1 ... Yeah, that's odd.

OK, so the `multiples` array is full of odd multiples, except for the first element, since it will be muliples of 2.

So perhaps that comment should be:

	 // multiples of corresponding prime.

Or perhaps we should change the name of the array to something like `primeMultiples` and drop the comment altogether.

Moving on to the next comment:

	            // Each iteration of this loop tests the candidate against one
	            // potential prime factor. Skip the first factor (2) since we
	            // only consider odd candidates.

That doesn't make a lot of sense.  The code it's talking about is:

	            for (int i = 1; i <= lastMultiple; i++) {
	                while (multiples[i] < candidate) {

The `multiples` array, as we have now learned, is an array of *multiples* of prime numbers.  This loop is not testing the candidate against prime *factors*, it's testing it against the current prime _multiples_.

Fortunately for me the third of fourth time I read this comment I realized that you really meant to use the word "multiples".  But the only way for me to know that was to understand the algorithm.  And when I understand the algorithm, why do I need the comment?

That left me with one final question.  What the deuce was the reason behind:

	multiples[primesFound] = candidate*candidate;

Why the square?  That makes no sense.  So I changed it to:

	multiples[primesFound] = candidate;

And it worked just fine.  So this must be an optimization of some kind.

Your comment to explain this is:

	            // Start with the prime's square here, rather than 3x the prime.
	            // This saves time and is safe because all of the intervening
	            // multiples will be detected by smaller prime numbers. As an
	            // example, consider the prime 7: the value in multiples will
	            // start at 49; 21 will be ruled out as a multiple of 3, and
	            // 35 will be ruled out as a multiple of 5, so 49 is the first
	            // multiple that won't be ruled out by a smaller prime.

The first few times I read this it made no sense to me at all.  It was just a jumble of numbers.

I stared at the ceiling, and closed my eyes to visualize. I couldn't see it.  So I went on a long contemplative bike ride during which I realized that the prime multiples of 2 will at one point contain 2*3 and then 2*5.  So the `multiples` array will at some point contain multiples of primes *larger* than the prime they represent.  _And it clicked!_

Suddenly it all made sense. I realized that the `multiples` array was the equivalent of the array of booleans we use in the *Sieve of Eratosthenes* -- but with a really interesting twist.  If you were to do the sieve on a whiteboard, you _could_ erase every number less than the candidate, and only cross out the numbers that were the next multiples of all the previous primes.

That explanation makes perfect sense to me -- now, but I'd be willing to bet that those who are reading it are puzzling over it.  The idea is just hard to explain.

Finally I went back to your comment and could see what you were saying.

###A Tale of Two Programmers
The bottom line here is that you and I both fell into the same trap.  I refactored that old algorithm 18 years ago, and I thought all those method and variable names would make my intent clear -- *because I understood that algorithm*.

You wrote that code awhile back and decorated it with comments that you thought would explain your intent -- *because you understood that algorithm*.

But my names didn't help me 18 years later.  They didn't help you, or your students either.  And your comments didn't help me.

We were inside the box trying to communicate to those who stood outside and could not see what we saw.

## Bob's Rewrite of PrimeGenerator2

**UB:**

When I saw your solution, and after I gained a good understanding of it.  I refactored it just a bit.  I loaded it into my IDE, wrote some simple tests, and extracted a few simple methods.

I also got rid of that *awful* labeled `continue` statement.  And I added 3 to the primes list so that I could mark the first element as *irrelevant* and give it a value of -1.  (I think I was still reeling from the even/odd confusion.)

I like this because the implementation of the `generateFirstNPrimes` method describes the moving parts in a way that hints at what is going on.  It's easy to read that implementation and get a glimpse of the mechanism.  I'm not at all sure that the comment helps.

I think it is just the reality of this algorithm that the effort required to properly explain it, and the effort required for anyone else to read and understand that explanation is roughly equivalent to the effort needed to read the code and go on a bike ride.

	package literatePrimes;

	public class PrimeGenerator3 {
	    private static int[] primes;
	    private static int[] primeMultiples;
	    private static int lastRelevantMultiple;
	    private static int primesFound;
	    private static int candidate;

	    // Lovely little algorithm that finds primes by predicting
	    // the next composite number and skipping over it. That prediction
	    // consists of a set of prime multiples that are continuously
	    // increased to keep pace with the candidate.

	    public static int[] generateFirstNPrimes(int n) {
	        initializeTheGenerator(n);

	        for (candidate = 5; primesFound < n; candidate += 2) {
	            increaseEachPrimeMultipleToOrBeyondCandidate();
	            if (candidateIsNotOneOfThePrimeMultiples()) {
	                registerTheCandiateAsPrime();
	            }
	        }
	        return primes;
	    }

	    private static void initializeTheGenerator(int n) {
	        primes = new int[n];
	        primeMultiples = new int[n];
	        lastRelevantMultiple = 1;

	        // prime the pump. (Sorry, couldn't resist.)
	        primesFound = 2;
	        primes[0] = 2;
	        primes[1] = 3;

	        primeMultiples[0] = -1;// irrelevant
	        primeMultiples[1] = 9;
	    }

	    private static void increaseEachPrimeMultipleToOrBeyondCandidate() {
	        if (candidate >= primeMultiples[lastRelevantMultiple])
	            lastRelevantMultiple++;

	        for (int i = 1; i <= lastRelevantMultiple; i++)
	            while (primeMultiples[i] < candidate)
	                primeMultiples[i] += 2 * primes[i];
	    }

	    private static boolean candidateIsNotOneOfThePrimeMultiples() {
	        for (int i = 1; i <= lastRelevantMultiple; i++)
	            if (primeMultiples[i] == candidate)
	                return false;
	        return true;
	    }

	    private static void registerTheCandiateAsPrime() {
	        primes[primesFound] = candidate;
	        primeMultiples[primesFound] = candidate * candidate;
	        primesFound++;
	    }
	}

## Test-Driven Development

**JOHN:**

Let's move on to our third area of disagreement, which is Test-Driven
Development. I am a huge fan of unit testing. I believe that unit tests are
an indispensable part of the software development process and pay for
themselves over and over. I think we agree on this.

However, I am not fan of Test-Driven Development (TDD), which dictates
that tests must be written before code and that code must be written
and tested in tiny increments. This approach has serious problems
without any compensating advantages that I have been able to identify.

**UB:**

As I said at the start I have carefully read _A Philosophy of Software Design_. I found it to be full of worthwhile insights, and I strongly agree with most of the points you make.

So I was surprised to find, on page 157, that you wrote a very short, dismissive, pejorative, and inaccurate section on _Test Driven Development_.  Sorry for all the adjectives, but I that's a fair characterization.  So my goal, here, is to correct the misconception you have that led you to write that section.

>"Test-driven development is an approach to software development where programmers write unit tests before they write code.  When creating a new class, the develper first writes unit tests for the class, based on its expected behavior.  None of these tests pass, since there is no code for the class.  Then the developer works through the tests one at a time, writing enough code for that test to pass.  When all of the tests pass, the class is finished."

This is just wrong.  TDD is quite considerably different from what you describe.  I describe it using three laws.

 1. You are not allowed to write any production code until you have first written a unit test that fails because that code does not exist.

 2. You are not allowed to write more of a unit test than is sufficient to fail, and failing to compile is failing.

 3. You are not allowed to write more production code than is sufficient to make the currently failing test pass.

 A little thought will convince you that these three laws will lock you into a cycle that is just a few seconds long.  You'll write a line or two of a test that will fail, you'll write a line or two of production code that will pass, around and around every few seconds.

 A second layer of TDD is the Red-Green-Refactor loop.  This loop is several minutes long.  It is comprised of a few cycles of the three laws, followed by a period of reflection and refactoring.  During that reflection we pull back from the intimacy of the quick cycle and look at the design of the code we've just written.  Is it clean?  Is it well structured?  Is there a better approach?  Does it match the design we are pursuing?  If not, should it?

 **JOHN:**

 Oops! I plead "guilty as charged" to inaccurately describing TDD.
 I will fix this in the next revision of APOSD. That said, your definition
 of TDD does not change my concerns.

 Let's discuss the potential advantages and disadvantages
 of TDD; then readers can decide for themselves whether they think TDD is a
 good idea overall.

 Before we start that discussion, let me clarify the approach I prefer as an
 alternative to TDD. In your online videos you describe the alternative to
 TDD as one where a developer writes the code, gets it fully working
 (presumably with manual tests), then goes back and writes the unit tests.
 You argue that this approach would be terrible: developers
 lose interest once they think code is working, so they wouldn't actually
 write the tests. I agree with you completely. However, this isn't the only
 alternative to TDD.

 The approach I prefer is one where the developer works in somewhat
 larger units than in TDD, perhaps a few methods or a class. The developer
 first writes some code (anwywhere from a few tens of lines to a few hundred
 lines), then writes unit tests for that code. As with TDD, the
 code isn't considered to be "working" until it has comprehensive unit
 tests.

 **UB:**

 How about if we call this technique "bundling" for purposes of this
 document?

 **JOHN:**

 Fine by me.

 The reason for working in larger units is to encourage design
 thinking, so that a developer can think about a collection of related
 tasks and do a bit of planning to come up with a good overall design
 where the pieces fit together well.
 Of course the initial design ideas will have flaws and refactoring
 will still be necessary, but the goal is to center the development
 process around design, not tests.

 To start our discussion, can you make a list of the advantages you
 think that TDD provides over the approach I just described?

 **UB:**
The advantages I usually describe when describing TDD are:

* Very little need for debugging.  After all, if you just saw everything working a minute or two ago, there's not much to debug.

* A stream of reliable low level documentation, in the form of very small and isolated unit tests.  Those tests describe the low level structure and operation of every facet of the system.  If you want to know how to do something in the system, there are tests that will show you how.

* A less coupled design which results from the fact that every small part of the system must be designed to be testable, and testability requires decoupling.

* A suite of tests that you trust with your life, and therefore supports fearless refactoring.

However, you asked me which of these advantages TDD might have over _your_ preferred method.  That depends on how big you make those larger units you described.  The important thing to me is to keep the cycle time short, and to prevents entanglements that block testability.

It seem to me that working in small units that you immediately write tests for after the fact can give you all the above advantages, so long as you are very careful to test every aspect of the code you just wrote.  I think a disciplined programmer could effectively work that way.  Indeed, I think such a programmer would produce code that I could not distiguish from code written by another programmer following TDD.

Above you suggested that bundling is to encourage design.  I think encouraging design is a very good thing.  My question for you is: Why do you think that TDD does not encourage design?  My own experience is that design comes from strategic thought, which is independent of the tactical behavior of either TDD or Bundling.  Design is taking one step back from the code and envisioning structures that address a larger set of constraints and needs.

Once you have that vision in your head it seems to me bundling and TDD will yield similar results.

**JOHN:**

First, let me address the four advantages you listed for TDD:

* Very little need for debugging? I think any form of unit testing can
  reduce debugging work, but not for the reason you
  suggested. The benefit comes because unit tests expose bugs earlier
  and in an environment where they are easier to track down. A
  relatively simple bug to fix in development can be very painful to
  track down in production. I'm not convinced by your argument that
  there's less debugging because "you just saw everything working a
  minute ago": it's easy to make a tiny change that exposes a really
  gnarly bug that has existed for a long time but hasn't yet been
  triggered. Hard-to-debug problems arise from the accumulated complexity
  of the system, not from the size of the code increments.
	
	>**UB:** True.  However, when the cycles are very short then the cause 
	of even the gnarliest of bugs have the best chance of being tracked down.
	The shorter the cycles, the better the chances.
* Low level documentation? I disagree: unit tests are a very poor form
  of documentation. Comments are a much more
  effective form of documentation, and you can put them right next to the
  relevant code. Searching through unit tests to figure out how a system
  works will be tedious and frustrating.
	
	>**UB:** Will be?  Or is?  Nowadays it's very easy to find the tests for
	a function by using the "where-used" feature of the IDE.  As for comments
	being better, if that were true then no one would publish example code.  
* A less coupled design? Possibly, but I haven't experienced this myself.
  It's not clear to me that designing for testability will produce the
  best design.
	
	>**UB:** Generally the decoupling arises because the test requires a mock
	of some kind.  Mocks tend to force abstractions that might otherwise not exist.
	
* Enabling fearless refactoring? BINGO! This is the where almost all of the
  benefits from unit testing come from, and it is a really really big deal.
	
	>**UB:** Agreed.

I agree with your conclusion that TDD and bundling are about the
same in terms of providing these benefits.

Now let me explain why I think TDD is likely to result in bad designs.
The fundamental problem with TDD is that it forces developers to work
too tactically, in
units of development that are too smalll; discourages design
thinking.  With TDD the basic unit of
development is one test: first the test is written, then the code to
make that test pass. However, the natural units for design are larger
than this: a class or method, for example. These units
correspond to multiple test cases. If a developer thinks only about
the next test, they are only considering part of a design problem at
any given time. It's hard to design something well if you don't think
about the whole design problem at once. TDD explicitly
prohibits developers from writing more code than is needed to pass
the current test; this discourages the kind of strategic thinking needed
for good design.

>**UB:** I haven't found this to be true.  We write code one line at a time.
That's immensely tactical and yet does not discourage design.  So why would one test at a time discourage it?  Wouldn't it be nice to know that the line you
just wrote actually does what you thought it would do?  That's the advantage of the test.  There is nothing about TDD that stops you from thinking.  

TDD does not provide adequate guidance to encourage design. You mentioned
the Red-Green-Refactor loop, which recommends refactoring after each step,
but there's almost no guidance for refactoring. How should developers
decide when and what to refactor? This seems to be left purely to their
own "judgment". For example, if I am writing a method that requires
multiple iterations of the TDD loop, should I refactor after every iteration
(which sounds pretty tedious) or wait until after several iterations so that
I can look at a bigger chunk of code when refactoring and hence be more
strategic? Without guidance it will be tempting for developers to keep
putting off refactoring.

>**UB:** If you familiarize yourself with the literature on TDD, you'll find
that putting off refactoring is strongly discouraged.  While thinking about design is strongly encouraged.  

TDD is similar to the One Thing Rule we discsused earlier in that it is
biased: it provides very strong and clear instructions pushing developers
in one direction (in this case, acting tactically) with only vague
guidance in the other direction (designing more strategically). As a result,
developers are likely to err on the side of being too tactical.

>**UB:** Again, once you learn more about TDD, the more you will realize that
refactoring and design are an integral part of the discipline.   

TDD guarantees that developers will initially write bad code. If you start
writing code without thinking about the whole design problem, the first code
you write will almost certainly be wrong. Design only
happens after a bunch of bad code has accumulated.
I watched your video on TDD, and
you repeatedly wrote the wrong code, then fixed it later. If the developer
refactors conscientiously (as you did) they can still end up with good
code, but this works against human nature. With TDD, that bad code will
actually work (there are tests to prove it!) and it's human nature not
to want to change something that
works. If the code I'm developing is nontrivial, I will probably have to
accumulate a lot of bad code with TDD before I have enough code in front
of me to understand what the design should have been.
It will be very difficult for me to force myself to throw away
all that work.

>**UB:** We all write bad code at the start.  The discipline of TDD gives us the opportunity and the safety to continuously clean it.  Design insights arise from those kinds of cleaning activities.  The discipline of refactoring allows bad designs to be transformed, one step at a time, into better designs.  

> The fears that you have expressed simply do not materialize in my, rather
long experience, with the discipline.

It's easy for a developer to believe they are doing TDD correctly while
working entirely tactically, layering on hack after hack with an
occasional minor refactor, without ever thinking about the overall design.

>**UB:** Only a developer who has not studied TDD would ever behave that way.

I believe that the bundling approach is superior to TDD because it focuses
the development process around design: design first, then code, then write
unit tests. Of course, refactoring will still be
required: it's almost never possible to get the design right the first time.
But starting with design will reduce the amount of bad code you write and
get you to a good design sooner. It is possible to produce equally good
designs with TDD; it's just harder and requires a lot more discipline.

>**UB:** It's not clear to me why the act of writing tests late is a better design choice.  There's nothing in TDD that prevents me from thinking through a design, long before I write the very first tested code.

Now let me ask you a couple of questions.

First, at a microscopic level, why on earth does TDD prohibit developers
from writing more code than needed to pass the current test? How does
enforcing myopia make systems better?

>**UB:** The goal of the discipline is to make sure that everything is tested.
One good way to do that is to refuse to write any code unless it is to make a failing test pass.  Also, working in such short cycles provides insights into 
the way the code is working.  Those insights often lead to better design decisions.

Second, at a broader level, do you think TDD is likely to produce better
designs than approaches that are more design-centric, such as the bundling
approach I described? If so, can you explain why?

>**UB:** My guess is that an adept at bundling, and an adept at TDD would produce very similar designs, with very similar test coverage.  I would also venture to guess that the TDDer would be somewhat more productive than the bundler if for no reason other than that the TDDer finds and fixes problems earlier than the bundler.