---
layout: post
title: JS Portfolio Project Flatiron School (mod4 project)!
---

<h2>JS project - HAM Logbook</h2>
![HAM Logbook](/assets/img/hamlogbook.jpg){:class="img-responsive"}

For my project I took the same subject as with the Sinatra project, as a hobby I have my radio amateur license. Especially for long distances we like to keep a logbook, maybe a bit to show off.

Because of this, the framework that I needed was not an issue and was clear in my head.

I log my contacts in two different places, most contacts I have made so far are on digital modes. The programs I use for this offer automated logging to online logbooks. The ones that I use are Logbook of the World (LotW) hosted by the ARRL (American Radio Relay League) and eQSL.cc. Both sites allow you to download your contacts in ADIF format, a format that was proposed in 1996 and used by many logging programs since 1997. This should make importing the data fairly easy.
For my project I used the LoTW data to seed my data into the postgresql database

Already while working on my Sinatra project I came across a ruby script adif_to_sql.rb. It is a command line script and needed some adjustments to get it working.
At least it got me from 
{% highlight sql %}
<eoh>

<APP_LoTW_OWNCALL:4>K5GT
<STATION_CALLSIGN:4>K5GT
<CALL:6>CT7ACG
<BAND:3>20M
<FREQ:8>14.07510
<MODE:3>FT8
<APP_LoTW_MODEGROUP:4>DATA
<QSO_DATE:8>20190801
<APP_LoTW_RXQSO:19>2019-08-01 22:16:03 // QSO record inserted/modified at LoTW
<TIME_ON:6>221400
<APP_LoTW_QSO_TIMESTAMP:20>2019-08-01T22:14:00Z // QSO Date & Time; ISO-8601
<QSL_RCVD:1>Y
<QSLRDATE:8>20200921
<APP_LoTW_RXQSL:19>2020-09-21 11:47:03 // QSL record matched/modified at LoTW
<DXCC:3>272
<COUNTRY:8>PORTUGAL
<APP_LoTW_DXCC_ENTITY_STATUS:7>Current
<PFX:3>CT7
<APP_LoTW_2xQSL:1>Y
<GRIDSQUARE:6>IM57VF
<CQZ:2>14
<APP_LoTW_CQZ_Inferred:1>Y // from DXCC entity
<ITUZ:2>37
<APP_LoTW_ITUZ_Inferred:1>Y // from DXCC entity
<eor>
{% endhighlight %}

to

