---
layout: post
title: "Convert Text field to DatePicker in React"
---

In my GCE webpages app I have a field for when a tour will take place, at the moment of writing this field is just a simple text input field. It would be nice to make that a calendar drop down.

In the back-end this will also have some implications as of now the database the type of the date field is also just a string field.

Choosing a datepicker

This wasn't really a though choice, first one I found was react-date-picker and looking at their github pages and examples I saw this would suffice my needs for a simple datepicker.

Installation is pretty simple with:
{% highlight javascript %}
npm install react-datepicker
{% endhighlight %}

To implement in the app all I had to do is to replace the text field I had with a call to the DatePicker component.
But at first the component has to be imported, so in the components that will be using the DatePicker I did a simple import:

{% highlight javascript %}
import DatePicker from 'react-date-picker'
{% endhighlight %}

The code in the component is added like:

{% highlight javascript %}
<DatePicker 
    value= {new Date(this.props.tour.date)}
    onChange={this.props.handleDate}
/>
{% endhighlight %}

If you look closely you see that for the value we create a new Date object, this is needed for react-date-picker needs object as input. For this we translate the given value in this.props.tour.date from a String to an Object.

Furthermore the date column in the tour database has to be of type datetime. When using just the type of date, it can lead to faulty displayed values. In my case when I entered the date 03/09/2002 this was displayed in the rendered page as 03/08/2002.

Only problem that occured was in the TourPage.js and TourList.js pages that the displayed date was including the time.

The date would be displayed like:
{% highlight javascript %}
2002-03-09T06:00:00.000Z
{% endhighlight %}

To overcome this and just display the date we can use the toLocaleDateString function. Because this function is expecting an Object we have to create an object of our tour.date.
So to create an Object of our tour.date we use the function new Date(tour.date) and this we will feed into the toLocaleDateString() function.
{% highlight javascript %}
{ (new Date(tour.date)).toLocaleDateString() }
{% endhighlight %}

Last change to make the field border the same color as the rest of the input fields we have to make a change in /node_modules/react-date-picker/dist/DatePicker.css

In the class definition 
{% highlight javascript %}
.react-date-picker__wrapper
{% endhighlight %}
we have to change the border to
{% highlight javascript %}
border: thin solid #ced4da
{% endhighlight %}
