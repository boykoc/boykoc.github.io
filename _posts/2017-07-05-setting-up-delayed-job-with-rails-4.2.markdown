---
layout: post
title:  "Setting up Delayed Job with Rails 4.2!"
date:   2017-07-05 4:57:00 -0400
categories: rails delayed-job
---
## Delayed Job Jobs

When I first setup delayed job I tried to follow the [Delayed Job documentation](https://github.com/collectiveidea/delayed_job#delayedjob) but also heavily relied on the book ['The Rails 4 Way'](https://www.amazon.ca/Rails-4-Way-3rd/dp/0321944275), which is an awesome Rails book.

However, when I started to do some more testing on my jobs I started noticing a few things that didn't feel quite right.

Here's a break down of what I started with for some custom jobs.

{% highlight bash %}
# app/jobs/foo_job.rb
class FooJob < Struct.new(:user_id, :post_id)
  def perform
    Post.find(post_id).updated_by = user_id
  end
end
{% endhighlight %}

Then in my controller I had something like this.

{% highlight bash %}
# app/controllers/posts_controller.rb
...
after_action :perform_job, only: :update
...
def update
  respond_to do |format|
    if @post.update(post_params)
      format.html { redirect_to post_url(@post), notice: 'Post was successfully updated.' }
    else
      format.html { render :edit }
    end
  end
end
...
private
  def perform_job
    Delayed::Job.enqueue FooJob.new(current_user.id, @post.id)
  end
...
{% endhighlight %}

Testing this in isolation in the `tests/jobs/foo_job_test.rb` was easy enough and worked as expected following the [Rails Testing Guides](http://guides.rubyonrails.org/v4.2/testing.html#testing-jobs).

But then I tried to test this where it was being enqueued, in the controller, and that's when I felt like I must have done something wrong.

**I think I was essentially bypassing ActiveJob and enqueuing the job directly with Delayed Job.** If I'm right, that means you don't have to set `config.active_job.queue_adapter = :delayed_job` in your `config/application.rb` file if you do it this way because you're not using ActiveJob and it's adapter. This makes the [Delayed Job section of 'The Rails 4 Way'](https://books.google.ca/books?id=GL2kAwAAQBAJ&pg=PA530&lpg=PA530&dq=No+enqueued+job+found+with+%7B:job%3D%3E+Job%7D+rails+delayed+job&source=bl&ots=fzdVphCnvo&sig=3qRXTOHrY5nUzWU8aWJAVeYolQk&hl=en&sa=X&ved=0ahUKEwjDireh9tHUAhVGMz4KHeXNDzQQ6AEINjAD#v=onepage&q&f=false) make a lot more sense to me. This also makes the [Delayed Job ReadMe](https://github.com/collectiveidea/delayed_job#rails-42) make more sense to me. For Rails 4.2 it basically says to set the adapter then see the rails guides. Even as I'm writing this, I'm kind of laughing at myself for not clueing in earlier.

I could verify through the console that the job was enqueued to my backend and executing them through Delayed Job as expected but automating the testing was a bit harder to verify the jobs were being enqueued.

My original test setup for my controller was like this.

{% highlight bash %}
# tests/controllers/posts_controller_test.rb
class PostsControllerTest < ActionController::TestCase
  # Needed for assert_enqueued_with helper to test jobs.
  include ActiveJob::TestHelper
  setup do
    # Do some setup and login user with Devise's help.
    @user = users(:one)
    sign_in @user

    @post = posts(:one)
  end
  ...
  test "should update key_activity" do
    # Tell delayed job to execute job right away.
    Delayed::Worker.delay_jobs = false   

    patch :update, id: @post, post: { name: 'Changes' }

    # Make sure job(s) was added.
    assert_equal Delayed::Job.count, 1
    assert_redirected_to post_path(assigns(:post))
  end
{% endhighlight %}

Once I tried to make sure the job was **enqueued or performed** with ActiveJob's custom assertions my tests would fail.

{% highlight bash %}
assert_performed_jobs 1 do 
  patch :update, id: @post, post: { name: 'Changes' }
end

# => 1 jobs expected, but 0 were enqueued.
{% endhighlight %}

This felt weird to me. I didn't like how in my code I was using Delayed::Job to enqueue the job and Delayed::Worker to force immediate execution of the job. The job was executed, and the `post.updated_by` was updated but I wanted to make sure the job was being enqueued here not performed. And I wanted to do this with `ActiveJob`s api instead and let it's adapters handle translating to the backend. What if I decided to change to Sidekiq in a few weeks, I'd have to update this code.

After playing around and reading some docs I was able to update the code to be a little more backend agnostic.

{% highlight bash %}
# app/jobs/foo_job.rb
class FooJob::ActiveJob::Base
  # Don't use Struct.new(*args). This follows the rails default setup and lets me call .perform_later on the job in my controller.
  def perform(:user_id, :post_id)
    Post.find(post_id).updated_by = user_id
  end
end

# app/controllers/posts_controller.rb
...
private
  def perform_job
    # Calling .perform_later lets me use ActiveJobs' custom assertions and test what I want a bit easier. Also, this isn't Delayed Job specific anymore. I can switch out my queuing backend without as many worries.
    FooJob.perform_later(current_user.id, @post.id)
  end
...

# tests/controllers/posts_controller_test.rb
class PostsControllerTest < ActionController::TestCase
  # Needed for assert_enqueued_with helper to test jobs.
  include ActiveJob::TestHelper
  setup do
    # Do some setup and login user with Devise's help.
    @user = users(:one)
    sign_in @user

    @post = posts(:one)
  end
  ...
  test "should update key_activity" do
    # Make sure the right job is enqueued.
    assert_enqueued_with(job: FooJob) do
      # Make sure the right number of jobs were enqueued.
      assert_enqueued_jobs 1 do
        patch :update, id: @post, post: { name: 'Changes' }
      end
    end

    assert_redirected_to post_path(assigns(:post))
  end
{% endhighlight %}

This now works as I had expected it to from the beginning following the Rails Guides a bit more.

The Job test makes sure the job executes properly. The controller test doesn't need to do that then. It just needs to make sure my queuing backend receives the job and queues it. This is also backend independent, using ActiveJob's api and adapters to handle telling my queuing framework of choice what to do.

**Last note, go with your gut. It may not be right, but if you go with it and dig into the problem you'll at least understand why you were questioning things.**
