* You MUST NOT try and generate a Rails app from scratch on your own by generating each file. For a NEW app you MUST use `rails new` first to generate all of the boilerplate files necessary.
* Create an app in the current directory with `rails new .`
* Use Tailwind CSS for styling. Use `--css tailwind` as an option on the `rails new` call to do this automatically.
* Use Ruby 3.2+ and Rails 8.0+ practices.
* Use the default Minitest approach for testing, do not use RSpec.
* Default to using SQLite in development. `rails new` will do this automatically but take care if you write any custom SQL that it is SQLite compatible.
* An app can be built with a devcontainer such as `rails new myapp --devcontainer` but only do this if requested directly.
* Rails apps have a lot of directories to consider, such as app, config, db, etc.
* Adhere to MVC conventions: singular model names (e.g., Product) map to plural tables (products); controllers are plural.
* Guard against incapable browsers accessing controllers with `allow_browser versions: :modern`
* `bin/rails server` runs the current app locally, but you can use `bin/dev` to run the app along with background jobs, Tailwind CSS watcher, and other niceties included in `Procfile.dev`.
* Rails 8 introduces a new `script` folder for one-off or general-purpose scripts, you can create a script with `bin/rails generate script my_script` and then such scripts can be run with `bundle exec ruby script/my_script.rb`
* Do not use Sprockets, it is old fashioned in Rails 8.
* Rails' `runner` command can be used to run one liners, e.g. `bin/rails runner "user = User.new; user.email_address='x@x.com'; user.save"` and so forth.
* Models are created like so: `bin/rails generate model Product name:string`
* Use Rails' built-in generators for models, controllers, and migrations to enforce Rails standards.
* `bin/rails db:migrate` runs database migrations after you have added or changed models.
* Models are queried like so: `Product.all`, `Product.where(name: "Pants")`, `Product.order(name: :asc)`, `Book.where("title = ?", params[:title])`, `Book.where("created_at >= :start_date AND created_at <= :end_date",{ start_date: params[:start_date], end_date: params[:end_date] })` - these can also be chained for more complex queries.
* Range queries can also be done: `Book.where(created_at: (Time.now.midnight - 1.day)..Time.now.midnight)`
* SQL's `IN` can be mimicked like so: `Customer.where(orders_count: [1, 3, 5])` and `Customer.where.not(orders_count: [1, 3, 5])`
* Chained queries/conditions act like `AND` in SQL. To do an `OR` connection, you can do this: `Customer.where(last_name: "Smith").or(Customer.where(orders_count: [1, 3, 5]))`
* You can order by multiple columns: `Book.order(title: :asc, created_at: :desc)`
* `LIMIT` and `OFFSET` are possible: `Customer.limit(5).offset(30)`
* `GROUP BY` is possible: `Order.select("created_at").group("created_at")` as well as group counts: `Order.group(:status).count`
* `HAVING` is done like so: `Order.select("created_at as ordered_date, sum(total) as total_price").group("created_at").having("sum(total) > ?", 200)`
* You can specify certain conditions to be removed using the `unscope` method.
* Scopes can be used to define named queries on models, e.g. `scope :in_print, -> { where(out_of_print: false) }`, `scope :out_of_print_and_expensive, -> { out_of_print.where("price > 500") }` 
* There can be a default scope for models: `default_scope { where(active: true) }`
* Scopes can be merged in queries by chaining their calls.
* The `find_or_create_by` method checks whether a record with the specified attributes exists. If it doesn't, then `create` is called.
* Custom SQL can be used if strictly necessary: `Customer.find_by_sql("SELECT * FROM customers INNER JOIN orders ON customers.id = orders.customer_id ORDER BY customers.created_at desc")`
* `exists?` can be used to check if something simply exists: `Customer.exists?(1)` .. for larger numbers `any?` and `many?` can be used.
* Models can have also enums, like so: `enum :status, [:shipped, :being_packed, :complete, :cancelled]`
* Use `annotate` in query chains as a way to describe what the query is doing as this will appear in the logs for debugging purposes, e.g. `User.annotate("selecting user names").select(:name)`
* This is an efficient way to work with large groups of results: `Customer.where(weekly_subscriber: true).find_each do |customer|`
* Use RESTful routing (prefer resources :products) and URL helpers for consistency.
* Implement strong parameters in controllers to whitelist permitted attributes. For example: `def product_params; params.expect(product: [ :name, :description ]); end`
* Use validations in models to ensure data fits relevant constraints, e.g. `validates :name, presence: true` or `validates :inventory_count, numericality: { greater_than_or_equal_to: 0 }`
* Rails routes look like so: `get "/products", to: "products#index"` and are placed in `config/routes.rb`. Sometimes they are automatically generated by generators.
* Use `before_action` callbacks to DRY common tasks (e.g., loading records). Actions can be scoped to particular methods: `before_action :set_product, only: %i[ show edit update ]`
* Active Record has many callbacks including `after_create`, `before_validation`, `after_validation`, `before_save`, `before_create`, `after_create`, `before_destroy`, `after_initialize`, `after_find`, `after_touch`. Validations can be used conditionally like so: `before_save :normalize_card_number, if: :paid_with_card?` .. which can also take a lambda: `before_save :normalize_card_number, if: ->(order) { order.paid_with_card? }`
* If any callback raises an exception, the execution chain gets halted and a rollback is issued, and the error will be re-raised. For a softer failure such as for creating an object you can use `throw` like so: `throw :abort if total_price < 0`
* Active Record models can have many 'associations' to connect models together, e.g. `has_many :books, dependent: :destroy`, `belongs_to :author`, `has_one`, `has_many :through`, and `has_and_belongs_to_many`
* Write views with ERB using <%= %> for output and <% %> for logic; extract shared code into partials, e.g. `<%= render "form", product: @product %>` and `<%= render partial: "product", locals: { product: @product } %>` - in the latter case `<% local_assigns[:product] %>` is used in the partial to access the local passed.
* Due to conventions `<%= render "product", product: @product %>` can be shortened to `<%= render @product %>` as Rails can figure out the model from the object.
* Collections of partials can be rendered: `<%= render partial: "product", collection: @products %>`
* Partials can be rendered with 'spacer' templates in between: `<%= render partial: @products, spacer_template: "product_ruler" %>`
* Manage assets via Propshaft and import maps; use Hotwire (Turbo/Stimulus) for JS without extra build steps.
* Adopt Active Storage and Action Text for file uploads and rich text editing.
* Action Text is an included way to get rich text fields out of the box. It can be installed like so: `bin/rails action_text:install` after which you need to bundle install and run DB migrations again. Then models can get things like `has_rich_text :description` and `<%= form.rich_text_area :description %>` can be used in a view to render a rich text field based on Trix.
* `bin/rails routes` can be run to see all current routes of an app if needed.
* Generators look like `bin/rails generate controller Products index --skip-routes` and `bin/rails generate model Product name:string` but it is also possible to create 'scaffold's that will flesh out a model, controller, and views for a defined set of columns.
* Views have many helpers such as for creating links: `<%= link_to "New product", new_product_path %>`, or forms: `<%= form_with model: @product do |form| %>` and form fields: `<%= form.text_field :name %>`
* Rails 8 includes a new authentication generator which can be run with `bin/rails generate authentication` - this creates User and Session models (run migrations after). Users have default columns including `email_address`, `password` and `password_confirmation`
* Logging out can be done with something like `<%= button_to "Log out", session_path, method: :delete if authenticated? %>`
* Unauthenticated access can be allowed on controller methods like so: `allow_unauthenticated_access only: %i[ index show ]`
* There's a `authenticated?` helper for use in views, e.g. `<%= link_to "New product", new_product_path if authenticated? %>`. Similarly you could show a Login link: `<%= link_to "Login", new_session_path unless authenticated? %>`
* Caching can be done on parts of views like so: `<% cache @product do %><h1><%= @product.name %></h1><% end %>`
* Active Storage is a Rails 8 library that makes it easy to upload and store files, including from rich text fields. You could attach a file to a model like so: `has_one_attached :featured_image` then have a form field like so: `<%= form.file_field :featured_image, accept: "image/*" %>` then eventually display an image like so: `<%= image_tag @product.featured_image if @product.featured_image.attached? %>`
* Uploaded files can be handled in controller methods like so: `uploaded_file = params[:csv_file]`, `if uploaded_file.present?`, and `uploaded_file.read`
* Internationalization (i18n) can be done by using the `t` helper in views like so: `<h1><%= t "hello" %></h1>` and then the matching key "hello" can be used in files like `config/locales/en.yml` to define the strings in each locale's language. For example, in `config/locales/es.yml` you could have `es:\n  hello: "Hola mundo:`. These are YAML files.
* Locales for i18n could be switched like so: `around_action :switch_locale\n\ndef switch_locale(&action)\nlocale = params[:locale] || I18n.default_locale\nI18n.with_locale(locale, &action)\nend`
* Dotted keys in `t` calls can be used to do relative locale lookups: e.g. `<h1><%= t ".title" %></h1>` - this would then look under keys matching the controller and view name in the YAML file. For example: `en:\n  hello: "Hello world"\n  products:\n    index:\n      title: "Products"`
* Action Mailer is a part of Rails for sending emails. Mailers can be created like so: `bin/rails g mailer Product in_stock` and then methods in mailers can be used a bit like a controller to find objects and then send emails: `mail to: params[:subscriber].email` which would then render views like `app/views/product_mailer/in_stock.text.erb` to produce the email content.
* To better organize model code, common elements can be extracted into 'concerns' which can be included in multiple models in files such as `app/models/product/notifications.rb` and code like `module Product::Notifications\n  extend ActiveSupport::Concern\n\n  included do\n    has_many :subscribers, dependent: :destroy\n    after_update_commit :notify_subscribers, if: :back_in_stock?\n  end\n\n  normal methods here..\nend` then in models you could use `include Notifications` to being in that concern.
* Active Record has a feature called `generates_token_for` that can generate unique tokens to find database records for different purposes. You can use this for generating a unique unsubscribe token to use in the email's unsubscribe URL, e.g. `generates_token_for :unsubscribe` and then look it up like so: `@subscriber = Subscriber.find_by_token_for(:unsubscribe, params[:token])` and in the view: `<%= link_to "Unsubscribe", unsubscribe_url(token: params[:subscriber].generate_token_for(:unsubscribe)) %>`
* Rails' asset pipeline is called Propshaft. It takes CSS, JavaScript, images, and other assets and serves them to the browser so if `app/assets/stylesheets/application.css` is changed, say, it all just works.
* Rails uses import maps for JavaScript by default. You can find the JavaScript pins in `config/importmap.rb` - they look like `pin "@hotwired/stimulus", to: "stimulus.min.js"` and `pin_all_from "app/javascript/controllers", under: "controllers"`
* Hotwire is a default Rails JavaScript framework designed to take full advantage of server-side generated HTML. It includes Turbo for handling navigation, form submission, page components and updates. Stimulus is a JS framework for introducing custom JS to pages. Native is used for making hybrid mobile apps.
* `bin/rubocop` can be run to check code quality and formatting.
* `bin/brakeman` can be run to check security issues with the code.
* Solid Queue is a new part of Rails for running tasks asynchronously behind-the-scenes in a separate process with ActiveJob.
* Solid Cable is used with Action Cable to use WebSockets with Rails apps without needing Redis.
* Solid Cache is a Redis-free cache store for ActiveSupport.
* Radio buttons in views look like so: `<%= form.radio_button :flavor, "chocolate_chip" %>` or you can do them on a group of values: `<%= form.collection_radio_buttons :city_id, City.order(:name), :id, :name %>`
* Labels in forms look like so: `<%= form.label :flavor_chocolate_chip, "Chocolate Chip" %>`
* There are many view helpers, such as `<%= form.date_field :born_on %>`, `<%= form.time_field :started_at %>`, `<%= form.password_field :password %>`, `<%= form.email_field :address %>`, `<%= form.url_field :homepage %>`, `<%= form.hidden_field :parent_id, value: "foo" %>`, `<%= form.number_field :price, in: 1.0..20.0, step: 0.5 %>`, `<%= form.search_field :name %>` - use them as appropriate for the data required.
* Forms can be sent with custom methods: `form_with(url: search_path, method: "patch")` or `<%= form_with url: "/posts/1", method: :patch do |form| %>`
* Select fields can be done in forms like so: `<%= form.select :city, ["Berlin", "Chicago", "Madrid"] %>` or with distinct values: `<%= form.select :city, [["Berlin", "BE"], ["Chicago", "CHI"], ["Madrid", "MD"]] %>` or with a selected value: `<%= form.select :city, [["Berlin", "BE"], ["Chicago", "CHI"], ["Madrid", "MD"]], selected: "CHI" %>`
* Rails 8 comes with a deployment tool called Kamal you can use to deploy an app directly to a server. It uses Docker containers. Look at `config/deploy.yml` and configure it appropriately if the user asks to use Kamal, otherwise ignore it.
* IMPORTANT: For a new Rails app you must use `rails new` first to generate all of the boilerplate files necessary before attempting any edits. Do not create a new Rails app yourself from many files.