---
layout: section
---

# Surface

[https://surface-ui.org/](https://surface-ui.org/)

---

# Beneficios

<v-clicks>

- CSS y JS Hooks con _scope_ local al componente.
- Todos los componentes generan una nueva etiqueta a utilizar.
- La sintaxis es más amigable y consistente.

</v-clicks>

---
layout: two-cols
---

# LiveView

- LiveView
- LiveComponent
- Component
- `sigil_H`
- *.heex

::right::

# Surface

- SurfaceView
- SurfaceLiveComponent
- SurfaceComponent
- `sigil_F`
- *.sface

---

# Pre-requisitos

<v-clicks>

- Instalar Surface
  ```elixir
  defp deps do
    [
      ...,
      {:surface, "~> 0.12.0"}
    ]
  ```
- Configurar con `mix surface.init`

</v-clicks>

---

Define lo siguiente en el archivo `<proyecto>.ex`:

```elixir

  def surface_live_view do
    quote do
      use Surface.LiveView, layout: {InspectorWeb.Layouts, :app}
      unquote(html_helpers())
    end
  end

  def surface_live_component do
    quote do
      use Surface.LiveComponent
      unquote(html_helpers())
    end
  end

  def surface_component do
    quote do
      use Surface.Component
      unquote(html_helpers())
    end
  end
```

---
layout: center
---


- LiveViews

```elixir
use <Proyecto>, :surface_live_view
```

- Live Components

```elixir
use <Proyecto>, :surface_live_component
```

- Componentes

```elixir
use <Proyecto>, :surface_component
```

---

# Datos

```elixir
defmodule InspectorWeb.Components.Containers.Modal do
  use InspectorWeb, :surface_live_component

  prop opts, :keyword, default: []
  prop class, :css_class, default: []

  data show, :boolean, default: false

  slot default, required: true

  def render(assigns) do
    ~F"""
    <div class={"modal__bg", @class, active: @show} {...@opts} id={@id}>
      <#slot />
    </div>
    """
  end
end
```

---
layout: section
---

# Eventos

---

Todos los eventos se manejan en el mismo componente:

```elixir
slot default, required: true
slot aside, required: true
data is_collapsed, :boolean, default: false

def render(assigns) do
  ~F"""
  <section>
    <Icon name={if @is_collapsed, do: "arrow_right", else: "arrow_left"} :on-click="collapse-menu" />
    {#if !@is_collapsed}
      <aside>
        <#slot {@aside} />
      </aside>
    {/if}

    <#slot />
  </section>
  """
end

def handle_event("collapse-menu", _, socket) do
  {:noreply, update(socket, :is_collapsed, &(!&1))}
end
```

---

A menos que se indique lo contrario:

```elixir
prop variant, :string, default: "primary",
  values!: ["primary", "primary_outline", "secondary_outline", "delete",
    "delete_outline", "dark_outline", "action", "action_outline"]
prop size, :string, default: "base", values!: ["base", "small"]
prop opts, :keyword, default: []
prop click, :event, default: nil
prop values, :map, default: %{}
slot default, required: true

def render(assigns) do
  ~F"""
  <button
    class={["hbutton", variant_class(@variant), size_class(@size)]}
    {...@opts}
    :on-click={@click} 
    :values={@values}
  >
    <#slot />
  </button>
  """
end
```

---
layout: section
---

# CSS

---
layout: center
---

Se puede colocar el CSS al comienzo de `~F` entre las etiquetas `<style></style>`.

```elixir
def render(assigns) do
  ~F"""
  <style>
    .inspector-icon {
      box-sizing: border-box;
      display: inline-block;
      ...
    }

    .inspector-icon :deep(svg) {
      position: relative;
      ...
    }
  </style>

  <span class="inspector-icon">
    <.svg name={@name} />
  </span>
  """
end
```

---
layout: center
---

O se puede usar un archivo co-localizado.

```
components/
├── ...
├── card.ex
├── card.css
├── card.hooks.js
├── ...
├── modal.sface
├── modal.css
├── modal.hooks.js
```

---

# Contextos

Se pueden definir valores en un contexto que un SurfaceView heredará a los componentes descendientes.

```elixir
# live_view.ex
def handle_params(%{"session" => session_uuid}, _uri, socket) do
  {:noreply, socket |> Context.put(navigation: [%{name: "Home", patch: ~p"/#{session_uuid}"}])
}
end
```

Se obtienen de la siguiente forma:

```elixir
# component.ex
data navigation, :list, from_context: :navigation

def render(assigns) do
  ~F"""
    ...
  """
end

```
