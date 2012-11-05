---
layout: cookbook
title: Class map on the fly
---

With default configuration, capifony will reinstall all your vendors for each deploy.
If you feel that is inefficient, you can manage to have your vendors just updated, not
reinstalled.

Add these lines to your `deploy.rb` file:

{% highlight ruby %}
set :shared_children, [app_path + "/logs", web_path + "/uploads"]

# Symfony2 2.0.x
before "symfony:vendors:install", "symfony:copy_vendors"

# Symfony2 2.1
before 'symfony:composer:update', 'symfony:copy_vendors'

namespace :symfony do
  desc "Copy vendors from previous release"
  task :copy_vendors, :except => { :no_release => true } do
    if Capistrano::CLI.ui.agree("Do you want to copy last release vendor dir then do composer install ?: (y/N)")
      capifony_pretty_print "--> Copying vendors from previous release"

      run "cp -a #{previous_release}/vendor #{latest_release}/"
      capifony_puts_ok
    end
  end
end
{% endhighlight %}

Now, each time you deploy, your vendors will be copied from your previous release,
and then vendors will be updated, not reinstalled from scratch. This means that, in
case you did not change your vendors, deploy will be faster.

Please notice that is not the same as putting your vendors folder in shared.
A shared vendor folder does not allow for real independent releases, and also causes
a downtime on your current release while deploying.

If you don't need a prompt asking you if you want to copy vendors, you can use
the following code:

{% highlight ruby %}
after 'symfony:composer:install', 'composer:class_map_generation'
after 'symfony:composer:update', 'composer:class_map_generation'

namespace :composer do
  task :class_map_generation do
    capifony_pretty_print "--> Generating class map with composer"

    run "#{try_sudo} sh -c 'cd #{latest_release} && #{composer_bin} dump-autoload --optimize'"
    capifony_puts_ok
  end
end
{% endhighlight %}
