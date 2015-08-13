
# Safely evaluating user-defined formulas and calculations

Few things have brought computing power to the masses more effectively than the humble spreadsheet program. By combining data, simple logic, and mathematical operations together in a single environment, it is possible to handle a very wide range of computational needs. Nearly every company you can think of has at least one person who knows how to work with spreadsheets, and some businesses seem to be completely powered by them. The ubiquity of the spreadsheet model of computation is both a blessing and a curse.

On the one hand, its wonderful for someone with business knowledge but only a basic level of technical proficiency to be able to process data and crunch numbers without needing to learn how to build full scale programs. 

Spreadsheets encourage ad-hoc exploration,  and also make it easy to share business processes and results throughout an organization. Because spreadsheets can be written by the people who actually understand the core problem to be solved, there isn't a risk of information being "lost in translation" in the way that often happens in full scale software projects.

On the other hand, there is a dark side to every powerful tool. Every spreadsheet is not only a business document, but is also an awkward data storage mechanism and a computer program. 

Data management practices around spreadsheets can be horrifyingly bad, to the point where keeping track of what files have what data, and which are up to date, which are kept for archival purposes, and which should be discarded entirely is nearly impossible to do without a very carefully designed business process in place.

Also, the tendency of spreadsheets to become "almost-but-not-quite" full featured computer programs over time can lead to some major maintenance headaches, even if a business manager wouldn't necessarily think of things that way.  There are things we take for granted as software developers (i.e. reusability, testability, changeability, revision control)  that wouldn't even enter into a spreadsheet author's mind even as they started to face major friction in their workflow.

The solution to this problem is not to seek some sort of killer application to revolutionize the way spreadsheets are built, nor is it to train those who write spreadsheets to become "full stack" programmers. 

Instead, what is needed is to find a proper middle ground whenever a business problem grows beyond the scope of what can be done by some smart folks who know how to write formulas and design basic computational models,  but can't or don't want to maintain full scale software applications on their own.

In this article, we'll explore one possible solution to the "spreadsheet scaling problem"... embedding a formula evaluator into a business application so that users can write their own Excel-like scripts to do computations, while programmers handle the data management and core business processes using standard application development practices and tools.


----------

Start w. a high level explanation of the two different kinds of boxes we're looking at, and how to compute their materials.

(i.e. a rectangular box is simple (but not TOO simple, remember the corners!), but what about a much more complicated example?, then explain yin-yang).

-----------

pi * diameter          = circumference = outer wall
pi * (diameter / 2) = length of inner wall  (obtained  by construction graph)
(pi * diameter) / 2  = pi * (diameter / 2 ) [associative property]
(pi * diameter) / 2 = circumference / 2


------------

* Show completed tables and describe layout of "Calm" and "Yinyang" gardens, and describe the customizable parameters for each.
* Break down the materials list and relevant formulas.
* 


# Formulas used

## Rectangular garden (calm)

Walls: `2*width*height + 2*length*height` (perimeter of box * height)
Base: `width*height` (area of rectangle)
Sand: `volume*fill` (% of volume of box)

`volume = length * width * height`

## Circular garden (yinyang)

Walls: `pi*diameter*height` (circumference of circle * height)
Base: `pi*radius^2` (area of circle)
Black sand: `cylinder * 0.5 * fill` (% of half the cylindrical volume of the circle)
White sand: `cylinder * 0.5 * fill` (% of half the cylindrical volume of the circle)

`cylinder = pi * radius^2 * height`

------------

## What is a formula processor?

* A tool for safely executing user defined computations
* An external domain-specific language for formula parsing and evaluation

### Use cases

* You want to give the user a very simple syntax / computation model to work with.

* You have a business domain where calculations may change much more often than the underlying data model, and/or the calculations aren't known in advance but the data model is well defined.

* You have technical business users that can write formulas but don't want to or can't maintain a full application's codebase.

* You want to create a hard barrier between user-provided computations and the main application to constrain data access, allow for safe execution of untrusted code, etc.

* You want to be able to store computations as data,  and update/execute them at runtime without access to the application’s source code or the need to redeploy code.

## Case Study: Zen garden cost calculator

> See the code for the project at: https://github.com/PracticingDeveloper/dentaku-zen-garden

We’re doing cost calculations for building zen gardens!

The idea being that there will be a common set of supplies for a store that sells zen gardens, and they will be reused across various different styles of gardens.

But the gardens are not sold in fixed sizes, the customer gets to specify the size they want, and the calculator will figure out how much materials are needed, and what those materials will cost (as well as a very basic shipping weight calculation)

