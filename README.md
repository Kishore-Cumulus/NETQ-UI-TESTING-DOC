# Introduction

We know writing tests for our applications are obviously important because we can verify our new features are working or the older features didn't break, at the end helps us deliver products with confidence and maintain the stability of the product. But writing tests will take time and effort, we will develop this document as a reference for testing our NetQ-UI codebase. For the initial part I will go through some concepts,  changes to the Angular's default setting  and in later parts will walk through specific testing parts and in the final part discussing some techniques where we can utilise this setup for our advantage to develop features faster without depending on the backend. 

For the first part, We will discuss some concepts and techniques available today in the testing ecosystem. All these concepts/techniques can be thought of web frontend application testing. There are three general types of testing 

1. Unit testing
2. Integration testing
3. E2E testing 

#### Unit testing

* These tests will be at the component level and tests the functionality of the single component without worrying about the class and template dependencies\(generally we achieve this by stubbing the dependencies\).
* Test cases for reducers, effects can also can come into this category.
* We will discuss this type of testing in detail in the coming pages.

#### Integration testing

* These tests will be testing a feature or module which can have multiple components interacting with each other. 
* In our project, an example would be testing a `upgrade switches` flow where there are multiple  components interactions like routing to the `lcmhome` via the hamburger menu and fetching the data from the backend for the switches, images, credentials and clicking the switches card.
* These tests are slower than unit testing and also require other dependent components to work properly.

#### E2E testing \(End To End\)

* These type of tests will be more similar to the real world usage.
* These tests will runs in a browser and makes call to the actual backend and the interactions would be similar to normal user.
* Slower compared to the above two testing schemes and would require an extensive setup at the backend for seeding the data and also the availability.

{% hint style="info" %}
Write more integration+e2e tests than unit tests
{% endhint %}

In our project we will using unit testing and a mix of integration testing + E2E where we can simulate the user interactions via the browser but instead of depending on the backend we mock the server so that our testing can be independent in that respect.

