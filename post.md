##1 Component-Based Directives in AngularJS

AngularDart and Angular 2.0 are shifting themselves to better embrace use of directives instead of standalone controllers. This means that component-based directives are the way to go, but how exactly can we build our app purely using directives in AngularJS 1.3? Let’s explore how we craft together reusable, portable and well-tested directives that embrace HTML while empowering us to make the most of our Angular applications.

###1.2 What are we here to learn?
We are here to learn how to construct portable and reusable directives in Angular.
The topic itself is quite extensive, so, to make things short and sweet, let's focus
on the three following points:
- How to port our verbose template HTML into reusable directives.
- How to unit test our newly crafted HTML component code.
- How to further refactor both our HTML and JS directive code to make things more flexible.

##2 Robust & Flexible Component-Directives in AngularJS

Directives in AngularJS are uber-powerful. Enough said. But the proper construction and use of directives overall, determines the flexibility of both the JavaScript and template code of an application. In other words, the better the directives, the more robust the application becomes.

With the feature-set that AngularJS has to offer, we can construct robust and flexible directives in our application by treating directives as if they are standalone, isolated components.

Let's explore the various ways that we can turn our directives into smart components and make the most of our template code.

###2.1 Directives as Components

Why does this article all of sudden start calling directives, **components**? Aren't they
the same thing? Not exactly. A directive is its own term, but a component is a
fragment of code that can be plugged into another body of code and behave as expected. In other words, a component is just a directive that works off of the inputs given and it can be used in any template anywhere.

Before we start exploring how to do this, let's take a detour and see what exactly
components are outside of AngularJS.

####2.1.1 The Future of the DOM: Web Components
HTML and JS are attempting to solve the problem of messy plugin-based code in websites
by introducing a new browser-based technology called Web Components. 

Web Components work
in the DOM, but they hardness a hidden layer of the DOM called the Shadow DOM. This mystical,
hidden layer of DOM elements acts as the hidden plumping to make custom and complex HTML elements
behave in an easy to use way. 

Does this already exist in some way? Yes. Just think about the `<video>`
element in HTML5. What does it do? The video element sets up buttons, a progress meter and a poster image.
If we were to break down the underlying HTML we would see a hidden underworld of button elements, custom-styled div tags and an img element. But it's hidden because it all exists in the shadow DOM. 

Now,
with Web Components, we can craft our own component code to make use of this DOM underworld.

If that isn't enough, Web Components also conveniently package all of the required code into remote asset files via something called HTML imports.  The idea here is that one developer can create a nice plugin which can then be used on another website without another developer's CSS/JS/HTML code interfering. Once this
feature goes mainstream, we have peace of mind while importing 3rd party code, free from worry about clashing CSS styles and messy HTML markup.

###2.2 Making a Component-Based Directive
So Web Components aren't out just yet right? But can we do something similar in Angular? Yes.
That's the end goal when it comes to crafting together components in Angular. 

Let's take an example of a directive written in AngularJS that we can turn into a reusable component.

We'll build out a multiplication times table and expand that
out based on the inputs to the directive. To get started, let's code this in our standard,
everyday AngularJS ng-repeat code.

<!--code lang=markup linenums=true-->
  
    <div class="multiplication-table" ng-controller="MultiplicationTableCtrl as ctrl">
      <div ng-repeat="row in ctrl.rows">
        <div ng-repeat="cell in row">{{ ($parent.$index + 1) * ($index + 1) }}
        </div>
      </div>
    </div>

The code doesn't work right away since we're missing the scope data. Let's mix that in
there.

<!--code lang=javascript linenums=true -->

    ngModule.controller('MultiplicationTableCtrl', [function() {
      this.rows = [
        [ 1, 1, 1, 1 ],
        [ 1, 1, 1, 1 ],
        [ 1, 1, 1, 1 ],
        [ 1, 1, 1, 1 ]
      ];
    }]);

<a href="http://plnkr.co/edit/wBb04Z4l5gLDEtKW3keK?p=preview" target="_blank">Click here to see this in action</a>

This works. Great. Amazing. But it's kinda useless. Yes, we can display a series of numbers
on the screen, but the controller code is static and the HTML code is horrendous. That being said,
what's the first step that we can take to make the code above more portable and useful?
Let's start by wrapping up the controller code into a directive.

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationTable', [function() {
      return {
        controllerAs : 'ctrl',
        controller : function() {
          this.rows = [
            [ 1, 1, 1, 1 ],
            [ 1, 1, 1, 1 ],
            [ 1, 1, 1, 1 ],
            [ 1, 1, 1, 1 ]
          ];
        }
      }
    }]);
