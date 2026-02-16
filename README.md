# UI5 App Skill

A comprehensive SAPUI5/Fiori development skill and coding standards guide for building enterprise-grade applications.

## Overview

This repository provides development standards, best practices, and code patterns for SAPUI5/Fiori applications. It covers:

- Project structure and naming conventions
- MVC architecture and BaseController patterns
- Formatters and data binding
- Routing and navigation
- i18n and localization
- OData v2 and v4 (RAP) integration
- UI5 CLI tooling and build processes

## Quick Start

### Prerequisites

- Node.js v20.11.0+ or v22.0.0+
- UI5 CLI (`npm install --save-dev @ui5/cli`)

### Creating a New Project

```bash
# Initialize project
npm init --yes
ui5 init
ui5 use sapui5@1.120.0
ui5 add sap.ui.core sap.m sap.ui.layout themelib_sap_fiori_3

# Start development server
ui5 serve --open index.html

# Production build
ui5 build --all --clean-dest
```

## Project Structure

```
webapp/
├── controller/
│   ├── BaseController.js          # Base controller with shared utilities
│   ├── App.controller.js          # Main app controller
│   ├── Main.controller.js         # Feature controllers
│   └── Detail.controller.js
├── view/
│   ├── App.view.xml
│   ├── Main.view.xml
│   └── Detail.view.xml
├── fragment/                      # Reusable UI fragments
│   └── FilterDialog.fragment.xml
├── model/
│   ├── formatter.js              # All formatter functions
│   └── models.js                 # Model factory helpers
├── i18n/
│   ├── i18n.properties           # Default translations
│   └── i18n_de.properties        # Additional locales
├── css/
│   └── style.css
├── localService/                  # Mock data for testing
│   ├── mockdata/
│   └── metadata.xml
├── Component.js                   # Component initialization
└── manifest.json                  # App descriptor
```

## Naming Conventions

### Project Names
- **Project name**: Lowercase alphanumeric, max 15 characters (ABAP Repository limit)
  - Example: `zpurchorders`
- **Namespace**: Reverse-DNS, 3 segments, lowercase
  - Pattern: `<company>.<module>.<appname>`
  - Example: `com.relacon.purchorders`
- **Git repository**: Lowercase with hyphens, max 30 chars

### Fiori Launchpad Integration
- **Semantic Object**: PascalCase business entity (e.g., `PurchaseOrder`)
- **Action**: camelCase verb (e.g., `display`, `manage`)
- **App Alias**: `<SemanticObject>-<action>` (e.g., `PurchaseOrder-display`)

## Core Development Patterns

### BaseController

Every controller must extend `BaseController`, never `sap/ui/core/mvc/Controller` directly.

```javascript
sap.ui.define([
    "com/example/app/controller/BaseController"
], function (BaseController) {
    "use strict";

    return BaseController.extend("com.example.app.controller.Main", {
        onInit: function () {
            // Initialize using inherited utilities
            var oRouter = this.getRouter();
            oRouter.getRoute("main").attachMatched(this._onRouteMatched, this);
        }
    });
});
```

### Formatters

All formatter functions live in `webapp/model/formatter.js`:

```javascript
sap.ui.define([], function () {
    "use strict";

    return {
        statusToState: function (sStatus) {
            var mStates = { "A": "Success", "B": "Warning", "C": "Error" };
            return mStates[sStatus] || "None";
        }
    };
});
```

### i18n (Internationalization)

All UI texts use i18n placeholders:

```properties
#XTIT: Application title
appTitle=Purchase Orders

#XBUT: Confirm button
btnConfirm=Confirm

#XMSG: Success message with parameter
msgSaveSuccess=Order {0} saved successfully.
```

Usage:
```xml
<Page title="{i18n>appTitle}">
    <Button text="{i18n>btnConfirm}"/>
</Page>
```

## OData Integration

### OData v2 (Classic Gateway)

```javascript
// Build entity path
var sPath = oModel.createKey("/PurchaseOrderSet", { OrderId: sOrderId });

// Bind element
this.getView().bindElement({
    path: sPath,
    parameters: { expand: "to_Items,to_Partner" }
});

// Submit changes
oModel.submitChanges({
    success: function () { /* handle success */ },
    error: function (oError) { /* handle error */ }
});
```