{% highlight sql %}
insert into contacts ('app_lotw_owncall','station_callsign','call','band','freq','mode','app_lotw_modegroup','qso_date','app_lotw_rxqso','time_on','app_lotw_qso_timestamp','qsl_rcvd','qslrdate','app_lotw_rxqsl','dxcc','country','app_lotw_dxcc_entity_status','pfx','app_lotw_2xqsl','gridsquare','cqz','app_lotw_cqz_inferred','ituz','app_lotw_ituz_inferred') values ('K5GT
','K5GT
','CT7ACG
','20M
','14.07510
','FT8
','DATA
','20190801
','2019-08-01 22:16:03 // QSO record inserted/modified at LoTW
','221400
','2019-08-01T22:14:00Z // QSO Date & Time; ISO-8601
','Y
','20200921
','2020-09-21 11:47:03 // QSL record matched/modified at LoTW
','272
','PORTUGAL
','Current
','CT7
','Y
','IM57VF
','14
','Y // from DXCC entity
','37
','Y // from DXCC entity
');
{% endhighlight %}

Finally I used Atom, with find and replace and deleting some info, to come to the seeds.rb file that I used to fill my database.

{% highlight sql %}
Contact.create(
    user_id: 1,
    owncall: 'K5GT',
    station_callsign: 'K5GT',
    my_gridsquare: 'EL15fx',
    call: 'CT7ACG',
    band: '20M',
    freq: '14.07510',
    mode: 'FT8',
    modegroup: 'DATA',
    qso_date: '20190801',
    time_on: '22:14:00',
    qsl_rcvd: 'Y',
    qsl_rdate: '20200921',
    dxcc: '272',
    country: 'PORTUGAL',
    gridsquare: 'IM57vf',
    cqz: '14',
    ituz: '37'
)
{% endhighlight %}

So that was it, at least I had a database. 

In my scope I saw a user that had many contacts and the contacts belonged to a user.

Now the next step, creating an user. Although it wasn't a project requirement I decided still to let the user login.
This brought me to user authentication, and keeping track of the user in the front end.

In my app, the moment a User is created and can be saved in the render json command, an auth_token is generated based on 'user_id: user.id' and exp_time.
{% highlight ruby %}
def create
    user = User.new(user_params)
    if user.save
        render json: {
            auth_token: JsonWebToken.encode({user_id: user.id}, exp_time),
            user: UserSerializer.new(user),
            success: "User created succesfully"
        }
    else
        render json: { 
            errors: user.errors.full_messages 
        }, status: :not_acceptable
    end 
end

private
    
def user_params
    params.require(:user).permit!
end
{% endhighlight %}

In here the JsonWebToken.encode class method is called. In this method we add the expiry time to the payload. The expiry time is based on Time.now and the @@expiry variable.
Because Time.now gives us a string we have to translate it to an integer.
{% highlight javascript %}
Time.now
will give us 2020-12-06 15:23:34 -0600
Time.now.to_i will give us
1607289835
{% endhighlight %}

{% highlight ruby %}
class JsonWebToken
    def self.encode(payload, expiration)
        payload[:exp] = expiration
        JWT.encode(payload, ENV['JWT_SECRET'],'HS256')
    end
    
    def self.decode(token)
        begin
            return JWT.decode(token, ENV['JWT_SECRET'],'HS256')[0]
        rescue
            'FAILED'
        end
    end   
end
{% endhighlight %}

Every time when the front-end communicates with the back-end the auth_token is exchanged. Except for creating a new User, and the login process because at that moment the frontend can not have a auth_token. This exception for the check of the token can be found in the back-end in the users and authentication controller in the skip_before_action :authorized, only: [:create]. In all other front- and back end interactions a check if the action requested is done by an authorized user is performed.

With every interaction after log in the auth_token is refreshed and stored in localStorage.jwt with localStorage.setItem("jwt", json.auth_token). When a request is made to the backend the auth_token is read with its co-companion getItem (localStorage.getItem("jwt")) and send in the back end as a Authorization: `Bearer: ${localStorage.getItem("jwt")`} header in the fetch request.

