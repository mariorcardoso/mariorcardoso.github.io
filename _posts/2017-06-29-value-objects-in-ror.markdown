---
layout: post
title:  "Value Objects in Ruby on Rails"
date:   2017-06-29
categories: rails patterns "value objects"
---

![](https://miro.medium.com/max/1400/1*luqPGefqe8_spk1vSncF8Q.jpeg)

Today I’m going to talk about value objects and how they can be used and identified in your Rails application.

The goal here is to have simple models and controllers by extracting some classes. Creating value objects is one way of achieving this. I would rather have a lot of small classes than have only a few big classes, since they are easier to test, change and understand.

I will explore some situations and give you some value object examples that should make your application simpler and easier to work with. These examples were taken from my own experience developing applications, and also several scenarios that I have come across in other great blog posts.

## What is a Value Object?

As stated by [Martin Fowler a value object is](https://martinfowler.com/eaaCatalog/valueObject.html):

> A small simple object, like money or a date range, whose equality isn’t based on identity.

A value object represents a simple entity whose equality is based on its value, meaning that two different objects are equal when they have the same value. Value objects should be immutable, so when two value objects are created equal they should remain equal. When we define a value object in Ruby we should define the `==` or `<=>` methods so we can compare them based in its value.

The primitive objects such as Symbol, String, Integer and Range defined in Ruby are the simplest examples.

## Why do we need them?

Identifying value objects in your Rails application can significantly simplify your system. The advantages of refactoring your code and making use of value objects are:

* the separation of concerns
* it allows you to combine behaviour with the data and add functionality to data without polluting the model
* by isolating functionality your code is easier to test
* removes duplication
* improves the understanding and organisation of code. Operations on particular data are now gathered in a single place, instead of disperse throughout the code

## How can we identify them?

I usually consider four situations that are candidates for creating a value object:

* Arguments together all the time
* One attribute with behaviour
* Two inseparable attributes value and unit
* Class enumerable

### Arguments together all the time

The first situation happens when we have two or more arguments that are passed and used together all the time, often known as a “Data Clump” (code smell). A date range is a common example, when `start_date` and `end_date` are passed together all the time in our methods. We can create a class DateRange with the attributes `start_date` and `end_date` and this class should be responsible for the `start_date` and `end_date` columns of a given ActiveRecord object and accommodate all the related behaviour. We could include methods like `include_date?(date)`, `include_date_range?(date_range)`, `overlap_date_range?(date_range)` and `to_s`. This class can look something like this:

{% gist dc592344f8e40e56440c542bc020c640 %}

This is just a standard Ruby object that does not inherit from ActiveRecord::Base. This class can be used, for example, with an Event model with the following columns: `name`, `description`, `address_city`, `address_state`, `starts_at`, `ends_at`. The Event model could look something like this:

{% gist dc592344f8e40e56440c542bc020c640 %}

With all this in place, we get the following usage:

{% gist dc592344f8e40e56440c542bc020c640 %}

Another simple scenario that is demonstrated in the book _The Rails 4 Way (3rd Edition),_ is a Person model with a single Address, that can be modeled using composition like we did in the previous example. The Person class has the attributes `name`, `address_city` and `address_state` and looks like this:

{% gist dc592344f8e40e56440c542bc020c640 %}

The value object Address looks something like this:

{% gist dc592344f8e40e56440c542bc020c640 %}

Again this is just a standard Ruby object. With all this in place we get a similar usage like we saw in the date_range example:

{% gist dc592344f8e40e56440c542bc020c640 %}

As I mentioned previously, one of the advantages of extracting code that usually goes in the model and create value objects is that you can reuse them and here is an example of that since we could use the Address object in the Event model as well.

### One attribute with behaviour

Another situation where a value object can be useful is when you have one simple attribute that needs some associated behaviour and such behaviour does not belong in the model. Imagine that you have a model Room that inherits from ActiveRecord::Base with a degrees attribute and then you add a Temperature class to answer some questions that your system may need related with the temperature value:

{% gist dc592344f8e40e56440c542bc020c640 %}

Besides the `cold?` and `hot?` methods we also defined the methods `<=>`, `hash`, and `eql?`which allow us to make use of operations like `sort` and `uniq`_._ We get the following usage:

{% gist dc592344f8e40e56440c542bc020c640 %}

### Two inseparable attributes value and unit

Another common situation is when you have value-unit pairs like temperature (degrees and unit), money (cents and currency), distance (value and unit), etc. A pair of this kind usually needs some conversions methods, from euros to dollars, from kelvin to fahrenheit, from miles to kilometres, etc. And not just conversions, but also math operations often require special handling. Since this is so common and involves some logic there are already a few gems that are basically value objects that you can include and start using them right away.

A very popular one is the [money gem](https://github.com/RubyMoney/money), which helps you deal with money and currency conventions by providing a Money class that encapsulates all information about a certain amount of money such as its currency and value. The gem readme file is very thorough and self-explanatory so if you are interested go ahead and take a look. You can use it in a model class like Product:

{% gist dc592344f8e40e56440c542bc020c640 %}

In this case when asking for or setting the cost of a product, we would use a Money instance.

{% gist dc592344f8e40e56440c542bc020c640 %}

The use of this gem as a value object is also covered in the book The Rails 4 Way (3rd Edition). Money is one of the most complex value objects of this type, specially the part related with conversions, but you can find simpler examples like the [weight value object](https://github.com/shemerey/weight).

### Class enumerable

It is common practice to define a value object in Rails models by creating an array class like this:

{% gist dc592344f8e40e56440c542bc020c640 %}

This practice is not good because the array values may be used in the model attributes but they have nothing to do directly with the model domain. Defining the value object like this has a few disadvantages like impossibility to add functionality to the value object without polluting the model and does not allow to reuse the object. So we can create an object to accommodate the data of that array and also add some useful methods if we need them. The value object class for the example above could look something like this:

{% gist dc592344f8e40e56440c542bc020c640 %}

We can get the set of possible sizes and have a method that can be used in select form fields. I think this is useful if you have more than one model that has the attribute size, if your model has a lot of those arrays and you want to slim your model or if you have logic associated with it. Another common example of this type is Color, when we need to have a set of colors that can be used in some persisted models.

## **Values**

As they say in the [github page](https://github.com/tcrayford/Values/tree/master):

> Values is a tiny library for creating value objects in ruby.

Values allows you to simplify your value objects definition by removing the need to write an initialiser, add a few handy methods like attribute accessors and forces your value object to be immutable.

Here is an example that I showed you before but now using the values gem:

{% gist dc592344f8e40e56440c542bc020c640 %}

## Conclusion

My goal here was to encourage you to extract some classes and make your models and controllers less bloated. I tried to do this by providing several value object examples and covering a variety of potential scenarios, which, I hope, will help identify when and how to use them in your own Rails applications.

## References

[https://www.sitepoint.com/value-objects-explained-with-ruby/](https://www.sitepoint.com/value-objects-explained-with-ruby/)  
[https://martinfowler.com/bliki/ValueObject.html](https://martinfowler.com/bliki/ValueObject.html)  
[http://wiki.c2.com/?ValueObject](http://wiki.c2.com/?ValueObject)  
[http://blog.jasonharrelson.com/blog/2014/03/07/how-i-design-rails-applications-part-1-value-objects/](http://blog.jasonharrelson.com/blog/2014/03/07/how-i-design-rails-applications-part-1-value-objects/)  
[https://makandracards.com/alexander-m/43081-value-object](https://makandracards.com/alexander-m/43081-value-object)
