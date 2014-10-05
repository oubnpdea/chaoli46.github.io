---

layout: post
title:  "RecursiveOpenStruct性能测试"
date:   2014-10-05 23:50:00
categories: ruby

---

今天看到RubyChina里的一个帖子（[Ruby 中的 OpenStruct 详解](https://ruby-china.org/topics/21617)），想起自己的自动化测试项目里也用到了OpenStruct的增强版[RecursiveOpenStruct](https://github.com/aetherknight/recursive-open-struct)，于是也照着帖子里的[stackoverflow回答](http://stackoverflow.com/questions/1177594/ruby-struct-vs-openstruct/4459132#4459132)做了一个`RecursiveOpenStruct`的性能测试。

实现：

```ruby
require 'benchmark'
require 'ostruct'
require 'recursive-open-struct'

REP = 100000

User = Struct.new(:name, :age)

USER = "User".freeze
AGE = 21
HASH = {name:USER, age:AGE}.freeze

Benchmark.bm 30 do |x|
  x.report 'RecursiveOpenStruct slow' do
    REP.times do |index|
      RecursiveOpenStruct.new(name:"User", age:21)
    end
  end

  x.report 'RecursiveOpenStruct fast' do
    REP.times do |index|
      RecursiveOpenStruct.new(HASH)
    end
  end

  x.report 'OpenStruct slow' do
    REP.times do |index|
       OpenStruct.new(name:"User", age:21)
    end
  end

  x.report 'OpenStruct fast' do
    REP.times do |index|
       OpenStruct.new(HASH)
    end
  end

  x.report 'Struct slow' do
    REP.times do |index|
       User.new("User", 21)
    end
  end

  x.report 'Struct fast' do
    REP.times do |index|
       User.new(USER, AGE)
    end
  end
end
```

输出：

```
                                     user     system      total        real
RecursiveOpenStruct slow         1.870000   0.020000   1.890000 (  1.884342)
RecursiveOpenStruct fast         1.790000   0.010000   1.800000 (  1.811618)
OpenStruct slow                  1.300000   0.010000   1.310000 (  1.305426)
OpenStruct fast                  1.250000   0.000000   1.250000 (  1.259883)
Struct slow                      0.040000   0.010000   0.050000 (  0.041694)
Struct fast                      0.030000   0.000000   0.030000 (  0.026384)
```

这里可以看出，`RecursiveOpenStruct`的性能比起Ruby自带的`Struct`慢了很多，毕竟一个是Ruby实现的，一个是C实现的哈。其实我在项目里应用`RecursiveOpenStruct`的原因很简单，就是为了提高代码的可读性。

具体效果如下：

```ruby
# old
$SYS.span(class: @oc['order_center']['node_class'], text: @oc['order_center'][@i18n])
.when_present.click

# new (use recursive-open-struct)
$SYS.span(class: @oc.order_center.node_class, text: @oc.order_center[@i18n])
.when_present.click
```

通过应用了`RecursiveOpenStruct`，读取symbol中的值就像使用一个获取值的方法一样，非常自然。现在看来，应用在内部的自动化测试项目没有问题，如果要应用在实际的生产项目中需要好好考量一下。
