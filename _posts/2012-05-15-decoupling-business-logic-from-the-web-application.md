---
layout: post
title: Decoupling business logic from the web application
---

<div class="alert">
This blog post is a continuation of a series of blog posts. Please see the parent blog post "<a href="/2012/05/15/dci-and-decoupling-business-logic-from-ruby-on-rails.html">DCI and decoupling business logic from ruby on rails</a>" if you haven't read the previous entries.
</div>

Uncle Bob's talk inspired me to try to isolate away my business rules into a separate gem. Here was the concept I came up with (more or less):

#### SApp Gem Directory Structure
<pre>
... other gem crap ...
|-- lib
|   `-- sapp
|       `-- authenticate_user.rb
|       `-- display_receipt.rb
|       `-- finish_account.rb
|       `-- finish_transaction.rb
|       ... more use cases/interactors ...
|   `-- sapp.rb
|-- test
|   `-- authenticate_user_test.rb
|   `-- display_receipt_test.rb
|   `-- finish_account_test.rb
|   `-- finish_transaction_test.rb
|   ... more use cases ... 
... other gem crap ...
</pre>

#### lib/sapp/authenticate_user.rb
{% highlight ruby %}
module SApp
  module AuthenticateUser
    class << self
      attr_reader :action
      def authenticate(account, username, password)
        @account, @username, @password = account, username, password

        find_account

        if account_found? && valid_credentials?
          view_success
        else
          view_login_error
        end
      end

      # some methods like this:
      def account_found?
      end

      def user_found?
      end

      def valid_credentials?
      end

      ### etc ###

      private

      def find_user
        @user = account.find_account(@username)
      end
    end
  end
end
{% endhighlight %}

Now you can imagine that my code could be interacted with from a rails controller or a sinatra application or a desktop application with this API:

#### Rails Example
{% highlight ruby %}
class SessionController < ApplicationController
  before_filter :find_account

  def create
    authenticate = SApp::AuthenticateUser.authenticate(account, params[:username], params[:password])
    action = authenticate.action
    if action != :login_error
      redirect_to "/#{action}"
    else
      flash[:error] = "Login failed"
      redirect_to :back
    end
  end

  protected
  # @account_subdomain extracted from the subdomain elsewhere. That logic should remain in the web application
  def find_account
    @account = SApp::FindAccount.find(@account_subdomain)
  end
end
{% endhighlight %}

#### Sinatra Example
{% highlight ruby %}
before '/authenticate' do
  @account = SApp::FindAccount.find(@account_subdomain)
end

post '/authenticate' do
  authenticate = SApp::AuthenticateUser.authenticate(@account, params[:username], params[:password])
  action = authenticate.action
  if action != :login_error
    redirect to("/#{action}")
  else
    flash[:error] = "Login failed"
    redirect to('/')
  end
end
{% endhighlight %}

#### test/authenticate_user_test.rb
{% highlight ruby %}
require 'test_helper'

describe SApp::AuthenticateUser do
  describe "authenticate" do
    let(:account) { FactoryGirl.create(:account) }
    let(:user) { FactoryGirl.create(:username, :account_id => account.id) }

    describe "user_found?" do
      before do
        user
      end

      it "is true when the username matches a user on the account" do
        authenticate = SApp::AuthenticateAccount.authenticate(account, "testusername", "testpassword")
        authenticate.user_found?.must_equal true
      end

      it "is false when the username cannot find a user on the account" do
        authenticate = SApp::AuthenticateAccount.authenticate(account, "notfoundusername", "testpassword")
        authenticate.user_found?.must_equal false
      end
    end
  end
  ### ... more tests ... ###
end
{% endhighlight %}

It is important to note that I intentionally did not pass the entire params hash to the interactor. The interactor should explicitly specify the required parameters it needs to complete the use case. As a result of this new design, I don't need to write acceptance tests that run through the web application because my core business layer has been decoupled and the API that my web application interacts with is dumb and dead simple. 

I intentionally left out the Entities from my examples and you might also notice that I am making calls to the database from my use cases. I will explain my reasoning in the next chapter on my quest for a better architecture... which is coming soon.

Here is a high level diagram of the idea:

<img src="/img/interactors.png">

As always, feedback is appreciated!