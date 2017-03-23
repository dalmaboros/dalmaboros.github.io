---
layout: post
title:  "How I Built My First Ruby Gem"
date:   2017-03-23 18:24:48 -0400
---


I built a Ruby command line interface (CLI) gem application as my first portfolio project in the [Online Web Development Program](https://flatironschool.com/programs/online-web-developer-career-course/) at [Flatiron School](https://flatironschool.com/). I created a program that displays the weekly schedule of bills to be debated on the United States House of Representatives floor. It pulls data from the [House Floor](http://docs.house.gov/floor/) web page, as well as from individual bill pages on [Congress.gov](https://www.congress.gov/). Interestingly, the House Floor schedule does not provide links to the individual bill pages. Fortunately, my program does!

Here is a video demonstrating how the gem works:

<iframe width="560" height="315" src="https://www.youtube.com/embed/6LL6yv0-u-4" frameborder="0" allowfullscreen></iframe>

# What is a gem??

A gem is a self-contained and distributable bundle of code in the form of either an application (program) or a library of code. So a gem can be one of two things: 1) a stand-alone program that functions autonomously, or 2) a library of code (i.e. a set of reusable functions) you can integrate into your own application. Libraries of code can save you, the developer, time spent writing code from scratch that may have already been written by someone else before. For example, my first Ruby gem makes use of the [Nokogiri](https://rubygems.org/gems/nokogiri/versions/1.6.8) gem.

Nokogiri adds to my application the (reusable) functionality of parsing HTML (and XML, SAX, and Reader) files. Theoretically,  I myself could have written the code that parses HTML files (*ha!*) for my application, but that would take a very long time, and it probably wouldn't have made my program any better. It's akin to making pasta from scratch when you can buy fresh pasta from the grocery store. Or spinning your own wool when you can buy yarn at the craft store. If all you want is spaghetti dinner, buying that ready-made pasta saves you a lot of time and effort. Likewise, you don't need to make yarn if you just want to knit a scarf. 

I didn't set out to create an HTML-parser, I wanted to make a program that displays the House floor schedule. Nokogiri has been around for over 8 years, it's open source and has had numerous contributors, so chances are it's a pretty legit, high-quality gem (spoiler alert: it is). I trust Nokogiri more than I trust myself attempting to write the same functionality it provides. 

**Why did I make a gem?**

If gems are useful for bundling libraries of code, why did I bother packaging my stand-alone application into a gem, instead of leaving it a plain Ruby program? For one, gems are a widely-used tool by Ruby developers, so it's probably a good idea to get some practice and become familiar with making one. Second, public gems are hosted and distrubuted in the RubyGems [public repository](https://rubygems.org/gems) at [RubyGems.org](https://rubygems.org/), a popular resource for Ruby developers. Instead of having my program live in isolation on my local drive, or in obscurity on my personal website, releasing my gem on RubyGems shares it with the world, gives it exposure, and gives it opportunity to be contributed to by other developers.

### How did I make my gem?

I used the [bundler](https://rubygems.org/gems/bundler) gem to create my house-floor-bills gem. To check if bundler is installed:

`$ gem list`

If it's not already installed, type into your terminal:

`$ gem install bundler`

You need to choose a name for your project before you make your gem. Since my gem displays the schedule of bills to be debated on the House of Representatives floor, I named it "house_floor_bills." Convention seems to be that you separate multiple words in a gem name with underscores. 

`$ bundle gem house_floor_bills`

Next I created an executable file "house-floor-bills" in the `bin` directory. In this file I included a shebang line with a directive to the Ruby interpreter:

```
#!/usr/bin/env ruby
```

And I declared some dependencies:

```
require "bundler/setup"
require "house_floor_bills"
```

I needed to grant this file executable permissions, so in the terminal I typed:

`$ chmod +x house-floor-bills`

At this point I created a remote repository on GitHub, and [added the URL](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/) for this remote repository to my local repository through the terminal:

`$ git remote add origin URL`
`$ git push -u origin master`

Next, I declared the environment load dependencies. The `lib/house_floor_bills.rb` file acts as the environment file where I declare my program's requirements: 

```
require "open-uri"
require "nokogiri"
require "pry"

require_relative "./house_floor_bills/version"
```

*Note that `lib/housefloorbills/version.rb` was created automatically by bundler when I created the gem.*

To successfully include [Nokogiri](https://rubygems.org/gems/nokogiri) and [Pry](https://rubygems.org/gems/pry) in my project, I needed to also add the dependancies in my `house_floor_bills.gemspec` Gemspec file:

```
spec.add_development_dependency "pry"
spec.add_dependency "nokogiri"
```

Now I was ready to start writing my program code. In reality, I went through several iterations of stubbing, hard-coding, and refactoring until I had what I think is well (enough) written object-oriented code. I won't go into the details of that process, but I'll talk about what I ended up with. 

### How did I code my program?

My program shows the weekly schedule of bills for consideration on the House Floor. It scrapes the data from the [US House of Representatives](http://docs.house.gov/floor/) website and presents it in a command line interface (CLI). I needed four collaborating objects:

1. A scraper object
2. A schedule object
3. A bill object
4. A CLI object

The scraper object is responsible for scraping both schedule data and bill data from the web. The schedule object is responsible for containing bills. The bill object is responsible for containing its own attributes. Finally, the CLI object is responsible for providing an interface for the user to access the schedule and bill data. So the scraper knows about both the schedule and bill objects, the schedule knows about the bill object, the bill only knows about itself, and the CLI only knows about the schedule object.

I started with the CLI object. I defined the `HouseFloorBills::CLI` class within the `lib/house_floor_bills/cli.rb` file:

```
class HouseFloorBills::CLI

  def call
    welcome
    list_bills
    menu
  end

  def welcome
    puts "\n************* Welcome to House Floor Bills *************"
    puts "\nSee which bills are scheduled for debate on the House of Representatives floor."
  end

  def list_bills
    ...
  end

  def menu
   ...
  end

end
```

Back in my `bin/house-floor-bills` executable file, I called the newly defined `#call` method:

```
HouseFloorBills::CLI.new.call
```

This is really the only method called in `cli.rb`, and this method initiates the whole program upon opening `bin/house-floor-bills`. But in order for this to work I needed to tell my program that this file exists, and it is dependent upon it. So in my `lib/house_floor_bills.rb` file I declared the dependency:

```
require_relative "./house_floor_bills/cli"
```

I repeated this for the other three objects:

1. I defined the `HouseFloorBills::Scraper` class in a file named `lib/house_floor_bills/scraper.rb`.
2. I defined the `HouseFloorBills::Schedule` class in a file named `lib/house_floor_bills/schedule.rb`.
3. I defined the `HouseFloorBills::Bill` class in a file named `lib/house_floor_bills/bill.rb`.

And I declared the dependencies in the `lib/house_floor_bills.rb` file:

```
require_relative "./house_floor_bills/bill"
require_relative "./house_floor_bills/scraper"
require_relative "./house_floor_bills/schedule"
```

The bill object is only responsible for containing its own attributes:

```
class HouseFloorBills::Bill
  attr_accessor :number, :name, :pdf, :url, :sponsor, :committees, :status, :summary

end
```

The schedule object is responsible for containing (reading, writing, and validating) bills:

```
class InvalidType < StandardError; end

class HouseFloorBills::Schedule
  attr_accessor :title, :week, :published, :last_updated

  def initialize
    @bills = []
  end

  def bills
    @bills.dup.freeze
  end

  def find_bill(id)
    @bills[id.to_i-1]
  end

  def add_bill(bill)
    if !bill.is_a?(HouseFloorBills::Bill) #&& !bill.title.empty?
      raise InvalidType, "must be a Bill"
    else
      @bills << bill
    end
  end
end
```

And the scraper object simply instantiates a schedule and assigns data to the schedule and bill attributes.

The code in its entirety is [viewable on GitHub](https://github.com/dalmaboros/house-floor-bills-cli-gem/). 

### How did I release my gem?

Once my program was completed, I was ready to publish. I really wanted my gem to be able to be executed with a simple command in the terminal, such as `$ house-floor-bills`. To achieve this, I changed the following line of code in the Gemspec file:

```
spec.executables = ["house-floor-bills"]
```

I also removed this line of code:

```
spec.bindir = "exe"
```

I updated these lines in the Gemspec:

```
spec.summary = %q{Bills to be considered on the house floor.}
spec.description = %q{Legislative bills scheduled for debate this week on the United States House of Representatives floor.}
spec.homepage = "https://github.com/dalmaboros/house-floor-bills-cli-gem"
```

And I commented out this whole section:

```
  # Prevent pushing this gem to RubyGems.org by setting 'allowed_push_host', or
  # delete this section to allow pushing this gem to any host.
  # if spec.respond_to?(:metadata)
  #   spec.metadata['allowed_push_host'] = "TODO: Set to 'http://mygemserver.com'"
  # else
  #   raise "RubyGems 2.0 or newer is required to protect against public gem pushes."
  # end
```

In order to publish a gem, all changes must be commited to your git repository. The gem must also be built and installed locally.

`$ rake build`

This builds `house_floor_bills-0.1.0.gem` in a `pkg` directory. 

`$ rake install pkg/house_floor_bills-0.1.0.gem`

This installs the gem into the local system gems.

`$ gem push pkg/house_floor_bills-0.1.0.gem`

This pushes the gem onto the [RubyGems public repository](https://rubygems.org/gems). You must have [an account](https://rubygems.org/sign_up) on RubyGems.org to release a gem on its platform. On your first gem release, you will be prompted to enter your RubyGem.org credentials. After you enter your password, your gem will be publicly released on [RubyGems.org](https://rubygems.org/)! 🎉

