---
layout: post
title: "To Optional, or not to Optional?"
date: 2020-08-20 11:00
description: To Optional, or not to Optional?
img: optional-image-1.jpg
tags: [optional, java, functional-programming]
---

Java’s `Optional` is quite a simple type. Most of the [API](https://download.java.net/java/early_access/jdk15/docs/api/java.base/java/util/Optional.html) can be easily understood and used. There are numerous examples on the internet which give us sufficient idea about what each and every method in `Optional` does. As it is so easy to use, there is a possibility that it gets overused.

Should we use it at every place where possible? This is where there are many different opinions in the community. If you follow that discussion, you can learn a lot from it. This blog enlists some of the important things to keep in mind while using Optional.

**Note:** There is another post on `Optional` which explores the important aspects of `Optional` API. You can find it [here](https://balarawool.me/Optional-concise-examples/).

## What is the purpose of Optional?

`Optional` helps in representing a case of missing value. For example, if you ask a person about their email address, they may or may not have an email. So if you create a `Person` class with `retrieveEmail()` method it should return a value that can represent the case of missing email. Traditionally `null` was used to indicate the no-email scenario. But this has many problems. It could mean that something went wrong while retrieving the email. Also someone might call a method on the returned null-email only to get `NullPointerException`. This is where you would use `Optional`. So the method returns `Optional<String>`. It can be empty which represents the missing-value scenario.

The most important benefit of `Optional` is that it triggers you to think about this missing-value scenario and to handle it properly. When you see a method returning `Optional`, it serves as a reminder that the method might return an empty value and we need to be prepared for it.

The original intention why `Optional` was introduced in Java was to make use of fluent API while handling `Stream`-s. (See [this discussion](http://mail.openjdk.java.net/pipermail/lambda-dev/2012-September/005952.html) for more info on that.) Without this, developers had to use `NoSuchElementException` and that’s very cumbersome. With `Optional`, it becomes ‘fluent’. For example, if you have a list of scores (in a fictitious game) and you want to present an award to the first player scoring a century, you can do so by

{% highlight java %}
scores.stream()
      .filter(Score::centuryOrHigher)
      .findFirst()
      .ifPresentOrElse(s -> System.out.println("Award goes to " + s.getPlayer()),
                      () -> System.out.println("Nobody gets an award"));
{% endhighlight %}

(Read more about the origin of the term fluent API here: [https://www.martinfowler.com/bliki/FluentInterface.html](https://www.martinfowler.com/bliki/FluentInterface.html))

**Note:** All source code for this post is present here: [https://github.com/balkrishnarawool/optional-examples](https://github.com/balkrishnarawool/optional-examples).

## Should you use get()?

`Optional` is a wrapper type and you obviously need to get the value out of its instance quite often. A straightforward way of doing that is to use `get()`. But a problem with `get()` is that it throws `NoSuchElementException`, if the instance is empty. So whenever we call `get()`, we have to make absolutely sure that it is not empty. We can do that by using `isPresent()` on the  instance. For example, if you want to send an email to a person if they have an email and do nothing otherwise, then it can be done by

{% highlight java %}
Optional<String> email = person.retrieveEmail();
if(email.isPresent()) {
    sendEmail(email.get());
}
{% endhighlight %}

But this is no better than using null and performing a null-check before using any object that can be null. Also it would have been great if compiler could prevent us from doing an unsafe `get()`. But that is not possible with the current `Optional` API. So if you forget to check before using an empty object, you'll reach the inevitable `NoSuchElementException`. Therefore it is best to avoid using `get()` as much as possible.

What's the alternative then? We need to think a bit further and see what we want to do with the value. Then depending on what we want to do, we can use `orElse()`, `orElseGet()`, `ifPresent()`, `filter()`, `map()` or `flatMap()`. In the above code, we just want to consume the value and use it to send an email. So here `ifPresent()` can be used

{% highlight java %}
person.retrieveEmail().ifPresent(email -> sendEmail(email));
{% endhighlight %}

## Difference between orElse() and orElseGet()

Sometimes we want to provide default value when the value is missing. This can be done by using `orElse()` or `orElseGet()`. For example, consider a website where `getUser()` gives current user and `getName()` gives their name if they are logged in and empty otherwise. In case it is empty, if we want to generate a name from random strings we can do so by

{% highlight java %}
getUser().getName().orElse(generateRandomName());
{% endhighlight %}

In this example even if user’s name is known, the program would still execute `generateRandomName()` which is unnecessary. And to avoid that, we can use `orElseGet()`. `orElseGet()` is lazy. It adds laziness by accepting a `Supplier`. So above example with `orElseGet()` becomes

{% highlight java %}
getUser().getName().orElseGet(() -> generateRandomName());
{% endhighlight %}

In this case `genrateRandomName()` will be executed only if name is empty.

So a general guideline would be: use `orElseGet()` if you have to calculate default value and use `orElse()` when the default is a literal value or is readily available.

## Optional as method return type

Using `Optional` as a return type for public methods is a recommended way of using it. This creates a clear contract for the users of the method. This makes it clear that they can expect an empty value and need to handle it properly. As an example, here's implementation of the `retrieveEmail()` method that was referred earlier

{% highlight java %}
public Optional<String> retrieveEmail() {
    return address.getType().equals(Address.Type.EMAIL))
        ? Optional.of(address.getValue())
        : Optional.empty();
}
{% endhighlight %}

But it is not recommended to be used as return type for POJO getters. Because this would mean we would have to deviate from convention. Also some frameworks and libraries require the getters to have same return type as their corresponding fields.

As stated earlier, `Optional` is a wrapper type and therefore requires the object to be wrapped/ unwrapped. So if you’re writing performance critical code, you can stay away from it. Also if your method is private, the `Optional` instances can be very short lived and would only add overhead. Here we can deal with nulls. We just have to ensure that null references are not crossing object boundaries.

## Optionals as fields

Using `Optional` as a type for fields in a class, is generally not a bad idea. But it is not recommended by JSR-335 Expert Group (the creators of `Optional`). You can read more about it [here](http://mail.openjdk.java.net/pipermail/jdk8-dev/2013-September/003274.html). As the introduction of `Optional` did not have the intention of using it this way, it may not work as expected in some scenarios. Below is a list of some of these.

If it is inside a `Serializable` class then serialization may not work. A `Serializable` class with `Optional` field would throw `NotSerializableException` when you try to serialize its instance. An explanation (and history behind) the decision of not making `Optional` serializable is [here](https://blog.codefx.org/java/dev/why-isnt-optional-serializable/).

If you want to convert a Java object to JSON, it creates an unnecessary wrapper.
For example, an object of this class

{% highlight javascript %}
public class Person {
    private String name;
    private Optional<String> email;
}
{% endhighlight %}

would produce a JSON like this

{% highlight javascript %}
{"name":"Bruce Wayne","email":{"present":true}}
{% endhighlight %}

That’s not exactly what you would want.
Although if you use [Jackson](https://github.com/FasterXML/jackson), you can correct it with [jackson-modules-java8](https://github.com/FasterXML/jackson-modules-java8)
So the JSON would become

{% highlight javascript %}
{"name":"Bruce Wayne","email":"bruce.wayne@wayneenterprises.com"}
{% endhighlight %}

If you use JPA (with ORM libraries like Hibernate), you can’t map `Optional` field to a column in database table.

So if you want to use Optional fields in classes be very careful of these scenarios.

## Optional as method parameter type

Using `Optional` as method parameter type is not really recommended as it can make the API more complex than it needs to be and method overloading is a better alternative.
Consider below method which searches for players with given country and optionally with a minimum rating.

{% highlight java %}
public static List<Player> searchFor(Country country, Optional<Integer> minimumRatingOptional) {
    int minimumRating = minimumRatingOptional.orElse(RATING_MIN_VALUE);
    return allPlayers.stream()
        .filter(p -> p.getCountry().equals(country))
        .filter(p -> p.getRating() >= minimumRating)
        .collect(Collectors.toList());
}
{% endhighlight %}

So when someone wants to call this method they can do so like this

{% highlight java %}
searchFor(Country.NL, Optional.empty())
searchFor(Country.IN, Optional.of(1000))
{% endhighlight %}

which is slightly obscured. In fact with method overloading it can be better written as

{% highlight java %}
public static List<Player> searchFor(Country country) {
    return searchFor(country, RATING_MIN_VALUE);
}
public static List<Player> searchFor(Country country, int minimumRating) {
    return allPlayers.stream()
        .filter(p -> p.getCountry().equals(country))
        .filter(p -> p.getRating() >= minimumRating)
        .collect(Collectors.toList());
    return players;
}
{% endhighlight %}

These methods can then be called as

{% highlight java %}
searchFor(Country.NL)
searchFor(Country.IN, 1000)
{% endhighlight %}

No need for creating `Optional` instances.

## Conclusion

These are some examples where one should or should not use `Optional`. But these are guidelines and are applicable in most of the cases. There can always be instances where deviation from these is better. But with these in mind one can make a more informed decision.

**Note:** All source code for this post is present here: [https://github.com/balkrishnarawool/optional-examples](https://github.com/balkrishnarawool/optional-examples).