<a href="http://plnkr.co/edit/Mjqo7uPySpNJnvgXTA96?p=preview" target="_blank">Click here to see this in action</a>

Now we can summon our multiplication table in our templates by using
`<div multiplication-table` instead of having to use `ng-controller`.

This is nice and glamorous, but we're still not any further into improving the quality of
our code. We've just swapped one controller container for another and the HTML code is still
out of whack.

So what can we do to make it more portable? Let's start by wrapping the HTML code into a remote template.

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationTable', [function() {
      return {
        templateUrl : 'multiplication-table-tpl.html',
        controllerAs : 'ctrl',
        controller : function() {
          //... same as before ...
        }
      }
    }]);

Excellent. Now if we want to instantiate a new instance of the multiplication component
then it's just a matter of using the AngularJS directive within the template.

<!--code lang=markup linenums=true -->

    <div multiplication-table></div>

<a href="http://plnkr.co/edit/U8Xysntbv4Sd4avfI8pb?p=preview" target="_blank">Click here to see this in action</a>


Our code is now a lot more portable. But it is far from reusable and it is not flexible
at all. I've got a question ... Should there always be a grid of 4x4? Is that a dumb question?
How can we maintain the sleekness
of the directive whilst allowing the user to pass in more parameters into the directive
itself via the HTML code? 

We can think of directives like functions within HTML. Their
purpose is to take in some data and render-out some DOM contents. We can think of the
attributes that go into the directive as function parameters. So if we had two attributes:
`x="..."` and `y="..."`, then we can have our directive controller code build the
multiplication table behind the scenes.

So what are our options for loading in **directive parameters**? We can either use the injected `$attrs` object or we can setup an isolated scope via the scope parameter within the directive definition. The latter
is easier to use and (with AngularJS 1.3) we can bind scope properties directly onto the directive controller.

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationTable', [function() {
      return {
        templateUrl : 'multiplication-table-tpl.html',
        // this is new to AngularJS 1.3
        bindToController: true,
        scope: {
          x: '=',
          y: '='
        },
        controllerAs : 'ctrl',
        controller : function() {
          var x = this.x || 0;
          var y = this.y || 0;

          var table = this.rows = [];
          for(var i=0;i&lt;y;i++) {
            var array = table[i] = [];
            for(var j=0;j&lt;x;j++) {
              array.push(1); 
            }
          }
        }
      }
    }]);

<a href="http://plnkr.co/edit/08UvCUfOteke4bP4JQtE?p=preview" target="_blank">Click here to see this in action</a>

Excellent! Now our directive is flexible. How we can make it extendible?   

The next thing we should do is allow the outside HTML code to provide inner HTML
code inside of each of the multiplication cells. Since we're already using a remote
template, any inner data within the directive will be wiped out. That means that at this stage,
we're out of luck to pass in any inner directive template code.

To retain the inner directive HTML code and place it somewhere within the contents of
the remote template code, we will need to use directive **transclusion**. This mechanism
will allow us to do something like this:

<!--code lang=markup linenums=true -->

    <div multiplication-table>
      Number {{ multiplication.value }}
    </div>

The first step in making this work is to hop into the remote template and add
in a element called `<div ng-transclude></div>`.

<!--code lang=markup linenums=true -->

    <div ng-repeat="row in ctrl.rows">
      <div ng-repeat="cell in row">
        <div ng-transclude></div>
      </div>
    </div>

And now for the directive code:

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationTable', [function() {
      return {
        transclude: true,
        // ... same as above ...
      }
    }]);

This works but `multiplication.value` doesn't exist. How can we make this work
without having to know the internals of the directive scope (like row[$index][$index])?
Let's hop back into the remote template file and create a new directive. 

<!--code lang=markup linenums=true -->

    <div ng-repeat="row in ctrl.rows">
      <div ng-repeat="cell in row">
        <div multiplication-cell
             x="$index+1"
             y="$parent.$index+1"
             ng-transclude></div>
      </div>
    </div>

You can probably guess that this new directive (called `multiplicationCell`) is designed
in some way to display the contents of the product of `x` and `y`. Yes. Let's code it up.

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationCell', [function() {
      // notice that we now injected $compile
      return {
        controllerAs : 'multiplication',
        controller : ['$attrs', '$scope', function($attrs, $scope) {
          this.x = $scope.$eval($attrs.x);
          this.y = $scope.$eval($attrs.y);
          this.value = x * y;
        }]
      }
    }]);

