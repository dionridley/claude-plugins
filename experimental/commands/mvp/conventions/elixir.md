# Elixir/Phoenix Conventions Reference

This file is read by the `/mvp build` orchestrator at the start of each build session when the stack is Elixir. The contents are injected into every agent prompt under a `## Elixir/Phoenix Conventions — MANDATORY` section.

---

## Mandatory Rules for Every Agent

Include this block verbatim in all Elixir agent prompts:

```
## Elixir/Phoenix Conventions — MANDATORY

**Date formatting:**
- NEVER use `Calendar.strftime(date, "%-d")` — the `%-d` directive breaks on macOS/BSD
- ALWAYS use `date.day` directly: `"#{Calendar.strftime(date, "%B")} #{date.day}, #{date.year}"`

**Form errors:**
- NEVER call `Phoenix.HTML.Form.translate_error/1` — it is private/undefined in Phoenix 1.8
- Use `elem(error, 0)` for direct access, or the `<.error>` core component

**LiveView mounts:**
- ALWAYS assign `:page_title` in every `mount/3` or `handle_params/3`:
  `|> assign(:page_title, "Page Name")`

**Association preloading:**
- ALWAYS preload the full association chain required by the view
- Wrong: `Repo.preload([:tags])` when view renders `tag.category.color`
- Correct: `Repo.preload([tags: :category])`
- Read the render template before writing context functions to determine required depth

**Flash messages:**
- Flash is handled by `root.html.heex` — do NOT add `<.flash_group>` inside individual LiveView renders
- `put_flash/3` calls will display automatically via the root layout

**Navigation:**
- Use `~p"/path/#{id}"` verified route sigil for ALL internal paths — never hardcoded strings
- `push_navigate(socket, to: ~p"/path")` for cross-LiveView navigation
- `push_patch(socket, to: ~p"/path?#{params}")` for same-LiveView param changes (modals, filters)

**Forms and submission:**
- Add `phx-disable-with="Saving..."` on all submit buttons for loading state feedback

**Reading before writing:**
- ALWAYS read an existing file with the Read tool before modifying it
- Only use Write directly for files that do not yet exist

**Seed data:**
- Faker is available: `Faker.Person.name()`, `Faker.Internet.email()`, `Faker.Lorem.paragraph(2..3)`
- Make seeds idempotent: delete-then-insert so re-running doesn't error
```

---

## Phoenix 1.8 Environment Reference

### What Ships by Default in Phoenix 1.8
- **DaisyUI 5.x** — `assets/vendor/daisyui.js`, use component classes: `btn`, `card`, `badge`, `modal`, `alert`, `input`, `select`, etc.
- **Heroicons** — `assets/vendor/heroicons.js`, use `<.icon name="hero-..." />` component
- **TailwindCSS 4.x** — configured via `assets/css/app.css` with `@plugin "../vendor/daisyui"` and `@plugin "../vendor/heroicons"`
- **LiveView** — already wired up, use `use MyAppWeb, :live_view`
- **`data-theme` attribute** — DaisyUI theming via HTML attribute on `<html>` tag

### Key File Locations
```
lib/[slug]/                       # Business logic — contexts, schemas
lib/[slug]_web/live/              # LiveView pages
lib/[slug]_web/components/        # Shared components (core_components.ex, layouts.ex)
lib/[slug]_web/router.ex          # Routes
priv/repo/migrations/             # Ecto migrations (timestamp-prefixed filenames)
priv/repo/seeds.exs               # Seed data
assets/css/app.css                # Tailwind + DaisyUI CSS entry point
assets/js/app.js                  # JS entry point
config/dev.exs                    # Dev config (port lives here)
config/runtime.exs                # Runtime config (prod port override — MUST be wrapped)
test/[slug]/                      # Context/unit tests
test/[slug]_web/live/             # LiveView interaction tests
test/support/factory.ex           # Test data factory helpers
```

### Standard LiveView Pattern
```elixir
defmodule MyAppWeb.SomeLive do
  use MyAppWeb, :live_view

  def mount(_params, _session, socket) do
    {:ok,
     socket
     |> assign(:page_title, "Page Name")
     |> assign(:data, load_data())}
  end

  def handle_params(%{"id" => id}, _uri, socket) do
    {:noreply,
     socket
     |> assign(:page_title, "Item Name")
     |> assign(:item, Context.get_item!(id))}
  end

  def handle_event("save", params, socket) do
    case Context.create_thing(params) do
      {:ok, thing} ->
        {:noreply,
         socket
         |> put_flash(:info, "Created successfully!")
         |> push_navigate(to: ~p"/things/#{thing.id}")}

      {:error, changeset} ->
        {:noreply, assign(socket, :changeset, changeset)}
    end
  end

  def render(assigns) do
    ~H"""
    <div>
      <h1 class="text-2xl font-bold"><%= @page_title %></h1>
    </div>
    """
  end
end
```

