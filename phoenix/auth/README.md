# Elixir/Phoenix with auth and sqlite and bootstrap and deploy


## Create application with sqlite

Updated to phoenix 1.6, it simplyfies a lot  nice!


[phoenix_auth](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Auth.html)


Create a phx application

    mix phx.new jle --database sqlite3

No needed --live anymore, and new option to sqlite3  :-)

Done!

Follow instructions...

```
We are almost there! The following steps are missing:

    $ cd jle

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server

```


## Add authentication

Easy now...

```
    mix phx.gen.auth Accounts User users
```

And follow instructions again

```
Please re-fetch your dependencies with the following command:

    $ mix deps.get

Remember to update your repository by running migrations:

    $ mix ecto.migrate

Once you are ready, visit "/users/register"
to create your account and then access to "/dev/mailbox" to
see the account confirmation email.
```

Add

```
    mix ecto.migrate
```

## Add bulma

In root.html.heex

```
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.3/css/bulma.min.css">
```


## Add bootstrap

[bootstrap phoenix](https://fullstackphoenix.com/tutorials/bootstrap-5-and-phoenix-liveview)

```elixir
# mix.exs
defp deps do
  [
    ...
    {:dart_sass, "~> 0.2", runtime: Mix.env() == :dev}
  ]
end
  
defp aliases do
  [
    ...
    "assets.deploy": [
      "esbuild default --minify",
      "sass default --no-source-map --style=compressed",
      "phx.digest"
    ]
  ]
end
```


```elixir
# config/config.exs
config :dart_sass,
  version: "1.42.1",
  default: [
    args: ~w(css/app.scss ../priv/static/assets/app.css),
    cd: Path.expand("../assets", __DIR__)
  ]
```

Inside the watchers list in the Endpoint configuration, add the following

```elixir
# config/dev.exs
    sass: {
      DartSass,
      :install_and_run,
      [:default, ~w(--embed-source-map --source-map-urls=absolute --watch)]
    }
```

```
mix sass.install
```

## Old version...



Follow instrucionts...

    We are almost there! The following steps are missing:

        $ cd jle
        $ cd assets && npm install && node node_modules/webpack/bin/webpack.js --mode development

    Then configure your database in config/dev.exs and run:

        $ mix ecto.create

    Start your Phoenix app with:

        $ mix phx.server

    You can also run your app inside IEx (Interactive Elixir) as:

        $ iex -S mix phx.server

Therefore

    $ cd jle
    $ $ cd assets && npm install && node node_modules/webpack/bin/webpack.js --mode development

I had to run npm with nix

node.nix file...

    with import <nixpkgs> {}; {
    teams = stdenvNoCC.mkDerivation {
        name = "node";
        buildInputs = [
            nodejs
            # libev
            #...
        ];
    };
    }

run

    $ nix-shell node.nix

Now to configure sqlite3

    Then configure your database in config/dev.exs


mix.exs

```elixir
    defp deps do
        {:ecto_sqlite3, "~> 0.5.5"}
    end
```

Remove it not necessary...

```elixir
      {:postgrex, ">= 0.0.0"},
```


Now got to repo.ex and...

```elixir
    defmodule Jle.Repo do
        # use Ecto.Repo,
        #   otp_app: :jle,
        #   adapter: Ecto.Adapters.Postgres
        use Ecto.Repo, otp_app: :jle, adapter: Ecto.Adapters.SQLite3
    end
```

Now configure phx

develop/testing... 

dev.exs

```elixir

    config :jle, JLE.Repo,
        adapter: Ecto.Adapters.SQLite3,
        database: "dev_database.db"
```

Comment the generated one

    # Configure your database
    # config :jle, Jle.Repo,
    #   username: "postgres",
    #   password: "postgres",
    #   database: "jle_dev",
    #   hostname: "localhost",
    #   show_sensitive_data_on_connection_error: true,
    #   pool_size: 10
        
        
Production...
???
prod.exs

    config :jle, JLE.Repo,
        database: "database.db"

It's time to configure the gen_authauth

```elixir
  defp deps do
    [
      {:phoenix, "~> 1.5.1"},
      {:phoenix_ecto, "~> 4.1"},
      {:ecto_sql, "~> 3.4"},
      {:postgrex, ">= 0.0.0"},
      {:phoenix_html, "~> 2.11"},
      {:phoenix_live_reload, "~> 1.2", only: :dev},
      {:phoenix_live_dashboard, "~> 0.2.0"},
      {:telemetry_metrics, "~> 0.4"},
      {:telemetry_poller, "~> 0.4"},
      {:gettext, "~> 0.11"},
      {:jason, "~> 1.0"},
      {:plug_cowboy, "~> 2.0"},
      {:ecto_sqlite3, "~> 0.5.5"},
      {:phx_gen_auth, "~> 0.7", only: [:dev], runtime: false}
```

Install and compile the dependencies

    $ mix do deps.get, deps.compile

It will require a time.
It will compile sqlite3.c library  ;-)


Running the generator

    $ mix phx.gen.auth Accounts User users
    
    $ mix ecto.create
    $ mix deps.get
    $ mix ecto.migrate

    $ mix test
    
    $ mix phx.server
    
BUG!!!

```elixir
    scope "/", JleWeb do
        pipe_through [:browser]

        delete "/users/log_out", UserSessionController, :delete
        get "/users/log_out", UserSessionController, :delete
```


If you put the database on config.exs the 

        export DATABASE_URL=ecto://./database.sqlite3

Probably is not relevant



Release

[release](https://hexdocs.pm/phoenix/releases.html)

```
    $ mix phx.gen.secret
    REALLY_LONG_SECRET
    $ export SECRET_KEY_BASE=REALLY_LONG_SECRET
    $ export DATABASE_URL=ecto://USER:PASS@HOST/database

    # Initial setup
    $ mix deps.get --only prod
    $ MIX_ENV=prod mix compile

    # Install / update  JavaScript dependencies
    $ npm install --prefix ./assets

    # Compile assets
    $ npm run deploy --prefix ./assets
    $ mix phx.digest

    $ MIX_ENV=prod mix release
    Generated my_app app
    * assembling my_app-0.1.0 on MIX_ENV=prod
    * skipping runtime configuration (config/releases.exs not found)

    Release created at _build/prod/rel/my_app!

        # To start your system
        _build/prod/rel/my_app/bin/my_app start

    ...

    config :my_app, MyApp.Endpoint, server: true

    $ MIX_ENV=prod mix release
```


## Getting the user for permisions

On live...

```elixir
  def mount(_params, _session, socket) do
    {an, _} = socket.private.assign_new
    user = an.current_user.email

    {:ok,
     assign(socket,
       debug: "#{inspect(socket |> Map.keys(), pretty: true)}",
       debug2: "#{inspect(socket.private.assign_new, pretty: true)}",
       debug3: "#{inspect(user, pretty: true)}"
     )}
  end
```