---
icon: lucide/rectangle-horizontal
tags:
  - Authoring
---

# Buttons

Zensical provides dedicated styles for primary and secondary buttons that can be
added to any link, `label` or `button` element. This is especially useful for
documents or landing pages with dedicated _call-to-actions_.

## Configuration

This configuration allows to add attributes to all inline- and block-level
elements with a simple syntax, turning any link into a button. Add the
following lines to your configuration:

=== "`zensical.toml`"

    ``` toml
    [project.markdown_extensions.attr_list]
    ```

=== "`mkdocs.yml`"

    ``` yaml
    markdown_extensions:
      - attr_list
    ```

See additional configuration options:

- [Attribute Lists]

  [Attribute Lists]: ../setup/extensions/python-markdown.md#attribute-lists

## Usage

### Add a button

In order to render a link as a button, suffix it with curly braces and add the
`.md-button` class selector to it. The button will receive the selected
[primary color] and [accent color] if active.

``` markdown title="Button"
[Subscribe to our newsletter](#){ .md-button }
```

<div class="result" markdown>

[Subscribe to our newsletter][Demo]{ .md-button }

</div>

  [primary color]: ../setup/colors.md#primary-color
  [accent color]: ../setup/colors.md#accent-color
  [Demo]: javascript:alert$.next("Demo")

### Add a primary button

If you want to display a filled, primary button (like on the [landing page]
of Zensical), add both, the `.md-button` and `.md-button--primary`
CSS class selectors.

``` markdown title="Button, primary"
[Subscribe to our newsletter](#){ .md-button .md-button--primary }
```

<div class="result" markdown>

[Subscribe to our newsletter][Demo]{ .md-button .md-button--primary }

</div>

  [landing page]: https://zensical.org

### Add a button with an icon

Of course, icons can be added to all types of buttons by using the [icon syntax]
together with any valid [icon shortcode]:

  [icon shortcode]: icons-emojis.md#included-icon-sets
  [icon syntax]: icons-emojis.md#use-icons

``` markdown title="Button with icon"
[Send :fontawesome-solid-paper-plane:](#){ .md-button }
```

<div class="result" markdown>

[Send :fontawesome-solid-paper-plane:][Demo]{ .md-button }

</div>

