# Implementing Email Functionality in Ruby on Rails

Action Mailer is the built-in email framework for Ruby on Rails. To use it, you need to configure it in your application.

First, you need to set the default host for your application. This is important because Action Mailer needs to know where to send the emails from. You can set the default host in your config/application.rb file:

```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
Next, you need to configure the mail delivery method. By default, Action Mailer will use the SMTP delivery method, but you can also use other methods like sendmail or letter_opener. To configure the SMTP delivery method, you can add the following code to your config/environments/development.rb file:

```ruby
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:              'smtp.gmail.com',
  port:                 587,
  user_name:            'your-email@example.com',
  password:             'your-email-password',
  authentication:       'plain',
  enable_starttls_auto: true
}
```
### Sending Emails

With Action Mailer set up, you can use it to send emails in your Ruby on Rails application. Here's an example of sending a simple email:

```ruby
class UserMailer < ApplicationMailer
  def welcome_email(user)
    @user = user
    mail(to: @user.email, subject: 'Welcome to My Awesome Site')
  end
end
```

In this example, we define a UserMailer class that inherits from ApplicationMailer. We then define a welcome_email method that takes a user parameter. Inside the method, we set an instance variable @user and then call the mail method to send the email to the user's email address.

### Conclusion
Action Mailer provides an easy way to send emails in Ruby on Rails applications. With Action Mailer, you can send simple text emails or HTML emails with ease. By following the examples above, you can add email functionality to your Ruby on Rails application in no time.