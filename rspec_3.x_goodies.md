# Rspec 3.x
# Code Examples

Describe things and context
============================

```ruby
# Use describe method to define an example group
describe "A new account" do
  it "should have a balance of 0" do
    account = Account.new

    expect(account.balance).to eq Money.new(0, :USD)
  end
end

module Authentication
  describe User, "with no roles assigned" do
    it "is not allowed to view protected content" do; end
  end
end

# context method is an alias for describe, use describe for things and context for context
# use the word when or with when describing a context
describe User do
  context "with no roles assigned" do
    it "is not allowed to view protected content" do; end
  end
end

# use . to describe class methods and # for instance methods
describe "#check_in" do; end
describe ".all" do; end
```

Pending Examples
=====================

```ruby
# simply write it with no block
it "is not allowed to view protetected content"

# specify pending inside the spec example, will stop execution all the following lines
it "should not be mixed with french fries" do
  pending "cleaning out the fryer"

  expect(fryer_with(:onion_rings)).to_not include(:french_fries)
end

# mark pending and executing the code with failsafe
# use it to mark an example and in case is already fixed
# will notice you to remove the pending

describe "an empty array" do
  it "should be empty" do
    pending("bug report 1897") do
      expect([]).to be_empty
    end
  end
end
```

Hooks
================

# prefer let and let! over before(:each)
# let is lazy evaluated, will be set until is accessed the first time
describe Stack do
  context "when empty" do
    let(stack) { Stack.new }

    it "returns an empty array"
  end
end

# use let! to save records to the database
describe Stack do
  context "with valid attributes" do
    let!(stack) { User.create(email: "acolin@incaztech.com") }

    it "creates a new db record"
  end
end

# execute code before, after & around

# use before(:each) to recreate context before each test
describe Stack do
  context "when empty" do
    before(:each) do
      @stack = Stack.new
    end
  end

  context "when almost empty" do
    before(:each) do
      @stack = Stack.new
      @stack.push 1
    end
  end
end

# dont use before(:all) is not recommended to share context among all contexts

# use after(:each) use in some special cases to restore a global value
before(:each) do
  @original_global_value =$some_global_value
  $some_global_value = temporary_value
end

after(:each) do
  $some_global_value = @original_global_value
end

# use after(:all) when closing database connections, browsers, closing sockets

Helper Methods
================

# use spec helper methods for repetitive tasks
module UserExampleHelpers
  def create_valid_user
    User.new(email: "email@example.com", password: "shhhh")
  end

  def create_invalid_user
    User.new(password: "sshh")
  end
end

describe User do
  include UserExampleHelpers
end

# initialize a module of helpers you want to include in all example groups
RSpec.configure do |config|
  config.include(UserExampleHelpers)
end

Shared Examples
==================

# use them when having objects that share behavior or roles

# First define a shared example group with shared_examples_for()
shared_examples "any pizza" do
  it "tastes really good" do
    @pizza.should taste_really_good
  end

  it "is available by the slice" do
    @pizza.should be_available_by_the_slice
  end
end

# Then include the behavior with it_behaves_like()
describe "New York style thin pizza" do
  before(:each) do
    @pizza = Pizza.new(region: "New York", style: "Thin crust")
  end

  it_behaves_like "any pizza"
end

# Expectations

Use expect instead of should
====================

# use should and should_not for expectations, avoid using != it does not work
# use == for value correctness and equal(obj) for object equality

expect(3 * 5).to eq 15

person = Person.create!(name: "David")
Person.find_by(name: "David").should equal(person)
expect(Person.find_by(name: "David"))to equal(person)

expect(3 * 5).to_not eq 16

Floating point calculations
=============================

# test will pass if value ranges from 5.25+-0.005
expect(result).to be_close(5.25, 0.005)

Expect for changes
=======================

# expecting a change in count
expect {
  User.create!(role: "admin")
}.to change{ User.admins.count }

# expecting a change in count by specif value by() to() from()
expect {
  User.create!(role: "admin")
}.to change { User.admins.count }.by(1)

expect {
  User.create!(role: "admin")
}.to change { User.admins.count }.to(1)

# change in ranges
expect {
  User.create!(role: "admin")
}.to change { User.admins.count }.from(0).to(1)

Expecting Errors
====================

# expect to raise any subclass of Exception
expect { do_something_risky }.to raise_error

# to match a specific Exception
expect {
  account.withdraw 75, :dollars
}.to raise_error(InsufficientFundsError)

# to match a specific message sent by the exception as regex
expect {
  account.withdraw 75, :dollars
}.to raise_error(/attempted to withdraw 75 dollars/)

