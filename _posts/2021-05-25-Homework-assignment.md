---
layout: post
title: Homeworks Assignment!
---

Well I guess I made it with hurdles. Sometimes live is happening and stands in the way of progress. Will not say too much about it, but it sure could have been a lot easier to hit the end of the Software Engineering Bootcamp. Only needed some cooperation of a person very close to me. Let's leave it with that.

To my surprise I passed my project 5 review on first try. I would have liked to be a little better prepared, but a nightly worktrip from 11pm till 630am sure didn't help. Guess after that I needed my beauty sleep.

For the livecoding the reviewer wanted me to change my 'ToursList' page to have a like button and a field to change the number of likes that were given per click. There was no need to persist it to the back-end. So, after a page refresh or leaving the page and coming back to the ToursList page, it was allowed to start over from 'no likes'.

I did make a start but we were running out of time. So at the end she did congratulate me with passing the review and that I had a good understanding of the Rails/Redux concepts.
As homework and a little bit more practice she advised me to carry on with the live-coding assignment.

Where to start.

At first I made a button in the ToursList itself but made a little mistake by including the brackets in the onClick eventListener. Result was that it did count when the page was first rendered but all the tours had an ascending number starting from the top without any user interaction. That was an easy fix though by removing the brackets in the function call.

Now a day later I sat down and started to think a little deeper about it.
The easiest way would be to create a separate component and call that when the tours are mapped in the ToursList component.

{% highlight javascript %}
import React, { Component } from 'react'

class LikeButton extends Component {
    constructor(props) {
        super(props) 
        this.state = {
            count: 0
        }
    }

    increaseLike = () => {
        let delta = 1
        let newCount = this.state.count + delta
        this.setState({
        count: newCount
    })

    }
    render() {
        return (
            <div>
                <button id={this.props.id} onClick={this.increaseLike}>Likes: {this.state.count}</button>
            </div>
        )
    }
}

export default LikeButton
{% endhighlight %}
LikeButton.js

This component we can as said before call when we map the tours.

{% highlight javascript %}
import React, { Component } from 'react'
import {Link} from 'react-router-dom';
import LikeButton from './LikeButton'

const ToursList = ({tours}) => {

    return (
        <ul>
            {tours.map(
                tour =>
                    <li key={tour.id}>
                        <Link to ={'/tours/' + tour.id}>{tour.name}</Link>
                        {tour.country ? ', ' + tour.country : null}
                        , {tour.date}
                        <LikeButton />
                    </li>
            )}
        </ul>
    )
}

export default ToursList
{% endhighlight %}
ToursList.js

As we call the LikeButton per mapped tour we create a separate LikeButton Object with its own local state for every tour. So the counter will only be updated on the button that is clicked. No need to keep track of id's to see which button is clicked. Every button is an object itself with its own eventListener.

Okay after all that was no rocket science, although seeing Starship prototypes soar through the sky nearby might have influenced me.

The next requirement was to add the field for the number of likes per click.
ToursList is just a render of an unnumbered list, so I didn't want to change too much on that component. I created the required field in the ToursPage component by just adding the following code

{% highlight javascript %}
Number of likes given per click: <input type='text' onChange={this.handleChange} name='numberOfLikes'></input>
{% endhighlight %}

Further I added a local state for the numberOfLikes

{% highlight javascript %}
state = {
        numberOfLikes: ''
    }
{% endhighlight %}

And created an eventListener to update this state if something was typed into the field.

{% highlight javascript %}
handleChange = event => {
    this.setState({ 
            [event.target.name]: event.target.value
    })
}
{% endhighlight %}