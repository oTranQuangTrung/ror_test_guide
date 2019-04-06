Đa phần các lập trình viên thường chỉ lo viết code mà ít ai chú ý đến việc viết test cho những đoạn code mà mình vừa viết ra. Điều này có thể làm bạn cảm thấy không thành vấn đề khi bạn làm việc một mình, nhưng khi bạn làm việc với những lập trình viên khác, với những dự án lớn hơn, nhiều chức năng phức tạp thì chắc chắn việc không kiểm soát được code của mình sẽ gây ra nhiều vấn đề khiến bạn đau đầu.

Một khi bạn dành nhiều thời gian để viết test thật cẩn thận thì sau này khi bạn refactor code sẽ dễ dàng hơn rất nhiều, đồng thời giảm thiểu được bugs của ứng dụng do đó sẽ nâng cao được chất lượng sản phẩm và tiết kiệm được rất nhiều thời gian để test lại khi bạn nâng cấp hoặc bổ sung thêm chức năng cho ứng dụng về sau.

Trong bài viết này, mình sẽ giới thiệu về cách sử dụng Rspec để kiểm thử chức năng của ứng dụng web Ruby on Rails. Có thể việc viết test sẽ gặp nhiều khó khăn khi mới bắt đầu, tuy nhiên cũng có khá nhiều resources hỗ trợ việc viết test dễ dàng hơn. Hi vọng bài viết này sẽ có ích với các bạn mới bắt đầu.

Trong bài viết này chúng ta sẽ tìm hiểu những vấn đề sau

1. Cài đặt RSpec, Capybara, Shoulda-Matchers, Database Cleaner.

2. Cài đặt và tạo dữ liệu test bằng dữ liệu giả sử dụng Factory Girl và Faker.

3. Viết test cho Models (Model specs).

4. Viết test cho Controllers (Controller specs).

5. Viết test cho Features (Feature specs).

6. Viết test cho Routings (Routing specs).

Bắt đầu
Tạo 1 project mới

$ rails new rspec-examples
1. Cài đặt RSpec, Capybara, Shoulda-Matchers, Database Cleaner.
1.1 Cài đặt RSpec

Thêm gem rspec-rails vào group :development và :test trong Gemfile

group :development, :test do
  gem 'rspec-rails'
end
Chạy bundle install và khởi tạo rspec

$ bundle install & rails generate rspec:install
Cấu trúc thư mục của Rspec

├── spec
│   ├── controllers
│   ├── helpers
│   ├── models
│   ├── rails_helper.rb
│   └── spec_helper.rb
Rspec phân chia các thư mục theo chức năng tương ứng

spec/controllers: chứa các file test cho controller.

spec/models: chứa các file test cho model.

…

1.2 Cài đặt Shoulda Matcher và Database Cleaner

Thêm vào Gemfile gem shoulda-matchers và database_cleaner

group :test do
  gem 'shoulda-matchers', '~> 3.1'
  gem 'database_cleaner'
end
Chạy bundle install

Shoulda Matchers

shoulda matchers giúp cho việc viết test dễ dàng hơn, tiết kiệm được thời gian khi viết các test dài và phức tạp, code ngắn gọn, dễ đọc. Tiếp theo, chúng ta cần cung cấp cho shoulda matcher:

– Test framework mà chúng ta sử dụng (rspec, minitest,…)

– Những matcher mà chúng ta muốn sử dụng (active-record, model, controller)

Để shoulda-matchers làm việc với RSpec thêm config sau vào rails_helper.rb

Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    # Chỉ định test framework, ở đây chúng ta đang làm việc với Rspec:
    with.test_framework :rspec
    # with.test_framework :minitest
    # with.test_framework :minitest_4
    # with.test_framework :test_unit
    # Chỉ định các thư viện:
    # with.library :active_record
    # with.library :active_model
    # with.library :action_controller
    # Or, choose the following (which implies all of the above):
    with.library :rails
  end
end
Database Cleaner

database cleaner thực hiện cleaning database giữa mỗi test, tức là sau khi chạy xong một test nó sẽ rollback database lại trạng thái trước khi chạy test, đảm bảo database luôn nhất quán trong suốt quá trình thực hiện tất cả các test.

Để tích hợp database-cleaner với RSpec, đầu tiên ta cần thay đổi config trong file spec/rails_helper.rb

config.use_transactional_fixtures = true
Thành

config.use_transactional_fixtures = false
Có nghĩa là chúng ta muốn disable việc rspec-rails ngầm định chạy test trong một database transaction.

Tạo thư mục spec/support để chứa file config cho database-cleaner

spec/support/database_cleaner.rb

