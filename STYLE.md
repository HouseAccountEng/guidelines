# Style

We aim to write code that is a pleasure to read, and we have opinions about how to do it well. 
We care about how code reads, how code looks, and how code makes you feel when you read it.

We love discussing code. 
If you have questions about how to write something, or detect some code smell, ask away.
A Pull Request is a great way to do this.

When writing new code, try to find similar code elsewhere to look for inspiration.

## Prefer many files over long files

When we open an existing file, we don't want to feel overwhelmed. 
Limiting the length of files to 50 lines increases readability and decrease headaches.

We picked 50 as an arbitrary number of lines that can fit on any screen at a good font size.
For the same reason, we set an arbitrary limit of 100 characters per line.
These limits includes blank lines and comments and are enforced by Rubocop rules.

When a model exceeds the limit, ask yourself: can I group some methods together into a concern?
When a controller exceeds the limit, ask yourself: would two separate controllers be clearer?
When a helper exceeds the limit, ask yourself: would splitting into multiple helpers be clearer?

# Don't wrap instance variables with accessors unless necessary

See https://www.codewithjason.com/dont-wrap-instance-variables-attr_reader-unless-necessary/

## Documentation is required

Every public class and public method must have YARD documentation.
This is enforced by the `bin/yardoc --list-undoc` command, which is required to pass before merging a PR.
Do not add documentation for private classes or private methods.

## Do not use meta-programming

Never call private methods using `.send` or `.public_send` on an object

## Make every database migration reversible

When some actions only happen in the `up` sense, use `up_only`, not `up` and `down`.
When some actions only happen in the `up` and `down` sense, use `reversible`, not `up` and `down`.

```ruby
# Bad
def up
  add_column :bookings, :satisfied, :boolean
  Booking.where(status: :liked).update_all satisfied: true, status: :fulfilled
  Booking.where(status: :disliked).update_all satisfied: false, status: :fulfilled
end

def down
  remove_column :bookings, :satisfied, :boolean
end

# Good
def change
  add_column :bookings, :satisfied, :boolean
  reversible do |dir|
    dir.up do
      Booking.where(status: :liked).update_all satisfied: true, status: :fulfilled
      Booking.where(status: :disliked).update_all satisfied: false, status: :fulfilled
    end
  end
end
```

## Follow REST

We model web endpoints as CRUD operations on resources (REST).
When an action doesn't map cleanly to a standard CRUD verb, we introduce a new resource rather than adding custom actions.

```ruby
# Bad
resources :cards do
  post :close
  post :reopen
end

# Good
resources :cards do
  resource :closure
end
```

## Use Active Record -- do not use Arel or SQL

When writing database operations, always resort to Active Record commands such as `find`,
`insert_all`, `update_all`. Never reach out to Arel commands such as `Arel.sql('count')`
and never reach out for SQL statements.


## Use `performs` to reduce cognitive overload for background jobs

## Create second-level controllers for nested routes

## Consider every endpoint to be turbo-enabled

* This means to add `allow_other_host: true` or status: see_other, or data: { turbo_frame: '_top' }


## Limit inline method declarations to these cases:

1. def show; end
2. def advice_params = params.expect booking_advice: [:content]

## Harness the power of nested layouts

- In other words, don't repeat yourself in views
- Even for framework helpers, extract them even with form builders

## Code optimistically

- Never add a method that is not invoked anywhere in the code
- If your code change removes the only usage of a method, remove that method
- Don't add rescue statements for errors that have never happened

## Do not write unit tests

- Never create or update files in test/models
- Never write unit tests for models

## Use short, meaningful names

Example: Provider, not ServiceProvider. Chat, not Conversation

## Use adjective to name concerns

Example:

```ruby
# Bad
module Booking::AppAssociation extend ActiveSupport::Concern
…
end

# Good
module Booking::Applied extend ActiveSupport::Concern
…
end

```


## Use acronyms when they are correct

Example: API, ZIP

## Do not use single-letter variables

The variable names should be meaningful. Example:

```ruby
# Bad
@vertical.specialties.each do |s|
  @provider.skills.find_or_create_by! specialty: s
end

# Good
@vertical.specialties.each do |specialty|
  @provider.skills.find_or_create_by! specialty: specialty
end
```

## Only use parentheses when needed

Example:

```ruby
# Bad
Skill.find_or_create_by!(specialty: specialty)

# Good
Skill.find_or_create_by! specialty: specialty
```


## For JavaScript, use Stimulus controllers as much as possible

## Visibility modifiers

We indent visibility modifiers at the level of `class` and we don't use special indentation for the
content under them.

```ruby
class SomeClass
  def some_method
    # ...
  end

private

  def some_private_method_1
    # ...
  end

  def some_private_method_2
    # ...
  end
end
```
