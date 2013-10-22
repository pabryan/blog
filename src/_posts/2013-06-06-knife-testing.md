---
layout: post
title: "Testing Chef Cookbooks with Knife"
comments: true
date: 2013-06-06 08:43
categories: ['chef', 'chef broiler plate', 'devops']
twitter: [chef, devops, chefsurvivalguide, knife]
---

*After a brief hiatus into other topics, we return with the seventh installment in a [series on the development of Chef Broiler Plate](http://neverstopbuilding.net/blog/categories/chef-broiler-plate/) in which we go over setting up a robust, TDD framework for Chef cookbook development.*

In the [last post](http://neverstopbuilding.net/foodcritic/) we added the Foodcritic to the framework in order to verify cookbook quality. This time we will add the ability to run the Knife cookbook testing utility and hook it up to CI. Thanks must go to Nathan Harvey for writing a [helpful article](http://www.nathenharvey.com/blog/2012/07/06/mvt-knife-test-and-travisci) on Knife testing.

##Prepare a Knife CI Configuration
Knife will need CI specific configuration so we will create an alternative `knife.rb` file under our test directory:

{% codeblock lang:ruby %}
current_dir             = File.dirname(__FILE__)
cookbook_path           ["#{current_dir}/../cookbooks"]
cache_type 'BasicFile'
cache_options(:path => "#{ENV['HOME']}/.chef/checksums")
{% endcodeblock %}

##Adding the Rake Task
Now add a simple rake task `knife_test` to our rake file to run the cookbook testing command and another rake task `knife_test_ci` that will use our customized `knife.rb` file:

{% codeblock lang:ruby %}
desc "Runs knife cookbook test against all the cookbooks."
task :knife_test do
  sh "bundle exec knife cookbook test -a"
end

desc "Runs foodcritic against all the cookbooks."
task :knife_test_ci do
  sh "bundle exec knife cookbook test -a -c test/knife.rb"
end
{% endcodeblock %}

The `-a` means that this will test all cookbooks.

##Update the Build Task
Now is about the time we will need to split our build task in to `build` and `build_ci` because certain ci tasks will have different reporting outputs, or need different configuration. While developing we should be running the build task all the time to do testing, so our build task will tend to grow toward something that is more useful for humans while that build_ci task will be better suited to machines for reporting or notifications.

Right now the only difference I am making is having the `build_ci` task use the `knife_test_ci` task in its run list:

{% codeblock lang:ruby %}
desc "Builds the package for ci server."
task :build_ci do
  Rake::Task[:knife_test_ci].execute
  Rake::Task[:foodcritic].execute
  Rake::Task[:chefspec].execute
end
{% endcodeblock %}

Note that the order is the reverse of the development of these tasks, this is because I want the suite to "fail fast" either for a syntax error or rule violation before it tries to run the specs.

##Update the CI Configuration
We also need to update the Travis file to use our new ci task, simple:

    script: "bundle exec rake build_ci"

Hurray! Now both our CI server and standard build process will run the "knife cookbook test" to ensure our cookbooks are top notch!

#Coming up…
In the next post we will set up a workflow management tool and integrate it into our processes.

{% c2a icon:"K" title:"Hungry for more? Get the Book!" action:"Check it out!" link:"https://leanpub.com/Chef-survival-guide?utm_source=nsb&utm_medium=blog&utm_campaign=Testing+Chef+Cookbooks+with+Knife" label:"Chef Survival Guide" %}
This and future posts related to the build out of Chef Broiler Plate are  consolidated in "The Chef Survival Guide." The book includes more detail and examples.
{% endc2a %}