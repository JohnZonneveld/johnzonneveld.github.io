---
layout: post
title: React/Redux Portfolio Project Flatiron School (mod5 project)!
---

<h2>React/Redux project - GCE pages</h2>

![GCE pages](/assets/img/gce_homepage.jpg){:class="img-responsive"}


For my project I took something that is close to me but yet so far away. I have been travelling all over Europe with a motorcycle group. Last year one of the members that was acting as webmaster stepped down, although I live in the US nowadays I volunteered to take over the helm if there was noone stepping up.

Officially this should have been my project in February, but life happened. Slept in my car for 5 weeks and the corona situation didn't make things easier as well. On top of it we had the Texas freeze.
But that is all behind us now and we're trying to pick up the pieces and make best of life.

Made a simple React/Redux project that fullfills the requirements that were set in stone, or at least I hope.
Javascript was a big change from what we started to learn in the first three modules,but I was able to make that a very impressing show case. This React/Redux project is just the start, think it can be impressing as well but would need some extra work.

At first glance while doing the assignments in the curriculum, React and later Redux added doesn't look like it is that difficult. Until you have to build something from scratch, yes I was struggling in the beginning with this project. But as time went by I was grasping the matter much better.
With some additional bootstrap styling it at least looks nice.

I tried to re-create the homepage of the (Kawasaki) GTR Club Europe. I gave it for now a static homepage and about page. The React/Redux magic is happening in the members and tours section.
There is a simple relationship between the two. Members could have been organizer of a tour and in that way can have many tours. But a tour can only have one organizer.

Members and Tours are loaded from the Rails backend the moment App.js is mounted and stored in the store.
When you click on members you will see a listing of all the members currently in the system with their repective role in the club. Used a ternary operator to display the separating comma and the role when it is available.
Same happens when you click on Tours, all tours in the system will be displayed with the starting date and the country where it will take place.

In the ADD and Edit page of the Tour I wanted to create a dropdown with all the members in the system to add the organizer.
Because at any moment someone can step up and organize a tour in their country so it had to be a dynamic dropdown.

{% highlight javascript %}
<Form.Label column sm="2">
    Organizer: 
</Form.Label>                        
    <Select 
        options={this.props.members.map(member => {
        return { value : member.name, label: member.name, target: {value: member.id, name: "member_id"}}
        })}
        onChange={event => this.props.onChange(event)} 
        name="member_id"
    />          
{% endhighlight %}

The members are loaded from the state through:
{% highlight javascript %}
export default connect(mapStateToProps)(TourForm)
{% endhighlight %}

This makes a connection to the store which is called by the mapStateToProps function.
{% highlight javascript %}
function mapStateToProps(state, ownProps) {
    return {
        members: state.members
    };
}
{% endhighlight %}

This connection to the store and the function mapStateToProps makes the member objects array available in the tourForm or MemberAdd class as props.

So to display the members in the dropdown we simply have to call the .map function on this.props.members
Bycoming problem, although it makes things easier, is that Select is only showing the value and label. So for the onChange we would have to write a separate function. To overcome this I added the target: 
{% highlight javascript %} {value: member.id, name: "member_id"} {% endhighlight %}in the return. This is returning exactly what we would need for {% highlight javascript %}[event.target.name] = event.target.value{% endhighlight %} Now we can use the onChange we use for all the other fields.

In the editForm we should be able to display the current selection. As the member.id is added to a tour as member_id and the loaded members in this.props.members we should be able to distill and show the name based on the member.id.
{% highlight javascript %}
const currentMemberObject = this.props.members.filter(member => member.id === this.props.tour.member_id){% endhighlight %}
Here we filter the member object array based on the member_id that matches member.id. This returns an array with 1 object, the member in question.
To be able to get to the key, value pairs of this mmember object we have to add the index in the newly created array [0]. The 'Select' uses defaultValue as parameter to display the current selection.

{% highlight javascript %}
    defaultValue={{label: currentMemberObject[0].name, value: currentMemberObject[0].name}}
{% endhighlight %}

![edit pages](/assets/img/editForm.png){:class="img-responsive"}


<h2>Conclusion</h2>

Coming up with a subject was no issue at all.If it was a wise decission only time will tell. Think this might be a good start to later bring into practice and give the 'live' webpage I am webmaster of a React/Redux overhaul. For a second I was lost but at the end pieces were coming together.