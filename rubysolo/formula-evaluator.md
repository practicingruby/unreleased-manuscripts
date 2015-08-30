**NOTE: THIS IS CURRENTLY BEING WORKED ON. DON'T EXPECT IT TO BE READY TO READ!**

Imagine that you're a programmer for a company that sells miniature zen gardens, and you've been asked to create a  small calculator program that will help determine the material costs of the various different garden designs in the company's product line.

The tool itself is conceptually simple: The dimensions of the garden to be built will be entered via a web form, and then calculator will output the quantity and weight of all the materials that are needed to construct the garden. 

In practice, the problem is a little more complicated, because the company offers many different kinds of gardens. Even though only a handful of basic materials are used throughout the entire product line, the gardens themselves can consist of anything from a basic rectangular design to very intricate and complicated layouts. For this reason, figuring out how much material is needed for each garden type requires the use of custom formulas.

> MATH WARNING: You don't need to try to understand the geometric computations being done throughout this article, unless you enjoy that sort of thing; just notice how all the formulas are nothing more than basic arithmetic expressions operating on a handful of variables.

The following diagram shows the formulas used for determining the material quantities for two popular products. *Calm* is a simple rectangular garden, while *Yinyang* is a more complex shape that requires working with circles and semicircles. 

![](/uploads/db0611/original/1X/b3dfbcd87dcf8ba9dadc1b5cf926e0d1d1b2a513.png)

In the past, material quantities and weights for new product designs were computed using Excel spreadsheets, which worked fine when the company only had a few basic garden layoutts. But to keep up with the incredibly high demand for bespoke desktop Zen Gardens, the business managers have insisted that their workflow become more Agile by moving all product design activities to a web application in THE CLOUD.

The major design challenge for building this calculator is that it would not be practical to have a programmer update the codebase whenever a new product idea was dreamt up by the product design team. Some days, the designers have been known to attempt at least 32 different variants on a "snowman with top-hat" zen garden, and in the end only seven or so make it to the marketplace. Dealing with these rapidly changing requirements would drive any reasonable programmer insane.

After reviewing the project requirements, you decide to build a program that will allow the product design team to specify project requirements in a simple, Excel-like format and then safely execute the formulas they define within the context of a Ruby-based web application.

