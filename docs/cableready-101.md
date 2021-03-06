# CableReady 101

Now that we have installed the library, verified its dependencies and created an ActionCable Channel in our app, it's time to actually make the magic happen.

## Include the Broadcaster

You can send CableReady broadcasts from just about anywhere in your application: ActiveJobs, controller actions, ActiveRecord model callbacks or state machine transitions, rake tasks, pub/sub workers, webhooks, you name it. The only thing you have to do to use the `cable_ready` method is include `CableReady::Broadcaster` in the class you're working in.

We can add CableReady support to all of your controllers:

```ruby
class ApplicationController < ActionController::Base
  include CableReady::Broadcaster
end
```

The one notable exception is that you must **not** include `CableReady::Broadcaster` in your Reflex classes, as StimulusReflex makes special versions of the CableReady methods available.

{% hint style="info" %}
If you perform a CableReady broadcast during a controller action, it will send the broadcast immediately; before the action has completed, before the view template has been rendered and before the HTML has been sent to the client. This can lead to people becoming convinced \(incorrectly\) that the broadcast did not work.

If you need the user executing the controller action to see the broadcast, you should use an [ActiveJob](https://guides.rubyonrails.org/active_job_basics.html) that has been delayed for a few seconds using the [`set`](https://edgeguides.rubyonrails.org/active_job_basics.html#enqueue-the-job) method.
{% endhint %}

## Broadcasting operations

There are three distinct aspects of every CableReady invocation:

1. Channel stream identifier\(s\): decide who \(or what\) will receive operations
2. Operation queueing: define one or more operations to broadcast
3. Broadcast: broadcast all queued operations immediately

```ruby
cable_ready["visitors"].console_log(message: "I hope you brought a towel.")
cable_ready.broadcast # send queued console_log operation to all ExampleChannel subscribers
```

Following the example started in the [Setup](setup.md#setup), the `ExampleChannel` will send any operations broadcasted to `visitors` to all currently subscribed clients.

{% hint style="info" %}
ActionCable can deduce `ExampleChannel` from `visitors` because only one Channel can stream from a given identifier. It is conceptually similar to Rails request routing, except that resolution possibilities are defined across all of your Channel classes.
{% endhint %}

You can call `cable_ready` multiple times to add more operations to the queue. Since `cable_ready` is a singleton instance, you can continue to add operations to the queue even across multiple methods, or a recursive function.

You can use different operations together, and each operation can have completely different arguments. Without a call to `broadcast`, operations will accumulate for the specified Channel stream identifier.

```ruby
cable_ready["visitors"].console_log(message: "We have more salad than we can eat.")
cable_ready["visitors"].set_style(selector: "body", name: "color", value: "red")
cable_ready["visitors"].set_style(selector: "#foo", name: "color", value: "blue")
```

## Method chaining

When you call `cable_ready["visitors"]` you are presented with a `CableReady::Channels` object, which supports method chaining. This is a fancy way of saying that you can link up as many operations in sequence as you want, and they will ultimately be broadcast in the order that they were created.

```ruby
cable_ready["visitors"].console_log(message: "1").console_log(message: "2")
```

The `broadcast` method can conclude the chain, meaning that you can send a console message to everyone looking at your site with:

```ruby
cable_ready["visitors"].console_log(message: "Welcome!").broadcast
```

![](.gitbook/assets/hasselhoff.jpg)

