Slop
====

Slop is a simple option collector with an easy to remember syntax and friendly API.

**NOTE**: Slop is changing! You can view changes in the
[fml branch](https://github.com/injekt/slop/tree/fml). These are the changes
that will be building Slop up to 1.0. There **will** be backwards incompatible
changes between master and the new branch. Until then this README will
display instructions for version 0.3.0

Installation
------------

### Rubygems

    gem install slop

### GitHub

    git clone git://github.com/injekt/slop.git
    cd slop
    rake install

Usage
-----

    s = Slop.parse(ARGV) do
      option(:v, :verbose, "Enable verbose mode", :default => false)
      option(:n, :name, "Your name", true) # compulsory argument
      option(:c, :country, "Your country", :argument => true) # the same thing

      option(:a, :age, "Your age", true, :optional => true) # optional argument
      option(:address, "Your address", :optional => true)   # the same

      # shortcut option aliases
      opt(:height, "Your height")
      o(:weight, "Your weight")
    end

    # using `--name Lee -a 100`
    s.options_hash #=> {:verbose=>false, :name=>"Lee", :age=>"100", :address=>nil}
    s.value_for(:name) #=> "Lee"

    # or grab the Option object directly
    option = s.option_for(:name)
    option.description #=> "Your name"

    # You can also use switch values to set options according to arguments
    s = Slop.parse(ARGV) do
      option(:v, :verbose, :default => false, :switch => true)
      option(:applicable_age, :default => 10, :switch => 20)
    end

    # without `-v`
    s[:verbose] #=> false

    # using `-v`
    s[:verbose] #=> true

    # using `--applicable_age`
    s[:applicable_age] #=> 20

Want these options back in a nice readable help string? Just call `Slop#to_s`
and Slop will return a nice indented option list. You can even add a banner to
the help text using the `banner` method like so:

    opts = Slop.new do
      banner("Usage: foo [options]")

      opt(:n, :name, "Your name", true)
    end

    puts opts

Returns:

    Usage: foo [options]
          -n, --name <name>      Your name

Callbacks
---------

If you'd like to trigger an event when an option is used, you can pass a
block to your option. Here's how:

    Slop.parse(ARGV) do
      opt(:v, :version, "Display the version") do
        puts "Version 1.0.1"
        exit
      end
    end

Now when using the `--version` option on the command line, the trigger will
be called and its contents executed.

Casting
-------

If you want to return values of specific types, for example a Symbol or Integer
you can pass the `:as` attribute to your option.

    s = Slop.parse("--age 20") do
      opt(:age, true, :as => Integer) # :int/:integer both also work
    end
    s[:age] #=> 20 # not "20"

Smart
-----

Slop is pretty smart when it comes to building your options, for example if you
want your option to have a flag attribute, but no `--option` attribute, you
can do this:

    opt(:n, "Your name")

and Slop will detect a description in place of an option, so you don't have to
do this:

    opt(:n, nil, "Your name")

You can also try other variations:

    opt(:name, "Your name")
    opt(:n, :name)
    opt(:name)

Lists
-----

You can of course also parse lists into options. Here's how:

    s = Slop.parse("--people lee,injekt") do
      opt(:people, true, :as => Array)
    end
    s[:people] #=> ["lee", "injekt"]

You can also change both the split delimiter and limit

    s = Slop.parse("--people lee:injekt:bob") do
      opt(:people, true, :as => Array, :delimiter => ':', :limit => 2)
    end
    s[:people] #=> ["lee", "injekt:bob"]

Arguments as keys
-----------------

If your application is expecting some command line arguments which are not
touched by Slop (for example a filename not given using a flag or option), you
can find these inside of the `Slop#argv` method. But what if you wanted a simple
method of mapping these values to keys, well.. Slop can do that for you.

Lets say your application takes two optional arguments. A filename source, and
a filename destination. Let's see how we might build that in Slop:

    s = Slop.parse(ARGV) do
      banner("Usage: convert <source> <destination>")
      argument(:source, :destination)

      opt(:r, :replace, "Replace the existing file?", default: false, switch: true)
    end

    # Now, using the command line string "foo.txt bar.txt -r" here's how
    # Slop builds your arguments:

    s.arguments #=> { :source => "foo.txt", :destination => "bar.txt" }
    s.options   #=> { :default => true }

    # You can also fetch these arguments just like you can options:
    s[:source] #=> "foo.txt"

Do note that Slop will prioritize options over arguments, so if you have an
option and argument with the same key, `Slop#[]` will return the option value,
not the argument value.

Contributing
------------

If you'd like to contribute to Slop (it's **really** appreciated) please fork
the GitHub repository, create your feature/bugfix branch, add specs, and send
me a pull request. I'd be more than happy to look at it.
