# Kaminari

A Scope & Engine based, clean, powerful, customizable and sophisticated paginator for Rails 3


## Features

### Clean

Does not globally pollute *Array*, *Hash*, *Object* or <tt>AR::Base</tt>.

### Easy to use

Just bundle the gem, then your models are ready to be paginated. No configuration required. Don't have to define anything in your models or helpers.

### Simple scope-based API

Everything is method chainable with less "Hasheritis". You know, that's the Rails 3 way.
No special collection class or anything for the paginated values, instead using a general <tt>AR::Relation</tt> instance. So, of course you can chain any other conditions before or after the paginator scope.

### Customizable engine-based I18n-aware helper

As the whole pagination helper is basically just a collection of links and non-links, Kaminari renders each of them through its own partial template inside the Engine. So, you can easily modify their behaviour, style or whatever by overriding partial templates.

### ORM & template engine agnostic

Kaminari supports multiple ORMs (ActiveRecord, Mongoid, MongoMapper) and multiple template engines (ERB, Haml).

### Modern

The pagination helper outputs the HTML5 \<nav\> tag by default. Plus, the helper supports Rails 3 unobtrusive Ajax.


## Supported versions

* Ruby 1.8.7, 1.9.2, 1.9.3 (trunk)

* Rails 3.0.x, 3.1 (edge)

* Haml 3

* Mongoid 2

* MongoMapper 0.9

* DataMapper 1.1.0

## Install

Put this line in your Gemfile:

    gem 'kaminari'

Then bundle:

    % bundle


## Usage

### Query Basics

* the *page* scope

  To fetch the 7th page of users (default *per_page* is 25)

    User.page(7)

* the *per* scope

  To show a lot more users per each page (change the *per_page* value)

    User.page(7).per(50)

  Note that the *per* scope is not directly defined on the models but is just a method defined on the page scope. This is absolutely reasonable because you will never actually use *per_page* without specifying the *page* number.

* the *padding* scope

  Occasionally you need to padding a number of records that is not a multiple of the page size.

    User.page(7).per(50).padding(3)

  Note that the *padding* scope is not directly defined on the models but is just a method defined on the page scope. This is absolutely reasonable because you can use *offset* otherwise.

### General configuration options

You can configure the following default values by overriding these values using <tt>Kaminari.configure</tt> method.

    default_per_page  # 25 by default
    window            # 4 by default
    outer_window      # 0 by default
    left              # 0 by default
    right             # 0 by default

There's a handy generator that generates the default configuration file into config/initializers directory.
Run the following generator command, then edit the generated file.

    % rails g kaminari:config

### Configuring default *per_page* value for each model

* *paginates_per*

  You can specify default *per_page* value per each model using the following declarative DSL.

    class User < ActiveRecord::Base
      paginates_per 50
    end

### Controllers

* the page parameter is in <tt>params[:page]</tt>

  Typically, your controller code will look like this:

    @users = User.order(:name).page params[:page]

### Views

* the same old helper method

  Just call the *paginate* helper:

    <%= paginate @users %>

  This will render several <tt>?page=N</tt> pagination links surrounded by an HTML5 <*nav*> tag.

### Helper Options

* specifing the "inner window" size (4 by default)

    <%= paginate @users, :window => 2 %>

  This would output something like <tt>... 5 6 7 8 9 ...</tt> when 7 is the current page.

* specifing the "outer window" size (0 by default)

    <%= paginate @users, :outer_window => 3 %>

  This would output something like <tt>1 2 3 4 ...(snip)... 17 18 19 20</tt> while having 20 pages in total.

* outer window can be separetely specified by *left*, *right* (0 by default)

    <%= paginate @users, :left => 1, :right => 3 %>

  This would output something like <tt>1 ...(snip)... 18 19 20</tt> while having 20 pages in total.

* changing the parameter name (:*param_name*) for the links

    <%= paginate @users, :param_name => :pagina %>

  This would modify the query parameter name on each links.

* extra parameters (:*params*) for the links

    <%= paginate @users, :params => {:controller => 'foo', :action => 'bar'} %>

  This would modify each link's *url_option*. :*controller* and :*action* might be the keys in common.

* Ajax links (crazy simple, but works perfectly!)

    <%= paginate @users, :remote => true %>

  This would add <tt>data-remote="true"</tt> to all the links inside.

