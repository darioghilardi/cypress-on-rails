# cypress-on-rails

Proof-of-Concept gem for using [cypress.io](http://github.com/cypress-io/) in Rails applications. It provides the following features:
* run ruby code in your application context before executing a test
* database cleaning before test run (using database_cleaner)
* ability to use RSpec Mocks for your Rails code

## Getting started

Add this to your Gemfile:
```
group :test, :development do
  gem 'cypress-on-rails'
end
```

The generate the boilerplate code using:
```
rails g cypress:install
```

Finally add the `cypress` package using yarn:
```
yarn add --dev cypress
```

If you are not using RSpec and/or database_cleaner look at `spec/cypress/cypress_helper.rb`.

## Usage

This gem provides the `cypress` command. When called without any arguments ie. `bundle exec cypress`, it will start the cypress.io UI. While running the UI you can edit both your application and test code and see the effects on the next test run. When run as `bundle exec cypress run` it runs headless for CI testing.

The generator adds the following files/directory to your application:
* `spec/cypress/cypress_helper.rb` contains your configuration
* `spec/cypress/integrations/` contains your tests
* `spec/cypress/scenarios/` contains your scenario definitions
* `spec/cypress/support/setup.js` contains support code

When writing End-to-End tests, you will probably want to prepare your database to a known state. Maybe using a gem like factory_girl. This gem implements two methods to achieve this goal:

### Using embedded ruby
You can embed ruby code in your test file. This code will then be executed in the context of your application. For example:

```
// spec/cypress/integrations/simple_spec.js
describe('My First Test', function() {
  it('visit root', function() {
    // This calls to the backend to prepare the application state
    cy.rails(`
      Profile.create name: "Cypress Hill"
    `)

    // The application unter test is available at SERVER_PORT
    cy.visit('http://localhost:'+Cypress.env("SERVER_PORT"))

    cy.contains("Cypress Hill")
  })
})
```

Use the (`) backtick string syntax to allow multiline strings.

### Using scenarios

Scenarios are named `before` blocks that you can reference in your test.

You define a scenario in the `spec/cypress/scenarios` directory:
```
# spec/cypress/scenarios/basic.rb
scenario :basic do
  Profile.create name: "Cypress Hill"
end
```

Then reference the scenario in your test:
```
// spec/cypress/integrations/simple_spec.js
describe('My First Test', function() {
  it('visit root', function() {
    // This calls to the backend to prepare the application state
    cy.setupScenario('basic')

    // The application unter test is available at SERVER_PORT
    cy.visit('http://localhost:'+Cypress.env("SERVER_PORT"))

    cy.contains("Cypress Hill")
  })
})
```

The `setupScenario` call does the following things:
* clears the database using database_cleaner (can be disabled)
* calls the optional `before` block from `spec/cypress/cypress_helper.rb`
* calls the scenario block associated with the name given

In the scenario you also have access to RSpec mocking functions. So you could do something like:
```
scenario :basic do
  allow(ExternalService).to receive(:retrieve).and_return("result")
end
```

An example application is available at https://github.com/konvenit/cypress-on-rails-example

# Limitations
This code is very much at the proof-of-concept stage. The following limitations are known:
* It requires yarn for the javascript dependency management
* Only tested on Rails 5.1
* Only works with RSpec and database_cleaner