Notice how we're using a different approach to loading in attribute data for the directive? The `$attrs` object contains the HTML param values that are placed on the element containing the directive. The call
to `$scope.$eval` evaluates the scope value based on the attribute key value. This way we can load in the attribute values without having to create an isolate scope.

This should work, but not just yet. We are almost there. Just one final thing to tweak. Since transclusion in AngularJS shields the
outer scope data from the inner scope data (the multiplication stuff), it means that the `multiplication`
controller is not available directly on the same scope. It is however on the $parent scope of the inner data
that is injected into the cell. Therefore to get this to work, we just need to prefix the expression with `$parent` to access
the data that we so nicely packaged in the multiplication controller.

<!--code lang=markup linenums=true -->

    <div multiplication-table x="5" y="5">
      {{ $parent.multiplication.value }}
    </div>
<a href="http://plnkr.co/edit/oNlZ8oRUk327R7yZH5k3?p=preview" target="_blank">Click here to see this in action</a>

Perfect. Now we have a super versatile component directive that is flexible and reusable. Now let's unit test it.

##3 Unit Testing Component-based directives
So we have our code. It works. But we have to apply to get insurance now. Why?
Because if our code breaks then we're screwed. When it comes to programming,
the insurance that you have to know that your code works as expected is the
quality of your unit tests. The more code that you test in your application then
the better your insurance coverage. The more simplistic and "to the point" your
unit tests are, the better your insurance premium is. 

When it comes to testing component-based directives, here's what we need to do:

- Compile the directive HTML with the remote template code
- Test out **how the data is rendered within the directive DOM**
- Review all the sticky-points of our code

There's a lot of advanced stuff mentioned above, but don't worry ... we'll go step by step.

###3.1 Configuring Karma
To orchestrate unit-testing in Angular (and with any level of front-end code)
we need to use a testing tool called `karma`. Karma is freaking amazing! When
karma is configured and ready to go it launches the associated browsers and tests
your code to see if things work.

First things first, let's install karma and then initialize it.

<!--code lang=bash linenums=true -->

    sudo npm install -g karma

    karma init

Let's stick to only using Chrome for testing and let's use Jasmine as the testing
framework. Let's leave the list of source files empty, but we should also enable Karma
to run the tests on change. 

Inside of `karma.conf.js` make sure your list of files looks like this.

<!--code lang=javascript linenums=true -->

    // Note: The file structure here is according to the github application
    // (this can be found at the bottom of the article).
    files: [
      'app/angular.js',
      'app/angular-mocks.js',
      'test/multiplication-app.js',
      'test/multiplicationSpec.js'
    ],

Now the most important thing that we need in order to test our directive components is
to setup Karma to load all of our template files into our unit tests. 

First,
install `karma-ng-html2js-preprocessor` and configure karma.conf.js to look like so:

<!--code lang=javascript linenums=true -->

    // list of files / patterns to load in the browser
    files: [
      'app/angular.js',
      'test/angular-mocks.js',
      'app/multiplication-app.js',
      'app/**/*-tpl.html',
      'test/multiplicationSpec.js'
    ],

    //add this in too
    ngHtml2JsPreprocessor: {
      stripPrefix: 'app/',
      moduleName: 'appTemplates'
    },


    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      '**/*.html': ['ng-html2js']
    }

Karma will now find each of the template files and concatenate them into a module and
load that into the unit test runner. This means that in our AngularJS unit tests we
don't have to mock-out any of the remote directive template files. 

But wait, hold on - isn't
it a bad idea to allow production-level template code to be used in unit tests? Yes and
no. Since our code is moving towards "component-based" directive code this means that
the remote template HTML code associated with each directive is very neutral. Not only
are none of the templates in any way knowledgeable of their parent template code, they
also ONLY work off the data that was prepared in the directive itself. Also, when any
additional HTML code gets added into the remote file, more directives are created.

This answer may be a bit complex to comprehend so let's revisit the question at the
end of the article once our code has been nicely refactored.

###3.2 The Unit Test
Now that our testing environment is setup, let's write our first test to test out
our `multiplication-table` directive.

<!--code lang=javascript linenums=true -->

    describe("MultiplicationApp", function() {
      // our collected remote templates
      beforeEach(module("appTemplates"));

      // our application module
      beforeEach(module("multiplicationApp"));

      describe("multiplication-table", function() {
        //...
      });
    });

This here is the foundation for our unit test code. Now let's test things out.

###3.3 Testing our Components

