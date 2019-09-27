---
layout: post
title: Hello Minitest
---

I have heard nice things about [Minitest][minitest] in the past, but have always worked with RSpec. Working on a small CLI utility recently, I wanted to minimize external dependencies, and since Minitest is built in to Ruby[^1] I thought I'd give it a shot.

Minitest allows you to [run the tests directly from the command line](http://docs.seattlerb.org/minitest/#label-Running+Your+Tests). I started with this approach, test-driving a specific piece of logic I had in my head. Later I followed [Bundler's excellent documentation for writing a gem][bundler-gem] and adopted a Rakefile.

# Spec or Unit Test Notation?

I started using the spec notation at first, but noticed that the error messages refer to [auto-generated methods](https://github.com/seattlerb/minitest/blob/master/design_rationale.rb). Consider the following test:

```ruby
require 'minitest/autorun'

describe 'Foo' do
  it 'fails' do
    1.must_equal 0
  end
end
```

Running it generates this output:

```
$ ruby foo_test.rb
Run options: --seed 64703

# Running:

F

Failure:
Foo#test_0001_fails [foo_test.rb:5]:
Expected: 0
  Actual: 1


ruby foo_test.rb:4



Finished in 0.001004s, 996.0157 runs/s, 996.0157 assertions/s.
1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

That failure description references `Foo#test_0001_fails`. I found this slightly disorienting even with a small amount of code, and it hinders navigating to the failing test by copying the method name from the error message and searching for it. I decided to switch to the [regular unit test notation](https://github.com/seattlerb/minitest#unit-tests):

```ruby
require 'minitest/autorun'

class Foo < Minitest::Test
  def test_fails
    assert_equal 0, 1
  end
end
```

It reminds me of how I first learned unit testing, through JUnit examples[^2]. The error report is clearer:

```
Failure:
Foo#test_fails [foo_test.rb:5]:
Expected: 0
  Actual: 1
```

The notation also affects the assertion syntax.

# Running Specific Tests

Once there were multiple test files, the need arose to run specific test files and specific tests on their own. A web search eventually led me to [a post by Weston Ganger][run-minitest] which provides the solution.

[How it works](https://github.com/ruby/rake/blob/1c22b490ee6cb8bd614fa8d0d6145f671466206b/lib/rake/testtask.rb): the `TEST` option tells `rake` which file to run, while the value of the `TESTOPTS` option is passed on to the test runner. [The options Minitest accepts are documented](http://docs.seattlerb.org/minitest/#label-Running+Your+Tests).

# References

- Minitest [documentation](http://docs.seattlerb.org/minitest/) and [source code][minitest]
- [_How to create a Ruby gem with Bundler_][bundler-gem]
- [_Minitest Quick Reference_][quick-reference] by Matt Sears
- [_Cheatsheet for Minitest_][cheatsheet] at devhints.io
- [_A Few Ways To Run Your MiniTest Tests_][run-minitest] by Weston Ganger

[minitest]: https://github.com/seattlerb/minitest
[bundler-gem]: https://bundler.io/v2.0/guides/creating_gem.html
[quick-reference]: http://mattsears.com/articles/2011/12/10/minitest-quick-reference/
[cheatsheet]: https://devhints.io/minitest
[run-minitest]: https://github.com/ruby/rake/blob/1c22b490ee6cb8bd614fa8d0d6145f671466206b/lib/rake/testtask.rb

[^1]: Minitest started shipping with Ruby 1.9. The [docs do mention, though, that not all gem functionality works](http://docs.seattlerb.org/minitest/#label-INSTALL-3A<Paste>) with the built-in Minitest, so installing the gem might be best.

[^2]: I actually first encountered unit testing in Python, but reading [Kent Beck's _Test-Driven Development: By Example_](https://www.amazon.com/dp/0321146530) really solidified things for me. I recommend this book to anyone starting out with unit testing.
