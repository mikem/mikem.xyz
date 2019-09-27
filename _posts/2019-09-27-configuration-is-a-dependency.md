---
layout: post
title: Configuration is a Dependency
---

Consider this Ruby code which generates a report for a subset of companies:

```ruby
class CustomReport
  def initialize(output:)
    @output = output
  end

  def companies
    DB.companies.query(vertical: 'airline').all
  end

  def generate
    report = # do something with companies

    File.open(@output, 'w') { |f| f.write(report) }
  end
end
```

```ruby
class CustomReportTest < Minitest::Test
  def setup
    @airline = DB.companies.create(vertical: 'airline')
    @gallery = DB.companies.create(vertical: 'fine_art')
  end

  def test_generate_report_includes_only_airlines
    CustomReport.new(output: '/tmp/custom_report.txt').generate
    report = File.read('/tmp/custom_report.txt')

    report_ids = extract_ids(report)
    expected_ids = Set.new([@airline.id])

    assert_equal expected_ids, report_ids
  end

  def extract_ids(report)
    # return a Set of IDs found in the report
  end
end
```

Our task is to generate the same report for another vertical. The simplest solution would be adding a second DB query with the new vertical hardcoded. Instead, let's make the algorithm generic to make supporting future verticals easier.

In our scenario, adding new verticals happens rarely and with ample lead time, so deploying code is a feasible workflow for adding more. A list of the verticals can be stored in a constant on `CustomReport` and iterated over. Adding a new report simply means adding a new element to the list:

```ruby
class CustomReport
  VERTICALS = ['airline', 'performance']

  def initialize(output:)
    @output_dirname, @output_filename = parse_dir_and_filename(output)
  end

  def companies_by_vertical
    VERTICALS.each_with_object({}) do |v, h|
      h[v] = DB.companies.query(vertical: v).all
    end
  end

  def generate
    companies_by_vertical.each do |vertical, companies|
      report = # do something with companies

      filename = "#{@output_dirname}/#{vertical}_#{@output_filename}"
      File.open(filename, 'w') { |f| f.write(report) }
    end
  end

  def parse_dir_and_filename(path)
    # return dirname, filename
  end
end
```

This approach introduces some tradeoffs in testing. The test _could_ check the same verticals defined in `CustomReport::VERTICALS`, but that means _changing the test each time a new vertical is added_.

The code and its tests are tightly coupled.

Decoupling would require modifying the value of `CustomReport::VERTICALS` in the test, which [can be done][redefine-class-constant-issue] but requires some push-ups. It's _an indicator that there's probably a better way_.

[redefine-class-constant-issue]: https://github.com/freerange/mocha/issues/13#issuecomment-509832

Let's use a class method instead, which is easier to stub in test:

```ruby
class CustomReport
  def self.verticals
    ['airline', 'performance']
  end

  def initialize(output:)
    @output_dirname, @output_filename = parse_dir_and_filename(output)
  end

  def companies_by_vertical
    self.class.verticals.each_with_object({}) do |v, h|
      h[v] = DB.companies.query(vertical: v).all
    end
  end

  def generate
    companies_by_vertical.each do |vertical, companies|
      report = # do something with companies

      filename = "#{@output_dirname}/#{vertical}_#{@output_filename}"
      File.open(filename, 'w') { |f| f.write(report) }
    end
  end

  def parse_dir_and_filename(path)
    # return dirname, filename
  end
end
```

The test can now stub `CustomReport.verticals` which achieves the decoupling we're after. However, that `self.class.verticals.each_with_object` looks onerous. And it does rely on stubbing, which incurs some risk. I don't quite like it.

The key here is that the verticals are _configuration_ for `CustomReport` and should live outside the class. This configuration is a _dependency_ for `CustomReport`. Let's _inject_ this dependency:

```ruby
class CustomReport
  def initialize(output:, verticals:)
    @output_dirname, @output_filename = parse_dir_and_filename(output)
    @verticals = verticals
  end

  def companies
    @verticals.each_with_object({}) do |v, h|
      h[v] = DB.companies.query(vertical: v).all
    end
  end

  def generate
    companies_by_vertical.each do |vertical, companies|
      report = # do something with companies

      filename = "#{@output_dirname}/#{vertical}_#{@output_filename}"
      File.open(filename, 'w') { |f| f.write(report) }
    end
  end

  def parse_dir_and_filename(path)
    # return dirname, filename
  end
end
```

The test now looks like this:

```ruby
class CustomReportTest < Minitest::Test
  def setup
    @verticals = ['airline', 'performance']
    @airline = DB.companies.create(vertical: 'airline')
    @gallery = DB.companies.create(vertical: 'fine_art')
    @theater = DB.companies.create(vertical: 'performance')
  end

  def test_generate_airline_report_includes_only_airlines
    CustomReport.new(
      output: '/tmp/custom_report.txt',
      verticals: @verticals
    ).generate
    report = File.read('/tmp/airline_custom_report.txt')

    report_ids = extract_ids(report)
    expected_ids = Set.new([@airline.id])

    assert_equal expected_ids, report_ids
  end

  def test_generate_theater_report_includes_only_theaters
    CustomReport.new(
      output: '/tmp/custom_report.txt',
      verticals: @verticals
    ).generate
    report = File.read('/tmp/performance_custom_report.txt')

    report_ids = extract_ids(report)
    expected_ids = Set.new([@theater.id])

    assert_equal expected_ids, report_ids
  end

  def extract_ids(report)
    # return a Set of IDs found in the report
  end
end
```

The test generates reports for airlines and theaters, but the production code can generate reports for food trucks and toy stores. The code and its tests are completely decoupled, and adding more verticals is as straightforward as the initial solution. The `CustomReport` class can be used elsewhere in the code for different verticals if required.

Pay attention to those code smells. If something looks off or is difficult to test, it's probably a nudge to a better design.
