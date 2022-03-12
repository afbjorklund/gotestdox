[![Go Reference](https://pkg.go.dev/badge/github.com/bitfield/gotestdox.svg)](https://pkg.go.dev/github.com/bitfield/gotestdox)[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)[![Go Report Card](https://goreportcard.com/badge/github.com/bitfield/gotestdox)](https://goreportcard.com/report/github.com/bitfield/gotestdox)[![CircleCI](https://circleci.com/gh/bitfield/gotestdox.svg?style=svg)](https://circleci.com/gh/bitfield/gotestdox)

```
go install github.com/bitfield/gotestdox/cmd/gotestdox@latest
```

![Writing gopher logo](img/gotestdox.png)

# `gotestdox`

`gotestdox` is a command-line tool for turning Go test names into readable sentences. For example, suppose we have some tests named like this:

```
TestRelevantIsTrueForTestPassOrFailEvents
TestRelevantIsFalseForNonPassFailEvents
```

We can transform them into straightforward sentences that express the desired behaviour, by running `gotestdox`:

**`gotestdox`**

This will run the tests, and print:

```
 ✔ Relevant is true for test pass or fail events (0.00s)
 ✔ Relevant is false for non pass fail events (0.00s)
```

# Why

I read a blog post by Dan North, which says:

> My first “Aha!” moment occurred as I was being shown a deceptively simple utility called `agiledox`, written by my colleague, Chris Stevenson. It takes a JUnit test class and prints out the method names as plain sentences.
>
> The word “test” is stripped from both the class name and the method names, and the camel-case method name is converted into regular text. That’s all it does, but its effect is amazing.
>
> Developers discovered it could do at least some of their documentation for them, so they started to write test methods that were real sentences.\
—Dan North, [Introducing BDD](https://dannorth.net/introducing-bdd/)

# How

The original [`testdox`](https://github.com/astubbs/testdox) tool (part of `agiledox`) was very simple, as Dan describes: it just turned a camel-case JUnit test name like `testFailsForDuplicateCustomers` into a space-separated sentence like `fails for duplicate customers`.

And that's what I find neat about it: it's so simple that it hardly seems like it could be of any value, but it is. I've already used the idea to improve a lot of my test names.

There are implementations of `testdox` for various languages other than Java: for example, [PHP](https://phpunit.readthedocs.io/en/9.5/textui.html#testdox), [Python](https://pypi.org/project/pytest-testdox/), and [.NET](https://testdox.wordpress.com/). I haven't found one for Go, so here it is.

`gotestdox` reads the JSON output generated by the `go test -json` command. This is easier than trying to parse Go source code, for example, and also gives us pass/fail information for the tests. It ignores all events except pass/fail events for individual tests (including subtests).

# Getting fancy

Some more advanced ways to use `gotestdox`:

## Exit status

If there are any test failures, `gotestdox` will report exit status 1.

## Colour

`gotestdox` indicates a passing test with a `✔` (check mark emoji), and a failing test with an `x`. These are displayed as green and red respectively, using the [`color`](https://github.com/fatih/color) library, which automagically detects if it's talking to a colour-capable terminal.

If not (for example, when you redirect output to a file), or if the [`NO_COLOR`](https://no-color.org/) environment variable is set to any value, colour output will be disabled.

## Test flags and arguments

`gotestdox`, with no arguments, will run the command `go test -json` and process its output.

Any arguments you supply will be passed on to `go test`. For example:

**`gotestdox -run ParseJSON`**

will run the command:

`go test -json -run ParseJSON`

You can supply a list of packages to test, or any other arguments or flags understood by `go test`. However, `gotestdox` only prints events about *tests* (ignoring benchmarks and examples). It doesn't report fuzz tests, since they don't tend to have useful names.

## Multiple packages

To test all the packages in the current tree, run:

**`gotestdox ./...`**

Each package's test results will be prefixed by the fully-qualified name of the package. For example:

```
github.com/octocat/mymodule/api:
 ✔ NewServer returns a correctly configured server (0.00s)
 ✔ NewServer errors on invalid config options (0.00s)

github.com/octocat/mymodule/util:
 ✔ LeftPad adds the correct number of leading spaces (0.00s)
 ```

## Multi-word function names

There's an ambiguity about test names involving functions whose names contain more than one word. For example, suppose we're testing a function `HandleInput`, and we write a test like this:

```
TestHandleInputClosesInputAfterReading
```

Unless we do something, this will be rendered as:

```
 ✔ Handle input closes input after reading
```

To let us give `gotestdox` a hint about this, there's one extra transformation rule: the first underscore marks the end of the function name. So we can name our test like this:

```
TestHandleInput_ClosesInputAfterReading
```

and this becomes:

```
 ✔ HandleInput closes input after reading
```

I think this is an acceptable compromise: the `gotestdox` output is much more readable, while the extra underscore in the test name doesn't seriously interfere with its readability.

The intent is not to *perfectly* render all sensible test names as sentences, in any case, but to do *something* useful with them, primarily to encourage developers to write test names that are informative descriptions of the unit's behaviour, and thus (as a side effect) read well when formatted by `gotestdox`.

In other words, `gotestdox` is not the thing. It's the thing that gets us to the thing, the end goal being meaningful test names (I like the term _literate_ test names).

## Filtering standard input

If you want to run `go test -json` yourself, for example as part of a shell pipeline, and pipe its output into `gotestdox`, you can do that too:

**`go test -json | gotestdox`**

In this case, any flags or arguments to `gotestdox` will be ignored, and it won't run the tests; it will act purely as a text filter. However, just like when it runs the tests itself, it will report exit status 1 if there are any test failures.

## As a library

See [pkg.go.dev/github.com/bitfield/gotestdox](https://pkg.go.dev/github.com/bitfield/gotestdox) for the full documentation on using `gotestdox` as a library package.

# So what?

Why should you care, then? What's interesting about `gotestdox`, or any `testdox`-like tool, I find, is the way its output makes you think about your tests, how you name them, and what they do.

As Dan says in his blog post, turning test names into sentences is a very simple idea, but it has a powerful effect. Test names *should* be sentences.

## Test names should be sentences

I don't know about you, but I've wasted a lot of time and energy over the years trying to choose good names for tests. I didn't really have a way to evaluate whether the name I chose was good or not. Now I do!

Naming your test as a sentence actually forces you to think about how your code is supposed to *behave*, given some input or circumstances. There needs to at least be a verb.

For example, suppose we have some function `Match` that tells you whether or not a given input matches the string you're looking for:

```go
func Match(input, substring string) bool {
```

What would we name a test for this function? We might instinctively name it:

```
TestMatch
```

Pretty standard, and no doubt it does test `Match` in some way, but *what* way, actually? How is `Match` supposed to behave, according to this test? Under what circumstances? Given what input? We don't know. Suppose we're running the tests and all we see is some failure like this:

```
--- FAIL: TestMatch (0.00s)
```

That's very unhelpful. To find out what this test thinks *should* have happened, but didn't, we need to dig into the code. Ideally, the name of the test itself should tell us everything we need to know!

## Describe the behaviour you want

Of course, a good test will also give us specific information about the failure: if `want` wasn't equal to `got`, it will tell us that. But we still don't know why `want` is *supposed* to equal `got`. In other words, we're missing some critical information: what is the test actually *about*?

We need to switch from thinking about the test name as a piece of useless paperwork to thinking about it as documentation. As soon as we do that, it's clear that the name should be a sentence expressing what happens when the code under test is correct:

```
Match is true for matching input
```

In fact, let's lean into this and call them “test sentences” instead of “test names”. That'll prompt us every time we write one: “What's the sentence for this test?”

## Under what circumstances?

When we see this test sentence, it's helpful, but it also immediately prompts us to think, “well, what about *non*-matching input?” Okay. That's another test, then:

```
Match is false for non matching input
```

We haven't just improved the names of our existing tests; we've actually generated new test cases. That's powerful. When we're forced to describe some particular case explicitly, it becomes obvious what *other* possibilities exist that we haven't yet tested.

> I see a lot of Go unit tests without the condition, just a list of expectations (or contrariwise, just a list of states without expectations). Either way, it can easily lead to blind spots, things you are not testing that you should be. Let’s include both condition and expectation:

> HandleCategory trims LEADING spaces from valid category\
> HandleCategory trims TRAILING spaces from valid category\
> HandleCategory trims LEADING and TRAILING spaces from valid category

> Immediately upon reading those, I bet you noticed we are missing some tests (and/or requirements!). What happens if we give an empty category? Or an invalid category? What constitutes a valid category?\
—Michael Sorens, [Go Unit Tests: Tips from the Trenches](https://www.red-gate.com/simple-talk/devops/testing/go-unit-tests-tips-from-the-trenches/)

## “Does”, not “should”

It's tempting to include “should” in every test sentence, especially if we're writing the test first:

```
Match should be true for matching input
```

I don't think that's necessary, and we can keep our test sentences short and to the point by omitting words like “should”, “must”, and “will”. Just say what it *does*. A good way to think of this is that every test sentence implicitly ends with the words “...when the code is correct”.

We wouldn't say that Match *should* be true when the code is correct, we would say it *is* true! This isn't an aspiration, it's a definition. The definition of “correct” is that Match is true for matching input, and false otherwise.

> The test itself has a name, which can convey useful information if we choose it wisely. It’s a good idea to name each test function after the behaviour it tests. You don’t need to use words like Should, or Must; these are implicit. Just say what the function does when it’s working correctly.\
—[The Power of Go: Tools](https://bitfieldconsulting.com/books/tools)

## The friendly manual

And that's it! Now we have two sentences that completely describe the important behaviour of `Match`, and `gotestdox` will format them nicely for us:

```
 ✔ Match is true for matching input (0.00s)
 ✔ Match is false for non matching input (0.00s)
 ```

As you accumulate more tests over time, your `gotestdox` output will be a more and more valuable user manual for your package. And that's the right way to think about it. Good tests should focus on *user-visible* behaviour: your public API. So your tests should be named using domain terms that users understand (“A user can log in”), not computer jargon (“Initialize persistent session”).

It might be interesting to show your `gotestdox` output to users, customers, or business folks, and see if it makes sense to them. If so, you're on the right lines. And it's quite likely to generate some interesting conversations (“Is that really what it does? But that's not what we asked for!”)

## Subtest names should complete a sentence

`gotestdox` encourages you to create tests and subtests with descriptive names, because the results read nicely. For example, here's a snippet of one of its own tests:

```go
func TestPrettify(t *testing.T) {
	t.Parallel()
	tcs := []struct {
		name, input, want string
	}{
		{
			name:  "correctly renders a well-formed test name",
			input: "TestSumCorrectlySumsInputNumbers",
			want:  "Sum correctly sums input numbers",
		},
		{
			name:  "preserves capitalisation of initialisms such as PDF",
			input: "TestFooGeneratesValidPDFFile",
			want:  "Foo generates valid PDF file",
		},
        ...
```

These subtests are rendered as:

```
 ✔ Prettify correctly renders a well-formed test name (0.00s)
 ✔ Prettify preserves capitalisation of initialisms such as PDF (0.00s)
 ...
```

In other words, it's a good idea to name each subtest so that it completes a sentence beginning with the name of the unit under test, describing the specific behaviour checked by that subtest.

If you find it weird at first writing super long test names like `TestRelevantIsFalseForOtherEvents`, don't worry. You'll get used to it quite quickly. We wouldn't want to use function names like this in our application code, certainly. But tests are different. We never *call* these functions, and users don't see them. So if it really doesn't matter what they're called, let's call them something meaningful!

When you've used `gotestdox` a little, it starts to feel perfectly natural to write your test names as descriptive sentences—which is the point, of course.

## Some examples

Here is the complete `gotestdox` rendering of its own tests (sorted for readability), in case it gives you any useful ideas:

```
github.com/bitfield/gotestdox:
 ✔ EventString formats pass and fail events differently (0.00s)
 ✔ ExecGoTest sets OK to false when command errors (0.02s)
 ✔ ExecGoTest sets OK to false when tests fail (0.76s)
 ✔ ExecGoTest sets OK to true when tests pass (0.63s)
 ✔ Filter keeps track of current package (0.00s)
 ✔ Filter sets OK to false if any test fails (0.01s)
 ✔ Filter sets OK to false on parsing error (0.00s)
 ✔ Filter sets OK to true if there are no test failures (0.01s)
 ✔ Filter skips irrelevant events (0.01s)
 ✔ NewTestDoxer returns testdoxer with standard IO streams (0.00s)
 ✔ ParseJSON errors on invalid JSON (0.00s)
 ✔ ParseJSON returns valid data for valid JSON (0.00s)
 ✔ Prettifier logs to debug writer (0.00s)
 ✔ Prettify (0.01s)
 ✔ Prettify accepts a single-letter test name (0.00s)
 ✔ Prettify accepts a single-word test name (0.00s)
 ✔ Prettify does not break words when a digit follows an '=' sign (0.00s)
 ✔ Prettify does not erase the final digit in words that end with a digit (0.00s)
 ✔ Prettify does not hang when name ends with initialism (0.00s)
 ✔ Prettify does not treat an underscore in a subtest name as marking the end of a multiword function name (0.00s)
 ✔ Prettify doesn't incorrectly title-case single-letter words (0.00s)
 ✔ Prettify eliminates any words containing underscores after splitting (0.00s)
 ✔ Prettify handles a test with no name, but with subtests (0.00s)
 ✔ Prettify handles multiple underscores, with the first marking the end of a multiword function name (0.00s)
 ✔ Prettify inserts a word break before subtest names beginning with a lowercase letter (0.00s)
 ✔ Prettify is okay with test names not in the form of a sentence (0.00s)
 ✔ Prettify keeps a trailing digit as part of an initialism (0.00s)
 ✔ Prettify keeps numbers within a hyphenated word (0.00s)
 ✔ Prettify keeps together digits in numbers that are standalone words (0.00s)
 ✔ Prettify keeps together hyphenated words with initial capitals (0.00s)
 ✔ Prettify knows that just 'test' is a valid test name (0.00s)
 ✔ Prettify preserves capitalisation of initialism when it is the first word (0.00s)
 ✔ Prettify preserves capitalisation of initialisms such as 'PDF' (0.00s)
 ✔ Prettify preserves capitalisation of two-letter initialisms such as 'OK' (0.00s)
 ✔ Prettify preserves initialisms containing digits (0.00s)
 ✔ Prettify preserves initialisms containing digits with two or more leading alpha characters (0.00s)
 ✔ Prettify preserves longer all-caps words (0.00s)
 ✔ Prettify recognises a dash followed by a digit as a negative number (0.00s)
 ✔ Prettify renders subtest names without the slash, and with underscores replaced by spaces (0.00s)
 ✔ Prettify replaces camel-case transitions with spaces (0.00s)
 ✔ Prettify retains apostrophised words in their original form (0.00s)
 ✔ Prettify retains capitalisation of initialisms in a multiword function name (0.00s)
 ✔ Prettify retains hyphenated words in their original form (0.00s)
 ✔ Prettify retains quoted words as quoted (0.00s)
 ✔ Prettify treats a single underscore as marking the end of a multiword function name (0.00s)
 ✔ Prettify treats a single underscore before the first slash as marking the end of a multiword function name (0.00s)
 ✔ Prettify treats consecutive underscores as a single word break (0.00s)
 ✔ Prettify treats numbers as word separators (0.00s)
 ✔ Prettify treats underscores as word breaks (0.00s)
 ✔ Relevant is false for non test pass fail events (0.00s)
 ✔ Relevant is true for test pass or fail events (0.00s)
```

# Links

- [Bitfield Consulting](https://bitfieldconsulting.com/)

<small>Gopher image by [MariaLetta](https://github.com/MariaLetta/free-gophers-pack)</small>
