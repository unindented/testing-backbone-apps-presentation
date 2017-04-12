# Testing Backbone Apps



## Who am I

Daniel ([@unindented](https://twitter.com/unindented))

Front-end developer at Yammer



## Testing at Yammer


### July 2012

```bash
git co `git rev-list -n 1 --before="2012-07-02" master`
```

- Lines of code: ~67600 <!-- .element: class="fragment" -->
- Tests: 673 (10 errors and 44 pending) <!-- .element: class="fragment" -->
- Time: 27 seconds <!-- .element: class="fragment" -->
- Coverage: ? <!-- .element: class="fragment" -->

Note:

We were really bad at this. We didn't have a culture of testing in the front-end team. We had very few tests, and they where slow.


### March 2014

```bash
git co master
```

- Lines of code: ~72700 <!-- .element: class="fragment" -->
- Tests: 5359 <!-- .element: class="fragment" -->
- Time: 31 seconds <!-- .element: class="fragment" -->
- Coverage: ~70% <!-- .element: class="fragment" -->

Note:

We are getting better, but we still have a lot to improve.

The message here is that, even if your project is in a bad situation right now, you can get out of that hole. Don't give up, because testing has too many benefits to pass.



## Why test?

> Tests are like vegetables. They're really good for other people. <!-- .element: class="fragment" -->


### Certainty

Note:

Passing tests give you some level of certainty that the changes you made didn't break anything.

That level of certainty is determined by the quality of your test suite.


### Courage

Note:

First reaction upon seeing messy code: "This is a mess that needs to be cleaned"

Second reaction: "I'm not touching it!"

Why? Because you know that if you touch it you risk breaking it; and if you break it, it becomes yours.

Tests help you make sure that your cleaning doesn't break anything. When you have a test suite you can trust, you lose fear of making changes. The code base steadily improves instead of rotting.


### Design

Note:

To test a piece of code, you first have to isolate it. It is difficult to test an entity if it is tightly coupled to others.

Tests force you to think about good, decoupled designs (especially if you do TDD).


### Documentation

Note:

A test is an example written in code, describing how the system should be used.

Unit tests are documents. They are unambiguous, accurate, written in a familiar language, and can be executed. They are the best kind of low-level documentation.


### Defect rate

[Microsoft Research: Realizing quality improvement through test driven development](http://research.microsoft.com/en-us/groups/ese/nagappan_tdd.pdf) <!-- .element: class="fragment" -->

Note:

"Defect density [...] decreased between 40% and 90% relative to similar projects that did not use the TDD practice."



## General advice



## Keep tests **isolated**


### Avoid global state

Note:

Global state is bad from theoretical, maintainability, and understandability point of view, but is tolerable at run-time as long as you have one instance of your application.

However, each test is a small instantiation of your application in contrast to one instance of application in production. Global state persists from one test to the next and creates mass confusion (e.g. tests fail together but problems can not be reproduced in isolation, order of the tests matters, etc.).


### Avoid singletons

Note:

All of the internal objects of the singleton are global as well (and the internals of those objects, and...). Singletons are just another flavor of global state.

Of course you can have objects that are instantiated only once in your application, but don't make them singletons by definition. In tests they should be instantiated as any other object.


```javascript
var MySingletonClass = function () {

  if (MySingletonClass.prototype._instance) {
    return MySingletonClass.prototype._instance;
  }
  MySingletonClass.prototype._instance = this;

};
```

Note:

Don't ever do shit like this, please.


### Clean up after yourself

Note:

Failing to do so could result in changes to the conditions of other tests.

Also, browsers could fail to garbage-collect your stuff, and memory usage could grow substantially.

We reached a point where our tests would crash IE8 in SauceLabs because of the amount of memory they required.



## Keep tests **simple**


### Don't mix concerns

Note:

Complex classes are hard to test, since there is so much functionality you need to cover:

  - Describe what the class does in one sentence, without using conjunctions.
  - Its code should be easy for other team members to read and quickly understand.


### Separate models from service objects

Note:

We all know we should be separating models and views. That's why we are using Backbone, right? But even then, we sometimes feel tempted to create fat models that have lots of functionality.

My advice is to separate models from service objects:

  - Models: These just have a bunch of getters and setters, and are very easy to construct. They are never mocked, and probably don't need an interface. Examples: `User`, `Message`, etc.
  - Service objects: These do the interesting work. Their constructors ask for lots of other objects for colaboration, are good candidates for mocking, tend to have an interface, and can even have multiple implementations. Example: `UserAuthenticator`, `LinkHydrator`, etc.

Models can be tested easily, as we can create them on the fly and assert on their state.

Service objects are harder to test since their state is not clear, and they are all about collaboration. As a result we are forced to use mocking, something which we want to minimize.

Mixing the two creates a hybrid which has none of the advantages of models, and all the disadvantages of service objects.


### Favor composition over inheritance

Note:

Two key ideas in the first chapter of Design Patterns:

  - Program to an interface, not an implementation.
  - Favor object composition over class inheritance.

You may think that these are things that only apply to languages like Java, but they are as important in JavaScript.

I'll flat out say that using inheritance for code reuse is wrong.

The base class is exposing its implementation details to all its subclasses. You have to actually know the internal implementation of the base class in order to correctly write the derived class. Any changes to the internal implementation of the base class could break the derived class in ways difficult to predict.

I put mixins in the same bucket as inheritance.


### Favor polymorphism over conditionals

Note:

If you see a `switch` statement you should think polymorphism.

If you see the same `if` condition repeated in many places in your class you should again think polymorphism.

Polymorphism will break your complex class into several simpler classes which clearly define which pieces of the code are related and execute together.



## Keep tests **fast**


### Don't do too much work in constructors

Note:

Almost every test in your suite instantiates an object. It is by far the most common operation you will do in tests, so make it easy on yourself and make the constructors do no work.

Any work you do in a constructor, you will have to successfully navigate through on every test.


### Don't attach to the document unless necessary

Note:

Attaching to the document means additional work for the browser engine, which translates into slower tests. Most of the stuff you want to test doesn't need to live inside the document.


### Fake anything that's unreliable or slow

Note:

Timers, animations, objects that do heavy DOM manipulation, anything external to your system...

Fake it all, but clean up after you.



## Know the tools


### Testing frameworks

- [Jasmine](http://jasmine.github.io/)
- [Mocha](http://mochajs.org/) + [Chai](http://chaijs.com/) + [SinonJS](http://sinonjs.org/)

Note:

Jasmine was probably the first popular testing framework in JavaScript. It's an all-in-one package, but has some limitations, and is not that easy to extend.

Mocha has a more do-it-yourself approach. Minimal (doesn't come with assertion or spy frameworks!) but really flexible. Usually combined with Chai for assertions, and SinonJS for spies.

SinonJS provides test spies, stubs and mocks, and works with any unit testing framework.


### Code coverage libraries

- [Istanbul](https://github.com/gotwarlost/istanbul)
- [Blanket](http://blanketjs.org/)

Note:

These tools instrument your code before running your test suite, so they know which parts of the code have been exercised.


### Headless browsers

- [PhantomJS](http://phantomjs.org/)
- [SlimerJS](http://slimerjs.org/)
- [trifleJS](http://triflejs.org/)

Note:

A headless browser is just a browser without a GUI. You control it through code.


### Test runners

- [Testem](https://github.com/airportyh/testem)
- [Karma](http://karma-runner.github.io/)

Note:

Tools focused on getting instant feedback from tests.


### Cloud testing platforms

- [SauceLabs](https://saucelabs.com/)
- [BrowserStack](http://www.browserstack.com/)

Note:

Run your tests in multiple browsers, but without having to maintain the VMs yourself.


### Continuous integration platforms

- [TravisCI](https://travis-ci.org/)
- [CircleCI](https://circleci.com/)
- [Jenkins](http://jenkins-ci.org/)
- [TeamCity](http://www.jetbrains.com/teamcity/)

Note:

Have a single source of truth.

Run your tests periodically, or after every push, to ensure that master is green.



## Structure of a test

1. Instantiate object graph <!-- .element: class="fragment" -->
2. Apply stimulus <!-- .element: class="fragment" -->
3. Assert a response <!-- .element: class="fragment" -->



## Testing **models**


### Testing validation

```javascript
describe('Message', function () {
  var message;

  describe('#isValid', function () {
    // ...
  });
});
```

Note:

Maybe validation is one of the most common things to test in a Backbone model.

There shouldn't be that much more to test for models, like we discussed before. All complex behavior should go into service objects.

I usually organize my test like this:

  - Root `describe` block with the name of the class under test.
  - Child `describe` blocks, one per public method. I follow the RSpec convention of starting with `#` for instance methods, and `.` for class methods.

Do whatever works for your project, but keep in mind that following the same conventions across the test suite helps people understand what's going on.


```javascript
describe('when it does not have body', function () {
  beforeEach(function () {
    message = new Message();
  });

  it('returns `false`', function () {
    expect(message.isValid()).toBe(false);
  });
});
```

Message #isValid when it does not have body returns false <!-- .element: class="fragment" -->

Note:

Most test runners I know concatenate the strings for each block when printing out the report.


```javascript
describe('when it has body', function () {
  beforeEach(function () {
    message = new Message({ body: 'Foo' });
  });

  it('returns `true`', function () {
    expect(message.isValid()).toBe(true);
  });
});
```

Message #isValid when it has body returns true <!-- .element: class="fragment" -->

Note:

So if you name things right, they read like sentences, and should describe the behavior of your system in plain English.


### Testing events

```javascript
initialize: function () {
  var recipients = this.get('recipients');

  recipients.on('add remove reset', function () {
    this.trigger('change:recipients');
  }, this);
}
```

Note:

You may also want to test custom events in your model. If you have a collection as a property of one of your models, maybe adding or removing from it should cause the parent model to trigger an event.


```javascript
describe('adding a recipient', function () {
  it('triggers a `change:recipients` event', function () {
    var spy = jasmine.createSpy();
    message.on('change:recipients', spy);

    message.get('recipients').add({ name: 'John' });
    expect(spy).toHaveBeenCalled();
  });
});
```

Message adding a recipient triggers a change:recipients event <!-- .element: class="fragment" -->

Note:

Events are usually pretty straightforward to test. I like them.


### Testing URLs

```javascript
describe('without ID', function() {
  it('returns the collection URL', function() {
    expect(message.url()).toEqual('/messages');
  });
});

describe('with ID', function() {
  it('returns the model URL', function() {
    message.id = 1;
    expect(message.url()).toEqual('/messages/1');
  });
});
```



## Testing **collections**


### Testing comparison

```javascript
describe('Recipients', function () {
  var recipients;

  describe('#comparator', function () {
    // ...
  });
});
```


```javascript
describe('with two recipients', function () {
  beforeEach(function () {
    recipients = new Recipients([
      { name: 'Bob' },
      { name: 'Alice' }
    ]);
  });

  it('sorts based on `name`', function () {
    expect(recipients.first().get('name')).toBe('Alice');
  });
});
```

Recipients #comparator sorts based on name <!-- .element: class="fragment" -->


### Testing syncing

```javascript
describe('#fetch', function() {
  it('hits the right URL', function() {
    messages.fetch();

    request = mostRecentAjaxRequest();
    expect(request.url).toBe('/messages');
  });
});
```

Note:

Mock ajax requests. You don't want to hit the real thing.


```javascript
describe('on success', function() {
  it('populates the collection', function() {
    messages.fetch();

    mostRecentAjaxRequest().response(successFixture);
    expect(messages.length)
      .toBe(successFixture.response.messages.length);
  });
});
```

Note:

Fixtures can help you simplify your tests.



## Testing **views**


### Setup and teardown

```javascript
describe('MessageView', function () {
  var view;

  afterEach(function () {
    view.remove();
  });
});
```

Note:

Be sure to clean up after creating your views. They are by far the most common source of leaks.


### Rendering tests

```javascript
describe('#render', function () {
  beforeEach(function () {
    view = new MessageView({ model: new Message() });
    view.render();
  });

  it('renders with class name `message`', function () {
    //expect(view.$el.hasClass('message')).toBe(true);
    expect(view.$el).toHaveClassName('message');
  });
});
```

Note:

Remember to use custom matchers, so that your error messages are descriptive. When you break the build, you want as much information as you can get.


```javascript
describe('#render', function () {
  describe('when the message is valid', function () {
    beforeEach(function () {
      model = new Message({ body: 'Foo' });
      view = new MessageView({ model: model });
      view.render();
    });

    // ...
  });
});
```

Note:

I usually define nested `describe` blocks to set up the conditions of the model or collection that the view will receive. It helps keep things structured.

Don't go too deep though, or it will be difficult to tell which block is nested where. I try to stop at this level.


### Visibility tests

```javascript
describe('#expand', function () {
  it('shows the dropdown', function () {
    view.expand();
    expect(view.$el).toHaveSelectorVisible('.dropdown');
  });
});
```

Note:

Checking for the visibility of elements requires you to attach the view to the document.

I usually avoid testing visibility through `is(':visible')`.


```javascript
Backbone.View.extend({
  show: function () {
    this.$el.show();
    this.trigger('show');
    return this;
  },

  hide: function () {
    this.$el.hide();
    this.trigger('hide');
    return this;
  }
});
```

Note:

If your base `View` class has something like this, you can test visibility by spying on these methods, or by listening to events. You no longer need to attach the view to the document, and your tests probably end up being more reliable.


### Interaction tests

```javascript
describe('clicking the X', function () {
  it('calls `remove`', function () {
    spyOn(view, 'remove');
    view.$('.remove-button').triggerHandler('click');
    expect(view.remove).toHaveBeenCalled();
  });
});
```

Note:

Differences between `trigger` and `triggerHandler`:

  - The `triggerHandler` method does not cause the default behavior of an event to occur.
  - While `trigger` will operate on all elements matched by the jQuery object, `triggerHandler` only affects the first matched element.
  - Events triggered with `triggerHandler` do not bubble up the DOM hierarchy. If they are not handled by the target element directly, they do nothing.

The `triggerHandler` method usually results in a more consistent behavior. We have had many flaky tests because of `trigger`, especially with `focus` and `blur` events.


```javascript
describe('pressing ENTER', function () {
  beforeEach(function () {
    evt = jQuery.Event('keydown');
    evt.which = 13; // ENTER
  });

  it('calls `submit`', function () {
    spyOn(view, 'submit');
    view.$('.input-field').trigger(evt);
    expect(view.submit).toHaveBeenCalled();
  });
});
```

Note:

Faking keyboard events involves creating them by hand, but it's not too much work. You can create a helper around this.


### Composition tests

```javascript
describe('#render', function () {
  beforeEach(function () {
    view = new MessageListView({ collection: ... });
    view.render();
  });

  it('renders `MessageView` children', function () {
    expect(view.$el)
      .toHaveSelector('.message', collection.length);
  });
});
```

Note:

If you are following the advice of favoring composition over inheritance, you will end up with views that are composed of views that are composed of... It's views all the way down.


```javascript
describe('#render', function () {
  beforeEach(function () {
    view = new MessageListView({ collection: ... });
    spyOn(MessageView.prototype, 'render')
      .andCallFake(function () { return this; });
    view.render();
  });

  // ...
});
```

Note:

Avoid doing unnecessary work. For example, you can stub out the `render` method of child views. You shouldn't care about what the child views render, as that should be exercised in different test.



## Testing **routers**


### Setup and teardown

```javascript
beforeEach(function () {
  router = new Router();
  Backbone.history.start();
});

afterEach(function () {
  Backbone.history.stop();
});
```


### Testing routes

```javascript
describe('/messages', function () {
  it('calls `#messages`', function () {
    spyOn(router, 'messages');

    router.navigate('messages', {
      trigger: true, replace: true
    });
    expect(router.messages).toHaveBeenCalled();
  });
});
```


### Testing methods

```javascript
describe('#messages', function () {
  beforeEach(function () {
    spyOn(MessagesView.prototype, 'initialize');
  });

  it('instantiates `MessagesView`', function () {
    router.messages();
    expect(MessagesView.prototype.initialize)
      .toHaveBeenCalled();
  });
});
```



## Misc stuff


### Custom matchers


```javascript
it('contains `foobar`', function () {
  expect(view.$('.foobar').length > 0).toBe(true);
});
```

Expected false to be true <!-- .element: class="fragment" -->


```javascript
beforeEach(function() {
  this.addMatchers({
    toHaveSelector: function (selector) {
      return (this.actual.find(selector).length > 0);
    }
  });
});

it('contains `foobar`', function () {
  expect(view.$el).toHaveSelector('.foobar');
});
```

Expected { ... } to have selector '.foobar' <!-- .element: class="fragment" -->


- [jasmine-jquery](https://github.com/velesin/jasmine-jquery)
- [jasmine-sinon](https://github.com/froots/jasmine-sinon)
- [chai-jquery](https://github.com/chaijs/chai-jquery)
- [sinon-chai](https://github.com/domenic/sinon-chai)


### jQuery.fx.off

```javascript
beforeEach(function() {
  $.fx.off = true;
});
```


### Fake timers

```javascript
beforeEach(function() {
  jasmine.Clock.useMock();

  timerCallback = jasmine.createSpy();
});

it("tests a timeout", function() {
  setTimeout(timerCallback, 100);

  jasmine.Clock.tick(101);

  expect(timerCallback).toHaveBeenCalled();
});
```


### Fake ajax requests

```javascript
beforeEach(function() {
  jasmine.Ajax.useMock();

  ajaxCallback = jasmine.createSpy();
});

it("tests an ajax request", function() {
  $.get('/api', ajaxCallback);

  mostRecentAjaxRequest().response({ status: 200 })
  expect(ajaxCallback).toHaveBeenCalled();
});
```


### Watch / LiveReload

![Monitoring changes through `grunt watch:test`](assets/watch.gif)

Note:

Having a tight feedback loop gives you confidence in the changes you are making.

If you are writing tests, then going back to the browser window and hitting reload, stop that!

A tool like Grunt combined with a headless browser like PhantomJS will give you great results. If you REALLY need to run your unit tests in real browsers, try something like Testem or Karma, which use LiveReload to automatically refresh.



## Parting thoughts


Keep tests isolated, simple and fast

Note:

Even though the talk is about testing Backbone apps, in my opinion the important stuff was in the "General advice" part of the presentation. If you keep the basics in mind, you will write good tests for any system.

Sure, learning the intricacies of testing with regards to the different pieces of your Backbone app will help you. But don't forget about keeping tests isolated, simple and fast. That's the key.



## Questions?

<https://unindented.github.io/testing-backbone-apps-presentation/> <!-- .element: class="fragment" -->
