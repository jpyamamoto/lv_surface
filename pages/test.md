---
layout: section
---

# Testing

---

## ¿Qué probar de un proyecto LiveView?

- Presencia de clases, datos o etiquetas HTML.
- Interacción (navegación o eventos).

---

# Verificar Contenido

```elixir
test "variant attr" do
  for variant <- @variants do
    content = "Contenido Botón"
    assigns = %{variant: variant, content: content}

    html =
      render_surface do
        ~F"""
        <Button variant={@variant}>
          {@content}
        </Button>
        """
      end

    assert html =~ "<button"
    assert html =~ content
    assert html =~ variant
  end
end
```

---
layout: two-cols
---

# Ventajas

- Sencillo de implementar.
- Se verifica el contenido.

::right::

# Desventajas

- Datos rígidos.
- No hay información semántica.

---
layout: section
---

# LiveViewTest + PhoenixTest

[https://hexdocs.pm/phoenix_test/PhoenixTest.html](https://hexdocs.pm/phoenix_test/PhoenixTest.html)

---
layout: two-cols
---

# Ventajas

- Contenido semántico en las pruebas.
- Estructura del código como un flujo de acciones.
- Utilidades sobre LiveViewTest.

::right::

# Desventajas

- Biblioteca en construcción.

---

# Ejemplos

```elixir
test "collapse/expand menu", %{conn: conn} do
  conn
  |> visit(~p"/")
  |> assert_has("ul li", text: "Session Status")
  |> click_button(".aside_button", "")
  |> refute_has("ul li", text: "Session Status")
  |> click_button(".aside_button", "")
  |> assert_has("ul li", text: "Session Status")
end
```

---

# Ejemplos

```elixir
test "filter participants", %{conn: conn} do
  conn
  |> visit(~p"/#{@existent_uuid}")
  |> assert_path("/#{@existent_uuid}")
  |> assert_has("h5", text: "Participants (3)")
  |> assert_has(".user-preview", count: 3)
  # filter stable
  |> select("Stable", from: "Filter")
  |> assert_has("h5", text: "Participants (1)")
  |> assert_has(".user-preview", count: 1)
  # filter unstable
  |> select("Unstable", from: "Filter")
  |> assert_has("h5", text: "Participants (1)")
  |> assert_has(".user-preview", count: 1)
  # filter broken
  |> select("Broken", from: "Filter")
  |> assert_has("h5", text: "Participants (1)")
  |> assert_has(".user-preview", count: 1)
end
```

---

# Ejemplos

```elixir
test "render", %{live: live} do
  live
  |> assert_has("h2", text: "Synchronization")
  |> assert_has("h4", text: "Events")
  |> assert_has("h4", text: "Incidences")
  |> assert_has("h5", text: @participant.user.email)
end
```