So the format we have right now for specifying materials and projects is just some CSV files…

For example, this is the materials list: https://github.com/PracticingDeveloper/formula-engine/blob/master/db/materials.csv

The weight column is a set of formulas… (@rubysolo got them from http://www.mojobob.com/roleplay/weight_chart.html)

In there… you see quantity, which is going to get defined elsewhere.

But here you see the basic idea behind using a formula engine… imagine instead of CSV, you were using a database to store this information.

Elsewhere, you’d define a project, which uses a subset of these materials.

For example, this is a rectangular zen garden box: https://github.com/PracticingDeveloper/formula-engine/blob/master/db/projects/calm.csv

Notice that the names here match that of the names on the materials list, and that the formulas being used are meant to determine the quantity used in the material weight calculations.

So we can have formulas built on top of other formulas… we can look at how that’s wired up later.

And here we have another project… which is a circular zen garden with two colors of sand (suitable for building a yin yang pattern… so I suppose it’s a Taoist garden, actually)

https://github.com/PracticingDeveloper/formula-engine/blob/master/db/projects/yinyang.csv

Notice here… that there are few different kinds of terms being used in the formulas: diameter, radius, cylinder

Some of these would be specified directly as variables… i.e. you’d enter a diameter for the circle

But then others can be defined by global rules… for example, that :radius => “diameter/2"

Or 'cylinder' => '3.1416 * radius^2 * height’ And so in this sense, you can define these kinds of calculations up front, then use them throughout your calculations.

When you glue all of this together, you get a simple workflow that basically asks the customer to pick a project, specify some basic dimensions (i.e. a diameter for a circular project, or a width and height for a rectangular one), and then Dentaku would churn through all of this and convert those measurements into areas and volumes, and then the areas and volumes into weights, and then all of that into costs.

## Why introduce an exernal DSL instead of writing formulas in Ruby?

* Safely evaluating custom Ruby code at runtime presents many security challenges,  at both the application and operating system level. Creating some sort of sandboxed environment is possible, but not trivial.

*  Ruby is a very complex language with rich syntax, and so it is likely to give much more power than what is actually needed for the purposes of supporting simple numeric and logical computation. This means both more for the users to learn, and more opportunities to run into problems.

* Debugging Ruby code that was evaluated at runtime can be very challenging. It is possible to add constraints to mitigate this problem somewhat, but doing so requires careful design consideration.

* Constraining valid and invalid rules in Ruby is challenging because of the many ways that Ruby allows you to circumvent its own access controls. By contrast, an external DSL can bake those rules directly into the language.

* Once you decide to allow user-defined Ruby scripts to be executed at runtime, you will inevitably end up spending as much time and effort thinking through that design decision as you do on building the actual computational model your application needs. In other words, you will end up dealing with many incidental implementation details rather than the functional requirements of the system you're building.

## Other useful Dentaku features

`TODO`

(Not sure where in the article this belongs, if its needed at all)
(Once we make a list, consider going back and working examples into case study, then kill this section)

## What are the challenges and downsides to look out for?

### Caveats for formula processors in general:

* Because all user-defined computations will be run through the formula processor rather than directly integrated with the application, you need to explicitly manage all data access, and also explicitly define any helper functions that you want to expose for use in formulas. This can be beneficial in that it forces you to make explicit choices about what capabilities to provide users with, but it also creates some maintenance overhead.

* By introducing an external DSL with its own syntax and semantics, you are effectively introducing an extra language into the mix, which forces you to translate concepts and structures between your main application and the formula processor. (i.e. data types need to be translated, parsing rules, error states, etc.) -- This is mainly a concern for application maintainers, not the users who are writing formulas.

* Figuring out how to store, load, update, and maintain your formulas becomes an open-ended problem, and you may end up losing some of the certain niceties that come along with a traditional development environment (or you'll need to recreate them)... i.e. things like revision control, syntax highlighting, rich text editing capabilities, debugging tools, etc. 

### Dentaku-specific caveats

Generally speaking, Dentaku works best with simple numerical formulae and small data sets (i.e. you could comfortably run computations in a loop with a few thousand data points, but not millions).

* Current parser is an unoptimized regular-expresssion based implementation, and Dentaku does not have a well-defined intermediate representation of a parsed formula.

* Parsing/evaluation is done in one step without caching, so the parsing cost is incurred every time a formula is executed. (i.e. the formula are always intrepreted rather than compiled)

* Dentaku does not currently support any collection data types, so doing things like `SUM`, `AVERAGE`, etc. on an array of numbers isn't possible... this needs to be done at the Ruby level instead.
