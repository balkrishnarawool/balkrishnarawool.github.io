---
layout: post
title: "Examples with Java's Optional"
date: 2020-08-23 11:00
description: Examples with Java's Optional
img: optional-image-2.jpg
tags: [optional, java, functional-programming]
---

`Optional` type from Java helps in representing the case of missing value. It also has some functional style API built into it which enables us to create concise code. In this post we will look at some such examples.

**Note:** There is another post on `Optional` which explores some important points to keep in mind while using `Optional` API. You can find it [here](https://balarawool.me/To-Optional-or-not-to-Optional/).

## Example 1

Consider an eCommerce company where `Customer`-s can have a `LoyaltyStatus` based on their usage of the company’s services.
If we have a method `getLoyaltyStatus()` which returns `Optional<LoyaltyStatus>` and we want to print a customer’s loyalty-status, we can do so simply by using` get()`

{% highlight java %}
public void printStatus() {
    Optional<LoyaltyStatus> loyaltyStatusOptional = getLoyaltyStatus();
    if(loyaltyStatusOptional.isPresent()) {
        System.out.println("Customer’s loyalty status is: " + loyaltyStatusOptional.get());
    } else {
        System.out.println("No loyalty status found for customer.");
    }
}
{% endhighlight %}

This is perfectly fine, but if we forget to put the `isPresent()` check, we might get `NoSuchElementException` from `get()`. In fact, we can write the same method concisely by using `ifPresentOrElse()` and we don't have to use `get()`.

{% highlight java %}
public void printStatus() {
    getLoyaltyStatus()
     .ifPresentOrElse(loyaltyStatus -> System.out.println("Customer’s loyalty status is: " + loyaltyStatus),
                     () -> System.out.println("No loyalty status found for customer."));
}
{% endhighlight %}

Here in this case we are interested in the empty-value case as well, so we used `ifPresentOrElse()`. But if we are not interested in the empty case, we can use `ifPresent()`.

**Note:** All source code for this post is present here: [https://github.com/balkrishnarawool/optional-examples](https://github.com/balkrishnarawool/optional-examples).

## Example 2

There are various promotions going on in the company and these promotions give reward points to the customers based on their loyalty-status.
Below is an example of a method that retrieves reward points for a customer.

{% highlight java %}
public int getCustomerLoyaltyStatusPoints() {
    Optional<LoyaltyStatus> loyaltyStatusOptional = getLoyaltyStatus();
    if(loyaltyStatusOptional.isPresent()) {
        LoyaltyStatus loyaltyStatus = loyaltyStatusOptional.get();
        return loyaltyStatus.getPoints();
    }
    return 0;
}
{% endhighlight %}

What we are actually doing here is, mapping a customer’s loyalty-status to reward points which can be also be done using `map()`.

{% highlight java %}
public int getCustomerLoyaltyStatusPoints() {
    return getLoyaltyStatus()
      .map(LoyaltyStatus::getPoints)
      .orElse(0);
}
{% endhighlight %}

We also see `orElse()` which is used to get value from Optional object with a default. There is also another method `orElseGet()`. For the difference between these methods see [this post]().

## Example 3

Free shipping is available to customers with `GOLD` and `PLATINUM` loyalty-status. So `isFreeShippingAvailable()` method can be written as

{% highlight java %}
public boolean isFreeShippingAvailable() {
    Optional<LoyaltyStatus> loyaltyStatusOptional = getLoyaltyStatus();
    if(loyaltyStatusOptional.isPresent()) {
        LoyaltyStatus loyaltyStatus = loyaltyStatusOptional.get();
        return loyaltyStatus.equals(GOLD) || loyaltyStatus.equals(PLATINUM);
    }
    return false;
}
{% endhighlight %}

Basically what we are doing here is, applying a condition. This can be easily done by using `filter()` method

{% highlight java %}
public boolean isFreeShippingAvailable() {
    return getLoyaltyStatus()
      .filter(loyaltyStatus -> loyaltyStatus.equals(GOLD) || loyaltyStatus.equals(PLATINUM))
      .isPresent();
}
{% endhighlight %}

## Example 4

There are different kinds of discounts available to customers. Customers with different loyalty status may or may not get discounts. This is represented by method `getDiscount()` in `LoyaltyStatus` which has return type `Optional<Discount>`. So `retrieveDiscount()` can be implemented like this

{% highlight java %}
public Optional<Discount> retrieveDiscount() {
    Optional<LoyaltyStatus> loyaltyStatusOptional = getLoyaltyStatus();
    if(loyaltyStatusOptional.isPresent()) {
        LoyaltyStatus loyaltyStatus = loyaltyStatusOptional.get();
        return loyaltyStatus.getDiscount();
    }
    return Optional.empty();
}
{% endhighlight %}

What we see here is that we are actually mapping loyalty status to discount. But if we simply use `map()` we will get an object of `Optional<Optional<Discount>>`. So we use `flatMap()` which gives the right type of object.

{% highlight java %}
public Optional<Discount> retrieveDiscountConcise() {
    return getLoyaltyStatus().flatMap(LoyaltyStatus::getDiscount);
}
{% endhighlight %}

Use `flatMap()` when you have an `Optional` object and the mapping function returns another `Optional` object.

## Example 5

For this example, consider a class `ShippingObject`. We can create a `ShippingObject` instance only when we have `length`, `width` and `height`. But we have methods that may or may not return these. i.e. we have three methods `retrieveLength()`, `retrieveWidth()` and `retrieveHeight()`, all returning `Optional<Integer>`. For simplicity they have been put into one type `DimensionsRetriever`. Now we can write `createShippingObject()` method like this

{% highlight java %}
public static Optional<ShippingObject> createShippingObject(DimensionsRetriever retriever) {
    Optional<Integer> length = retriever.retrieveLength();
    Optional<Integer> width = retriever.retrieveWidth();
    Optional<Integer> height = retriever.retrieveHeight();

    if(length.isPresent() && width.isPresent() && height.isPresent()) {
        return Optional.of(new ShippingObject(length.get(), width.get(), height.get()));
    }
    return Optional.empty();
}
{% endhighlight %}

Here we have to map the `Optional` objects to `ShippingObject`, but we have three of them. So we will have to chain the `map()` and `flatMap()` calls like this

{% highlight java %}
public static Optional<ShippingObject> createShippingObjectConcise(DimensionsRetriever retriever) {
    return retriever.retrieveLength().flatMap(length ->
              retriever.retrieveWidth().flatMap(width ->
                  retriever.retrieveHeight().map(height -> new ShippingObject(length, width, height))));
}
{% endhighlight %}

This works for more than three `Optional` objects as well, we just have to extend the chain.

We have see some commonly occurring use cases with `Optional` and how we can write them concisely with the `Optional` API. Hope this is useful.

**Note:** All source code for this post is present here: [https://github.com/balkrishnarawool/optional-examples](https://github.com/balkrishnarawool/optional-examples).
