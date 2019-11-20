# Project Description

## 1. Creating the Project

```
$ mix phx.new rumbl
$ cd rumbl
$ mix ecto.create
$ mix phx.server
```
to make sure it’s working, and point your browser to
http://localhost:4000/

## 2. A Simple Home Page

Make your lib/rumbl_web/templates/page/index.html.eex look like this:

```html
<section class="phx-hero">
  <h1><%= gettext "Welcome to %{name}!", name: "Rumbl.io" %></h1>
  <p>Rumbl out loud.</p>
</section>
```
Now we have a home page started. Notice that your browser has already changed as shown in the figure below.

![alt text]( assets/images/img-1.png "home index page")

## 3. Working with Contexts
We’ll need a data structure for representing
a user,
so create a new file in lib/rumbl/accounts/user.ex and key this in:

```elixir
defmodule Rumbl.Accounts.User do
  defstruct [:id, :name, :username]
end
```

### Elixir Structs
A
struct is Elixir’s main abstraction for working with structured data.

elixir maps:
```elixir
iex> alias Rumbl.Accounts.User

iex> user = %{usernmae: "jose"}
%{usernmae: "jose"}

iex> user.username
** (KeyError) key :username not found in: %{usernmae: "jose"}
```

elixir structs:
```elixir
iex> jose = %User{name: "Jose Valim"}
%Rumbl.Accounts.User{id: nil, name: "Jose Valim", username: nil}

iex> jose.name
"Jose Valim"
```
Now, if we misspell a key, we’re protected:
```elixir
iex> chris = %User{nmae: "chris"}
** (KeyError) key :nmae not found in:
%Rumbl.Accounts.User{id: nil, name: nil, username: nil}
```
A struct is a map that has
a `__struct__` key:
```elixir
iex> jose.__struct__
Rumbl.Accounts.User
```
With our user in place, let’s define our Accounts context. We will add a couple
of functions which will allow user account fetching. Let’s create a new file in
lib/rumbl/accounts.ex and key this in:
```elixir
defmodule Rumbl.Accounts do
  @moduledoc """
  The Accounts context.
  """

  alias Rumbl.Accounts.User

  def list_users do
    [
      %User{id: "1", name: "José", username: "josevalim"},
      %User{id: "2", name: "Bruce", username: "redrapids"},
      %User{id: "3", name: "Chris", username: "chrismccord"}
    ]
  end

  def get_user(id) do
    Enum.find(list_users(), fn map -> map.id == id end)
  end

  def get_user_by(params) do
    Enum.find(list_users(), fn map ->
      Enum.all?(params, fn {key, val} -> Map.get(map, key) == val end)
    end)
  end
end
```
Let’s take the context for a spin. Start the console with iex -S mix . The -S mix
option starts IEx in the context of the Mix script, giving us access to all
modules in our application directly in IEx:

```elixir
iex> alias Rumbl.Accounts
iex> alias Rumbl.Accounts.User
iex> Accounts.list_users()
[
%Rumbl.Accounts.User{
id: "1",
name: "José",
username: "josevalim"
},
%Rumbl.Accounts.User{
id: "2",
name: "Bruce",
username: "redrapids"
},
%Rumbl.Accounts.User{
id: "3",
name: "Chris",
username: "chrismccord"
}
]
iex> Accounts.get_user("1")
%Rumbl.Accounts.User{
id: "1",
name: "José",
username: "josevalim"
}
iex> Accounts.get_user_by(name: "Bruce")
%Rumbl.Accounts.User{
id: "2",
name: "Bruce",
username: "redrapids"
}
```
## 4. Building a Controller

Specifically, we need two routes. UserController.index
will show a list of users, and UserController.show will show a single user. As
always, create the routes in rumbl/lib/rumbl_web/router.ex :
```elixir
scope "/", RumblWeb do
  pipe_through :browser
  get "/users", UserController, :index
  get "/users/:id", UserController, :show
  get "/", PageController, :index
end
```
Let’s take a closer look at the :index route:
```elixir
get "/users", UserController, :index
```

Let’s create a controller in lib/rumbl_web/controllers/user_controller.ex . Initially, we’ll
include one function called index to find the users from our Accounts context:
```elixir
defmodule RumblWeb.UserController do
  use RumblWeb, :controller
  alias Rumbl.Accounts
  def index(conn, _params) do
    users = Accounts.list_users()
    render(conn, "index.html", users: users)
  end
end
```
