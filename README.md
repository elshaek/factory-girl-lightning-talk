# Factory Girl Gem
- Create templates of _valid_ objects to be used during testing
- Keep object creation _flexible_
- Customization when instantiating objects
- Used in place of fixtures

- Fixtures: predefined sample data used for testing; 
            independent of database; 
            written in YAML (YAML Ain't Markup Language).


## Factories vs Fixtures

#### With fixtures...
```ruby
# users.yml
marko:
  first_name: Marko
  last_name: Anastasov
  phone: 555-123-6788
```
used in test:
```ruby
describe User do
  it "has a first name" do
    user = users(:marko)
    user.first_name.should eql("Marko")
  end
end
```
- How was user set up?
- What are the correct values to test for? (Have to refer back to predefined data set)

These are usually addressed by writing comments in test code:
```ruby
describe "#show" do
  before do
    # User with preferences to view posts about kittens
    # and in the group with special access to Burmese cats
    # with 4 friends that like ridgeback dogs.
    @user = users(:kitten_fan)
  end
end
```

#### With factories...
```ruby
# spec/factories/user.rb
FactoryGirl.define do
  factory :user do
    first_name "Andy"
    last_name  "Lindeman"
    phone "555-123-6788"
  end
end
```
used in test:
```ruby
describe User do
  it "has a first name" do
    user = FactoryGirl.create(:user, first_name: "Marko", last_name: "Anastasov", phone: "555-123-6788")
    user.first_name.should eql("Marko")
  end
end
```
- We know how user is set up
- Can generate an instance and test it without having to remember details of predefined data
- Note: build vs create
    + Using FactoryGirl.build(:factory_name) does not persist to the db and does not call save!, so the ActiveRecord validations will not run: faster, but no validations
    + Using FactoryGirl.create(:factory_name) will persist to the db and will call ActiveRecord validations: slower but can catch validation errors

#### Using Factories
```ruby
# Returns a User instance that's not saved
# ActiveRecord validations will not run: faster, but no validations
user = FactoryGirl.build(:user)

# Returns a saved User instances
# will persist to the db and will call ActiveRecord validations: slower but can catch validation errors
user = FactoryGirl.create(:user)

# Returns a hash of attributes that can be used to build a User instance
attrs = FactoryGirl.attributes_for(:user)

# Returns an object with all defined attributes stubbed out
stub = FactoryGirl.build_stubbed(:user)

# Passing a block to any of the methods above will yield the return object
create(:user) do |user|
  user.posts.create(attributes_for(:post))
end
```

#### Aliases
```ruby 
factory :user, aliases: [:author, :commenter] do
  first_name    "John"
  last_name     "Doe"
  date_of_birth { 18.years.ago }
end

factory :post do
  author
  # instead of
  # association :author, factory: :user
  title "How to read a book effectively"
  body  "There are five steps involved."
end

factory :comment do
  commenter
  # instead of
  # association :commenter, factory: :user
  body "Great article!"
end
```

#### Callbacks

- after(:build) - called after a factory is built (via FactoryGirl.build, FactoryGirl.create)
- before(:create) - called before a factory is saved (via FactoryGirl.create
- after(:create) - called after a factory is saved (via FactoryGirl.create)
- after(:stub) - called after a factory is stubbed (via FactoryGirl.build_stubbed)
- Note: Calling create will invoke both after_build and after_create callbacks.
```ruby
# Define a factory that calls the generate_hashed_password method after it is built
factory :user do
  after(:build) { |user| generate_hashed_password(user) }
end
```

## Alternative factory gems (less popular):
- [Fabrication](https://github.com/paulelliott/fabrication)
- [Cranky](https://github.com/ginty/cranky)
- [Machinist](https://github.com/notahat/machinist)


## References:
- [Rails Testing Antipatterns: Fixtures and Factories](https://semaphoreci.com/blog/2014/01/14/rails-testing-antipatterns-fixtures-and-factories.html)
- [Factories not Fixtures](http://railscasts.com/episodes/158-factories-not-fixtures-revised)
- [Factory Girl Github](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)