{% highlight javascript %}
function submitAddContact() {
    const contactData = readContactForm()
    fetch(baseUrl+`/contacts/`, {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
            Authorization: `Bearer: ${localStorage.getItem("jwt")}`
        },
        body: JSON.stringify(contactData)
    })
    .then(response => response.json())
    .then(json => {
        localStorage.setItem("jwt", json.auth_token)
        if (json.message) {
          etc..
{% endhighlight %}


In my project I have two actions of CRUD for the user and all four for the contact. The user's profile page is displayed with information returned from the backend after a successful login and not a read request. A user can create a login by registering and can update the user profile. Contacts can be created, displayed (read), updated and deleted.

Biggest challenge in the whole javascript project was the placement of the addEventListeners. Reason for this is that they can only be declared after the object they are monitoring is written in the DOM otherwise they don't exist and can not be monitored.

<h2>Grid validation, multiple variations possible</h2>
For the Maidenhead grid, because its importance, I added a validation to the values. The Maidenhead grid we need  to calculate the distance and to display the location correctly on the Google Map.
The Maidenhead grid is agreed upon in 1999 when a more global gridsystem was needed as the before used QRA locator grid only covered Europe. The Maidenhead grid is based on the World Geodetic System 1984. My Maidenhead grid is EL15fx62. In each pair (EL, 15, fx, 62) the first character encodes the longitude and the second the latitude. In this grid EL defines the 'field', 15 defines the square, fx defines the sub-square and 62 the extended square. It goes even a level deeper that gives more precision.

![Maidenhead Fields](/assets/img/Maidenhead_Locator_Map.png){:class="img-responsive"}

The size of these fields is 20º longitude and 10º latitude. A sphere is 360º around its equator and 180º south to north. This makes that we divide the globe in 324 fields and can code them by using the letters A-R.
The fields themselves are again divided in 100 squares (10x10) each 2º in longitude and 1º in latitude. The squares can be coded by 2 digits 00-99. At a moderate latitude this will give a square of approximately 70x100 miles.

![Maidenhead Squares](/assets/img/maidenhead_squares.jpg){:class="img-responsive"}

Each square again is divided in 576 sub-squares, coded with 'aa' to 'xx'. Sub-squares cover an area of approxumately 4x6 miles. This is the most used format in the radio amateur world, but still is an approximate location.
A further division is available as the sub-square can be divided again in 100 extended squares which will give a grid of approximately 0.4x0.6 miles, this is as far as I go in my app. 

So for the gridsqaure we have three acceptable formats, in my case EL15, EL15fx and EL15fx62

![EL15](/assets/img/EL15.jpg){:class="img-responsive"}*EL15*

![EL15fx](/assets/img/EL15fx.jpg){:class="img-responsive"}*EL15fx*

![EL15fx62](/assets/img/EL15fx62.jpg){:class="img-responsive"}*EL15fx62*

As you can see, in the above images the more characters in the gridsquare notation the more accurate the position will be. As I mentioned before all these three different notations are valid and have to be accepted in the forms.
In Javascript I added the following validation pattern="[A-R]{2}[0-9]{2}([a-x]{2})?([0-9]{2})?", this will give us a HTML validation and together with some css in the form of:
{% highlight javascript %}
input:invalid {
    background-color: coral;
}
{% endhighlight %}
we can make the background turn coral-red so the user can easier see which value is of a wrong format.

In the regex expression the first part is mandatory and the two later sections ([a-x]{2})? and ([0-9]{2})? are optional. Tried the same expression in my rails validation, but that didn't quite worked like I wanted.
In Rails I ended up with a little longer expression
{% highlight ruby %}
validates_format_of :my_qth, with: /\A([A-R]{2}[0-9]{2})\z|\A([A-R]{2}[0-9]{2}[a-x]{2})\z|\A([A-R]{2}[0-9]{2}[a-x]{2}[0-9]{2})\z/, on: [:create, :update]
{% endhighlight %}
In Rails I had to write out all three possible combinations divided with an 'or' and preceded with \A for start of string and followed by \z for end of string.

For both the distance and the path display on the Google Map we calculate the center of the square determined by the Maidenhead gridsquare. As the 8-character notation still gives us (at moderate latitudes) a square of 0.4x0.6 miles there is no need to use decimals in the distance calculations as the location is still approximate.

<h2>One form for Edit and Add, need for classes</h2>
Both for the User and the Contact I was able to use one form for the edit and create action.
For this I needed to create a JavaScript Object that I could use in the form. The User was the easiest because for the User I didn't need default values. On the other hand for the create Contact I needed some default values when creating the JavaScript Contact object. For this I used the state.page value, this I use also to render the different pages in the render() function. 

{% highlight javascript %}
function contactForm() {
    let title
    if (state.page == "addContact") {
        contact = new Contact
        title = "Add contact"
    } else {
        contact = contactDetail
        title = "Edit contact"
    }
{% endhighlight %}

To overcome empty values in the new object I used a ternary expression to display either nothing or the stored value. See the below example

{% highlight javascript %}
value="${(typeof contact.call == 'undefined') ? "":contact.call}"
{% endhighlight %}

Depending on the state.page I either write the HTML for a button with an id="submitAddContact" or id="submitEditContact.  Also the title of the form is adapted in the same way.

![Contact detail screen](/assets/img/contact_detail.jpg){:class="img-responsive"}

<h2>Conclusion</h2>
Some other projects gave me a hard time of coming up with a subject and how to get it started. This project, because it is hobby related made things a lot easier. Most time I have spend on things that were not a project requirement, like how to use the JsonWebToken for the user authentication. Also implementing the Google maps API was challenging, you have to have a space defined on the page before you can call the map-script. Of course a lot of examples can be found on the world wide web, but never exactly what you are looking for.
All in all it was fun to do, and while I was already near completion of this project, still some things to add or change came up after listening to the online lectures.