Our newly created components need to be unit tested. The goal here is not to test
out things in every possible way, but more so to get a level of coverage that is
satisfactory when things go wrong.

When it comes to figuring out exactly "what" to test, we should always stick to
placing our test expectations on points of our code that are unlikely to change in
the future. Think about it - if we were to test out our multiplication table, then we
would place our assertions on things like the numbers and ordering of numbers instead
of testing out classes and HTML elements (because those could change at any point).

First we need to compile the component HTML code into our tests, and then apply assertions
directly on the created data (or the DOM structure if we have to).

Before we do that however, we need to list out the different kinds of "sticky points" our
tests have:

<!--code lang=javascript linenums=true -->

    describe("MultiplicationApp", function() {
      // ... same as above ...

      describe("multiplication-table", function() {
        it("should list out a table of 0x0");
        it("should list out a table of 1x1");
        it("should list out a table of 2x2");
        it("should list out a table of NxM");
      });
    });

Now for each of these tests we should have a shared `beforeEach` which compiles
the table directive, sets up the scope and provides some shared data across all
tests.

<!--code lang=javascript linenums=true -->

    describe("multiplication-table", function() {
      var scope, element, render
      beforeEach(inject(function($rootScope, $compile) {
        scope = $rootScope.$new();
        var compileFn = $compile(
          '<div multiplication-table x="x" y="y">' +
          '  x={{ $parent.multiplication.x }},' +
          '  y={{ $parent.multiplication.y }},' +
          '  v={{ $parent.value }}' + 
          '  </div>' +
          '</div>'
        );
        render = function() {
          element = compileFn(scope);
          $rootScope.$digest();
        }
      }));

      it("should list out a table of 0x0");
      it("should list out a table of 1x1");
      it("should list out a table of 2x2");
      it("should list out a table of NxM");
    });

Before each of the tests are run (inside of the describe block for `multiplication-table`), the `beforeEach` function is triggered which builds up the test DOM data. 

We then are
left with scope and a render function that, when called, will apply the evaluate data onto
the HTML data. This means that our first test is very small and consists of setting data,
calling render and then asserting the output.

Here are some of the helper functions we need to make the tests more simple:

<!--code lang=javascript linenums=true -->

    function s(value) {
      return value.replace(/\s+/g, '');
    }

    function lastCell(table) {
      var row = table[0];
      var cell = row.children[row.children.length-1];
      if (cell.children.length) {
        cell = cell.children[cell.children.length-1];
      }
      return angular.element(cell);
    }

    function totalCells(element) {
      return element[0].querySelectorAll('[multiplication-cell]').length;
    }

And now for test #1:

<!--code lang=javascript linenums=true -->

    it("should list out a table of 0x0", function() {
      scope.x = 0;
      scope.y = 0;
      render();
      expect(totalCells(element)).toBe(0);
    });

Excellent. And now for the next test:

<!--code lang=javascript linenums=true -->

    it("should list out a table of 1x1", function() {
      scope.x = 1;
      scope.y = 1;
      render();
      expect(totalCells(element)).toBe(1);
      expect(s(element.text())).toEqual("x=1,y=1,v=1");
    });

The 3rd test is pretty much the same as the first, however now it's actually examining
the output data. In this case we're checking to see if the data provided is being compiled
in the right way. (One thing to note is that we're using a function called s() which removes
all of whitespace from the provided string.)

Now for test #3:

<!--code lang=javascript linenums=true -->

    it("should list out a table of 2x2", function() {
      scope.x = 2;
      scope.y = 2;
      render();
      expect(totalCells(element)).toBe(4);
      var cell = lastCell(element);
      expect(s(cell.text())).toEqual("x=2,y=2,v=4");
    });

This test is finding the very last cell and testing the output to match x=2 and y=2.

The next test is `NxM` so we only care about when n!=m. Therefore, combinations
like `3x2` and `2x3` are suitable. But in order to have two different sets of
data then we should change around the test description to also test for when
the scope data changes.

<!--code lang=javascript linenums=true -->

    it("should list out a table of NxM and rerender itself when the scope data changes", function() {
      scope.x = 2;
      scope.y = 3;
      render();

      var cell = lastCell(element);
      expect(s(cell.text())).toBe("x=2,y=3,v=6");

      scope.x = 3;
      scope.y = 2;
      scope.$digest();
      
      cell = lastCell(element);
      expect(s(cell.text())).toBe("x=3,y=2,v=6");
    });