RSpec.configure do |config|

  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
end
1.3 Cài đặt Capybara

Capybara là một framework tự động, giúp chúng ta kiểm tra các ứng dụng web bằng cách mô phỏng cách mà một người dùng thực sự tương tác với ứng dụng của chúng ta.

Thêm gem capybara vào group development và test trong Gemfile

gem 'capybara'
Chạy bundle install

Mở file spec/spec_helper.rb và thêm require capybara

require 'capybara/rspec'
2. Cài đặt Factory Girl và Faker
Faker rất hữu ích trong việc tạo dữ liệu giả như tên, địa chỉ, số điện thoại,… một cách ngẫu nhiên để phục vụ cho việc test. Factory girl cho phép chúng ta tạo ra các object cần thiết cho việc test với các giá trị mặc định, kết hợp cùng với Faker chúng ta có thể tạo ra các object(factory) với gía trị ngẫu nhiên thay vì chỉ sử dụng giá trị mặc định.

Thêm gem factory_girl_rails và faker vào Gemfile

group :development, :test do
  gem 'factory_girl_rails'
  gem 'faker'
end
$ bundle install
Cách tạo một factory Tạo thư mục factories trong thư mục spec để chứa các file tương ứng với model. Ví dụ: spec/factories/user.rb

FactoryGirl.define do
  factory :user do
    name        { Faker::Name.name }
    email       { Faker::Internet.email }
    phone_number  { Faker::PhoneNumber.phone_number }
    address       { Faker::Address.street_address }
  end
end
Xem đầy đủ các thuộc tính mà Faker hỗ trợ tại đây.

Factory girl sẽ sử dụng các giá trị ngẫu nhiên được sinh ra từ faker để tạo ra các factories sử dụng trong quá trình test và chúng ta sẽ không cần phải quan tâm đến việc tạo hàng loạt dữ liệu bằng cách manual để test nữa.

Tạo hai model User và Article sau đó thực hiện migrate database

$ rails g model User name:string email:string password:string address:text
$ rails g model Article title:string content:text user_id:integer published_at:datetime
$ rake db:migrate && rake db:test:prepare
Bây giờ chúng ta đã có thể chạy test thử

$ rake spec
hoặc

$ rspec spec
Mặc định khi chạy lệnh trên RSpec sẽ thực thi toàn bộ file trong thư mục spec

Ta cũng có thể chạy từng phần riêng biệt

Chỉ chạy specs cho model

$ rspec spec/models
Chỉ chạy spec cho UsersController

$ rspec spec/controllers/users_controller_spec.rb
3. Model Specs
Sau đây là một số ví dụ đơn giản về cách viết test cho model.

3.1 Kiểm tra database schema

spec/models/user_spec.rb

require 'rails_helper'

RSpec.describe User, type: :model do

  describe "db schema" do
    context "columns" do
      it { should have_db_column(:email).of_type(:string) }
      it { should have_db_column(:name).of_type(:string) }
      it { should have_db_column(:password).of_type(:string) }
      it { should have_db_column(:address).of_type(:text) }
    end
  end
end
Chạy lại spec để xem kết quả

$ rspec spec/models
Finished in 0.01508 seconds (files took 6.61 seconds to load)
4 examples, 0 failures
3.2 Kiểm tra model validations

Thêm vào file spec/models/user_spec.rb phần kiểm tra các validations

require 'rails_helper'

RSpec.describe User, type: :model do
  describe "validations" do
    it { should validate_presence_of(:name) }
    it { should validate_presence_of(:email) }
    it { should validate_presence_of(:password) }
    it { should validate_length_of(:password) }
  end
end
Chạy lại spec

$ rspec spec/models

Finished in 0.17217 seconds (files took 1.85 seconds to load)
5 examples, 4 failures

Failed examples:

rspec ./spec/models/user_spec.rb:15 # User validation should validate that :name cannot be empty/falsy
rspec ./spec/models/user_spec.rb:16 # User validation should validate that :email cannot be empty/falsy
rspec ./spec/models/user_spec.rb:17 # User validation should validate that :password cannot be empty/falsy
rspec ./spec/models/user_spec.rb:19 # User validation should have a secure password
Kết quả trả về failed vì chúng ta chưa thêm bất cứ validation nào vào model User.

Thêm validations vào app/models/user.rb

class User < ActiveRecord::Base

  validates :name, presence: true
  validates :email, presence: true
  validates :password, presence: true, length: { in: 6..15 }
end
Chạy lại spec và sẽ không còn thông báo lỗi.

3.3 Kiểm tra associations

Bây giờ, chúng ta sẽ thêm association vào model User và Article sau đó tiến hành viết spec cho association này

