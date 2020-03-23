# decidim_berlin

Free Open-Source participatory democracy, citizen participation and open government for Berlin and its organizations.
This is the open-source repository for decidim_berlin, based on [Decidim](https://github.com/decidim/decidim).

## Setting up the application

You will need to do some steps before having the app working properly on your machine:

1. Open a Rails console in the server: `bundle exec rails console`
1. Create a System Admin user:
```ruby
user = Decidim::System::Admin.new(email: <email>, password: <password>, password_confirmation: <password>)
user.save!
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

You should not do this on a public facing instance, of course.

## Gems

`Gems` are ruby's libraries - other people's code that we incorporate into our codebase. Ruby is an interpreted language so most of the gems are just the source code copied to a folder on the machine running the app. Ruby comes with a `gem` command (rubygems), that allows to install gems on your machine.

`Bundler` is the standard dependency manager for ruby. People have been thinking about incorporating `bundler` into rubygems, but that hasn't happened yet. So for now `bundler` is a gem that manages gem dependencies and builds a "bundle" of gems that work for your app. For standard rails app `bundler` is the only gem that is installed with rubygems directly. After `bundler` is installed rubygems will usually only be invoked only by `bundler` when you run `bundle install`. There are two files that `bundler` uses for two different purposes.
`Gemfile` is where we define the gems we want for our app and allows `bundler` to find fitting versions for them. `Gemfile.lock` is where bundler stores these fitting versions using the newest available gems at the time of running bundler and the limitiations defined in `Gemfile`.

In `Gemfile` we can set a maximum and minimum for the allowed versions of any gem, or define the version exactly. We can also set gems to be used only for a specific environment. For example is it unnecessary for our production instance to install any gems we use only for testing. `Bundler` will use `Gemfile` to find compatible versions for all the specified gems stop execution if it can't. An example: The newest version of gem a depends on gem b with exactly version 1.12, the newest version of gem c depends on gem b with version greater than 1.14. We have gem a and gem c in our gemfile. Bundler can't find a version of gem b that satisfies our requirements. So it stops and tells us. We can than set specific versions for gem a and gem c that have other requirements for versions to resolve the version conflict.

Using `Gemfile.lock` allows to avoid surprises coming from different gem versions on different machines. If we run bundler with an existing `Gemfile.lock` on two different machines it will fetch exactly the same versions defined by `Gemfile.lock`. So production and all our development machines will have exactly the same gem code. To have bundler look for the newest version of a gem again run `bundle update <gemname>`. This will adhere to the limitations in `Gemfile` and update `Gemfile.lock`.

The exception to gems that are just copied source code are gems that use C bindings or extensions. Ruby allows this to have a way to deal with very performance-critical tasks and some gems make use of that. That means that sometimes during gem installation you will see a message that "native extensions" are being "built". Installing these gems takes a lot longer due to this extra compilation step and may require C-libraries that you might need to install on your system. Homebrew is often recommended on MAC for doing this. On Linux the distro's package manager is usually the best choice. Installing `nokogiri` (ruby's most widely used XML-parser) is the most common case rubyist have to deal with C-libraries. Read the error messages very carefully. Often they state the missing libraries. If that doesn't help googling the error message will usually lead to good information.

Gems play a more important role for this app than in most other Rails apps, because all (maybe this will be "most" in the future) of our views and the database structure rely on the decidim gems.

See https://bundler.io and https://guides.rubygems.org/rubygems-basics for more information.

## Deployment

We use Capistrano (https://github.com/capistrano/capistrano) for deployment.
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
