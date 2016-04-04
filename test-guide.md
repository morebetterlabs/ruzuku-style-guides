# Testing
## General Rules

Tests are easier to read and understand if the main testing steps are written
in a procedural style. Object oriented techniques should only be used to create
logically simple and reusable steps, test assertions and setup methods.

* Do not DRY up tests by combining multiple steps into a single step
  ```Ruby
  # bad - four spaces
  it 'logs a user into a course and creates a post' do
    login_as user
    goto_discussion_and_post_a_remark(discussion,'This is the body of my post')
  end

  # good
  def some_method
    login_as user
    visit discussion_url(discussion)
    page.fill_in 'remark_body', with: 'This is the body of my post'
    page.click_button 'Post'
  end

  # good
  def some_method
    login_as user
    visit discussion_url(discussion)
    page.post_remark 'This is the body of my post'
  end
  ```
* Combine complicated assertions into simple tests
* Combine complex setups into methods when they are reusable
* No boolean tests in test steps or setup.  They are discouraged but occasionally
necessary in assertions

## Ruby/Rspec


### General

* Avoid the `private` keyword in specs.
* Avoid checking boolean equality directly. Instead, write predicate methods and
  use appropriate matchers. [Example][predicate-example].
* Prefer `eq` to `==` in RSpec.
* Separate setup, exercise, verification, and teardown phases with newlines.
* [Should syntax] or [`expect` syntax] is ok but start switch to expect.
* Use RSpec's [stub and return syntax] over the [`allow` syntax] for method stubs.
* Use `not_to` instead of `to_not` in RSpec expectations.
* Prefer the `have_selector` matcher to the `have_css` matcher in Capybara assertions.

[Should syntax]: https://github.com/rspec/rspec-expectations/blob/master/Should.md
[stub and return syntax]: https://relishapp.com/rspec/rspec-mocks/v/2-99/docs/method-stubs
[`expect` syntax]: http://myronmars.to/n/dev-blog/2012/06/rspecs-new-expectation-syntax
[`allow` syntax]: https://github.com/rspec/rspec-mocks#method-stubs
[predicate-example]: predicate_tests_spec.rb

### Structure

* Capybara tests should only be in the /integration directory
* Blueprints should be placed in a relevant file in the /blueprints directory
* In most cases the tests should be placed in the directory that corresponds to the code location
in the /app directory

### Acceptance Tests

* Avoid scenario titles that add no information, such as "successfully."
* Avoid scenario titles that repeat the feature title.
* Use names like `ROLE_ACTION_spec.rb`, such as
  `user_changes_password_spec.rb`, for feature spec file names.
* Use only one `feature` block per feature spec file.
* Use scenario titles that describe the success and failure paths.
* Use spec/integration directory to store feature specs.

### Unit Tests
* private methods should not be tested in isolation


[Imperative mood]: http://en.wikipedia.org/wiki/Imperative_mood
[subject for one-liners example]: unit_test_spec.rb#6