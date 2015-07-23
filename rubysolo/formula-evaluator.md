# Safely evaluating user-defined formulas and calculations

There are some applications where you need to support lots of small and simple calculations, the sort of stuff that anyone familiar with Excel would be comfortable writing.

If you take these simple computations and put them all directly into an application's source code, it means that whenever a small change is needed, a programmer needs to modify and redeploy the application. If we assume that there is at least one user of the application who is comfortable writing Excel-style formulas to cover the basic computations related to their business, giving them a way to do this kind of work without modifying the application's source code can be very useful.

To support this level of flexibility at runtime, some sort of runtime scripting capability is needed, whether it means storing code in a database or uploading files with custom calculations to extend the functionality of a deployed application. However, it is risky for a number of reasons to use Ruby code for this purpose:

* Safely evaluating custom Ruby code at runtime presents many security challenges,  at both the application and operating system level. Creating some sort of sandboxed environment is possible, but not trivial.
*  Ruby is a very complex language with rich syntax, and so it is likely to give much more power than what is actually needed for the purposes of supporting simple numeric and logical computation. This means both more for the users to learn, and more opportunities to run into problems.
* Debugging Ruby code that was evaluated at runtime can be very challenging. It is possible to add constraints to mitigate this problem somewhat, but doing so requires careful design consideration.
* Once you decide to allow user-defined Ruby scripts to be executed at runtime, you will inevitably end up spending as much time and effort thinking through that design decision as you do on building the actual computational model your application needs. In other words, you will end up dealing with many incidental implementation details rather than the functional requirements of the system you're building.


------------------------------------------------------------------------------------------------------------

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

## SETUP SCENARIO:
Startup business model is the "Etsy" of DIY zen gardens. (Market viability left as an exercise for the reader) Users of the site can post projects that define a shopping list for the materials to build their particular design of zen garden, but the purchaser can customize the dimensions of the garden, so quantities must be calculated at runtime based on the user-supplied dimensions.

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