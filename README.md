# i18n with Ruby on Rails wiki

## Overview
The following information is based on the latest Ruby on Rails Guide for i18n and developing a standard Rails 5 application.

If you are using Rails and start a new app it provides support for English and similar languages out of the box and makes it easy to customize and extend everything for other languages.

These Rails API features are covered by Rails 5:

* looking up translations
* interpolating data into translations
* pluralizing translations
* using safe HTML translations (view helper method only)
* localizing dates, numbers, currency, etc.

Generally it is very simple to internationalize your app by using the `t (translate)` helper and **.yml files**. The default `en.yml` locale in the config/locales directory contains a sample pair of translation strings. With the `l (localize)` helper you can localize date and time objects to local formats.

Here is an example of how to use **t()** and **l()**:
### Untranslated view
`<h1>Hello World</h1>`

`<p><%= Time.now %></p>`

### Internationalized view
`<h1><%= t 'hello_world' %></h1>`

`<p><%= l Time.now %></p>`

## Tutotial

This quick tutorial shows you how to set up your app for localization.

**Step 1:** Set your default locale (language code), if not **:en (Default = English)** in config/application.rb. (Create an according YAML file, e.g. it.yml, otherwise you will get an exception: ActionView::Template::Error :it is not a valid locale).

**Step 2:** Create a YAML file with the correct locale as **root key** in the config/locales directory. You can name it after the language’s ISO code, e.g. it.yml:

```
it:
  hello: "Ciao mondo"
```

**Step 3:** Use the i18n key in your views or controllers.

View
`<h1><%= t 'hello' %></h1>`

Controller
`flash[:notice] = t('hello')`

**Note:**
Rails adds a `t (translate)` helper method to your views so that you do not need to spell out `I18n.t` all the time. Additionally this helper will catch missing translations and wrap the resulting error message into a `<span class="translation_missing">`.

**Step 4:** Manage the locale across requests by defining a before action in the `ApplicationController`.
```
before_action :set_locale

def set_locale

I18n.locale = params[:locale] || I18n.default_locale

end
```
If the locale is set via the URL to the de locale `(http://localhost:3000?locale=it)`, the response renders the Italian strings.

Getting the locale from params and setting it accordingly is not hard; including it in every URL and thus passing it through the requests is. To not include an explicit option in every URL, e.g. `link_to(root_url(locale: I18n.locale))`, a convenient helper in the ApplicationController comes to the rescue:

```
def default_url_options
  { locale: I18n.locale }
end
```
If you want to have an URL like `http://localhost:3000/it`, which is RESTful and in accord with the rest of the World Wide Web, it does require a little bit more work to implement. Just update the config/routes.rb for an optional **:locale scope** (= subdirectory in this case).

```
scope "(:locale)", locale: /en|it/ do
  root to: 'welcome#index'
end
```
If you want to make Italian your default language for the application. Just update your config/application.rb

```
 config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
 config.i18n.default_locale = :it 
```

For other solutions, like setting the locale from the domain name or getting an user language from the database, please see the official [guide](http://edgeguides.rubyonrails.org/i18n.html#managing-the-locale-across-requests).


### Tips and Tricks

### “Lazy” Lookup – Key Structure
It is common practice to structure your keys according to your views’ files. Rails implements a convenient way to look up the translated text inside **views**. For instance, if you have the following dictionary:

```
it:
  books:
    index:
      title: "Titolo"
```
You can look up the `books.index.title` value inside `app/views/books/index.html.erb` template like this (note the dot):

```
<%= t '.title' %>
```

“Lazy” lookup can also be used in controllers, which is useful for setting flash messages.

## Use ActiveRecord scope to translate (labels of) a Model

If you have a form where you use the helper `form_for` to update a model you can use ActiveRecord keys to define label texts. For instance you have an user model and want to have the email label translated:

```
<% form_for(@user) do |f| %>
  <%= f.label :email %> <%= f.email_field :email %>
<% end %>
```

Just add this to your YAML file – Rails will look it up under the activerecord scope:
```
en:
  activerecord:
    attributes:
      user:
        email: "Your Email Address"
```

## Don’t forget your Numbers and Dates
Internationalization always comes along with localization. Once your application is ready to be localized consider some to localize even the tiniest bit of your product to create an immaculate user experience. When it comes to timesteps it is recommendable to use the **UTC format** to have a unified format. It guarantees that all related information will be displayed correctly.

Throughout the world different formats are used to display dates and time, that’s why Rails has most of the possible formats already [defined per locale](https://github.com/svenfuchs/rails-i18n/tree/master/rails/locale) and they work out of the box. If you want to change or add a format of a specific locale, you can do that by overwriting:

```
it:
  time:
    formats:
      short: "%H:%M"
```

There are also View Helper methods for numbers, like `number_to_currency`, `number_with_precision`, and `number_to_percentage`, etc.

## Detect missing Translations
Sometimes you unintentionally delete i18n keys. Just create an initializer that throws an exception when opening a page with a missing translation in development or test environment.
Find the appropriate code [here](https://stackoverflow.com/questions/8551029/how-to-enable-rails-i18n-translation-errors-in-views/9690449#9690449).

## Migrate from Rails 4 to Rails 5
There is only one difference between these two rails (i18n) versions, that you should know about. If you have used `errors.messages.record_invalid` for overwriting this specific error message, you should change this key to `activerecord.errors.messages.record_invalid` (adds activerecord to the scope). The same applies also for the `restrict_dependent_destroy` key.

## Use other i18n gems
Some Ruby gems like _devise_ for authentication, _will-paginate_ for pagination, etc. are just serving English texts. Due to this restriction, there are other gems available which extend them with additional languages: [devise-i18n](https://github.com/tigrish/devise-i18n), [will-paginate-i18n](https://github.com/tigrish/will-paginate-i18n)

Also when searching for translations of e.g. country names, there are gems available, that help you show your country select box in the user’s language: [i18n-country-translation](https://github.com/onomojo/i18n-country-translations)
