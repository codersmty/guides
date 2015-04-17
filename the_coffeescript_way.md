# CoffeeScript

Comments
==========

``coffeescript
# single line comment

###
  multiline comment, perhaps a LICENSE.
###
``

Variables and Scope
===================

```coffeescript
# All variables are by default scoped locally
  my_var = "Hello world"
  pi = 3.1416

# Attach a variable to the global or window object to make it global
  window.my_var = "Hello world"
```

Functions
=========

```coffeescript
# use the slim arrow to declare a function
func = -> "bar"

sumXtoTen = (x) ->
  10 + x

# arguments are specified in parenthesis before the arrow
times = (a, b) -> a * b

# you can specify default arguments
times = (a = 10, b = 15) -> a * b

# use splats to receive N arguments via an array
sum = (nums...) ->
  result = 0
  nums.reduce (total, num) -> total + num

# you can avoid parethesis in function calls with at least 1 argument
alert "hello world"
console.log "this is the end"

# on the contrary you cannot call a function without parenthesis and zero
# arguments

# this fails
alert
# this is O.K.
alert()

# use the fat arrow to bound a function call to the local context
# mostly used in callbacks
# avoid using the conventional self = this is not very coffeescriptish

this.clickHandler = -> alert "clicked"
element.addEventListener "click", (e) => this.clickHandler(e)
```

Objects Literals and Array Definition
=====================================

```coffeescript
# Object literals

  # regular Javascript syntaxis
  object1 = {one: 1, two: 2}

  # without braces
  object2 = one: 1, two: 2

  # using new lines instead of commands
  object3 =
    one: 1
    two: 2

# Arrays

  # avoid the trailing comma
  array1 = [1, 2, 3]

  # using new lines
  array2 = [
    1
    2
    3
  ]
```

Flow Control
============

```coffeescript
# ruby style one liners if and unless
alert("Hello world") if sayHi is yes
alert("Hello world") unless sayHi isnt yes

# one liner if then or ternary
if true isnt true then "Panic"
if true isnt true then "Panic" else "Everything just works!"

# use plain old english for not, is, isnt, and, or
if not true then "Panic"
if true is true then "Everything's gonna be allright"
if true isnt true then "We'll burn in hell!!"
if total is 0 and discount is 0 then "God save us all!"
```

String Interpolation
=====================

```coffeescript
  # oh good old ruby and multiline strings also!
  favoriteColor = "Blue. No, yel..."
  question = "Bridgekeeper: What... is your favorite color?
              Galahad: #{favoriteColor}
              Bridgekeeper: Wrong!
              "
```

Loops and Comprehensions
========================

```coffeescript
# Iterate over the array's elements
for name in ["Roger", "Roderick"]
  alert "Release #{name}"

# using index
for name, i in ["Roger", "Roderick"]
  alert "#{i} - Release #{name}"

# one liner
release prisioner for prisioner in ["Roger", "Roderick"]

# use keyword when to filter or select
release prisioner for prisioner in ["Roger", "Roderick"] when prisioner[0] is "R"

# use comprehensions to iterate over an object's properties
names =  sam: seaborn, donna: moss
alert("#{first} #{last}") for first, last of names

# use low leve while be aware that it returns an array of results like a map
num = 6
minstrel = while num -= 1
  num + " Brave Sir Robin ran away"

# slicing an array/ string with ranges
firstTwo  = ["one", "two", "three"][0..1]
my = "my string"[0..1]

# includes or checking existence
words = ["rattled", "roudy", "rebbles", "ranks"]
alert "Stop wagging me" if "ranks" in words

# a map
names = (muppet.name for muppet in muppets)

# a filter or select
names = (muppe.name for muppet in muppets when muppet.name isnt "peggy")

# property iteration over an object
object = {one: 1, two: 2}
alert("#{key} = #{value}") for key, value of object
```

Aliases and the Existencial Operator
====================================

```coffeescript
# @ is an alias for this
  this.saviour = true
  @saviour = true

# :: is an alias for prototype
  User.prototype.first = -> @records[0]
  User::first = -> @records[0]

# the ? symbols returns true unless null or undefined
  praise if brian?

# use ? in place of || operator
  velocity = souther ? 40

# use ? before accessing a property, similar to Active Support's try method
blacknight.getLegs()?.kick()

# check if a property is a function and callable
blacknight.getLegs().kick?()

# use it for initialization
hash ?= {}

# Modules: you can create modules using do notation
FormHelpers = do ->
  textFieldTag: () ->
  selectTag: () ->
  labelTag: () ->
```

Classes
===========

```coffeescript
# basic class structure
class Animal
  # constant
  @BASE_PRICE: 100.00

  # basic initializer or constructur
  constructor: (args = {}) ->
    # instance variables
    @name = args.name
    @species = args.species

  # instance method
  sell: (customer) ->
    # accessing constant
    @constructor.BASE_PRICE

  # class method
  @find: (name) ->

# use fat arrow in callbacks to keep the right context
class Animal
  price: 5

  sell: =>
    alert "Give me #{@price} shillings!"

animal = new Animal
$("#sell").click(animal.sell)

# Inheritance and Super

# use extends to inherit all the instance properties
class Animal
  constructor: (@name) ->

  alive: ->
    false

class Parrot extends Animal
  constructor: ->
    # use super to call the parents method
    super("Parrot")

    dead: ->
      not @alive()

# Mixins to share common behavior or roles
# there's no native support in coffee but you can implement it

extend = (obj, mixin) ->
  obj[name] = method for name, method of mixin
  obj

include = (klass, mixin) ->
  extend klass.prototype, mixin

# usage
include Parrot,
  isDeceased: true

(new Parrot).isDeceased
```

Rails, Coffeescript and KnockoutJS

# First: Define an MV structure for your javascript code under
# app/assets/javascripts

# money_collector == your_appname
money_collector/
  models/
  views/
  utils/
  money_collector.coffee

# Second: Define global variables/namespaces to load each class
# inside my_appname.coffee

```coffeescript
#=require_self
#=require jquery
#=require knockout
#=require_tree

window.MoneyCollector:
  Models: {}
  Views:
    Users: {}
    Posts: {}
  Utils: {}
```

# Third: Place every model, view or utility class in their folder
# use class MoneyCollector.Models.User, class MoneyCollector.Views.Users.Create
# pattern to defined each class name and keeping them under their scopes

class MoneyCollector.Models.User
  constructor: (args = {}) ->

class MoneyCollector.Views.Users.Show
  constructor: ->

# Fourth: Create objects and apply knockout bindings in the views show.html.erb

<!-- ko with: user --!>
  <span data-bind="text: firstName"></span>
  <span data-bind="text: lastName"></span>
<!-- /ko --!>

<script>
  ko.applyBindings(new MoneyCollector.Views.Users.Show());
</script>

# Fifth and last, require your_appname.coffee script inside your rails application.js
//=require money_collector
