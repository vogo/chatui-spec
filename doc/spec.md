# vjsonui Specification

A JSON schema UI specification designed to be simple, easy to render, and adaptive to WeChat mini-programs.

## Overview

vjsonui defines a declarative JSON format for describing user interfaces. The specification prioritizes:

- **Simplicity**: Minimal learning curve with intuitive structure
- **Portability**: Renders consistently across web, mobile, and WeChat mini-programs
- **Extensibility**: Easy to add custom components while maintaining compatibility

## Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Schema Structure

### Root Object

```json
{
  "version": "1.0",
  "root": { /* Component */ },
  "data": {}
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| version | string | REQUIRED | Specification version |
| root | Component | REQUIRED | Root component of the UI tree |
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
  "children": []
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| type | string | REQUIRED | Component type identifier |
| id | string | OPTIONAL | Unique identifier for the component |
| props | object | OPTIONAL | Component-specific properties |
| model | string | OPTIONAL | Path to data model for two-way binding |
| style | object | OPTIONAL | Styling properties |
| children | array | OPTIONAL | Child components |

## Components

### composite

A layout container that holds multiple children components. Used to group related UI elements together.

```json
{
  "type": "composite",
  "id": "userInfo",
  "props": {
    "direction": "vertical" | "horizontal",
    "gap": "8px"
  },
  "children": [
    { "type": "text", "props": { "content": "Name:" } },
    { "type": "input", "props": { "placeholder": "Enter name" } },
    { "type": "button", "props": { "text": "Save" } }
  ]
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| direction | string | OPTIONAL | Layout direction: `vertical` (default) or `horizontal` |
| gap | string | OPTIONAL | Spacing between children |

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

| Field | Type | Required | Description |
|------|------|----------|-------------|
| props.columns | array | REQUIRED | Column definitions |
| props.columns[].key | string | REQUIRED | Data field key |
| props.columns[].title | string | REQUIRED | Column header text |
| props.columns[].width | string | OPTIONAL | Column width |
| props.bordered | boolean | OPTIONAL | Show table borders |
| props.striped | boolean | OPTIONAL | Alternate row colors |
| props.hoverable | boolean | OPTIONAL | Highlight row on hover |
| model | string | OPTIONAL | Path to data model for two-way binding |

### form

Container for form elements with validation.

```json
{
  "type": "form",
  "id": "loginForm",
  "props": {
    "name": "loginForm",
    "layout": "vertical" | "horizontal" | "inline",
    "labelWidth": "100px"
  },
  "children": []
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| name | string | OPTIONAL | Form identifier |
| layout | string | OPTIONAL | Form layout style |
| labelWidth | string | OPTIONAL | Label width for horizontal layout |


### button

Interactive button element with action URL for click handling.

```json
{
  "type": "button",
  "id": "submitButton",
  "props": {
    "text": "Submit",
    "variant": "primary" | "secondary" | "outline",
    "disabled": false,
    "action_url": "/api/submit",
    "action_data": { "source": "web", "version": "1.0" }
  }
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| text | string | REQUIRED | Button label text |
| variant | string | OPTIONAL | Button style variant |
| disabled | boolean | OPTIONAL | Disable button interaction |
| action_url | string | OPTIONAL | URL to call when button is clicked |
| action_data | object | OPTIONAL | Default POST data; form data SHOULD override matching fields |

> The `action_url` will POST data to the specified endpoint when clicked. The posted data is merged from `action_data` and the parent form's model, with form data taking precedence over `action_data` for overlapping fields.

### link

Navigation link for redirecting to a URL.

```json
{
  "type": "link",
  "id": "homeLink",
  "props": {
    "text": "Go to Home",
    "href": "/home",
    "target": "_self" | "_blank"
  }
}
```

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| text | string | REQUIRED | Link display text |
| href | string | REQUIRED | Target URL |
| target | string | OPTIONAL | Open behavior: `_self` (default) or `_blank` |

#### form-item

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
| name | string | REQUIRED | Field name for form data |
| required | boolean | OPTIONAL | Mark field as required |
| rules | array | OPTIONAL | Validation rules |

## Styling

### Style Object

Styles use a subset of CSS properties in camelCase:

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
  "root": {
    "type": "form",
    "id": "loginForm",
    "props": {
      "name": "loginForm",
      "layout": "vertical"
    },
    "children": [
      {
        "type": "text",
        "props": { "content": "Login" },
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
          "action_url": "/api/login"
        }
      }
    ]
  }
}
```

## WeChat Mini-Program Compatibility

The specification maps directly to WeChat mini-program components:

| vjsonui | WeChat |
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
| composite | view |
| table | custom component |
| form | form |
| form-item | custom component |

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | - | Initial specification |
