---
layout: post
title: Developing Products with Email and Static Maps
---

_"If a user is only going to spend 10-15 seconds with your map without interacting, why spend two weeks wrestling with your Javascript?"_

This quote from Brian Timoney's post [Few Interact With Our Interactive Maps–What Can We Do About It?](http://mapbrief.com/2017/04/06/few-interact-with-our-interactive-maps-what-can-we-do-about-it/) got me thinking - what are the best approaches to building geospatial software products?

While it is exciting to work on interactive web mapping applications with all of the bells and whistles, this approach is costly and time consuming. You are also likely to see a poor return on investment.

# You might be thinking:

_"I have this cool new idea..."_

or

_"The customer is asking for ZXY."_

and

_"We can develop this tool , with a button, that does this thing!"_

and

_"A web mapping app is the answer!"_

Sure, it might be. But how do you know? Is there product market fit? Are there people willing to pay for what you're building? What is the ROI? These are questions you should be asking and answering before you start.

In Product Management its important to have a strong grasp of the problem you are solving and the people you are solving for. The key here is to provide a solution to a problem many people have. There are numerous approaches to launching products, specifically geospatial products that don't require building a web application. In many cases a static map will suffice.

This article will focus on one of these approaches - Email Products and Static Maps!

# Email Products

You're probably thinking _"Email? BORING!"_ , or _"Email? Does anyone even open those?"_

