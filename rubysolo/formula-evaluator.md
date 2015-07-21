We’re doing cost calculations for building zen gardens!

The idea being that there will be a common set of supplies for a store that sells zen gardens, and they will be reused across various different styles of gardens.

But the gardens are not sold in fixed sizes, the customer gets to specify the size they want, and the calculator will figure out how much materials are needed, and what those materials will cost (as well as a very basic shipping weight calculation)

So the format we have right now for specifying materials and projects is just some CSV files…

For example, this is the materials list: https://github.com/PracticingDeveloper/formula-engine/blob/master/db/materials.csv

The weight column is a set of formulas… no clue if @rubysolo tried to pick realistic values for those, but it’s unimportant.

In there… you see `quantity`, which is going to get defined elsewhere.

But here you see the basic idea behind using a formula engine… imagine instead of CSV, you were using a database to store this information.

Elsewhere, you’d define a project, which uses a subset of these materials.

For example, this is a rectangular zen garden box: https://github.com/PracticingDeveloper/formula-engine/blob/master/db/projects/calm.csv

Notice that the names here match that of the names on the materials list, and that the formulas being used are meant to determine the `quantity` used in the material weight calculations.

So we can have formulas built on top of other formulas… we can look at how that’s wired up later.

And here we have another project… which is a circular zen garden with two colors of sand (suitable for building a yin yang pattern… so I suppose it’s a Taoist garden, actually)

https://github.com/PracticingDeveloper/formula-engine/blob/master/db/projects/yinyang.csv

Notice here… that there are few different kinds of terms being used in the formulas: `diameter`, `radius`, `cylinder`

Some of these would be specified directly as variables… i.e. you’d enter a diameter for the circle

But then others can be defined by global rules… for example, that `:radius => “diameter/2"`

Or ` 'cylinder' => '3.1416 * radius^2 * height’`
And so in this sense, you can define these kinds of calculations up front, then use them throughout your calculations.

When you glue all of this together, you get a simple workflow that basically asks the customer to pick a project, specify some basic dimensions (i.e. a diameter for a circular project, or a width and height for a rectangular one), and then Dentaku would churn through all of this and convert those measurements into areas and volumes, and then the areas and volumes into weights, and then all of that into costs.

That’s pretty much what we have so far.

## Limitations

 For right now just as a tentative plan, I think we’ll create a section about “limitations” and include in it things like performance, the lack of aggregate functions, and whatever else we come up with.
 
 Current state: If you're dealing with no more than thousands or tens of thousands of data points, things will probably work fine (e.g. similar to a typical Excel sheet)... if you're dealing with millions of records, processing
 is going to be pretty slow (verify w. benchmark)
 
 ## Implementation details
 
 - Parser is plain Ruby (as opposed to something like Racc)
 - Parsing and evaluation done in one step, could probably optimize significantly by introducing precompilation
 - Built-ins are not currently implemented using extension point for custom function definitions, but probably could be.
 - Missing a list (Array) datatype.


--------------------------------------------------

@srdjan: I built an app that replaced a bunch of excel spreadsheets with embedded formulas that the business sharing with dropbox.  The requirement was to keep the existing formulas as much as possible and allow for the end-users to update the formulas as needed without code changes.

rubysolo [2:12 PM]
The domain was construction, so the formulas were for calculating things like the number of sheets of drywall required to cover walls given a perimeter and ceiling height.

rubysolo [2:12 PM]
@gregory_brown: they should follow the same rules — basically some identifier and a list of arguments in parentheses.

gregory_brown [2:13 PM] 
Nice! Then yeah, it’d be great to make the system use its own extension points

rubysolo [2:13 PM] 
stepping away for a minute...

gregory_brown [2:13 PM] 
No problem, thanks for your help!

gregory_brown [2:14 PM]
@srdjan: I’ve run into this problem a few times myself… in a publishing company’s application where we ended up having a Treetop parser or something, and in a food distribution domain where you were doing things like shipping cost calculations, fuel cost estimates, etc.

gregory_brown [2:15 PM]
It makes sense when you have technical users (i.e. business analysts), but when their experience level isn’t much higher than that of using some Excel functions.

paulh16 [2:15 PM] 
joined #general

gregory_brown [2:15 PM] 
A formula engine basically gives them a sandbox to write certain kinds of dynamic calculations in, without security concerns because it’s a parsed external DSL rather than evaluated code.

gregory_brown [2:16 PM]
And in my case… it usually meant a bunch of complex queries and data transformations were being done in the Ruby/Rails app, and then rolled up and sanitized bits of that data were being passed into the formula engine to be operated on.

srdjan [2:16 PM] 
right

gregory_brown [2:17 PM] 
In one situation we actually embedded a Lua interpreter (locked down) into a Rails app, and then preloaded a bunch of custom functions and some data tables

gregory_brown [2:17 PM]
then the output of that would be converted back into Ruby values and used within the Rails app./

srdjan [2:17 PM] 
interesting

gregory_brown [2:18 PM] 
What @rubysolo has done here with Dentaku is give folks a formula parser that has some builtins and can be extended with your own functions, written in pure Ruby w. no dependencies

gregory_brown [2:18 PM]
Which is something I wish I had when I did this a few years ago, because that would have saved me from building brittle, creaky underplumbing.

srdjan [2:19 PM] 
I was going to ask, heh, why not use Lisp-style evaluation: `(* 2 (+ 1 3))`

srdjan [2:19 PM]
but i'm guessing that switching the evaluation style would freak out business analysts

gregory_brown [2:19 PM] 
No harm in doing that… but keep in mind the target audience for this kind of thing is usually an excel user

gregory_brown [2:19 PM]
also, formulas are simple enough and syntax is simple enough that style doesn’t make much of a difference here.

packetmonkey [2:21 PM] 
We do a similar thing in some cases where we try to aggregate costing information but let our business consultants run various projects and options with a front end we hacked together using parslet.

gregory_brown [2:22 PM] 
I guess another theoretical upside of using an existing formula engine would be that it’d be easy to share custom extensions.

gregory_brown [2:23 PM]2:23
And of course… gain from whatever gets added to the core engine by the community.

---------------------------------------------------

## SETUP SCENARIO:
Startup business model is the "Etsy" of DIY zen gardens. (Market viability left
as an exercise for the reader) Users of the site can post projects that define a
shopping list for the materials to build their particular design of zen garden,
but the purchaser can customize the dimensions of the garden, so quantities must be calculated at runtime based on the user-supplied dimensions.

## INTRO:
One option to solve this sort of problem is to allow the users to specify
material quantites as formulas based on the dimensions, and use a formula
evaluation library like Dentaku to perform the runtime calculation. Dentaku
takes a string that represents a formula, along with a hash specifying variable
values to be substituted into the formula, and returns the evaluated result of
the formula.

For example, the formula "length * width" evaluated with the context
{ 'length' => 5, 'width' => 3 } would evaluate to 15.

## MAIN:
Walk through development of solution...
  - high level:
    - model projects as CSV files
    - prompt user to select project
    - use Dentaku to determine required variables
    - prompt user for variable values
    - use Dentaku to solve for quanities, total weight
    - present Bill of Materials, shipping cost

  - potential points to discuss:
    - solving set of multiple dependent formulas automatically
    - performance vs direct Ruby implementation
    - alternate approaches/implementation
    - Dentaku internals
