---
layout: post
title: Returning Consecutive Values in Jasmine
---

{% highlight js %}
spyOnAndReturnConsecutive(foo, 'getBar', [1, 2, 3]);

expect(foo.getBar()).toEqual(1);
expect(foo.getBar()).toEqual(2);
expect(foo.getBar()).toEqual(3);
expect(foo.getBar()).toEqual(3);
{% endhighlight %}

Jasmine provides the ability to stub methods and return specific values. You can even [return different values based on
the arguments](http://stackoverflow.com/questions/16198353/any-way-to-modify-jasmine-spies-based-on-arguments).
So how do we get the snippet above to work?

<!--more-->

You can leverage Jasmine's ` andCallFake ` and ` andReturn ` to modify the return value every time the stub is called.
Drop the following function definitions into ` spec_helper.js ` and start stubbing!

{% highlight js %}
function spyOnAndReturnConsecutive(object, method, values) {
  spyOn(object, method)
    .andCallFake(returnConsecutive(object, method, values));
}

function returnConsecutive(object, method, values) {
  var nextValue = values.shift();

  return function () {
    if (values.length > 1) {
      object[method]
        .andCallFake(returnConsecutive(object, method, values));
    } else {
      object[method].andReturn(values[0]);
    }

    return nextValue;
  };
}
{% endhighlight %}

## How does it work?

Let's take a look at the first function:

{% highlight js %}
function ... {
  spyOn(object, method)
    .andCallFake(returnConsecutive(object, method, values));
}
{% endhighlight %}

This simply creates the initial spy on the object and returns the result of our other function - ` returnConsecutive `.

<br>

The ` returnConsecutive ` function does two things:

  1. returns the next value
  2. modify what the stub returns

This is what it looks like with just returning the next value:

{% highlight js %}
function ... {
  var nextValue = values.shift();

  return function () {
    ...

    return nextValue;
  };
}
{% endhighlight %}

Now let's take a look at where the chaining is happening:

{% highlight js %}
function ... {
  ...

  return function () {
    if (values.length > 1) {
      object[method]
        .andCallFake(returnConsecutive(object, method, values));
    } else {
      object[method].andReturn(values[0]);
    }

    ...
  };
}
{% endhighlight %}

If there are more than one value that still needs to be returned, we modify the stub to call ` returnConsecutive ` again to
return the next set of values.

If there is only a single value left to be returned, we set the stub to statically return the last value using
` andReturn `. This means the stub will keep returning the last value in the array after all of them have been returned.

## Full Example

{% highlight js %}
'use strict';

describe('foo', function () {

  var foo;

  beforeEach(function () {
    foo = {
      getBar: function () {
        return -1;
      }
    };
  });

  it('returns consecutive values', function () {
    spyOnAndReturnConsecutive(foo, 'getBar', [1, 2, 3]);

    expect(foo.getBar()).toEqual(1);
    expect(foo.getBar()).toEqual(2);
    expect(foo.getBar()).toEqual(3);
    expect(foo.getBar()).toEqual(3);
  });

  // Helper functions

  function spyOnAndReturnConsecutive(object, method, values) {
    spyOn(object, method)
      .andCallFake(returnConsecutive(object, method, values));
  }

  function returnConsecutive(object, method, values) {
    var nextValue = values.shift();

    return function () {
      if (values.length > 1) {
        object[method]
          .andCallFake(returnConsecutive(object, method, values));
      } else {
        object[method].andReturn(values[0]);
      }

      return nextValue;
    };
  }

});
{% endhighlight %}