Yes, yes and yes. Email is hot right now! [The Skimm](http://theskimm.com/?r=4E76W) raised over 8 million dollars in Series B from 20th Century Fox and has a loyal readership in the millions. [The Hustle](http://ambassadors.thehustle.co/?ref=d463affa46) raised over 1 million dollars in seed funding and also has a loyal readership in the hundreds of thousands. I read both of these articles daily. Will a geospatial email product ever get funding or readership like that? No, never, but we can learn a few things from the mainstream consumer products.

## Fail Fast

From a technical perspective, developing an email product is easy. You don't have to worry about wrestling with javascript. You also don't have to think about complex issues that arise in application development, for example scalability, load times, security or authentication. With the help of some open source libraries you needn't worry about responsive design or cross-browser/email client support. Email can be a great avenue to get your content in the hands of your users quickly.

## Understand your users

The beautiful thing about email is the tools are vast and fairly simple to use. Mailchimp, Hubspot, Emma, Sendgrid, etc. all come with email analytics out of the box. You can see exactly who subscribes, unsubscribes, opens and interacts with your email. This is valuable information as you seek to understand your users. Are people picking up what you're putting down? The key performance indicators will flesh out whether or not you really do have product market fit. If your subscribers aren't opening your emails, let alone interacting with them, you better go back to the drawing board before even thinking about developing a web or mobile application.

Now you're probably thinking _"Email! Sweet! Lets do this!"_

## The Tools for the Job

At [RedZone Software](http://redzone.co) we have developed many email products. We now know that people only look at a map for 10 seconds, get the information they need and move on with their day. Show your customers what they need to know to make a decision. What are they interested in? At RedZone our customers want to know where is the fire? and how close is the house?

![_config.yml]({{ site.baseurl }}/images/fire_map.png)

Throw some data on a static map, email it to them and call it a day. _That was easy._

I've learned a few tricks a long the way that will help you get started. I've come to rely on a few technologies for my workflows. I'll go in to detail on what these technologies do and how they all fit together.

### Mapbox Static API

I'm a huge fan of the [Mapbox Static API](https://www.mapbox.com/api-documentation/#introduction). It is quite powerful yet so simple. If you're working with a simple dataset with a few points and a line on a map (Like a Lyft or Uber Email Receipt) you can add those features as [overlays](https://blog.mapbox.com/static-api-with-overlays-932ffc5fcf3d). If you're like RedZone where you would like to include an map of a fire perimeter polygon with a thousand vertices, the overlay will not work due to request URI size restrictions. Lucky for us you can reference a Mapbox Style URL in these Static Map requests.

```
"https://api.mapbox.com/styles/v1/{username}/{style_id}/static/{overlay}/{lon},{lat},{zoom},{bearing},{pitch}{auto}/{width}x{height}{@2x}
```

Where username is your Mapbox account and style_id is a custom map style you've create. Upload your data as a tileset and add it to a custom style and voilà!

Is your data dynamic and constantly being updated? You can programmatically upload new data to a tileset with the Mapbox Upload API and the changes will be reflected in the style that references it. Using Node.js you can upload data with the npm package [mapbox-upload](https://github.com/mapbox/mapbox-upload).

Say you get an update and new data is saved to your database. You ultimately want to get this into your mapbox style. You can set up a process or trigger to grab the new data and upload it to Mapbox. You would use this API to get it done. In just a few lines of code!

```
var upload = require('mapbox-upload');
var progress = upload({
    file: __dirname + '/test.mbtiles', // Path to mbtiles file on disk. You can also use geojson!
    account: 'test', // Mapbox user account.
    accesstoken: 'validtoken', // A valid Mapbox API secret token with the uploads:write scope enabled.
    mapid: 'test.upload', // The identifier of the map to create or update.
    name: 'My upload' // Optional name to set, otherwise a default such as original.geojson will be used.
});
```

So we have the data and know how to create static maps. Lets create an email!

### MJML.io + Handlebars.js

Developing responsive and well designed emails used to be difficult. Developers once had to think about all of the various email clients and how their css would be effected by each of them. [MJML](http://mjml.io) eliminates all of this headache. MJML makes developing clean, responsive and well designed emails a breeze. Its as simple as marking up HTML.

Assuming you want to build a product that is dynamic and flexible, one that doesn't depend user input, you will need a templating system. Using Handlebars.js you can tokenize data points in your mjml and generate your final HTML email to send.

Basic MJML/Handlebars.js template:

```handlebars
<mjml>
  <mj-body>
    <mj-container>
      <mj-section>
        <mj-column>
          <mj-image width="100" src="/assets/img/logo-small.png"></mj-image>
          <mj-text> {% raw %}{{title}}{% endraw %}</mj-text>
          <mj-text> {% raw %}{{date}}{% endraw %}</mj-text>
          <mj-divider border-color="#F45E43"></mj-divider>
          <mj-text font-size="20px" color="#F45E43" font-family="helvetica"> {% raw %}{{info}}{% endraw %}</mj-text>
        </mj-column>
        <mj-column width="600">
            <mj-image class="email-map" src="{% raw %}{{map}}{% endraw %}.png"></mj-image>
        </mj-column>
      </mj-section>
    </mj-container>
  </mj-body>
</mjml>
```

### Nodemailer

Send out emails with node.js and [Nodemailer](https://nodemailer.com/about/).

Lets take a look at an example:

```javascript

//Load in some node modules
var Handlebars = require('handlebars'),
    mjml = require('mjml').mjml2html;

var nodemailer = require('nodemailer'),
    sendmailTransport = require('nodemailer-sendmail-transport'),
    emailConf = "/usr/sbin/sendmail";

var transporter = nodemailer.createTransport(sendmailTransport({
    path: emailConf
}));

//Get some data
var data = {
    template_data : {
        title: "Template Example",
        Date: "02/04/2018",
        info: "This is really useful information",
        map:"https://api.mapbox.com/styles/v1/mapbox/outdoors-v10/static/-104.98150,39.73542,10.1,0,0/300x200?access_token=pk.eyJ1IjoicmVkem9uZXNvZnR3YXJlIiwiYSI6IkNKNXhPNkUifQ.yPI1bZL3nEvDFbdQjeTB-A"
    },
    email_data: [{
        to: "olamb@redzone.co",
        subject: "This is a test email"
    }]
};

//Compile the mjml/handlebars template
var template = Handlebars.compile(PATH_TO_MJML_TEMPLATE);

//Call function to add data to the template and convert to HTML
var html = mjmlToHtml(template, data.template_data);


// Set mail options
var mailOptions = {
    to: data.email_data.to,
    from: 'no-reply@email.com',
    subject: data.email_data.subject,
    html: html
};

//Send It!
transporter.sendMail(mailOptions, function(err, result) {
    if (err) {
        console.log("Uh Oh")
    } else {
        console.log("Success!")
    }
});

function mjmlToHtml(template, data) {
    var content = template(data),
        htmlOut = mjml(content);
    return htmlOut.html
}
```

And there you have it! That should send out a nice responsive email with a static map :)
