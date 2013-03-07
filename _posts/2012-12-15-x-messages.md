---
layout: post
title: X-Messages
date: 2012-12-15
category:
tags: ['http', rails']
---

<div class='callout github'>
  An example application for this post is available <a href='http://github.com/jpsilvashy/x-messages'>on Github</a>. The application itself is deployed <a href='http://x-messages.herokuapp.com'>on Heroku</a>.
  {% include octocat.svg %}
</div>

## Using headers to provide additional response information

While working on [Flyer](http://flyer.io), I wanted to try something different with handling flash messages. I've always wanted to get more creative with the usage of response headers so I decided to try to use those as a transport for the flash messages typically handled in the views by just rendering some html messages at the top of the page.

If you'd like to checkout the "mostly" working example, head over to [github.com/jpsilvashy/x-headers](http://github.com/jpsilvashy/x-messages), or check out the app running on heroku at [x-messages.herokuapp.com](http://x-messages.herokuapp.com).

### Why? You should just render some JavaScript in the response body instead!

So the biggest reason why I wanted to sort of ignore the "Rails Way" on this one is to try to extend the functionality beyond a rails application. The major idea here is that we can use a header named `X-Messages` to deliver more universal information about the response. I feel there is so much more informtion than just a 200 status that should be represented in a familiar place amongts all HTTP applications.

By using a common header like `X-Messages` all our applications can send additional response information for any format, not just HTML or JSON.  Lastly our clients will all know to look in a specific place for any sort of request.

In this example I've built a Rails application which returns the `X-Messages` header with the same information that `ActionController::Flash` would via the `flash` method.

There isn't much to it, and it won't affect the current functionality of you app either, add this to your `application_controller.rb`:


<pre class="prettyprint lang-ruby">
# After filter to insert the headers in the response
after_filter :x_messages

# Also make available as a helper so we can render
# the errors of the user doesn't have js
helper_method :x_messages

private

# If there is a flash present, insert into header
def x_messages
  if flash.present?
    response.headers['X-Messages'] = flash.to_hash.to_json
    flash.to_hash
  end
end
</pre>

Then in your view you can do a variety of things, I created a helper which would render the errors at the top of the page if the user doesn't have JS (in the case where the client is obviously a web browser), but we also render an array of the same objects in the JS near the bottom of the page so we can just access that when the DOM is ready.

Here is the helper I made to render the messages in the page for the users which don't have JS:

<pre class="prettyprint lang-ruby">
def render_x_messages
  if x_messages
    content_tag :ul, :class => 'x-messages' do
      x_messages.collect do |type, message|
        concat content_tag(
          :li, message, :class => "message-#{type}"
        )
      end
    end
  end
end
</pre>

Put that helper anywhere in your layout where you want to render the messages. Then by the footer I'm also inserting some JavaScript for us to iterate over in our JS after the page loads.

<pre class="prettyprint lang-ruby">
// You probably use erb or haml, this is just plain ruby
if x_message
  var x_messages = x_messages.to_json.html_safe
end
</pre>

Make sure to update the manifest for the JS files and include Alertify.js, then lets start handling the response for AJAX requests.

This function reads the header from the XHR response and parses the content in `X-Messages` as JSON.

<pre class="prettyprint">
var get_x_messages = function(response) {

  var x_messages_header =
    response.getResponseHeader('X-Messages');

  // Parse the header if there is one
  if (typeof x_messages_header !== "undefined") {
    return $.parseJSON(x_messages_header)
  }
}
</pre>

This is a pretty straight-forward function for creating an Alertify log message for each message in the parse `X-Messages` header.

<pre class="prettyprint">
// For each message, alert the user
var handle_messages = function(messages) {
  $.each(messages, function(type, message) {
    alertify.log(message, type);
  });
}
</pre>

This function is the main way we intercept the headers when they are returned to the client. This allows us to watch for ajax events, and when they are done handle the response with the first function we created, `get_x_messages()`.

<pre class="prettyprint">
// Get messages from response header
$(document).ajaxComplete(function(event, response) {
  handle_messages(get_x_messages(response))
});
</pre>

Lastly when the DOM is loaded we want to hide (or `remove()` for simplicty) the flash messages that were rendered in the page.

Meanwhile, we also want to see if there are messages on the page that just loaded, and since the reponse would be for standard HTML, we wouln't have the XHR object available to us. This stems from the way the DOM and BOM were created to seperate the two. There would be no real way to read the response headers "after the fact" with JavaScript, unless we made another request to our current path with XHR. The solution I found was to render the messages in the application layout as a JSON object.

<pre class="prettyprint">
$(function() {

  // hide these since we have alertify to show our messages
  $('.x-messages').remove();

  if (typeof x_messages !== "undefined") {
    handle_messages(x_messages);
  }
})
</pre>

This allows us to handle existing errors that come with the page's response, and any subsequent responses from remote requests in the same way. This is because the reponses made while already on the page with XHR will be caught by the `ajaxComplete()` event handler and the other ones will be handled when the page loaded.