class User < ActiveRecord::Base
  has_many :articles
end

class Article < ActiveRecord::Base
  belongs_to :user
end
spec/models/user_spec.rb

# Test associations
require 'rails_helper'

RSpec.describe User, type: :model do
  describe "associations" do
    it { should has_many(:articles) }
  end
end
spec/models/article_spec.rb

require 'rails_helper'

RSpec.describe Article, type: :model do
  describe "associations" do
    it { should belong_to(:user) }
  end
end
3.4 Kiểm tra instance methods

Thêm một instance method vào model Article để kiểm tra một article đã được đăng hay chưa

app/models/article.rb

def published?
  self.published_at.present?
end
Viết test cho method này như sau spec/models/article_spec.rb

describe "instance methods" do
  it { should respond_to(:published?) }
end
4. Controller Specs
Tạo 1 controller

$ rails g controller Articles
Các file spec của controller sẽ được đặt trong thư mục spec/controllers Mở file spec/controllers/articles_controller_spec.rb và bắt đầu viết test cho action new

require 'rails_helper'

RSpec.describe ArticlesController, type: :controller do
  describe "GET new" do
    it "assigns a blank article to the view" do
      get :new
      expect(assigns(:article)).to be_a_new(Article)
    end
  end
end
Chạy spec

$ rspec spec/controllers
Chương trình sẽ báo lỗi No route matches {:action=>"new", :controller=>"articles"} bởi vì chưa có route tương ứng với controller.

Thêm vào file config/routes.rb

resources :articles
Chạy lại spec và chúng ta sẽ vẫn nhận thông báo lỗi vì chưa định nghĩa action new trong controller

class ArticlesController < ApplicationController
  def new
    @article = Article.new
  end
end
Định nghĩa action new trong ArticleController và thêm file new.html.erb vào thư mục view của articles. Chạy lại spec một lần nữa và không có lỗi

1 example, 0 failures
Tương tự, chúng ta sẽ tiếp tục viết spec cho các actions show and create

app/controllers/articles_controller.rb

class ArticlesController < ApplicationController

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(article_params)

    respond_to do |format|
      if @article.save
        format.html { redirect_to @article }
        format.json { render :show, status: :created, location: @article }
      else
        format.html { render :new }
        format.json { render json: @article.errors }
      end
    end
  end

  def show
    @article = Article.find(params[:id])
  end

  private

  def article_params
    params.require(:article).permit(:title, :content, :user_id)
  end
end
5. Feature Specs
Bây giờ, chúng ta thử viết test cho chức năng tạo mới article, user sẽ nhập vào title và content, sau đó nhấn nút “Create” để tạo article mới, khi thành công sẽ chuyển sang màn hình show với thông điệp “Article was successfully created!”.

views/articles/new.html.erb

<%= form_for @article, url: articles_path do |f| %>
  <p>Title</p>
  <%= f.text_field :title %>
  <p>Content</p>
  <%= f.text_area :content, size: "60x8" %>
  <p><%= f.submit "Create" %></p>
<% end %>
views/articles/sh.html.erb

<h3>Article was successfully created!</h3>
Tạo thư mục features/articles trong thư mực spec để chứa spec cho các features của article. Viết test cho chức năng create article trong file spec/features/articles/create.rb

require 'rails_helper'

RSpec.feature "Article", type: :feature do
  describe "Create a new article" do
    visit "/articles/new"

    fill_in "article_title", with: "Create an article"
    fill_in "article_content", with: "How to create a great article?"

    click_button "Create"

    expect(page).to have_text("Article was successfully created!")
  end
end
6. Routing Specs
Tạo thư mục spec/routing sau đó tạo file articles_routing_spec.rbđể viết test cho articles routing

articles_routing_spec.rb

require "rails_helper"

RSpec.describe "routes for Widgets", type: :routing do
  it "routes /articles  to the articles controller with action index" do
    expect(get("/articles")).
      to route_to("articles#index")
    #hoặc to route_to(controller: "articles", action: "index")
  end
end
Ví dụ chúng ta có thêm 1 section admin thì cách viết test cho namespace :admin như sau

it "routes /admin/articles to the admin/articles controller" do
    expect(get("/admin/articles")).
      to route_to("admin/articles#index")
  end
P/S Trên đây chỉ là một phần giới thiệu đơn giản về RSpec, cách cài đặt môi trường và các gem cần thiết để hỗ trợ việc viết test, hi vọng sẽ giúp các bạn mới bắt đầu sẽ có được cái nhìn tổng quan. Để có thể viết test cho các trường hợp phức tạp hơn, các bạn có thể tham khảo thêm

