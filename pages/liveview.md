---
layout: section
---

# LiveView

Porque es la opción sensata para Phoenix.

---

# Pre-requisitos

<v-clicks>

- Crear la aplicación bajo la *umbrella* de Kamisama.
- Instalar LiveView
  ```elixir
  defp deps do
    [
      ...,
      {:phoenix_live_view, "~> 0.20.2"}
    ]
  ```

</v-clicks>

---
layout: center
---

# 3 ingredientes principales

Live Views, Components, Live Components

---

# Live View

Una vista corresponde a un proceso de Elixir.

- Estado propio.
- Manejadores de eventos: `handle_info`, `handle_event`, `handle_params`.

---
layout: section
---

# ¿Cuándo utilizar `LiveView`s?

---
layout: center
---

Como el punto de entrada de una URL.
```elixir
scope "/", InspectorWeb do
  pipe_through(:browser)

  live("/", SessionStatus.Live.Main, :index)
  live("/:session", SessionStatus.Live.Main, :index)
end
```

---
layout: center
---

Cuando se requiere acceso a los parámetros de la URL.
```elixir
def handle_params(%{"session" => session_uuid}, _uri, socket) do
  {:noreply, socket |> assign(:session_uuid, session_uuid)}
end
```

---
layout: center
---

Cuando se espera recibir mensajes desde otros procesos:
```elixir
@impl true
def handle_info({:upload_manifest, manifest, errors}, socket) do
  {:noreply,
   socket
   |> assign(manifest: manifest)
   |> assign(errors: errors)}
end
```

---

## Errores Comúnes

<br />

### Los módulos de vistas deben incluir la palabra `Live` en el nombre.

<br />

2 maneras de lograrlo:
- Añadir `Live` al nombre: `Proyecto.MainLive`
- Utilizar un módulo `Live` a manera de scope: `Proyecto.Live.Main`

---

# Component

Un componente (usualmente nombrado "componente funcional") es una función que retorna una plantilla de HEEx.

- Reutilización de código.
- Permite definir atributos y "slots".

---
layout: section
---

# ¿Cómo utilizar `Component`s?

<br />

```html
<div>
  <UI.icon name="user" />
  <Text.email class="small">mail@user.com</Text.email>
</div>
```

---
layout: section
---

# ¿Cuándo utilizar `Component`s?

---
layout: center
---

Cuando se quiere encapsular código sencillo para su reutilización:
```elixir
attr :variant, :string,
    default: "primary",
    values: ["primary", "primary_outline", "secondary_outline",
             "delete", "delete_outline", "dark_outline",
             "action", "action_outline"]

  attr :size, :string, default: "base", values: ["base", "small"]
  attr :rest, :global

  slot :inner_block, required: true

  def button(assigns) do
    assigns =
      assigns
      |> assign(variant_class: variant_class(assigns.variant))
      |> assign(size_class: size_class(assigns.size))

    ~H"""
    <button class={["hbutton", @variant_class, @size_class]} {@rest}>
      <%= render_slot(@inner_block) %>
    </button>
    """
  end
```

---
layout: center
---

Cuando se quiere colocar varios componentes (sencillos) bajo un mismo módulo:
```elixir
defmodule InspectorWeb.Components.Text do
  use Phoenix.Component

  attr :rest, :global
  slot :inner_block, required: true

  def h1(assigns) do
    ~H"""
    <h1 class="htext__title1" {@rest}>
      <%= render_slot(@inner_block) %>
    </h1>
    """
  end

  attr :rest, :global
  slot :inner_block, required: true

  def h2(assigns) do
    ~H"""
    <h2 class="htext__title2" {@rest}>
      <%= render_slot(@inner_block) %>
    </h2>
    """
  end
end
```

---

# LiveComponent

Un Live Component es un punto medio entre los componentes funcionales y las vistas.

- No levantan un proceso propio.
- Permiten responder a eventos.
- Se pueden realizar acciones durante su ciclo de vida.

---
layout: section
---

# ¿Cómo utilizar `LiveComponent`s?

<br />

```html
<div>
  <.live_component module={Modal} id="modal" >
    <h1>Título Modal</h1>
    <.live_component module={Counter} init={0} id="counter" />
  </.live_component>
</div>
```

---
layout: section
---

# ¿Cuándo utilizar `LiveComponent`s?

---
layout: center
---

Cuando se quiere manejar eventos/estados dentro del componente:
```elixir
def render(assigns) do
  ~H'''
  <span><%= @count %></span>
  <button phx-click="inc" phx-target={@myself}>+</button>
  '''
end

def handle_event("inc", _params, socket) do
  %{count: count} = socket.assigns
  {:noreply, assign(socket, count: count + 1)}
end
```

---

## Errores Comúnes

<br />

### El target por defecto es el Live View padre

<br />

```elixir
def render(assigns) do
  ~H'''
  <span><%= @count %></span>
  <button phx-click="inc" phx-target={@myself}> + </button>
  <button phx-click="dec"> - </button>
  '''
end

# "inc" lo maneja el live component
# "dec" lo maneja el live view ascendiente más cercano
```

---

## Errores Comúnes

<br />

### Los LiveComponents no pueden recibir mensajes

<br />

No se puede definir `handle_info` en este nivel, pues los LiveComponents no son un proceso. Los mensajes que se envíen al componente, llegarán al LiveView padre.

---

## Errores Comúnes

<br />

### Los LiveComponents siempre esperan un id

<br />

```html
<div>
  <.live_component module={SearchForm} id="search-form" />
</div>
```

---
layout: section
---

# ¿Cómo comunicar un `LiveComponent` y un `LiveView`?

---
layout: center
---

De la vista hacia el componente, se deben actualizar los atributos dados:

```elixir
def render(assigns) do
  ~H'''
  <div>
    <.live_component module={Counter} val={@count} id="counter" />
  </div>
  '''
end

def handle_event("update", %{counter: count}, socket) do
  {:ok, assign(socket, count: count)}
end
```

---
layout: center
---

Del componente hacia la vista, se debe enviar un mensaje:

```elixir
# component.ex
def handle_event("delete", %{user: user_id}, socket) do
  send self(), {:delete, user_id}
  {:ok, socket}
end

# view.ex
def handle_info({:delete, user_id}, socket) do
  # ...
  {:noreply, socket}
end
```

---

# Recomendaciones

<v-clicks>

- Haz uso de las rutas para la navegación.
- **Evita** usar vistas anidadas.
- Separa lo más posible tus componentes.
- Usa la herramienta más simple posible (componente, live component, live view).

</v-clicks>

---

# Problemas

<v-clicks>

- El CSS afecta globalmente al proyecto.
- Resulta tedioso necesitar de la etiqueta `live_component` para utilizar un Live Component.
- La sintaxis de HEEx no es muy agradable.

</v-clicks>