### Standard Context Pattern
```elixir
defmodule MyApp.Things do
  alias MyApp.Repo
  alias MyApp.Things.Thing

  def list_things do
    Repo.all(Thing)
    |> Repo.preload([:association])        # preload what views need
  end

  def get_thing!(id) do
    Repo.get!(Thing, id)
    |> Repo.preload([nested: :deep])       # preload full chain
  end

  def create_thing(attrs) do
    %Thing{}
    |> Thing.changeset(attrs)
    |> Repo.insert()
  end

  def update_thing(%Thing{} = thing, attrs) do
    thing
    |> Thing.changeset(attrs)
    |> Repo.update()
  end

  def delete_thing(%Thing{} = thing), do: Repo.delete(thing)

  def change_thing(%Thing{} = thing, attrs \\ %{}), do: Thing.changeset(thing, attrs)
end
```

### Writing Migration Files Directly
Skip `mix ecto.gen.migration` (produces empty files requiring a read-edit cycle). Write migration files directly using the `Write` tool.

Generate timestamp:
```bash
date -u +"%Y%m%d%H%M%S"
```

Filename: `priv/repo/migrations/[timestamp]_create_[table].exs`

```elixir
defmodule MyApp.Repo.Migrations.CreateThings do
  use Ecto.Migration

  def change do
    create table(:things) do
      add :name, :string, null: false
      add :description, :text
      add :status, :string, default: "active"

      timestamps()
    end

    create index(:things, [:status])
  end
end
```

### ExUnit Test Patterns
```elixir
# test/my_app/things_test.exs
defmodule MyApp.ThingsTest do
  use MyApp.DataCase, async: true
  import MyApp.Factory

  alias MyApp.Things

  describe "create_thing/1" do
    test "creates with valid attrs" do
      assert {:ok, thing} = Things.create_thing(%{name: "Test"})
      assert thing.name == "Test"
    end

    test "returns error with invalid attrs" do
      assert {:error, changeset} = Things.create_thing(%{})
      assert "can't be blank" in errors_on(changeset).name
    end
  end
end
```

```elixir
# test/my_app_web/live/thing_live_test.exs
defmodule MyAppWeb.ThingLiveTest do
  use MyAppWeb.ConnCase, async: true
  import Phoenix.LiveViewTest
  import MyApp.Factory

  describe "Index" do
    test "lists things", %{conn: conn} do
      thing = insert(:thing)
      {:ok, _view, html} = live(conn, ~p"/things")
      assert html =~ thing.name
    end

    test "creates a thing", %{conn: conn} do
      {:ok, view, _html} = live(conn, ~p"/things/new")

      view
      |> form("#thing-form", thing: %{name: "New Thing"})
      |> render_submit()

      assert_redirect(view, ~p"/things")
    end
  end
end
```

### Factory Pattern (`test/support/factory.ex`)
```elixir
defmodule MyApp.Factory do
  alias MyApp.Repo

  def build(:thing, attrs \\ %{}) do
    %MyApp.Things.Thing{
      name: Faker.Lorem.word(),
      description: Faker.Lorem.sentence()
    }
    |> Map.merge(Enum.into(attrs, %{}))
  end

  def insert(factory_name, attrs \\ %{}) do
    factory_name
    |> build(attrs)
    |> Repo.insert!()
  end
end
```

### DaisyUI Component Quick Reference
```html
<!-- Buttons -->
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary btn-sm">Small</button>
<button class="btn btn-outline btn-error">Outline Error</button>

<!-- Cards -->
<div class="card card-bordered shadow-sm bg-base-100">
  <div class="card-body">
    <h2 class="card-title">Title</h2>
    <p>Content</p>
    <div class="card-actions justify-end">
      <button class="btn btn-primary">Action</button>
    </div>
  </div>
</div>

<!-- Badges -->
<span class="badge badge-success">Active</span>
<span class="badge badge-error badge-outline">Error</span>

<!-- Form inputs -->
<label class="form-control w-full">
  <div class="label"><span class="label-text">Name</span></div>
  <input type="text" class="input input-bordered w-full" phx-debounce="300" />
</label>

<!-- Alert/Flash -->
<div class="alert alert-success"><span>Success!</span></div>

<!-- Empty state -->
<div class="hero min-h-[200px] bg-base-200 rounded-box">
  <div class="hero-content text-center">
    <div>
      <h1 class="text-2xl font-bold">Nothing here yet</h1>
      <p class="py-4 text-base-content/70">Get started by creating one.</p>
      <.link navigate={~p"/new"} class="btn btn-primary">Create</.link>
    </div>
  </div>
</div>
```
