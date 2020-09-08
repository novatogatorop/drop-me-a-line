## Contact Form for Rails App
Let's assume that you already have a Rails application. 

1. Create model

   run `rails g model contact name email message:text`
   
2. Run `rails db:migrate`

3. Run `ga . && gc -m 'generate contact model`

4. Create controller

   run `rails g controller contacts new create`
   
5. Run `ga . && gc -m 'generate contacts controller`

6. Add bootstrap 

   run `yarn add bootstrap`

7. Open Gemfile and add:

   ```ruby
   gem 'simple_form'
   gem 'mail_form'
   gem 'invisible_captcha'
   ```
   
8. Run `bundle install`

9. Run the generator of simple_form and mail_form

   ```ruby
   rails g simple_form install --bootstrap
   rails g mail_form
   ```

10. Open `app/asset/stylesheets/application.css`

    rename the file into `application.scss`
    
    then add `@import "bootstrap/scss/bootstrap";`
 
11. Open `routes.rb` and add:

    ```ruby
    Rails.application.routes.draw do
      root to: "contacts#new"
      post "/contacts", to: "contacts#create"
    end
    ```
    
12. Remove or uncomment this line in `development.rb` and `production.rb`:

    `config.action_mailer.raise_delivery_errors = false`
    
13. Open `development.rb` and add:

    ```ruby
    config.action_mailer.default_url_options = { host: 'localhost:3000' }
    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = {
      address: 'smtp.gmail.com',
      port: 587,
      domain: 'gmail.com',
      authentication: 'plain',
      enable_starttls_auto: true,
        user_name: ENV['GMAIL_EMAIL'],
      password: ENV['GMAIL_PASSWORD']
    }
    ```
    
14. Open `production.rb` and add:

    ```ruby
    config.action_mailer_default_url_options = { host: 'https://gmail.com' }
    Rails.application.routes.default_url_options[:host] = 'https://gmail.com'
    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = {
      address: 'smtp.gmail.com',
      port: 587,
      domain: 'gmail.com',
      authentication: 'plain',
      enable_starttls_auto: true,
      user_name: ENV['GMAIL_EMAIL'],
      password: ENV['GMAIL_PASSWORD']
    }
    ```
15. Open Gemfile and add:

    `gem 'dotenv-rails', groups: [:development, :test]`
    
16. Run `bundle install`

17. Create env file to store your email and password, run:

    ```ruby
    touch .env
    echo '.env*' >> .gitignore
    ```
    
18. Run `ga . && gc -m 'create dotenv'`

19. Open `.env` and add:

    ```ruby
    GMAIL_EMAIL=your-email@gmail.com
    GMAIL_PASSWORD=abcdefghij
    ```
    
    * You should share `app password`, ~~not actual password~~. Follow [this](https://support.google.com/mail/answer/185833?hl=en-GB) to create one (for gmail).
    
 20. Setup model. Open `contact.rb` and add:
 
     ```ruby
     class Contact < MailForm::Base
       attribute :name, validate: true
       attribute :email, validate: /\A([\w\.%\+\-]+)@([\w\-]+\.)+([\w]{2,})\z/i
       attribute :message, validate: true
     
       # Declare the e-mail headers. It accepts anything the mail method
       # in ActionMailer accepts.
       def headers
         {
           subject: "Contact Form",
           to: "your-email@gmail.com",
           from: %("#{name}" <#{email}>)
         }
       end
     end
     ```
 
21. Run `ga. && gc -m 'setup contact model`

22. Setup controller. Open `contacts_controller.rb` and add:

    ```ruby
    class ContactsController < ApplicationController
      require 'mail_form'
      invisible_captcha only: [:create, :update], honeypot: :subtitle
    
      def new
        @contact = Contact.new
      end
    
      def create
        @contact = Contact.new(contact_params)
        if @contact.deliver
          flash.now[:error] = nil
        else
          flash.now[:error] = 'Cannot send message'
          render 'contacts/new'
        end
      end
    
      private
    
      def contact_params
        params.require(:contact).permit(:name, :email, :message, :subtitle)
      end
    end
    ```
    
24. Run `ga. && gc -m 'setup contacts controller`
    
25. Create partial form file, run:

    `touch app/views/contacts/_form.html.erb`
    
    then add:
    
    ```ruby
    <%= simple_form_for @contact, :html => {:class => 'form-horizontal' } do |f| %>
      <div class="form-inputs">
        <%= f.input :name, required: true %>
        <%= f.input :email, required: true %>
        <%= f.input :message, as: :text, required: true %>
        <%= f.invisible_captcha :subtitle %>
      </div>
      <div class="form-actions mt-5">
        <a href="https://64.media.tumblr.com/tumblr_m2oxi5nGWI1rr3l61o1_500.png" class="btn-c btn-nm mr-auto">Never Mind</a>
        <%= f.button :submit, "Send", class: "btn-c btn-send" %>
      </div>
    <% end %>
    ```
    
26. Open `app/views/contacts/new.html.erb` and add:

    ```ruby
    <div class="form-container">
      <div class="form-box">
        <h1 class="form-title">Drop me a line here!</h1>
        <%= render 'form', contact: @contact %>
      </div>
    </div>
    ```
    
27. Open `app/views/contacts/create.html.erb` and add:

    ```ruby
    <div class="form-container">
      <div class="form-box d-flex flex-column">
          <h1 class="form-title">Thanks for your message!!</h1>
          <p class="mb-5">I will get back to you soon.</p>
          <div class="mt-5"><%= link_to 'Drop Me A Line', root_path, class: "btn-c btn-send" %></div>
        </div>
    </div>
    ```
    
28. Run `ga. && gc -m 'setup contacts controller and view`
