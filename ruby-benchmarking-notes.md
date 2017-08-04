# Ruby Benchmarking Notes

## Working with Ruby benchmark

You can get basic benchmarking information about a piece of Ruby code with the standard library Benchmark tool:

```ruby
require 'benchmark'

measurement = Benchmark.measure do
  (0..100_000_000).each { |n| n }
end
puts measurement

#=>  3.890000   0.000000   3.890000 (  3.894084)
```

Broken down into parts:

1. User CPU time = time spent executing code
1. System CPU time = time spent executing kernel code
1. User + System CPU time
1. Ellapsed real time (wall clock time)

Often times you want to compare two implementations of something

```ruby
require 'benchmark'

ITERATIONS = 1_000_000

# the 15 is the width of the report label
Benchmark.bm(15) do |bm|
  bm.report('each') do # 'each' is the report lable
    (0..ITERATIONS).each { |n| n }
  end

  bm.report('each_with_index') do
    (0..ITERATIONS).each_with_index { |n| n }
  end
end

#                       user     system      total        real
# each              0.040000   0.000000   0.040000 (  0.034467)
# each_with_index   0.050000   0.000000   0.050000 (  0.048852)
```

However, it's also helpful to get a sense of how these number increase as load increases to figure out if the time increase is linear or curved.

```ruby
require 'benchmark'

ITERATION_SETS = [100_000, 1_000_000, 10_000_000, 100_000_000]
ITERATION_SETS.each do |iterations|
  puts "Benchmarking with iterations #{iterations}"
  Benchmark.bm(15) do |x|
    x.report(:each) do
      (1..iterations).each { |n| n }
    end

    x.report(:each_with_index) do
      (1..iterations).each_with_index { |n, i| n }
    end
  end
  puts ''
end

# Benchmarking with iterations 100000
# user     system      total        real
# each              0.000000   0.000000   0.000000 (  0.003345)
# each_with_index   0.000000   0.000000   0.000000 (  0.004895)

# Benchmarking with iterations 1000000
# user     system      total        real
# each              0.030000   0.000000   0.030000 (  0.033715)
# each_with_index   0.060000   0.000000   0.060000 (  0.052931)

# Benchmarking with iterations 10000000
# user     system      total        real
# each              0.330000   0.000000   0.330000 (  0.334299)
# each_with_index   0.490000   0.000000   0.490000 (  0.488597)

# Benchmarking with iterations 100000000
# user     system      total        real
# each              3.270000   0.000000   3.270000 (  3.273185)
# each_with_index   4.720000   0.000000   4.720000 (  4.728551)
```

You may want to graph this data

```ruby
require 'gruff'

g = Gruff::Line.new
g.labels = []
  0 => 100_000,
  1 => 1_000_000,
  2 => 10_000_000,
  3 => 100_000_000,
}
g.data('each', [0.003345, 0.033715, 0.334299, 3.273185])
g.data('each_with_index', [0.004895, 0.052931, 0.488597, 4.72855])
g.write('output.png')
```

Here are the two combined together

```ruby
require 'benchmark'
require 'gruff'

ITERATION_SETS = [100_000, 1_000_000, 10_000_000, 100_000_000]
graph_data = {
  label_series: [],
  benchmarks: Hash.new([]),
}
ITERATION_SETS.each do |iterations|
  benchmark_reports = Benchmark.bm(15) do |x|
    x.report(:each) do
      (1..iterations).each { |n| n }
    end

    x.report(:each_with_index) do
      (1..iterations).each_with_index { |n, _| n }
    end
  end

  graph_data[:label_series] << iterations
  benchmark_reports.each do |b|
    graph_data[:benchmarks][b.label.to_sym] += Array(b.real)
  end
end

g = Gruff::Line.new
label_series = graph_data[:label_series]
g.labels = (0..label_series.size - 1).to_a.zip(label_series).to_h
graph_data[:benchmarks].each { |label, values| g.data(label, values) }
g.write('output.png')
```

## Benchmark IPS

This is a gem that runs your code as much as it can for five seconds and then outputs the iterations per second number.  The bigger the number the better, obviously.

```ruby
require 'benchmark/ips'
require 'set'

list = ('a'..'zzzz').to_a
set = Set.new(list)

Benchmark.ips do |bm|
  bm.report('set access') do
    set.include?('foo')
  end

  bm.report('array access') do
    list.include?('foo')
  end

  bp.compare!
end
```


You can also grab the output of the benchmark reports and use them to do things:

```ruby
require 'benchmark/ips'
require 'set'

list = ('a'..'zzzz').to_a
set = Set.new(list)

old_stdout = $stdout
$stdout = StringIO.new

report = Benchmark.ips do |bm|
  bm.report('set access') do
    set.include?('foo')
  end

  bm.report('array access') do
    list.include?('foo')
  end
end

$stdout = old_stdout

puts report.entries.map { |entry| [entry.label, entry.ips] }
```

Or... graphing multiple results

```ruby
require 'benchmark/ips'
require 'set'
require 'gruff'

ITERATION_SETS = [100_000, 1_000_000, 10_000_000, 100_000_000]

list = ('a'..'zzzz').to_a
set = Set.new(list)

graph_data = {
  label_series: [],
  benchmarks: Hash.new([]),
}
ITERATION_SETS.each do |iterations|
  report = Benchmark.ips do |bm|
    bm.report('set access') do
      set.include?('foo')
    end

    bm.report('array access') do
      list.include?('foo')
    end
  end

  graph_data[:label_series] << iterations
  report.entries.map do |entry|
    graph_data[:benchmarks][entry.label.to_sym] += Array(entry.ips)
  end
end

g = Gruff::Line.new
label_series = graph_data[:label_series]
g.labels = (0..label_series.size - 1).to_a.zip(label_series).to_h
graph_data[:benchmarks].each { |label, values| g.data(label, values) }
g.write('ips_output.png')
```