---
title: "Chefspec Basics - Initial notes"
date: 2014-10-31 16:10
classes: wide
categories: general
---

This post sums up the various aspects of writing Chefspec tests so that when I 
forget I can quick check here. Hopefully this will be of use to others. All examples 
refer to [an example cookbook](https://github.com/ftclausen/chefspec_example)
on Github.

# Layout

Chefspec conforms to the rspec layout and requires your tests to end with 
"_spec.rb" (unless you override that) and placed under the "spec" directory. For 
example (not all files shown)

    chefspec_example/
    ├── README.md
    ├── files
    ├── libraries
    ├── metadata.rb
    ├── providers
    ├── recipes
    ├── resources
    ├── spec
    │   ├── recipes
    │   │   └── default_spec.rb
    │   └── spec_helper.rb
    └── templates

# Spechelper File

Since you need to require some common libraries for every test it makes sense to
put these in a separate file that can easily be included in all tests

{% highlight ruby %}
# spec/spec_helper.rb
require 'chefspec'
require 'chefspec/berkshelf'
ChefSpec::Coverage.start!
{% endhighlight %}

I've also activated the test coverage checker on the last line. It gives you a
nice compliment if you get 100% test coverage :-)

# Anatomy of a Test

First I'll post a complete test then break it down

{% highlight ruby %}
# spec/recipes/default_spec.rb
require 'spec_helper'

describe 'chefspec_example::default' do
  let(:chef_run) do
    ChefSpec::ServerRunner.new(platform: 'ubuntu', version: '12.04')
      .converge('chefspec_example::default')
    end

  it 'should create a chefspec_example resource' do
    expect(chef_run).to go_and_doit('widget1')
  end

  it 'should create the template' do
    expect(chef_run).to create_template('/etc/example.conf')
      .with(
        owner: 'example',
        group: 'example',
        mode: '0755'
      )
  end

  it 'should create the file' do
    expect(chef_run).to create_cookbook_file('/var/lib/example.dat')
      .with(
        owner: 'example',
        group: 'example',
        mode: '0755'
      )
  end

  it 'should install the package' do
    expect(chef_run).to install_package('example')
  end

  it 'should install the service' do
    expect(chef_run).to enable_service('example')
  end
end
{% endhighlight %}

#### Line 1 - Require helper file
This is simply to get the appropriate libraries loaded
{% highlight ruby %}
require 'spec_helper'
{% endhighlight %}

#### Line 4 - Create an "example group" 
This is how you can group a set of related tests using *describe*
{% highlight ruby %}
describe 'chefspec_example::default' do
{% endhighlight %}

#### Line 10 Onwards - The tests
Then come the meat of the matter and the whole point of this exercise, namely,
the test themselves. Taking the file test as an example :

{% highlight ruby %}
  it 'should create the file' do
    expect(chef_run).to create_cookbook_file('/var/lib/example.dat')
      .with(
        owner: 'example',
        group: 'example',
        mode: '0755'
      )
  end
{% endhighlight %}

The **it** keyword describes an **example** which is another way of saying 
"the thing I want to test". In this case we are testing for the presence of a static
file with the given owner of "example" and mode "0755". The other tests (aka **examples**) 
test for common Chef attributes such as templates,
packages and services. 

Note that we can also test our own custom resources (in this case a
[LWRP](https://docs.getchef.com/lwrp.html) called "chefspec_example"). This 
was done in a way very similar to the built in Chef resources 

{% highlight ruby %}
it 'should create a chefspec_example resource' do
expect(chef_run).to create_chefspec_example('widget1').with(
  action: [ :doit ]
)
end
{% endhighlight %}

in the above snippet we are testing for the presence of the "chefspec_example" resource 
that has the "doit" attribute.

More on how to define custom matchers shortly.

# Actually Executing the Tests

To run the tests simply go to the base of your cookbook and call rspec since
Chefspec is an extension of rspec :

    user@example.com chefspec_example $ rspec

This will run your tests and hopefully you'll be basking in 
show you glorious green. You can get more information about passing tests 
(to watch the progress) 

    user@example.com chefspec_example $ rspec -f d

# Testing your own LWRPs

You may have noticed that you can test that things happened, such as

* create_cookbook_file
* install_package
* enable_service

how might I test my own resource you ask? That is done with **custom matchers**. 
These are placed in the **libraries** directory of your cookbook with the special name 
**matchers.rb**. Thus **chefspec_example/libraries/matchers.rb**. The contents of which
takes the form

{% highlight ruby %}
# libraries/matchers.rb
if defined?(ChefSpec)
  def create_chefspec_example(resource_name)
    ChefSpec::Matchers::ResourceMatcher
      .new(:chefspec_example, :doit, resource_name)
  end
end
{% endhighlight %}

The first thing to note is that all custom matcher libraries need to be wrapped in a 
check for Chefspec. Then we create a matcher method which will match the test we defined 
above using **create_chefspec_example**.

# Other Observations

## Fauxhai

There are lots of examples on the 'net that show Fauxhai being explicitly loaded
like this

{% highlight ruby %}
# The "before" stuff does not seem to be required
describe 'chefspec_example::default' do
  before do
    Fauxhai.mock(platform: 'ubuntu', version: '12.04')
  end
{% endhighlight ruby %}

however *this does not seem necessary*. All you need to do is create your Runner instance 
with the desired platform and version; Fauxhai will then automagically be invoked

{% highlight ruby %}
    describe 'chefspec_example::default' do
      let(:chef_run) do
        ChefSpec::ServerRunner.new(platform: 'ubuntu', version: '12.04')
          .converge('chefspec_example::default')
        end
    # rest of test omitted
{% endhighlight ruby %}

## Quoting of numbers

For permissions you need to be consistent when quoting numbers specified in resource
attributes. Either quote both the test and recipe or neither; for example, 

{% highlight ruby %}
# recipes/default.rb
template '/etc/example.conf' do
  source 'example.conf.erb'
  owner 'example'
  group 'example'
  mode '0755'
end
{% endhighlight ruby %}

{% highlight ruby %}
# spec/recipes/default_spec.rb
it 'should create the template' do
expect(chef_run).to create_template('/etc/example.conf')
  .with(
    owner: 'example',
    group: 'example',
    mode: '0755'
  )
end
{% endhighlight ruby %}

Notice that both examples are quoted. Having one quoted and the 
other not will lead to failed tests since it is technically a 
different resource.

