# decidim_berlin

Free Open-Source participatory democracy, citizen participation and open government for Berlin and its organizations.
This is the open-source repository for decidim_berlin, based on [Decidim](https://github.com/decidim/decidim).

## Working on decidim_berlin

### Setting up the application locally

You will need to do some steps before having the app working properly on your machine:

1. Open a Rails console in the server: `bundle exec rails console`
1. Create a System Admin user:
```ruby
Decidim::System::Admin.create!(email: <email>, password: <password>, password_confirmation: <password>)
```
1. Visit `<your app url>/system` and login with your system admin credentials
1. Create a new organization. Check the locales you want to use for that organization, and select a default locale.
1. Set the correct default host for the organization, otherwise the app will not work properly. Note that you need to include any subdomain you might be using.
1. Fill the rest of the form and submit it.

You're good to go!

Locally you can also use the provided seeds, but they seem to be a bit flaky. Just try `bin/rails db:seed` again a couple of times if they fail.

If you use the seeds you can:

- Browse the main interface at http://localhost:3000, and log in as: user@example.org | decidim123456
- Browse the admin interface at http://localhost:3000/admin, and log in as: admin@example.org | decidim123456
- Browse the system interface at http://localhost:3000/system, and log in as: system@example.org | decidim123456

⚠️ You should not do this on a public facing instance, of course. ⚠️

### Activating modules

Possible modules are listed at https://github.com/decidim/decidim. Find the one you need, click on the link and follow the instructions in your development environment. If everything checks out commit your changes and start the deployment as described below.

### Deployment

We use [Capistrano](https://github.com/capistrano/capistrano) for deployment.
You can run `bundle exec cap production deploy` to trigger the deployment process in the app folder.
For this to work you need an ssh key that is both registered with the server you're deploying to and have access to the repository on github.

Capistrano will then:
1. Ssh into the server
1. Checkout github's master branch in the `releases`-folder
1. Ensure all necessary gems are installed with `bundle install`
1. Run pending migrations
1. Compile the assets if necessary
1. Link files and directories that should be shared between releases (like the `database.yml`)
1. Link `/home/decidim/current` (which is targeted by the nginx-passenger) to the new release
1. Restart nginx-passenger to ensure the new release is used

If something goes wrong use `cap production deploy:rollback` to link `current` to the folder of the previous release. Capistrano keeps the last 5 releases.

## Setting up your own decidim instance

### Installation

At the time of this writing there are two installation manuals mentioned on the github repository for decidim:

1. The **official** installation manual at `https://github.com/decidim/decidim/blob/master/docs/manual-installation.md`

2. An **inofficial** installation manual at `https://github.com/Platoniq/decidim-install/blob/master/decidim-focal.md` with it's follow-up "basic configuration" (https://github.com/Platoniq/decidim-install/blob/master/basic-config.md)

The inofficial manual has **a lot more detail** and is **recommended** for people not experienced with Rails and for people that don't have a server set up yet on which they want to run decidim. The only outdated part of this guide is about using the gem `figaro`. Decidim currently uses Rails >= 5.2. which supports "Rails Credentials", which makes figaro obsolete. See `https://developer.epages.com/blog/coding/no-more-hassle-with-rails-secret-management`.
While the unofficial guide is very detailed and aims to get you going from nothing to a running decidim instance there is also no guide for setting up your **domain** and its DNS records. You can use [gandi](https://gandi.net) as a registrar and [their tutorial](https://docs.gandi.net/en/domain_names/common_operations/link_domain_to_website.html) for example.

The official manual is **heavily outdated** but is more focussed on decidim so it can be helpful if you have **specific requirements for your infrastructure**. If you're not entirely sure we heavily recommend not following the official installation instructions and follow the unofficial one instead.

One thing we found unintuitive are the different user types. So here's a little overview to reduce confusion. There are:

1. **System admins** (also called super admins)
These are admins that can manage your installation instance, organizations and should have technical knowledge. You'll probably won't need more than one or two of those. Possibly those that set up the instance. So you - the reader - probably ;-) You'll likely don't need to talk about this role with non-technical users.
The url to login as a system admin is: `https://<domain>/system/admins/sign_in`

1. **Organization admins** (which is a user with the admin flag, that belongs to an organization)
These are admins of one organization (You can think of an organization as a group of people that want to make decisions together, an organization always has a unique host). Organization admins don't need a technical background. They can set up modules/processes/conferences/... and are exposed to all of the core features of decidim. Organization admins need to understand what decidim is about, what it can do, and how to do it. The [admin-manual](https://decidim.org/pdf/Decidim_AdminManual_EN_0.10.pdf) is a good place to get familiar with decidim as an admin even though it's from 2018.
The url to log in as an organization admin is: `https://<organization-host>/users/sign_in`

1. **Organization users** which take part in the decision making process. The interface is supposed to be as self-explanatory as possible, but expect people to need support from admins.
They url to log in as organization admins: `https://<organization-host>/users/sign_in`.

Take note that the organization-host and the domain can be the same. The organization host is either the `<domain>` or `<subdomain>.<domain>`. That means that a common setup for manual checks would be to have three accounts:
1. `<domain>/system/admins/sign_in` to sign in as a system admin (and configure organizations).
1. `<domain>/users/sign_in` with an admin account to test as an organization admin. The documentation mentions `https://decidim.de/admin`, but the request is redirected to `<domain>/users/sign_in`.
1. `<domain>/users/sign_in` with a different user account to test as an organization user.

### Deployment

What's missing in both guides is a way to modify and deploy customizations. There is a part called "Going pro (optional):" in the unofficial guide, which sets up a git repository for your decidim instance. We heavily recommend following it, but it still assumes all changes are made on the (production) server directly.
If you want to have more safety while customizing (or updating) your decidim instance (including translations, see below), we recommend setting up [Capistrano](https://capistranorb.com/documentation/getting-started/installation) for deployment so you can **change code locally** before your production site is effected. Docker is an alternative as well, but the official guide marks the docker setup as "experimental", so if you prefer that you probably shouldn't need this guide anyway.

### Customizing Translations

Decidim translations are community-sourced with crowdin, but the state of german decidim translations is unintuitive at best and confusing at worst. Some similar but different concepts have the same name, other translations sound weird and even others seem to have been defined for a specific use case in mind that didn't fit our case and possibly won't fit yours. Especially because of the last issue and possibly long-term discussions associated with it we decided to customize translations for our use cases.

To do that you can create a translation file called `z9_custom.yml` in `config/locales`. `z9...` so that it's loaded last thus overwriting tranlsations from previous files. We included the translations used by decidim grouped by modules in our app's translations so it's easier to find the keys for translations you want to change. This way you can see a snippet of text on your decidim page, search for it in the translation files, look for the key, add the key to your `z9_custom.yml` and add the text you want to see instead. Feel free to copy the extracted translation files from https://github.com/decidim-de/decidim_berlin/tree/master/config/locales for your setup as well.

⚠️ With this custom translations setup it's important to ensure the translations are correct when updating the decidim version as translation keys might change during development of decidim. Another reason to ensure a proper setup that allows for local changes with an additional and separate deployment step. ⚠️

## General Information for people unfamiliar with ruby

### What are gems?

`Gems` are ruby's libraries - other people's code that we incorporate into our codebase. Ruby is an interpreted language so most of the gems are just the source code copied to a folder on the machine running the app. Ruby comes with a `gem` command (rubygems), that allows to install gems on your machine.

`Bundler` is the standard dependency manager for ruby. People have been thinking about incorporating `bundler` into rubygems, but that hasn't happened yet. So for now `bundler` is a gem that manages gem dependencies and builds a "bundle" of gems that work for your app. For standard rails app `bundler` is the only gem that is installed with rubygems directly. After `bundler` is installed rubygems will usually only be invoked only by `bundler` when you run `bundle install`. There are two files that `bundler` uses for two different purposes.
`Gemfile` is where we define the gems we want for our app and allows `bundler` to find fitting versions for them. `Gemfile.lock` is where bundler stores these fitting versions using the newest available gems at the time of running bundler and the limitiations defined in `Gemfile`.

In `Gemfile` we can set a maximum and minimum for the allowed versions of any gem, or define the version exactly. We can also set gems to be used only for a specific environment. For example is it unnecessary for our production instance to install any gems we use only for testing. `Bundler` will use `Gemfile` to find compatible versions for all the specified gems stop execution if it can't. An example: The newest version of gem a depends on gem b with exactly version 1.12, the newest version of gem c depends on gem b with version greater than 1.14. We have gem a and gem c in our gemfile. Bundler can't find a version of gem b that satisfies our requirements. So it stops and tells us. We can than set specific versions for gem a and gem c that have other requirements for versions to resolve the version conflict.

Using `Gemfile.lock` allows to avoid surprises coming from different gem versions on different machines. If we run bundler with an existing `Gemfile.lock` on two different machines it will fetch exactly the same versions defined by `Gemfile.lock`. So production and all our development machines will have exactly the same gem code. To have bundler look for the newest version of a gem again run `bundle update <gemname>`. This will adhere to the limitations in `Gemfile` and update `Gemfile.lock`.

The exception to gems that are just copied source code are gems that use C bindings or extensions. Ruby allows this to have a way to deal with very performance-critical tasks and some gems make use of that. That means that sometimes during gem installation you will see a message that "native extensions" are being "built". Installing these gems takes a lot longer due to this extra compilation step and may require C-libraries that you might need to install on your system. Homebrew is often recommended on MAC for doing this. On Linux the distro's package manager is usually the best choice. Installing `nokogiri` (ruby's most widely used XML-parser) is the most common case rubyist have to deal with C-libraries. Read the error messages very carefully. Often they state the missing libraries. If that doesn't help googling the error message will usually lead to good information.

Gems play a more important role for this app than in most other Rails apps, because all (maybe this will be "most" in the future) of our views and the database structure rely on the decidim gems.

See https://bundler.io and https://guides.rubygems.org/rubygems-basics for more information.