# or both Exception and message as string
expect {
  account.withdraw 75, :dollars
}.to raise_error(InsufficientFundsError,
    "attended to withdraw 75 dollars from an account with 50 dollars")

Expecting a Throw
====================

course = Course.new(seats: 20)
20.times { Course.register Student.new }
lambda {
  course.register Student.new
}.should throw_symbol(:course_full)

Predicate Matches
======================

# A predicate method is one that ends with "?" and returns a Boolean response
# use be_ to match predicate methods

expect(array).to be_empty

# there is a variation using be_a be_an to express better intent

expect(user).to be_an_instance_of(User)
expect(user).to be_a_kind_of(Player)

Expecting true or false
========================

# In ruby everything evals to true except false and nil

# to match things that will evaluate to true and false
expect(true).to be_true
expect(0).to be_true

expect(false).to be_false
expect(nil).to be_false

# to match the exact true and false values
expect(true).to equal(true)
expect(false).to equal(false)

Expecting for things to have or contain other things
======================================================

# Generic collections
expect(request.parameters).to have_key(:id)

# Owned collections
# Before
field.players.select { |p| p.team == home_team }.length.should == 9

# After
expect(home_team).to have(9).players_on(field)

# Unowned collections
expect(collection).to have(30).items

# Use have_at_least, have_at_most or have_exactly
expect(day).to have_exactly(24).hours
expect(dozen_bagels).to have_at_least(12).bagels
expect(internet).to have_at_most(2037).killing_social_networking_apps

Expectations with standard operators
======================================
expect(result).to =~ /some regex/
expect(result).to be == 7
expect(result).to be < 7
expect(result).to be > 7
expect(result).to be >= 7

Generated descriptions
==========================

# use 'specify' instead of 'it' to help you generate descriptions
# the following example will display
# A new chess board
# should have 32 pieces

describe "A new chess board" do
  before(:each) { @board = Chess::Board.new }
  specify { expect(@board).to have(32).pieces }
  specify { @board.should have(32).pieces }
end

Subjectivity
==================