Uh oh. Our test fails because our table fails to update itself when the `x` and `y` values change. Let's
go ahead and fix this bug in our application code:

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationTable', [function() {
      return {
        templateUrl : 'multiplication-table-tpl.html',
        controllerAs : 'ctrl',
        transclude: true,
        bindToController: true,
        scope: {
          x : '=',
          y : '='
        },
        controller : ['$scope', function($scope) {
          var ctrl = this;
          $scope.$watch(function() { return ctrl.x; }, render);
          $scope.$watch(function() { return ctrl.y; }, render);
          
          function render() {
            var x = ctrl.x || 0;
            var y = ctrl.y || 0;

            var table = ctrl.rows = [];
            for(var i=0;i&lt;y;i++) {
              var array = table[i] = [];
              for(var j=0;j&lt;x;j++) {
                array.push(1); 
              }
            }
          }
        }]
      }
    }]);

<a href="http://plnkr.co/edit/kVYi4DKtP8iDJBOx49F3?p=preview" target="_blank">Click here to see this in action</a>

Excellent. We just found a serious bug and took care of it. Good stuff.

There we have it. Our tests are done. Now we have some proper insurance on our code.

This means that we're free to refactor the HTML code as well as the directive code
to make things more flexible and possibly more efficient. Let's go ahead and refactor
out our remote template HTML code to NOT use two levels of ng-repeat.

##4 Refactoring Components

The beauty of using remote HTML code for directives and simple input attributes is that
all of the code within is neutral to the point that we can comfortably test it. If we
can comfortably test it, then we can definitely refactor it to improve the quality of the
code.

When was the last time you refactored your HTML code? Refactoring itself is not very
effective when we don't have any tests on our code, but now that we've tested out our
multiplication table component properly using unit tests, we can very easily change
the internals and be informed if something breaks.

So first things first, let's rip out the nested ng-repeat code and only use one.

<!--code lang=markup linenums=true -->

    <div ng-repeat="cell in ctrl.cells">
      <div multiplication-cell
           x="cell.x"
           y="cell.y"
           ng-transclude>
      </div>
    </div>

If we run the tests for our code, then things break. This is because the directive-level
code is still the same as before. So let's go ahead and refactor that out.

<!--code lang=javascript linenums=true -->

    ngModule.directive('multiplicationTable', [function() {
      return {
        templateUrl : 'multiplication-table-tpl.html',
        controllerAs : 'ctrl',
        transclude: true,
        bindToController: true,
        scope: {
          x : '=',
          y : '='
        },
        controller : ['$scope', function($scope) {
          var ctrl = this;
          $scope.$watch(function() { return ctrl.x; }, render);
          $scope.$watch(function() { return ctrl.y; }, render);
          
          function render() {
            var x = ctrl.x || 0;
            var y = ctrl.y || 0;

            var table = ctrl.cells = [];
            for(var i=0;i&lt;y;i++) {
              for(var j=0;j&lt;x;j++) {
                table.push({
                  x : j + 1,
                  y : i + 1
                });
              }
            }
          }
        }]
      }
    }]);

The tests are now green again. This is good. We've made the HTML code smaller and
easier to deal with. Now let's do a few more optimizations to the HTML code and see
if the tests work.

<!--code lang=markup linenums=true -->

    <div ng-repeat="cell in ctrl.cells"
         multiplication-cell
         x="cell.x"
         y="cell.y"
         ng-transclude>
    </div>

<a href="http://plnkr.co/edit/sS4BwmsDPSjRZdYSR6bN?p=preview" target="_blank">Click here to see this in action</a>


That's amazing! This would have been so much more difficult if we didn't have components
that were unit tested. What's even better is that on the outside of the component (the
homepage view that loads in the multiplication table) nothing has changed.

<a href="https://github.com/yearofmoo-articles/airpair-components-article" target="_blank">Click here for the github repo</a>

##5 Going Further
If you're interested in learning more about how components work and how we can use them
for other areas of our application such as forms and views, then have a look at the videos
below:

** Complex Forms with Advanced Directives in AngularJS**

<a href="https://www.youtube.com/watch?v=G5MzkDJQkoQ" target="_blank">https://www.youtube.com/watch?v=G5MzkDJQkoQ</a>

**Component-based Directives in AngularJS**

<a href="https://www.youtube.com/watch?v=gE4hyIfUD-A" target="_blank">https://www.youtube.com/watch?v=gE4hyIfUD-
A</a>


##6 Please Follow!
If you're also interested in learning about other AngularJS topics such as animations, forms
and testing then please follow <a href="//twitter.com/yearofmoo" target="_blank">@yearofmoo</a> on twitter or visit Matias' website at
http://www.yearofmoo.com.