### I18n and labels

The default labels for 'previous', '...' and 'next' are stored in the I18n yaml inside the engine, and rendered through I18n API. You can switch the label value per I18n.locale for your internationalized application.
Keys and the default values are the following. You can override them by adding to a YAML file in your <tt>Rails.root/config/locales</tt> directory.

    en:
      views:
        pagination:
          previous: "&laquo; Prev"
          next: "Next &raquo;"
          truncate: "..."

### Customizing the pagination helper

Kaminari includes a handy template generator.

* to edit your paginator

  Run the generator first,

    % rails g kaminari:views default

  then edit the partials in your app's <tt>app/views/kaminari/</tt> directory.

* for Haml users

  Haml templates generator is also available by adding the <tt>-e haml</tt> option (this is automatically invoked when the default template_engine is set to Haml).

    % rails g kaminari:views default -e haml

* themes

  The generator has the ability to fetch several sample template themes from
  the external repository (https://github.com/amatsuda/kaminari_themes) in
  addition to the bundled "default" one, which will help you creating a nice
  looking paginator.

    % rails g kaminari:views THEME

  To see the full list of avaliable themes, take a look at the themes repository,
  or just hit the generator without specifying *THEME* argument.

    % rails g kaminari:views

* multiple themes

  To utilize multiple themes from within a single application, create a directory within the app/views/kaminari/ and move your custom template files into that directory.

    % rails g kaminari:views default (skip if you have existing kaminari views)
    % cd app/views/kaminari
    % mkdir my_custom_theme
    % cp _*.html.* my_custom_theme/

  Next reference that directory when calling the paginate method:

    <%= paginate @users, :theme => 'my_custom_theme' %>

  Customize away!

  Note: if the theme isn't present or none is specified, kaminari will default back to the views included within the gem.

### Paginating a generic Array object

Kaminari provides an Array wrapper class that adapts a generic Array object to the <tt>paginate</tt> view helper.
However, the <tt>paginate</tt> helper doesn't automatically handle your Array object (this is intentional and by design).
<tt>Kaminari::paginate_array</tt> method converts your Array object into a paginatable Array that accepts <tt>page</tt> method.

    Kaminari.paginate_array(my_array_object).page(params[:page]).per(10)

## Creating friendly URLs and caching

Because of the `page` parameter and Rails 3 routing, you can easily generate SEO and user-friendly URLs. For any resource you'd like to paginate, just add the following to your `routes.rb`:

    resources :my_resources do
      get 'page/:page', :action => :index, :on => :collection
    end

This will create URLs like `/my_resources/page/33` instead of `/my_resources?page=33`. This is now a friendly URL, but it also has other added benefits...

Because the `page` parameter is now a URL segment, we can leverage on Rails page [caching](http://guides.rubyonrails.org/caching_with_rails.html#page-caching)!

NOTE: In this example, I've pointed the route to my `:index` action. You may have defined a custom pagination action in your controller - you should point `:action => :your_custom_action` instead.

## Sinatra/Padrino supports

After December 2011, kaminari started to support Sinatra or Sinatra-based frameworks experimentally.

To use kaminari and its helpers with these frameworks, please just

    require 'kaminari/sinatra'

or edit gemfile:

    gem 'kaminari', :require => 'kaminari/sinatra'

More features are coming, and please raise an [issue](https://github.com/amatsuda/kaminari/issues) if something would be occurred around Sinatra.

## For more information

Check out Kaminari recipes on the GitHub Wiki for more advanced tips and techniques.
https://github.com/amatsuda/kaminari/wiki/Kaminari-recipes

## Build Status {<img src="https://secure.travis-ci.org/amatsuda/kaminari.png"/>}[http://travis-ci.org/amatsuda/kaminari]

== Build Status {<img src="https://secure.travis-ci.org/rails/rails.png"/>}[http://travis-ci.org/rails/rails]

## Questions, Feedback

Feel free to message me on Github (amatsuda) or Twitter (@a_matsuda)  ☇☇☇  :)


## Contributing to Kaminari

* Fork, fix, then send me a pull request.


## Copyright

Copyright (c) 2011 Akira Matsuda. See LICENSE.txt for further details.
