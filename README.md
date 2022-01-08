# Hotwire: Dynamic forms with Stimulus

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.png)][heroku-deploy-app]

[heroku-deploy-app]: https://heroku.com/deploy?template=https://github.com/thoughtbot/hotwire-example-template/tree/hotwire-example-stimulus-dynamic-forms

Imagine a page to share a Document with varying levels of access. Marking a
Document as "publish" grants public access, marking it as "draft" limits access
to the document's creator, and marking it as "passcode protected" grants access
to anyone who knows the passcode.

A missing "passcode" value shouldn't block the creation of a Document with
"publish"- or "draft"-level access. Likewise, a Document marked with "passcode
protected" access without a "passcode" value is invalid.

Interactivity is a core selling point for client-side rendering frameworks.
Their value propositions are at their most compelling when they demonstrate
their ability to change a page's shape and content in response to end-user
actions.

If we built this page with a client-side rendering framework, our form could
store the selected level of access in-memory as a JavaScript object. We could
use that stored value to determine whether or not to render parts of the page to
collect a "passcode" value. In response to a change in the access level, our
framework could re-render the pertinent portions of the page, all without
additional communication with the server.

Unfortunately, server-side rendering frameworks don't have that luxury. The
server renders the page once, and only once when responding to an HTTP request.
If we built a version of this feature with a server-side rendering framework,
what would it take to achieve a similar level of interactivity and network
efficiency?

## Progressively enhancing server-generated HTML

We'll start by establishing a baseline version that renders HTML retrieved over
HTTP without any JavaScript. Our version will rely on the server to _always_
render _all_ of the form's fields, including those that collect the "passcode".

Our initial version will rely on full-page navigations and round-trips to the
server to fetch updated HTML. It'll even work in the absence of JavaScript.

Once we've established a foundation, we'll [progressively enhance][] the form
with JavaScript, making it more interactive with each incremental improvement.

The code samples shared in this article omit the majority of the application’s
setup. The initial code was generated by executing `rails new`. The rest of the
[source code][] from this article (including a [suite of tests][]) can be found
on GitHub, and is best read [commit-by-commit][].

[progressively enhance]: https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement
[source code]: https://github.com/thoughtbot/hotwire-example-template/tree/hotwire-example-stimulus-dynamic-forms
[suite of tests]: https://github.com/thoughtbot/hotwire-example-template/tree/hotwire-example-stimulus-dynamic-forms/test
[commit-by-commit]: https://github.com/thoughtbot/hotwire-example-template/compare/hotwire-example-stimulus-dynamic-forms

## Our starting point

We'll declare a `Document` model backed by [Active Record][]. The `Document`
class defines an [enumeration][] to outline the possible levels of access,
accepts [Action Text][] `content`, and declares presence [validations][]:

```ruby
class Document < ApplicationRecord
  enum :access, publish: 0, draft: 1, passcode_protected: 2

  has_rich_text :content

  with_options presence: true do
    validates :content
    validates :passcode, if: :passcode_protected?
  end
end
```

[Active Record]: https://guides.rubyonrails.org/active_record_basics.html
[Action Text]: https://edgeguides.rubyonrails.org/action_text_overview.html
[enumeration]: https://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html
[validations]: https://edgeguides.rubyonrails.org/active_record_validations.html

Our `Document` records are managed by a conventional `DocumentsController`
class:

```ruby
# app/controllers/documents_controller.rb

class DocumentsController < ApplicationController
  def new
    @document = Document.new
  end

  def create
    @document = Document.new document_params

    if @document.save
      redirect_to document_url(@document)
    else
      render :new, status: :unprocessable_entity
    end
  end

  def show
    @document = Document.find params[:id]
  end

  private

  def document_params
    params.require(:document).permit(
      :access,
      :passcode,
      :content,
    )
  end
end
```

The `app/views/documents/new.html.erb` template collects the `Document` records'
access level through a [group][] of [`<input type="radio">`][radio] elements,
collects the `content` through an Action Text-powered [`<trix-editor>`][trix]
element, and submits the [`<form>`][form] element as a `POST` request to the
`DocumentsController#create` action:

[group]: https://edgeapi.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html#method-i-collection_radio_buttons
[trix]: https://trix-editor.org
[radio]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio
[form]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form

```erb
<%# app/views/documents/new.html.erb %>

<section class="w-full max-w-lg">
  <h1>New document</h1>

  <%= form_with model: @document do |form| %>
    <%= render partial: "errors", object: @document.errors %>

    <%= field_set_tag "Access" do %>
      <%= form.collection_radio_buttons :access, Document.accesses.keys, :to_s, :humanize do |builder| %>
        <span>
          <%= builder.radio_button %>
          <%= builder.label %>
        </span>
      <% end %>
    <% end %>

    <%= field_set_tag "Passcode protected" do %>
      <%= form.label :passcode %>
      <%= form.text_field :passcode %>
    <% end %>

    <%= form.label :content %>
    <%= form.rich_text_area :content %>

    <%= form.button %>
  <% end %>
</section>
```

![A form collecting information about a Document, including its access level and content](https://images.thoughtbot.com/blog-vellum-image-uploads/gVLCYBB9QQq0Nyw2ASxy_150657727-09919557-322c-4697-b529-0703b418c470.png)

When the submission is valid, the record is created, the data is written to the
database, and the controller serves an [HTTP redirect response][redirect] to the
`DocumentsController#show` action.

When the submission's data is invalid, the controller responds with a [422
Unprocessable Entity][422] status and re-renders the `app/views/documents/new.html.erb`
template to include the `app/views/application/_errors.html.erb` partial. That
partial's [source code][_errors] is omitted here, but draws inspiration from the
template that [Rails scaffolds for new models][scaffolds]:

[redirect]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections
[422]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/422
[scaffolds]: https://github.com/rails/rails/blob/984c3ef2775781d47efa9f541ce570daa2434a80/railties/lib/rails/generators/erb/scaffold/templates/_form.html.erb.tt#L2-L12
[_errors]: https://github.com/thoughtbot/hotwire-example-template/blob/hotwire-example-stimulus-dynamic-forms/app/views/application/_errors.html.erb

![Validation error messages rendered above the form's fields](https://images.thoughtbot.com/blog-vellum-image-uploads/VMkuufpQWuUuureWAcof_150657724-98d59bc0-4eda-4f75-bc83-3e2f587e3ec8.png)
