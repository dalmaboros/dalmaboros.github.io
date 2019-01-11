---
layout: post
title:      "What Is `remote: true` And Why Can't I Use It?"
date:       2019-01-11 14:45:49 +0000
permalink:  what_is_remote_true_and_why_cant_i_use_it
---


Amongst the many technical requirements for the Rails With JavaScript portfolio project, one in particular stands out near the top of the list, in bold monospaced type:

**Do not use `remote: true` in this application.**

I immediately want to use `remote: true`. But I don't know what it is. If I at one point learned what this is, I can't remember. A Google search for "remote: true" yields some [barely-there Rails documentation](https://guides.rubyonrails.org/working_with_javascript_in_rails.html#link-to) on Rails Guides, and a handful of more promising articles entitled [*AJAX and Rails: Demystifying :remote => true*](https://medium.com/@AdamKing0126/ajax-and-rails-demystifying-remote-true-fe51ba2ce819), [*How to Use remote: true to Make Ajax Calls in Rails*](https://medium.com/@codenode/how-to-use-remote-true-to-make-ajax-calls-in-rails-3ecbed40869b), and [*:remote => true in Rails Forms*](http://www.korenlc.com/remote-true-in-rails-forms/).

By this limited information I can gather that `remote: true` is used to make AJAX calls in Rails. But this inference doesn't give me a solid grasp on the concept, so I am forced to read at least one of these articles.

## What is `remote: true`?

`remote: true` is a Rails helper. This helper can be added to any other Rails helper that generates a web request. For example, `remote: true` can be added to the `link_to` and `button_to` helpers, like so:

`<%= link_to "an article", @article, remote: true %>`

or

`<%= button_to "An article", @article, remote: true %>`

**So, what does `remote: true` do?**

The `remote: true` helper prevents the default action of the object from occuring, and submits an asynchronous web request to the object's associated controller action. Using the examples above, a `remote: true` helper added to either the `link_to` or `button_to` helper would submit an asynchronous request to the `show` action of the `articles` controller.

**What does `remote: true` not do?**

The `remote: true` helper does nothing on the server side. It doesn't handle the AJAX request, nor the server's response, and it does not manipulate the DOM. These things must still be coded manually in the controller and with JavaScript on the clientside in order to render data from the server without a page refresh.

## And Why Can't I Use It?

The `remote: true` helper is a "Rails magic" shortcut for sending asynchronous web requests to the server. It eliminates the need to code out explicit asynchronous web requests, such as:

```
$(function(){
  $("#my_form").on("submit", function(event){
    $.ajax({
      type: "POST",
      url: this.action,
      data: $(this).serialize(),
      success: function(response) {
        //update the DOM
      }
    });
    event.preventDefault();
  });
})
```

Instead, the `remote: true` helper can be added to a `form_for` helper:

```
<%= form_for @article, :remote => true do |f| %>
  <%= f.text_field :title %>
  <%= f.submit %>
<% end %>
```

Both methods allow form data to be submitted to the server without refreshing the page.

The Rails with JavaScript portfolio project is an exercise in understanding how to add dynamic features to a Rails application that are possible only through JavaScript and a JSON API. Without the use of `remote: true`, we are forced to explicitly code  asynchronous web requests such as `jQuery.ajax()` or `fetch()`. Like many other "Rails magic" shortcuts introduced to us in the [Learn.co](http://learn.co) curriculum, we don't use it so we can understand how it works. 
