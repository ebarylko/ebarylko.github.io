{:title "Acceptance testing"
 :layout :post
 :tags  ["cljs" "testing"]
 :toc true}


## Overview

This article covers what acceptance testing is, why it is useful, and an example of acceptance testing in a project.


## Types of testing

It is common to hear about preforming unit tests, acceptance (end-to-end) tests, and integration tests. How do all these appraoches differ?

Unit testing is about testing a function, using the inputs and verifying the outputs. All external dependencies should be [mocked](https://microsoft.github.io/code-with-engineering-playbook/automated-testing/unit-testing/mocking/). 

[Integration](https://www.techtarget.com/searchsoftwarequality/definition/integration-testing) testing is about testing the interaction of functions and external dependencies.

### What is acceptance testing?

Acceptance testing is a way to verifying that your site works correctly when the user interacts with it in a certain way. It's a method of simulating user actions, such as clicking, scrolling, filling in text, and seeing whether those actions produce a specific result, such as a message appearing, being moved to another page, a write to the database being executed, etc. 

## Why is it useful?

For example, on a website acceptance testing simulates what a user would do on your site. It is a way of checking that under certain scenarios, the user's interaction produce the result you thought it would.

For example, the website application I am working on includes user registration and login. I want to check that when the user is registered and they login, they see a welcome message that identifies them by their email.

The process should look like this.

First, the user enters the website and sees the landing page.

![Landing page](/img/landing-page.png)

Then, after clicking the login button they should be moved to another page which contains a welcome message.

![Before logging in](/img/before-logging-in.png)

Finally, after clicking the load-document button they should see the welcome message change to contain their email.

![After logging in](/img/after-logging-in.png)

Here's the code below that I use to test the entire process.

```clojure
(t/deftest message-test
  (t/testing "When the admin user exists"
    (doto driver
      (e/go "http://localhost:5000/")
      (e/screenshot "Screenshots/message-test-home-page.png")
      (e/click-visible {:tag :button :data-test-id "login"})
      (e/click-visible {:tag :button :id "load-document"})
      (e/wait-has-text-everywhere admin-login-message))
    (let [actual (e/get-element-text driver {:id "user"})]
      (e/screenshot driver "screenshots/message-test-when-the-admin-exists.png")
      (t/is (= admin-login-message actual)))))
```

I first head to my site, and then click on the login button. After that, I click on the load document button and wait for the message to appear. If the message does not appear I recieve an error that the message could not be found. If it appears I check that it matches the message the user should recieve.

By testing the process from start to end, from entering the site to logging in, It assures that this feature of the application works. Furthermore, It also tests that the user can actually interact with the site. In order for the message to appear the user had to navigate to the site and click two buttons. If the buttons are hidden by other objects or are in some obscure part of the page, the user might not be able to find it.

By doing acceptance tests, it assures that the components of the application work together successfully with a real database. It shows that from start to end, the application responds correctly given a specific scenario and certain user input.







