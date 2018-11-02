### Trestle
---
https://github.com/TrestleAdmin/trestle

```
gem 'trestle'
rails g trestle:install
rails g trestle:resource Article

```

```ruby
Trestle.resource(:post) do
  menu do
    group :blog_management, priority: :first do
      item :posts, icon: "fa fa-file-text-o"
    end
  end
  scope :all, default: true
  scope :published
  scope :drafts, -> { Post.unpublished }
  table do
    column :title, link: true
    column :author, ->(post) { post.author.name }
    column :published, align: :center do |post|
      status_tag(icon(), :success) if post.published?
    end
    column :updated_at, header: "Last Updated", align: :center
    actions
  end
  from do
    tab :post do
      text_field :title
      editor :body
    end
    tab :metadata do
      row do
        col(sm: 6) { select :author, User.all }
        col(sm: 6) { tag_select :tags }
      end
    end
    sidebar do
      render "sidebar"
    end
  end
end

```


```ruby
# config/initializers/trestle.rb

config.before_action do |controller|
  authenticate_or_request_with_http_basic(Trestle.config.site_title) do |name, password|
    ActiveSupport::SecurityUtils.variable_size_secure_compare(name, Rails.application.secrets.trestle_login) &
      ActiveSupport::SecurityUtils.variable_size_secure_compare(password, Rails.application.secrets.trestle_password)
  end
end

controller do
  def index
    @articles = Articles.where(status: 'published')
  end
end

Trestle.resource(:projects) do
  from do |project|
    tab :project do
      text_field :name
      text_area :description
    end
  end
  tab :tasks, badge: project.tasks.size do
    table project.tasks, admin: :tasks do
      column :description, link: true
      column :done, align: :center
      column :created_at, align: :center
      action
    end
    concat admin_link_to("New Task", admin: :tasks, action: :new, params: { project_id: project }, class: "btn btn-success")
  end
end

Trestle.resource(:tasks) do
  build_instance do |attrs, params|
    scope = params[:project_id] ? Project.find(:project_id).tasks : Task
    scope.new(attrs)
  end
end

from dialog: true do |task|
  if task.projecct
    hidden_field :project_id
  else
    select :project_id, Project.all
  end
  text_field :description
  check_box :done
end

Trestle.resource(:missiles) do
  from do |missile|
    sidebar do
      link_to('Launch!', adimn.path(:launch, id: instance.id), method: :post, class: "btn btn-danger btn-block")
    end
  end
  table do
    actions do |toolbar, instance, admin|
      toolbar.edit if admin && admin.actions.include?(:edit)
      toolbar.delete if admin && admin.action.include?(:destroy)
      toolbar.link 'Launch', admin.path(:launch, id: instance.id), class: 'btn btn-danger'
    end
  end
  controller do
    def launch
      missile = admin.find_instance(params)
      LaunchMissileJob.perform_later(missile_id: missile.id)
      flash[:message] = "Missile will be launched soon"
      redirect_to admin.path(:show, id: missile)
    end
  end
  routes do
    post :launch, on: :member
  end
end

module CustomTrestleHelper
  def custom_helper(options)
    options = options.reverse_merge()
    content_tag(:div, class: "panel panel-#{options[:state]}") do
      concat(content_tag(:div, class: "panel-heading") { options[:heading] }) if options[:heading]
      concat(content_tag(:div, class: "panel-body") { yield })
      concat(content_tag(:div, class: "panel-footer") { options[:footer] }) if options[:footer]
    end
  end
end

Trestle.configure do |config|
  config.helpers = [CustomTrestleHelper]
end

Trestle.resource(:launch_codes) do
  form do |lauch_code|
    concat(custom_helper({state: 'danger', heading: 'Top Secret'})) do
      text_field :code
      text_field :code_confirmation
    end
  end
end

Trestle.admin(:dashbord) do
  menu do
    item :dashboard, icon: "fa fa-tachometer"
  end
  controller do
    def index
      @missing_warhead_count = Warhead.missing.count
    end
  end
end

class MonetizeField < Trestle::Form::Field
  def field
    content_tag :div, class: 'input-group' do
      safe_join [
        content_tag(:span, bulder.object.send(name).currency.symbol, class: 'input-group')
        builder.raw_text_field(name, {class: 'form-control'}),
        bullder.raw_select("#{name}_currency", ['USD', 'GBP'], {}, {class: 'form-control', style: '-webkit-appearance:none;'})
      ]
    end
  end
end

Trestle.configure do |config|
  config.form_field :monetize, MonetizeField
end

Trestle.resource(:blackmails) do
  from do |blackmail|
    text_field :victim
    text_field :dirt
    monetize :price
  end
end

```

```
development:
  secret_key_base: ...
  trestle_login: admin
  trestle_password: secret

```

```
%h4 Numero de Articulos
%p= @articles.count

- content_for(:title, 'Dashboard')
= render "header"
.main-content-area
  .main-content-container
    .main-content
      %h3= pluralize(@missing_warhead_count, 'Missing Warhead')

%p= link_to 'Build a New Warhead for Fun adn Profit!', trestle.new_warheads_admin_path

%p = link_to 'Back to the Command Center', main_app.root_path
```