Fortunately, the [Dentaku](https://github.com/rubysolo/dentaku) formula parsing and evaluation library was built with this exact use case in mind. Just like you, Solomon White also really hates figuring out snowman geometry, and would prefer to leave that as an exercise for the user.

## First steps with the Dentaku formula evaluator

The purpose of Dentaku provide a safe way to execute user-defined mathematical formulas within a Ruby application.  For example, consider the following code:

```ruby
require "dentaku"

calc = Dentaku::Calculator.new
volume = calc.evaluate("length * width * height", 
                       :length => 10, :width => 5, :height => 3)

p volume #=> 150
```

Not much is going on here -- we have some named variables, some numerical values, and a simple formula: `length * width * height`.  Nothing in this example appears to be sensitive data, so on the surface it may not be clear why safety is a key concern here. 

To understand the risks, you consider an alternative implementation that allows mathematical formulas to be evaluated directly as plain Ruby code. You implement the equivalent code without the use of an external library, just to see what it would look like:

```ruby
def evaluate_formula(expression, variables)
  obj = Object.new
  context = obj.send(:binding)

  variables.each { |k,v| eval("#{k} = #{v}", context) }
  eval(expression, context)
end

volume = evaluate_formula("length * width * height",
                  :length => 10, :width => 5, :height => 3) 

p volume #=> 150
```

Although conceptually similar, it turns out these two code samples are worlds apart when you consider the implementation details:

* When using Dentaku, you're working with a very basic external domain specific language, which only knows how to represent simple numbers, variables, mathematical operations, etc. No direct access to the running Ruby process or its data is provided, and so formulas can only operate on what is explicitly provided to them whenever a `Calculator` object is instantiated.

* When using `eval` to run formulas as Ruby code, by default any valid Ruby code will be executed. Every instantiated object in the process can be accessed, system commands can be run, etc. This isn't much different than giving users access to the running application via an `irb` console.

This isn't to say that building a safe way to execute user-defined Ruby scripts isn't possible (it can even be practical in certain circumstances), but if you go that route, safe execution is something you need to specifically design for. By contrast, Dentaku is safe to use with minimally trusted users, because you have very fine-grained control over the data and actions those users will be able to work with.

You sit quietly for a moment and ponder the implications of all of this. After some very serious soul searching, you decide that for the existing and forseeable future needs of our overworked but relentlessly optimistic Zen garden designers... Dentaku should work just fine.

## Building the web interface

You spend a little bit of time building out the web interface for the calculator, using Sinatra and Bootstrap. It consists of only two screens, both of which are shown below:

![](/uploads/db0611/original/1X/d6012db2aae6eed8b2440d45dee96c753c64f3a4.png)

People who mostly work with Excel spreadsheets all day murmur that you must be some sort of wizard, and compliment you on your beautiful design. You pay no attention to this, because your mind has already started to focus on the more interesting parts of the problem.

## Defining garden layouts as simple data tables

With a basic idea in mind for how you'll implement the calculator, your next task is to figure out how to define the various garden layouts as a series of data tables.

You decide to start with a weight calculation lookup table, as it's one of the more simple computations that needs to be done. In practice, this boils down to minor variants on the `mass = density * volume` equation:

![](/uploads/db0611/original/1X/1ed4d2962e804418a86738098a49c4265e5ee539.png)

This material weight lookup table is suitable for use in all of the product definitions, but the `quantity` value will vary based both on the dimensions of the garden to be built and the physical layout of the garden.

With that in mind, you turn your attention to the tables that determine how much material is needed for each project, starting with the Calm rectangular garden as an example.

Going back to the diagram from earlier, you can see that the quantity of materials needed by the Calm project can be completely determined by the length, width, height, and desired fill level for the sandbox:

![](/uploads/db0611/original/1X/ccd891b1b858d184b3162ff5b74910bd904f9684.png)

You could directly use these formulas in project specifications, but it would feel a little too low-level. Project designers will need to work with various box-like shapes often, and so it would feel more natural to describe the problem with terms like `perimeter`, `area`, `volume`, etc. Knowing that the Dentaku formula processing engine provides support for creating helper functions, you come up with the following definitions for the materials used in the Calm project:

![](/uploads/db0611/original/1X/8028c3fd2969891df022fe4ba6c43c911ce847ab.png)

With this work done, you turn your attention to the Yinyang circular garden project. Even though it is much more complex than the basic rectangular design, you notice that it too is defined entirely in terms of a handful of simple variables -- diameter, height, and fill level:

![](/uploads/db0611/original/1X/e921184d0a1e72fbd3297f51e00cffb3f977c4cc.png)

As was the case before, it would be better from a product design perspective to describe things in terms of circular area, cylindrical volume, and circumference rather than the primary dimensional variables, so you design the project definition with that in mind:

![](/uploads/db0611/original/1X/85ca472049bcf7976ccdf0ccf2db6de22cbf2420.png)

**FIXME: ADD THE COMMON FORMULA TABLE HERE**

At this point, you realize that you have enough raw data to start working on implementing the calculator program. You may need to change the domain model at some point in the future to support more complex use cases, but many different garden layouts can already be represented in this basic format.

> **SOURCE FILES:** calm.csv // yinyang.csv // materials.csv // common_formulas.csv

## Implementing the formula processor

You start off by building a utility class for reading all the relevant bits of project data that will be needed by the calculator. For the most part, this is another boring chore, involving basic manipulation of CSV and JSON data into arrays and hashes.

After a bit of experimentation, you end up implementing the following interface:

```text
>> Project.available_projects
=> ["calm", "yinyang”]

>> Project.variables("calm")
=> ["length", "width", "height”]

>> Project.weight_formulas["black sand"]
=> "quantity * 2.000”

>> Project.quantity_formulas("yinyang")
          .select { |e| e["name"] == "black sand" }
=> [{"name" => "black sand", 
     "formula" => "cylinder_volume * 0.5 * fill", 
     "unit" => "cu cm”}]

>> Project.common_formulas["cylinder_volume"]
=> "circular_area * height”
```

Down the line, the `Project` class will probably read from a database rather than text files, but this is largely an implementation detail. Rather than getting bogged down in ruminations about the future, you shift your attention to the heart of the problem -- the Dentaku-powered `Calculator` class.

**TODO: CLEANUP IRB SAMPLE SHOWN BELOW**

```text
>> calc = Calculator.new("yinyang", :diameter => 20, :height => 5); nil
=> nil
>> pp calc.materials.map { |e| [e['name'], e['quantity'].ceil, e['unit']] }
[["1cm thick flexible strip", 472, "sq cm"],
 ["granite slab", 315, "sq cm"],
 ["white sand", 550, "cu cm"],
 ["black sand", 550, "cu cm"]]
=> [["1cm thick flexible strip", 472, "sq cm"], ["granite slab", 315, "sq cm"], ["white sand", 550, "cu cm"], ["black sand", 550, "cu cm"]]
>> pp calc.shipping_weight
4006
=> 4006
>> calc.shipping_weight
=> 4006
```

```ruby
class Calculator
  def initialize(project_name, params={})
    @params = Hash[params.map { |k,v| [k,Dentaku(v)] }]

    @quantity_formulas = Project.quantity_formulas(project_name)
    @common_formulas   = Project.common_formulas
    @weight_formulas   = Project.weight_formulas
  end

  # ...
end
```

```ruby
# class Calculator

  def materials
    calculator = Dentaku::Calculator.new

    @common_formulas.each { |k,v| calculator.store_formula(k,v) }
    
    @quantity_formulas.map do |material|
      amt = calculator.evaluate(material['formula'], @params)

      material.merge('quantity' => amt)
    end
  end
```

This is some text

```ruby
# class Calculator

  def shipping_weight
    calculator = Dentaku::Calculator.new

    # Sum up weights for all materials in project based on quantity
    materials.reduce(0.0) { |s, e| 
      weight = calculator.evaluate(@weight_formulas[e['name']], e)

      s + weight
    }.ceil
  end
```

----

* Performance implications
* Error handling (could be better or worse, but going to be different)
* Second system, w. its own semantics
* Limited range of types to work w. (feature and limitation)

> A COMPLETE, RUNNABLE SAMPLE PROJECT IS AVAILABLE AT [REPO]

---

Possibly recommend exercises.

---

**TODO: Everything else discussed in the [article outline](http://discourse.practicingruby.com/t/outline-safely-evaluating-user-defined-formulas-and-calculations/40).**