# Explicity subject
describe Person do
  subject { Person.new(birthdate: 19.years.ago }
  # should syntax
  specify { subject.should be_elegible_to_vote }
  # expect syntax
  specify { expect(subject).to be_elegible_to_vote }
end

# Delegate to subject
describe Person do
  subject { Person.new(birthdate: 19.years.ago }
  it { is_expected_to be_elegible_to_vote }
end

# Implicit Subject
# this will create a new instance of RSpecUser
describe RSpecUser do
  it { is_expected_to be_elegible_to_vote }
end

# Rspec Mocks

Test Doubles
===================

# an object that stands in for another object

# creating a double
user_double = double("user")
user_stub = stub("user")
user_mock = mock("user")

# use Method Stubs to return a predefined response
describe Statement do
  it "uses customer's name in the header" do
    customer = double("customer")
    customer.stub(:name).and_return("Aslak")
    statement = Statement.new(customer)

    statement.generate.should =~ /^Statement for Aslak/
  end
end

# creating a double that reseembles an actual object
book = instance_double("Book", pages: 250)

Message Expectations
========================

# A message stub that will raise an error if never called
describe Statement do
  it "logs a message on generate" do
    customer = double("customer")
    allow(customer).to receive(:name) { "Aslak" }
    logger = mock("logger")
    statement = Statement.new(customer)

    expect(logger).to receive(:log).with(/Statement generated for Aslak/)

    statement.generate
  end
end

Test Specific Extensions
==========================

# Partial Stubing. Use it when you want to stub certain behavior of an existing
# object

describe WidgetsController do
  describe "PUT update with valid attributes" do
    it "redirects to the list of widgets" do
      widget = Widget.new
      allow(Widget).to receive(:find) { widget }
      allow(widget).to receive(:update_attributes) { true }
      put :update, id: 37

      expect(response).to redirect_to(widgets_path)
    end
  end
end

# Partial Mocking: set message expectations on the class or instance
describe WidgetsController do
  describe "PUT update with valid attributes" do
    let(:widget) { Widget.new }

    it "finds the widget" do
      allow(widget).to receive(:update_attributes) { true }

      expect(Widget).to receive(:find).with("37") { widget }

      put :update, id: 37
    end

    it "udpates the widget's attributes" do
      allow(Widget).to receive(:find) { widget }

      expect(widget).to receive(:update_attributes) { true }

      put :update, id: 37
    end
  end
end

More on Method Stubs
======================

# One line shortcut
# Use this shortcut to specify values on the double
user_double = double("user", first_name: "Ricky", last_name: "Martinez")

# When having a stub method called more than once with different return values
ages = double("ages")

allow(ages).to receive(:age) do |what|
  return 21 if what == "drinking"
  return 18 if what == "voting"
end

More on messages Expectations
================================

# use count when expecting a specific number of method calls
mock_account.should_receive(:withdraw).exactly(1).times
expect(mock_account).to receive(:withdraw).exactly(1).times

expect(mock_account)to receive(:withdraw).at_most(1).times
expect(mock_account).to receive(:withdraw).at_least(1).times

expect(mock_account).to receive(:withdraw).once
expect(mock_account).to receive(:withdraw).twice

# negative expectation when a method should not be received
expect(network_double).to receive(:open_connection).never
expect(network_double).to_not receive(:open_connection)

# specify expected arguments
expect(checking_account).to receive(:transfer).with(50, savings_account)

# specify expected arguments with 'instance_of'
expect(source_account).to receive(:transfer).
  with(target_account, instance_of(Fixnum))

# receiving an argument that we do not care what it is use 'anything'
expect(source_account).to receive(:transfer)
  .with(anything, 50)

# explicitly specify that any arguments are acceptable with 'any_args'
expect(source_account).to receive(:transfer)
  .with(any_args, 50)

# or if we do not want args we specify 'no_args'
expect(source_account).to receive(:transfer)
  .with(no_args, 0)

# specifying a  hash including certain elements
expect(mock_account).to receive(:add_payment_accounts)
  .with(has_including("Electric" => "123", "Gas" => "234"))
# specifying a hash not including certain elements
  .with(has_not_including("Electric" => "123", "Gas" => "234"))

# Custom Argument Matchers
class GreaterThanMatcher
  def initialize(expected)
    @expected = expected
  end

  def description
    "a number greater than #{@expected}"
  end

  def ==(actual)
    actual > @expected
  end
end

def greater_than(floor)
  GreaterThanMatcher.new(floor)
end

# And we call it like this
allow(calculator).to receive(:add).with(greather(37))

# Returning multiple values in a row, simply specify more than one return value
expect(@network_connection).to receive(:open_connection).
  and_return(nil, nil, nil)

# Raising an exception from a mock
expect(account_double).to receive(:withdraw).and_raise
expect(account_double).to receive(:withdraw).and_raise(InsufficientFunds)

# When to use Test Doubles

Isolation from Dependencies
=================================

1. Don't create stubs from easy setup objects
2. Create stubs for objects that have external dependencies and are hard to
   setup and can slow runtime. i.e. ActiveRecord, DB connections, Network.

Isolation from Nondeterminism
=================================

1. Create stubs to avoid specs failing from other objects that behave randomly,
   i.e. connection errors, timeout errors.

Making Progress without Implemented Dependencies
=================================================

1. Use stubs if you know other object's APIS but the implementation is not ready
   yet.

Interface Discover
====================

1. Use stubs to discover new objects or interfaces.
2. Is easy to come by with new objects or interfaces when describing behavior.
3. Introduce a new mock and create the object/method later.

Focus on Roles not Objects
===========================

1. When creating a stub focus on what the interface should be, focus on the
   role.
2. A logger could be called a recorder, a reporter or anything we want as long
   as it act out the role of a logger.

Focus on interaction rather than state
=======================================

1. Avoid referencing the internals of an object
2. Use message expectations instead of stubs when possible
3. Describe the behavior and interaction

# Before
describe Statement do
  it "uses the customer's name in the header (with a stub)" do
    customer = stub("customer", name: "Dave Astels")
    statement = Statement.new(customer)

    expect(statement.header).to eq "Statement for Dave Astels"
  end
end

# After
describe Statement do
  it "uses the customer's name in the header (with a stub)" do
    customer = mock("customer")
    expect(customer).to receive(:name) { "Dave Astels" }
    statement = Statement.new(customer)

    expect(statement.header).to eq "Statement for Dave Astels"
  end
end

# Custom Matchers
require 'rspec/expectations'

# Always scope your rspec helpers
module BoletiaHelpers
  RSpec::Matchers.define :be_a_multiple_of do |expected|
    # implements the match logic
    match do |actual|
      actual % expected == 0
    end

    description do
      "be multiple of #{expected}"
    end

    failure_message_for_should do |actual|
      "expected that #{actual} would be a multiple of #{expected}"
    end

    failure_message_for_should_not do |actual|
      "expected that #{actual} would not be a multiple of #{expected}"
    end
  end
end
