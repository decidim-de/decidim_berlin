# decidim_berlin

Free Open-Source participatory democracy, citizen participation and open government for cities and organizations

This is the open-source repository for decidim_berlin, based on [Decidim](https://github.com/decidim/decidim).

## Setting up the application

You will need to do some steps before having the app working properly once you've deployed it:

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

## Deployment

We use capistrano for deployment.
You can run `bundle exec cap production deploy` to trigger the deployment process.
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