### OData v4 (RAP)

```javascript
// Context-based API
var oContext = this.byId("objectPage").getBindingContext();

// Read data
oContext.requestObject().then(function (oData) { /* use data */ });

// Update property
oContext.setProperty("Status", "Approved");

// Submit batch
oModel.submitBatch("$auto");
```

### RAP Draft Handling

```javascript
// Edit - create draft
oModel.bindContext("com.sap.namespace.EditAction(...)", oActiveContext)
    .invoke("$auto", false, null, true)
    .then(function (oDraftContext) {
        oObjectPage.setBindingContext(oDraftContext);
    });

// Activate (Save)
oModel.bindContext("com.sap.namespace.ActivationAction(...)", oDraftContext)
    .invoke("$auto");

// Discard
oDraftContext.replaceWith(oActiveContext);
oDraftContext.delete("$auto");
```

## Routing & Navigation

Configure routing in `manifest.json`:

```json
{
  "routing": {
    "config": {
      "routerClass": "sap.m.routing.Router",
      "viewType": "XML",
      "controlId": "app",
      "controlAggregation": "pages"
    },
    "routes": [
      { "pattern": "", "name": "home", "target": "home" },
      { "pattern": "orders/{orderId}", "name": "orderDetail", "target": "orderDetail" }
    ]
  }
}
```

Navigation in controller:

```javascript
// Navigate with parameters
this.getRouter().navTo("orderDetail", { orderId: encodeURIComponent(sOrderId) });

// Navigate and replace history
this.getRouter().navTo("home", {}, true);

// Back navigation (with history support)
this.onNavBack();  // Inherited from BaseController
```

## Hungarian Notation (Variable Naming)

| Prefix | Type | Example |
|--------|------|---------|
| `s` | string | `sOrderId`, `sTitle` |
| `i` | integer | `iCount`, `iIndex` |
| `f` | float | `fPrice`, `fAmount` |
| `b` | boolean | `bVisible`, `bEditable` |
| `o` | object | `oModel`, `oEvent`, `oView` |
| `a` | array | `aItems`, `aFilters` |
| `_` | private | `_loadData`, `_oDialog` |

## Performance Best Practices

- **Async views**: Set `"async": true` in manifest for rootView and routing
- **Component preloading**: Generate `Component-preload.js` with `ui5 build --all`
- **Lazy loading**: Load fragments on demand with `this.loadFragment()`
- **Growing lists**: Use `growing="true" growingThreshold="20"` on tables
- **Batch requests**: Let ODataModel batch requests automatically
- **Avoid**: `sap.ui.getCore()` for model access, `setTimeout` in UI5 code

## Documentation

For complete development standards, patterns, and detailed examples, see [SKILL.md](SKILL.md).

The SKILL.md file includes:
- Complete folder structure guidelines
- manifest.json configuration
- Component.js setup
- MVC conventions and lifecycle hooks
- Detailed BaseController implementation
- Formatter patterns and usage
- Complete routing configuration
- i18n best practices
- Model and data binding patterns
- Comprehensive OData v2 and v4/RAP integration guides
- Fragment usage patterns
- XML view conventions
- UI5 CLI tooling reference
- Quick reference tables

## OData Cheat Sheet

| Task | OData v2 | OData v4 (RAP) |
|------|----------|----------------|
| Build path | `oModel.createKey("/Set", {Key: val})` | `"/Set('" + val + "')"` |
| Read property | `oModel.getProperty(path)` | `oContext.getObject()` |
| Set property | `oModel.setProperty(path, val)` | `oContext.setProperty("Field", val)` |
| Save | `oModel.submitChanges()` | `oModel.submitBatch("groupId")` |
| Action | `oModel.callFunction("/Action", {...})` | `oModel.bindContext("/ns.Action(...)").invoke()` |

## References

- [SAP UI5 SDK](https://ui5.sap.com)
- [SAP Fiori Design Guidelines](https://experience.sap.com/fiori-design-web/)
- [UI5 CLI Documentation](https://ui5.github.io/cli/stable/)
- [DSAG UI5 Best Practice Guide](https://www.dsag.de/)

## License

This skill documentation is provided as a reference guide for SAPUI5/Fiori development.

---

**Version**: 1.0.0  
**Last Updated**: 2026-02-16
