---
layout: post
title:  "Convert API Response Into a Model in Rails"
tags: rails
---

Let's get down straight to the business.

Let's take an example of showing the invoices to the user from Stripe.

First, let's write **what we would like to have** without worrying about the implementation.  
<br/>

I like **short and clean** controllers. Something like the following would do.

`app/controllers/invoices_controller.rb`

```ruby
class InvoicesController < ApplicationController
  def index
    @invoices = Invoice.find_all_by_user(current_user)
  end

  def show
    @invoice = Invoice.new(params[:id])
    render
  end
end
```
<br />
  
Below is what I would like in my view: `app/views/invoices/_invoice.html.erb`
```erb
<% unless (invoice.total == 0) %>
  <tr>
    <td><%= link_to invoice.number, invoice_path(invoice.id) %></td>
    <td><%= invoice.date %></td>
    <td><%= number_to_currency(invoice.total, negative_format: "(%u%n)") %></td>
    <td><%= invoice.period_start %> to <%= invoice.period_end %></td>
    <td><%= invoice.paid? ? 'Paid' : 'Unpaid' %></td>
  </tr>
<% end %>
```
<br />

I have experienced that writing the interface first, as we did above, gives me a lot of clarity during implementation.
Now, let's start implementing it in a model.

`app/models/invoice.rb`
```ruby
class Invoice

  attr_reader :stripe_invoice

  def self.find_all_by_user(user)
    if user.present?
      stripe_invoices_for_user(user).map do |invoice|
        new(invoice)
      end
    else
      []
    end
  end

  def initialize(invoice_id_or_object)
    if invoice_id_or_object.is_a? String
      @stripe_invoice = retrieve(invoice_id_or_object)
    else
      @stripe_invoice = invoice_id_or_object
    end
  end

  def to_partial_path
    "invoices/#{self.class.name.underscore}"
  end

  def id
    stripe_invoice.id
  end

  def number
    stripe_invoice.number
  end

  def total
    cents_to_dollars(stripe_invoice.total)
  end

  def date
    convert_stripe_time(stripe_invoice.date)
  end

  def paid?
    stripe_invoice.paid
  end

  def subscription
    stripe_invoice.subscription
  end

  def period_start
    convert_stripe_time(stripe_invoice.period_start)
  end

  def period_end
    convert_stripe_time(stripe_invoice.period_end)
  end

  def user
    @user ||= User.find_by(stripe_customer_id: stripe_invoice.customer)
  end

  def balance
    if paid?
      0.00
    else
      amount_due
    end
  end

  def amount_due
    cents_to_dollars(stripe_invoice.amount_due)
  end

  def subtotal
    cents_to_dollars(stripe_invoice.subtotal)
  end

  def amount_paid
    if paid?
      amount_due
    else
      0.00
    end
  end

  def plan
    stripe_invoice.lines.data[0].plan.name
  end

  def plan_amount
    cents_to_dollars stripe_invoice.lines.data[0].plan.amount
  end

  def pay
    stripe_invoice.pay
  end

  def self.pay_if_pending(user)
    invoices = find_all_by_user(user)
    unless invoices.empty? || invoices.first.paid?
      invoices.first.pay
    end
  end

  def self.upcoming(user)
    new(Stripe::Invoice.upcoming(customer: user.stripe_customer_id) || nil )
  end

  private

  def self.stripe_invoices_for_user(user)
    Stripe::Invoice.all(customer: user.stripe_customer_id).data
  end

  def retrieve(invoice_id)
    Stripe::Invoice.retrieve(invoice_id)
  end

  def convert_stripe_time(time)
    Time.zone.at(time).strftime('%D')
  end

  def cents_to_dollars(amount)
    amount / 100.0
  end
end
```

Here, the [ Stripe ](https://github.com/stripe/stripe-ruby){:target="blank"} gem takes exposes many methods on the response. But we can take any vanilla JSON response any do the same.

Many years ago, I learned this pattern form [Upcase](https://thoughtbot.com/upcase){:target="blank"}. Since then I am always parsing API responses like this. Upcase is free now and definitely worth a look.

Download the code as [gist](https://gist.github.com/tannakartikey/d9f2b7cb8a473319f65fa325790c52dd){:target="blank"}.
