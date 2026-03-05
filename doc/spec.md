# chatui Specification

A JSON schema UI specification designed to be simple, easy to render, and mainly used for chatbot ui.

## Overview

chatui defines a declarative JSON format for describing user interfaces. The specification prioritizes:

- **Simplicity**: Minimal learning curve with intuitive structure
- **Portability**: Renders consistently across web, mobile, and WeChat mini-programs
- **Extensibility**: Easy to add custom components while maintaining compatibility

## Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Schema Structure

### Chat UI Object

```json
{
  "version": "1.0",
  "body": { /* Component */ },
  "data": {}
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| version | string | REQUIRED | Specification version (e.g., `"1.0"`). Renderers SHOULD accept documents whose major version matches the renderer's supported version. A renderer supporting version `1.x` MUST accept any `1.y` document where `y <= x` |
| body | Component | REQUIRED | Root component of the UI tree |
| data | object | OPTIONAL | Initial data for form fields and table rows |

### Component

Every UI element is a Component with the following structure:

```json
{
  "type": "component-type",
  "id": "unique-id",
  "props": {},
  "model": "data.path.to.value",
  "style": {},
  "children": [],
  "visible": "form.showAdvanced"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| type | string | REQUIRED | Component type identifier. In addition to the built-in types defined below, custom type values MAY be used for user-defined rendering. Custom types SHOULD use a `x-` prefix (e.g., `x-chart`, `x-map`) to avoid conflicts with future built-in types |
| id | string | OPTIONAL | Unique identifier for the component |
| props | object | OPTIONAL | Component-specific properties |
| model | string | OPTIONAL | Path to data model for two-way binding |
| style | object | OPTIONAL | Styling properties |
| visible | string \| boolean | OPTIONAL | Controls component visibility. When `true` or omitted, the component is rendered. When `false`, the component is hidden. When a string, it is evaluated as a model path — the component is visible if the resolved value is truthy. A string prefixed with `!` negates the condition (e.g., `"!form.isCollapsed"` means visible when the value is falsy) |
| children | array | OPTIONAL | Child components |

## Forward Compatibility

Renderers MUST follow these rules to ensure forward compatibility:

1. **Unknown component types**: Renderers MUST ignore components with unrecognized `type` values and MUST NOT treat them as errors. If the unrecognized component has `children`, the renderer MAY render the children as if the parent were a `view` component.
2. **Unknown properties**: Renderers MUST ignore unrecognized fields in `props`, the Component object, and the Chat UI Object without raising errors.

## Components

### view

A container element. Used as a generic wrapper for layout and grouping.

```json
{
  "type": "view",
  "id": "container",
  "props": {
    "direction": "vertical",
    "gap": "8px"
  },
  "style": {
    "padding": "16px",
    "backgroundColor": "#f5f5f5"
  },
  "children": []
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| direction | string | OPTIONAL | Layout direction: `vertical` (default) or `horizontal` |
| gap | string | OPTIONAL | Spacing between children |

### scroll-view

A scrollable container for content that may exceed the visible area.

```json
{
  "type": "scroll-view",
  "id": "scrollArea",
  "props": {
    "direction": "vertical",
    "height": "300px"
  },
  "children": []
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| direction | string | OPTIONAL | Scroll direction: `vertical` (default) or `horizontal` |
| height | string | OPTIONAL | Fixed height of the scroll container. REQUIRED when `direction` is `vertical` |

### text

Displays static or dynamic text content.

```json
{
  "type": "text",
  "id": "greeting",
  "props": {
    "text": "Hello, World!",
    "format": "plain"
  }
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| text | string | REQUIRED | Text content to display. Supports template interpolation using `{{model.path}}` syntax to embed dynamic values from `data` |
| format | string | OPTIONAL | Text format: `plain` (default) or `markdown` |

**Template interpolation**: Within `text`, `{{path}}` references are resolved against the `data` object using the same path rules as `model`. For example, `"Hello, {{form.username}}!"` renders as `"Hello, Alice!"` when `data.form.username` is `"Alice"`. If the path resolves to `undefined`, the placeholder SHOULD be replaced with an empty string.

### image

Displays an image from a URL.

```json
{
  "type": "image",
  "id": "avatar",
  "props": {
    "src": "/images/avatar.png",
    "alt": "User avatar",
    "mode": "cover"
  }
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| src | string | REQUIRED | Image source URL |
| alt | string | OPTIONAL | Alternative text for accessibility |
| width | string | OPTIONAL | Image display width (e.g., `"120px"`, `"100%"`) |
| height | string | OPTIONAL | Image display height (e.g., `"80px"`, `"auto"`) |
| mode | string | OPTIONAL | Image fit mode: `cover`, `contain`, `fill`, `none`. Default is `cover` |

### table

Displays tabular data with rows and columns.

```json
{
  "type": "table",
  "id": "userTable",
  "props": {
    "columns": [
      { "key": "name", "title": "Name", "width": "120px" },
      { "key": "age", "title": "Age", "width": "80px" },
      { "key": "email", "title": "Email" }
    ],
    "bordered": true,
    "striped": false,
    "hoverable": true
  },
  "model": "tableData"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| columns | array | REQUIRED | Column definitions |
| columns[].key | string | REQUIRED | Data field key |
| columns[].title | string | REQUIRED | Column header text |
| columns[].width | string | OPTIONAL | Column width |
| bordered | boolean | OPTIONAL | Show table borders |
| striped | boolean | OPTIONAL | Alternate row colors |
| hoverable | boolean | OPTIONAL | Highlight row on hover |

The `model` path MUST resolve to an array of objects. Each object represents a row, with property keys matching the `columns[].key` values. For example, if `model` is `"tableData"`, then `data.tableData` should be `[{"name": "Alice", "age": 30}, ...]`.

### form

Container for form elements with validation.

```json
{
  "type": "form",
  "id": "loginForm",
  "props": {
    "name": "loginForm",
    "layout": "vertical",
    "labelWidth": "100px"
  },
  "children": []
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| name | string | OPTIONAL | Form identifier |
| layout | string | OPTIONAL | Form layout style: `vertical` (default), `horizontal`, or `inline` |
| labelWidth | string | OPTIONAL | Label width for horizontal layout |


### button

Interactive button element with action handling.

```json
{
  "type": "button",
  "id": "submitButton",
  "props": {
    "text": "Submit",
    "variant": "primary",
    "disabled": false,
    "actionUrl": "/api/submit",
    "actionMethod": "POST",
    "actionData": { "source": "web", "version": "1.0" }
  }
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| text | string | REQUIRED | Button label text |
| variant | string | OPTIONAL | Button style variant: `primary`, `secondary`, `danger`, `link`. Default is `secondary` |
| disabled | boolean | OPTIONAL | Disable button interaction |
| actionUrl | string | OPTIONAL | URL to call when button is clicked |
| actionMethod | string | OPTIONAL | HTTP method for the request: `POST` (default), `GET`, `PUT`, `DELETE` |
| actionData | object | OPTIONAL | Static data merged with form-bound data; form-bound values take precedence for overlapping keys |

#### Action Behavior

When a button with `actionUrl` is clicked, the renderer MUST perform the following:

1. **Request method**: The HTTP method specified by `actionMethod` (default: `POST`)
2. **Content-Type**: `application/json`
3. **Request body**: A JSON object merged from `actionData` and the nearest ancestor `form`'s bound data (resolved via `model` paths of all form children). Form data MUST take precedence over `actionData` for overlapping fields.
4. **Loading state**: The button SHOULD be set to a loading/disabled state while the request is in flight, preventing duplicate submissions.
5. **Response handling**: The endpoint SHOULD return a JSON response. The renderer MUST handle the following:
   - **Success (HTTP 2xx)**: If the response body contains a Chat UI Object, the renderer SHOULD replace the current UI with the returned object. Otherwise, no action is taken.
   - **Client error (HTTP 4xx)**: The renderer SHOULD display the response message to the user.
   - **Server error (HTTP 5xx)**: The renderer SHOULD display a generic error message.
   - **Network failure**: The renderer SHOULD display a network error message and restore the button to its original state.

### link

Navigation link for redirecting to a URL.

```json
{
  "type": "link",
  "id": "homeLink",
  "props": {
    "text": "Go to Home",
    "href": "/home",
    "target": "_self"
  }
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| text | string | REQUIRED | Link display text |
| href | string | REQUIRED | Target URL |
| target | string | OPTIONAL | Open behavior: `_self` (default) or `_blank` |
| disabled | boolean | OPTIONAL | Disable link interaction |

### input

Single-line text input field.

```json
{
  "type": "input",
  "id": "emailInput",
  "props": {
    "placeholder": "Enter email",
    "type": "text",
    "maxLength": 100,
    "disabled": false
  },
  "model": "form.email"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| placeholder | string | OPTIONAL | Placeholder text when empty |
| type | string | OPTIONAL | Input type: `text` (default), `password`, `number`, `email`, `tel` |
| maxLength | number | OPTIONAL | Maximum character length |
| readonly | boolean | OPTIONAL | Make input read-only (value visible but not editable) |
| disabled | boolean | OPTIONAL | Disable input interaction |

### textarea

Multi-line text input field.

```json
{
  "type": "textarea",
  "id": "bioInput",
  "props": {
    "placeholder": "Tell us about yourself",
    "rows": 4,
    "maxLength": 500,
    "disabled": false
  },
  "model": "form.bio"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| placeholder | string | OPTIONAL | Placeholder text when empty |
| rows | number | OPTIONAL | Number of visible text lines. Default is `3` |
| maxLength | number | OPTIONAL | Maximum character length |
| readonly | boolean | OPTIONAL | Make textarea read-only (value visible but not editable) |
| disabled | boolean | OPTIONAL | Disable textarea interaction |

### checkbox

A checkbox for boolean or multi-select input. Supports two modes depending on props:

**Boolean mode** (single checkbox): When `options` is not provided, the checkbox binds to a boolean value.

```json
{
  "type": "checkbox",
  "id": "agreeCheckbox",
  "props": {
    "label": "I agree to the terms",
    "disabled": false
  },
  "model": "form.agreed"
}
```

**Multi-select mode**: When `options` is provided, the checkbox group binds to an array of selected values.

```json
{
  "type": "checkbox",
  "id": "hobbiesCheckbox",
  "props": {
    "options": [
      { "label": "Reading", "value": "reading" },
      { "label": "Sports", "value": "sports" },
      { "label": "Music", "value": "music" }
    ],
    "direction": "vertical",
    "disabled": false
  },
  "model": "form.hobbies"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string | OPTIONAL | Text label for single boolean checkbox. Ignored when `options` is provided |
| options | array | OPTIONAL | List of checkbox options for multi-select mode. When provided, `model` binds to an array of selected values |
| options[].label | string | REQUIRED | Display text for the option |
| options[].value | string \| number | REQUIRED | Value added to the bound array when selected |
| direction | string | OPTIONAL | Layout direction for multi-select mode: `vertical` (default) or `horizontal` |
| disabled | boolean | OPTIONAL | Disable checkbox interaction |

### radio

A radio button group for single-select input.

```json
{
  "type": "radio",
  "id": "genderRadio",
  "props": {
    "options": [
      { "label": "Male", "value": "male" },
      { "label": "Female", "value": "female" }
    ],
    "direction": "horizontal",
    "disabled": false
  },
  "model": "form.gender"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| options | array | REQUIRED | List of radio options |
| options[].label | string | REQUIRED | Display text for the option |
| options[].value | string \| number | REQUIRED | Value submitted when selected |
| direction | string | OPTIONAL | Layout direction: `vertical` (default) or `horizontal` |
| disabled | boolean | OPTIONAL | Disable all radio options |

### select

A dropdown selector for choosing from a list of options.

```json
{
  "type": "select",
  "id": "citySelect",
  "props": {
    "placeholder": "Select a city",
    "options": [
      { "label": "Beijing", "value": "bj" },
      { "label": "Shanghai", "value": "sh" },
      { "label": "Guangzhou", "value": "gz" }
    ],
    "disabled": false
  },
  "model": "form.city"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| placeholder | string | OPTIONAL | Placeholder text when no option is selected |
| options | array | REQUIRED | List of selectable options |
| options[].label | string | REQUIRED | Display text for the option |
| options[].value | string \| number | REQUIRED | Value submitted when selected |
| disabled | boolean | OPTIONAL | Disable select interaction |

### switch

A toggle switch for boolean input.

```json
{
  "type": "switch",
  "id": "notifySwitch",
  "props": {
    "label": "Enable notifications",
    "disabled": false
  },
  "model": "form.notify"
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string | OPTIONAL | Text label displayed next to the switch |
| disabled | boolean | OPTIONAL | Disable switch interaction |

### form-item

Wrapper for form fields with label and validation.

```json
{
  "type": "form-item",
  "id": "usernameItem",
  "props": {
    "label": "Username",
    "name": "username",
    "required": true,
    "rules": [
      { "type": "required", "message": "Username is required" },
      { "type": "minLength", "value": 3, "message": "Min 3 characters" }
    ]
  },
  "children": [
    {
      "type": "input",
      "props": { "placeholder": "Enter username" }
    }
  ]
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| label | string | OPTIONAL | Field label text |
| name | string | REQUIRED | Field identifier used for validation error display and accessibility. The actual form data binding is determined by the child component's `model` field, not by `name` |
| required | boolean | OPTIONAL | Mark field as required |
| rules | array | OPTIONAL | Validation rules (see below) |

**Relationship between `name` and `model`**: The `name` prop identifies the form field for validation and label association. The child component's `model` field determines the data binding path. When constructing the request body for `actionUrl`, the renderer collects values from all `model`-bound descendants within the form, using their `model` paths as keys. The `name` prop does NOT affect data binding or request body construction.

#### Validation Rules

Each rule in the `rules` array is an object with the following fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| type | string | REQUIRED | Rule type (see supported types below) |
| value | any | OPTIONAL | Parameter for the rule (e.g., minimum length). REQUIRED for types that need a comparison value |
| message | string | REQUIRED | Error message displayed when validation fails |

**Supported rule types:**

| Rule Type | Value Type | Description |
|-----------|-----------|-------------|
| `required` | — | Field must not be empty |
| `minLength` | number | Minimum character length |
| `maxLength` | number | Maximum character length |
| `min` | number | Minimum numeric value |
| `max` | number | Maximum numeric value |
| `pattern` | string | Regular expression pattern the value must match |
| `email` | — | Value must be a valid email address |

When validation fails, the renderer SHOULD display the `message` text near the corresponding form field. Validation SHOULD be triggered on form submission and MAY also be triggered on field blur.

## Data Binding

### Model Path

The `model` field on a Component establishes two-way data binding between the component and the `data` object in the Chat UI Object. The value of `model` is a **dot-separated path** relative to the `data` object.

**Path resolution rules:**

1. Paths MUST be dot-separated strings (e.g., `"form.username"`)
2. Each segment represents a property key in the `data` object
3. The path root is always the `data` object — a model value of `"form.username"` resolves to `data.form.username`
4. Array elements are accessed using numeric segments (e.g., `"items.0.name"` resolves to `data.items[0].name`)
5. If the resolved path does not exist in `data`, the bound value SHOULD be treated as `undefined` (or the component's default)
6. Renderers MUST create intermediate objects along the path when a user modifies a bound value
7. Path segments MUST NOT contain dot (`.`) characters. Property keys containing dots are not supported
8. Path segments MUST match the regular expression `[a-zA-Z_][a-zA-Z0-9_]*` or be non-negative integers (for array access)
9. Paths MUST NOT traverse prototype chains. Only own properties of the `data` object hierarchy are accessible

**Example:**

```json
{
  "data": {
    "form": {
      "username": "",
      "tags": ["vip"]
    }
  },
  "body": {
    "type": "input",
    "model": "form.username"
  }
}
```

In this example, the input component binds to `data.form.username`. User edits update `data.form.username`, and changes to `data.form.username` update the displayed value.

## Styling

### Style Object

Styles use CSS properties in camelCase notation. Renderers MUST support the following core properties. Additional CSS properties MAY be supported.

**Core properties (MUST support):**

| Category | Properties |
|----------|-----------|
| Sizing | `width`, `height`, `minWidth`, `minHeight`, `maxWidth`, `maxHeight` |
| Spacing | `padding`, `paddingTop`, `paddingRight`, `paddingBottom`, `paddingLeft`, `margin`, `marginTop`, `marginRight`, `marginBottom`, `marginLeft` |
| Flexbox | `display` (`flex`, `block`, `none`), `flexDirection`, `justifyContent`, `alignItems`, `flexWrap`, `flex`, `gap` |
| Typography | `fontSize`, `fontWeight`, `color`, `textAlign`, `lineHeight` |
| Background | `backgroundColor` |
| Border | `border`, `borderRadius`, `borderColor`, `borderWidth`, `borderStyle` |
| Overflow | `overflow` |
| Position | `position`, `top`, `right`, `bottom`, `left` |
| Other | `opacity` |

**Example:**

```json
{
  "style": {
    "width": "100%",
    "height": "auto",
    "padding": "16px",
    "margin": "8px",
    "backgroundColor": "#ffffff",
    "borderRadius": "8px",
    "fontSize": "14px",
    "color": "#333333",
    "display": "flex",
    "flexDirection": "row",
    "justifyContent": "center",
    "alignItems": "center"
  }
}
```

Values MUST be strings. Numeric values MUST include units (e.g., `"16px"`, `"1.5em"`) except for unitless properties like `opacity` and `flex`.


## Example

Complete example of a login form:

```json
{
  "version": "1.0",
  "data": {
    "form": {
      "username": "",
      "password": ""
    }
  },
  "body": {
    "type": "form",
    "id": "loginForm",
    "props": {
      "name": "loginForm",
      "layout": "vertical"
    },
    "children": [
      {
        "type": "text",
        "props": { "text": "Login" },
        "style": { "fontSize": "24px", "marginBottom": "24px" }
      },
      {
        "type": "form-item",
        "id": "usernameItem",
        "props": {
          "label": "Username",
          "name": "username",
          "required": true
        },
        "children": [
          {
            "type": "input",
            "id": "usernameInput",
            "props": { "placeholder": "Enter username" },
            "model": "form.username",
            "style": { "marginBottom": "16px" }
          }
        ]
      },
      {
        "type": "form-item",
        "id": "passwordItem",
        "props": {
          "label": "Password",
          "name": "password",
          "required": true
        },
        "children": [
          {
            "type": "input",
            "props": {
              "placeholder": "Enter password",
              "type": "password"
            },
            "model": "form.password",
            "style": { "marginBottom": "24px" }
          }
        ]
      },
      {
        "type": "button",
        "id": "submitButton",
        "props": {
          "text": "Sign In",
          "variant": "primary",
          "actionUrl": "/api/login"
        }
      }
    ]
  }
}
```

## Security

Renderers MUST follow these security requirements to prevent common vulnerabilities:

### Content Sanitization

- **Markdown rendering**: When the `text` component's `format` is `markdown`, the renderer MUST sanitize the output HTML. Raw HTML tags within markdown content MUST be escaped or stripped. Only safe markdown constructs (headings, lists, bold, italic, links, code blocks, images) SHOULD be rendered.
- **User-generated text**: All text content bound via `model` MUST be escaped before rendering to prevent XSS attacks.

### URL Validation

- The `actionUrl`, `link.href`, and `image.src` properties MUST only accept the following URL schemes: `http`, `https`, and relative paths (starting with `/`).
- URLs with `javascript:`, `data:`, `vbscript:`, or other executable schemes MUST be rejected by the renderer.

### Same-Origin Policy

- `actionUrl` values SHOULD be same-origin or explicitly allowed by the server's CORS policy.
- Renderers MUST NOT bypass same-origin restrictions when sending requests via `actionUrl`.

### Template Interpolation

- Template paths (`{{path}}`) MUST be resolved using the same rules and restrictions as `model` paths.
- Renderers MUST escape the resolved values before inserting them into the DOM to prevent XSS attacks.

### Input Handling

- Renderers MUST NOT execute any script content received from data binding or user input.
- Form data sent via `actionUrl` SHOULD be transmitted over HTTPS in production environments.

## WeChat Mini-Program Compatibility

The specification maps directly to WeChat mini-program components:

| chatui | WeChat |
|---------|--------|
| view | view |
| scroll-view | scroll-view |
| text | text |
| image | image |
| button | button |
| input | input |
| textarea | textarea |
| checkbox | checkbox |
| radio | radio-group + radio |
| select | picker |
| switch | switch |
| link | navigator |
| table | custom component |
| form | form |
| form-item | custom component |

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-06-01 | Initial specification |
