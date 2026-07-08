# Bigger Bookmarks

A Monitor-style bookmark launcher with optional icons, descriptions, and link arrows, without health checking or service monitoring.

## Widget Type

`custom-api`

## Preview

![Preview](./preview.png)

## Options

Unlike Glance's native Bookmarks widget, *Bigger Bookmarks* does not support groups. Each configured link is rendered as an independent Monitor-style row.

| Name         | Type    | Required | Default |
| ------------ | ------- | -------- | ------- |
| `color`      | string  | No       | —       |
| `hide-arrow` | boolean | No       | `false` |
| `links`      | array   | Yes      | —       |

### `color`

Sets the color of the right-side `↗` indicator.

Use Glance's HSL value format, such as `20 50 50`. When omitted, the arrow uses Glance's standard highlight color.

### `hide-arrow`

Controls whether the right-side `↗` indicator is shown by default.

Individual links may override this setting through `links[].hide-arrow`.

### `links`

A list of bookmark entries. Links are rendered in the same order as the YAML configuration.

| Name          | Type    | Required | Default                       |
| ------------- | ------- | -------- | ----------------------------- |
| `title`       | string  | Yes      | —                             |
| `url`         | string  | Yes      | —                             |
| `icon`        | string  | No       | —                             |
| `description` | string  | No       | —                             |
| `same-tab`    | boolean | No       | `false`                       |
| `target`      | string  | No       | `_blank`                      |
| `hide-arrow`  | boolean | No       | Inherits `options.hide-arrow` |

#### `title`

The displayed bookmark name.

#### `url`

The destination URL. The entire row is clickable.

#### `icon`

An optional direct image URL or local image path, such as `https://example.com/favicon.ico` or `/assets/icons/example.svg`.

Glance icon shorthand such as `si:immich`, `mdi:home`, `di:immich`, and `sh:glance` is not supported.

#### `description`

Optional subdued text displayed below the title.

#### `same-tab`

Opens the link in the current tab when set to `true`.

#### `target`

Sets the HTML link target, such as `_blank`, `_self`, `_parent`, or `_top`.

When both `target` and `same-tab` are configured, `target` takes precedence.

#### `hide-arrow`

Overrides the widget-level `hide-arrow` setting for this link.

An explicit `false` also overrides a global `hide-arrow: true`.

## Configuration

When adding multiple *Bigger Bookmarks* widgets, define the template once with a YAML anchor and reuse it with an alias. This keeps the configuration compact and ensures every *Bigger Bookmarks* widget uses the same rendering logic.

```yaml
- type: custom-api
  title: Bigger Bookmark
  options:
    color: 20 50 50
    hide-arrow: false
    links:
      - title: Title
        url: https://example.com/
        icon: ./static/02b8858d2d/app-icon.png
        description: Description
        same-tab: false
        target: _blank
        hide-arrow: false
    template: &bigger-bookmarks-template |
      {{ $links := index .Options "links" }}
      {{ $globalHideArrow := .Options.BoolOr "hide-arrow" false }}
      {{ $color := .Options.StringOr "color" "" }}

      <style>
        .bigger-bookmarks-link {
          width: 100%;
          min-height: 5rem;
          padding: 0.75rem 0;
          border-radius: var(--border-radius);
          text-decoration: none;
        }

        .bigger-bookmarks-icon {
          display: block;
          opacity: 0.8;
          filter: grayscale(0.4);
          object-fit: contain;
          aspect-ratio: 1 / 1;
          width: 3.2rem;
          position: relative;
          top: -0.1rem;
          transition: filter 0.3s, opacity 0.3s;
          flex-shrink: 0;
        }

        .bigger-bookmarks-link:hover .bigger-bookmarks-icon {
          opacity: 1;
        }

        .bigger-bookmarks-link:hover .bigger-bookmarks-icon:not(.flat-icon) {
          filter: grayscale(0);
        }

        .bigger-bookmarks-status-icon {
          flex-shrink: 0;
          margin-left: auto;
          width: 2rem;
          height: 2rem;
          display: flex;
          align-items: center;
          justify-content: center;
        }

        .bigger-bookmarks-arrow {
          display: block;
          text-align: center;
          font-size: 1.15em;
          line-height: 1;
        }
      </style>

      <ul class="dynamic-columns list-gap-20 list-with-separator">
      {{ range $link := $links }}

          {{/* Native Bookmarks target semantics */}}
          {{ $target := "_blank" }}
          {{ if index $link "same-tab" }}
          {{ $target = "" }}
          {{ end }}
          {{ with $configuredTarget := index $link "target" }}
          {{ $target = $configuredTarget }}
          {{ end }}

          {{/* Link-level hide-arrow overrides widget-level hide-arrow,
              including an explicit false override. */}}
          {{ $hideArrow := $globalHideArrow }}
          {{ $rawHideArrow := index $link "hide-arrow" }}
          {{ if eq (printf "%T" $rawHideArrow) "bool" }}
          {{ $hideArrow = $rawHideArrow }}
          {{ end }}

          <li style="min-width: 0;">
            <a
              class="bigger-bookmarks-link flex items-center gap-15"
              href="{{ index $link "url" }}"
              {{ if $target }}target="{{ $target }}"{{ end }}
              rel="noopener noreferrer"
              title="{{ index $link "title" }}"
            >
              {{ with $icon := index $link "icon" }}
                <img
                  class="bigger-bookmarks-icon"
                  src="{{ $icon }}"
                  alt=""
                  loading="lazy"
                >
              {{ end }}

              <div class="grow min-width-0">
                <span class="size-h3 color-highlight text-truncate block">
                  {{ index $link "title" }}
                </span>

                {{ with $description := index $link "description" }}
                  <ul class="list-horizontal-text">
                    <li>{{ $description }}</li>
                  </ul>
                {{ end }}
              </div>

              {{ if not $hideArrow }}
                <span class="bigger-bookmarks-status-icon" aria-hidden="true">
                  <span
                    class="bigger-bookmarks-arrow{{ if not $color }} color-primary{{ end }}"
                    {{ if $color }}style="color: hsl({{ $color }});"{{ end }}
                  >↗</span>
                </span>
              {{ end }}
            </a>
          </li>

      {{ end }}
      </ul>
```

The widget does not define a `url`, so Glance does not perform a server-side request or health check. External icons are still loaded normally by the browser.
