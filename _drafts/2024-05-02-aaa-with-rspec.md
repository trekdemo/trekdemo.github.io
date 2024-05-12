---
categories: [Testing, RSpec]
date: 2024-05-02 9:00:01 +0100
title: Arrange, act and assert with RSpec
excerpt: |-
  There is this amazing pattern that helps writing easy to read and maintainable tests.
  Following this pattern in RSpec, a DSL for writing tests, is a breeze and will improve your quality of life.
---

Automated tests are beneficial for many reasons. For one, they replace the tedious manual testing steps whenever I change code. It's not even required to write structured, "good-looking" tests to get this benefit. But I must see the test fail for the right reason. (I beg you never to skip seeing the test fail.)

Automated tests also act as living documentation of your system and its behaviour. To earn this benefit, they *must serve the reader*. The reader should understand the test's prerequisites, its purpose, and the expected outcomes.

[RSpec](https://rspec.info) is a testing framework for Ruby, providing a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) to write expressive, human-readable automated tests. Tests written using RSpec reads almost as common English.

## The pattern

We humans are great at [recognising patterns](<https://en.wikipedia.org/wiki/Pattern_recognition_(psychology)>).  The way you structure your tests will allow its readers to understand them quicker and, when needed, change and maintain them with *confidence*.

In the last fifteen years, I found one pattern that made my tests notably better for their readers. In a 2011 article, [Bill Wake](https://xp123.com/articles/3a-arrange-act-assert/) called this pattern *arrange-act-assert* (AAA or 3A). You might recognise a strikingly similar pattern in [behaviour-driven development](https://leanpub.com/bddbooks-formulation) books,  the given-when-then structure.

1. The *"arrange"* step sets up the test case, instantiates any necessary objects, configures settings, and prepares the database, ...
1. The *"act"* step covers the main *action*. Usually, this means a method call in a unit test, an HTTP call in integration tests and browser interactions within system tests.
1. The *"assert"* step compares the results with the expectations. Is the return value equal to the expected? Did the expected side-effect happen?

Unfortunately, we won't find any of these three terms (arrange, act, assert) in RSpec's DSL, but that does not stop us from structuring your tests into the three steps of 3A or GWT.

## Let's Arrange

RSpec has several helpers to express and define the necessary steps to prepare for a test.
The [`#before`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/Hooks.html#before-instance_method) method is RSpec's trivial way of saying *arrange*. Within its block, you can set up everything that is required by the test.

```ruby
RSpec.describe "A social robot" do
  before do
    I18n.locale = :en
    @name = "Bob"
    @test_subject = Robot.new(@name)
  end
  # ...
end
```

What about teardown? RSpec has us covered with the [`#after`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/Hooks.html#after-instance_method) and [`#around`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/Hooks.html#around-instance_method) methods. Like `#before`, these methods also accepts arguments to further specify when to execute them. They allow us to clean up state, roll back the database, and delete files created by the tests...

### Breaking up "arrange"

RSpec gets more sophisticated than that. It has words to "talk about" the object you're testing, your test *subject*, and its *collaborators*. So when you *arrange* your test, you can give them names.

I'll start with the [`#subject(name = nil) {}`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/MemoizedHelpers/ClassMethods.html#subject-instance_method) method. I use `#subject` to tell future me, my colleagues and RSpec about the object under testing. Specifying the *subject* this way will also let us reference it implicitly. This is why you can write, for example, `it { is_expected.to eq("foo") }`.

I prefer to name these subjects, so when I'm writing more complex expectations, I can use descriptive names. Naming things is especially handy when the subject is different between the test cases.

```ruby
RSpec.describe "A social robot" do
  subject(:robot) { Robot.new(@name) } # 👈 Extracted from #before

  before do
    I18n.locale = :en
    @name = "Bob"
  end

  # ...
end
```

Let's follow it up with [`#let(name, &block)`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/MemoizedHelpers/ClassMethods.html#let-instance_method). Using `#let`, you can extract, name (and memoize) the *collaborators* of your test subject.

```ruby
RSpec.describe "A social robot" do
  subject(:robot) { Robot.new(name) } # 👈 Uses the memoized method instead of the ivar
  let(:name) { "Bob" }                # 👈 Extracted from #before

  before do
    I18n.locale = :en
  end

  # ...
end
```

### A word of warning about let

Use `#let` [sparingly](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/MemoizedHelpers/ClassMethods.html#let-instance_method)! A couple can improve readability, but more will quickly deteriorate your test. If I find a test surrounded by a bunch of *let* statements, usually there is a whiff of the [Long Parameter List](https://refactoring.guru/smells/long-parameter-list) code smell too.

Using *let* to store expectations will force the reader to jump back and forth in the test code. Keep them together with assertions.

Joe Ferries wrote an excellent [article](https://thoughtbot.com/blog/lets-not) about potential overuse issues. I think, the dose makes the poison, so be mindful using *let*.

## Act and Assert

This is the meat of the test. This is where things go from red to green and gives you confidence that the system should work as intended.

In RSpec, act and arrange goes into the [`#it` or `#specify`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/ExampleGroup.html#it-class_method) block.

- [ ] Write about the two different ways of asserting!
  What to do to assert commands (side effects) and queries (return values).
  Sandy Metz has a really good talk about testing.

### Assert return values

If a return value is being tested, having it on a separate line is unnecessary. Simply make the assertion. RSpec calls its assertion helpers [matchers](https://rspec.info/documentation/3.12/rspec-expectations/RSpec/Matchers.html).

```ruby
RSpec.describe "A social robot" do
  # ...

  it "introduces themselves" do
    expect(robot.introduction).to eq("Hi! I'm Bob")
  end
end
```

If you are a fan of single-line tests, you can go even further by setting your test subject to the return value. In this case, you can use the [`#is_expected`](https://rspec.info/documentation/3.12/rspec-core/RSpec/Core/MemoizedHelpers.html#is_expected-instance_method), which is just a shorthand for the `expect(subject)` expression.

```ruby
RSpec.describe "A social robot" do
  # ...

  it "introduces themselves" do
    subject { robot.introduction }
    it { is_expected.to eq("Hi! I'm Bob") }
  end
end
```

### Assert side effects and behaviour

Writing expectations for side effects

```ruby
RSpec.describe "an example" do
  # ...

  it "introduces itself" do
    expect(person.introduction).to eq("Hi! I'm Bob")
  end
end
```
