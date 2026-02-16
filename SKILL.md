---
name: sapui5-fiori
description: >
  Comprehensive SAPUI5/Fiori development skill covering project structure,
  naming conventions, MVC best practices, BaseController pattern, formatter
  usage, routing, i18n, OData integration, and UI5 CLI tooling.
  Use when creating, extending, or reviewing any SAP UI5 / Fiori freestyle
  application. Covers both classic SAPUI5 (AMD/sap.ui.define) and modern
  approaches compatible with UI5 Tooling v4.
metadata:
  version: 1.0.0
  lastUpdated: 2026-02-16
  officialDocs:
    - https://ui5.sap.com
    - https://experience.sap.com/fiori-design-web/
    - https://ui5.github.io/cli/stable/
---

# SAP UI5 / Fiori Development Skill

## Table of Contents
- [Project Naming & Application ID](#project-naming--application-id)
- [Folder Structure](#folder-structure)
- [Fiori Launchpad Integration (App Alias)](#fiori-launchpad-integration-app-alias)
- [manifest.json Essentials](#manifestjson-essentials)
- [Component.js](#componentjs)
- [MVC Conventions](#mvc-conventions)
- [BaseController Pattern](#basecontroller-pattern)
- [Formatters](#formatters)
- [Routing & Navigation](#routing--navigation)
- [i18n & Resource Model](#i18n--resource-model)
- [Models & Data Binding](#models--data-binding)
- [OData Integration](#odata-integration)
  - [OData v2: Read & Bind Patterns](#odata-v2-read--bind-patterns)
  - [OData v4 / RAP: Read & Bind Patterns](#odata-v4--rap-read--bind-patterns)
  - [OData v4 / RAP: Draft Handling](#odata-v4--rap-draft-handling-edit--activate--discard)
  - [OData v4 / RAP: Actions and Function Imports](#odata-v4--rap-actions-and-function-imports)
  - [OData v4 / RAP: Deep Entity Navigation](#odata-v4--rap-deep-entity-navigation-expand)
- [Fragments](#fragments)
- [JS Variable & Function Naming](#js-variable--function-naming)
- [XML View Conventions](#xml-view-conventions)
- [Performance Best Practices](#performance-best-practices)
- [UI5 CLI Tooling Reference](#ui5-cli-tooling-reference)

---

## Project Naming & Application ID

- **Project name**: lowercase alphanumeric, max 15 characters (ABAP Repository limit), no hyphens.  
  Example: `zpurchorders`
- **Namespace (reverse-DNS)**: 3 segments, all lowercase, descriptive.  
  Pattern: `<company>.<module>.<appname>`  
  Example: `com.relacon.purchorders`
- **ABAP BSP application name**: same as project name → makes SICF lookups consistent.
- **Git repository name**: lowercase alphanumeric + hyphens, max 30 chars.
- Keep namespace segments short but readable. Avoid abbreviations that are not universally understood.

---

## Folder Structure

Standard `webapp/` directory layout for a Fiori freestyle app:

```
webapp/
├── controller/
│   ├── BaseController.js          ← NOT BaseController.controller.js
│   ├── App.controller.js
│   ├── Main.controller.js
│   └── Detail.controller.js
├── view/
│   ├── App.view.xml
│   ├── Main.view.xml
│   └── Detail.view.xml
├── fragment/
│   └── FilterDialog.fragment.xml
├── model/
│   ├── formatter.js               ← all formatter functions
│   └── models.js                  ← model factory helpers (optional)
├── i18n/
│   ├── i18n.properties            ← default (English)
│   └── i18n_de.properties         ← German translation
├── css/
│   └── style.css
├── localService/                  ← mock data for local testing
│   ├── mockdata/
│   └── metadata.xml
├── Component.js
└── manifest.json
```

**Rules:**
- One view per controller, same base name: `Main.view.xml` ↔ `Main.controller.js`
- `BaseController.js` has no `.controller.` infix — it is never directly referenced by a view.
- Fragments go in `fragment/` (not inside `view/`).
- Formatters always go in `model/formatter.js` — never inline in controllers.
- Models factory logic (createDeviceModel, createODataModel) goes in `model/models.js`.

---

## Fiori Launchpad Integration (App Alias)

The **semantic object + action** pair is the Fiori Launchpad identity of the app.

| Field | Convention | Example |
|---|---|---|
| Semantic Object | PascalCase business entity | `PurchaseOrder` |
| Action | camelCase verb | `display`, `manage`, `create` |
| App Alias (tile) | `<SemanticObject>-<action>` | `PurchaseOrder-display` |
| Technical ID (manifest) | reverse-DNS namespace | `com.relacon.purchorders` |

- Semantic objects map to real business entities — reuse SAP standard semantic objects where possible.
- Actions must be lowercase.
- App alias must be unique across all apps in the Fiori Launchpad catalog.
- App title and subtitle always from `i18n` — never hard-coded in manifest.

---

## manifest.json Essentials

```json
{
  "_version": "1.59.0",
  "sap.app": {
    "id": "com.relacon.purchorders",
    "type": "application",
    "title": "{{appTitle}}",
    "description": "{{appDescription}}",
    "applicationVersion": { "version": "1.0.0" },
    "dataSources": {
      "mainService": {
        "uri": "/sap/opu/odata/sap/ZPURCHORDER_SRV/",
        "type": "OData",
        "settings": { "odataVersion": "2.0" }
      }
    }
  },
  "sap.ui": {
    "technology": "UI5",
    "deviceTypes": { "desktop": true, "tablet": true, "phone": true }
  },
  "sap.ui5": {
    "rootView": {
      "viewName": "com.relacon.purchorders.view.App",
      "type": "XML",
      "async": true,
      "id": "app"
    },
    "dependencies": {
      "minUI5Version": "1.120.0",
      "libs": {
        "sap.ui.core": {},
        "sap.m": {},
        "sap.ui.layout": {}
      }
    },
    "models": {
      "": {
        "dataSource": "mainService",
        "preload": true,
        "settings": { "defaultBindingMode": "TwoWay" }
      },
      "i18n": {
        "type": "sap.ui.model.resource.ResourceModel",
        "settings": { "bundleName": "com.relacon.purchorders.i18n.i18n" }
      }
    },
    "routing": { }
  }
}
```

**Rules:**
- `appTitle` and `appDescription` use `{{key}}` placeholders — resolved from `i18n.properties`.
- Always declare all libraries used in `dependencies.libs` — do not rely on implicit loading.
- Pin `minUI5Version` to the minimum version your app requires.
- The default (unnamed) model `""` is the OData model.
- The i18n model must always be named `"i18n"`.

---

## Component.js

```javascript
sap.ui.define([
    "sap/ui/core/UIComponent",
    "sap/ui/Device",
    "com/relacon/purchorders/model/models"
], function (UIComponent, Device, models) {
    "use strict";

    return UIComponent.extend("com.relacon.purchorders.Component", {

        metadata: {
            manifest: "json"
        },

        init: function () {
            // call the base component's init function — ALWAYS first
            UIComponent.prototype.init.apply(this, arguments);

            // set device model
            this.setModel(models.createDeviceModel(), "device");

            // initialize the router — ALWAYS last in init
            this.getRouter().initialize();
        },

        destroy: function () {
            UIComponent.prototype.destroy.apply(this, arguments);
        }
    });
});
```

**Rules:**
- `UIComponent.prototype.init.apply(this, arguments)` must be the **first** line in `init`.
- `this.getRouter().initialize()` must be the **last** line in `init` — never call it before models are set.
- Never put business logic in `Component.js` — it is bootstrapping only.
- Do not use `createContent()` — routing-based apps use `rootView` in manifest instead.

---

## MVC Conventions

- **Always use XML Views** — JS views are obsolete (< 2% usage in modern apps).
- One view ↔ one controller, same base name.
- No business logic in views — views are declaration only.
- No DOM manipulation — always use UI5 APIs (`byId`, `bindElement`, etc.), never `document.getElementById`.
- Lifecycle hooks to use:

| Hook | Purpose |
|---|---|
| `onInit` | One-time initialization, model setup, router subscription |
| `onBeforeRendering` | Pre-render logic (e.g., disable fields based on settings) |
| `onAfterRendering` | Post-render DOM-dependent logic |
| `onExit` | Clean up: detach event handlers, destroy models |

---

## BaseController Pattern

Every controller **must** extend `BaseController`, never `sap/ui/core/mvc/Controller` directly.

**`webapp/controller/BaseController.js`**:

```javascript
sap.ui.define([
    "sap/ui/core/mvc/Controller",
    "sap/ui/core/routing/History",
    "sap/ui/core/UIComponent"
], function (Controller, History, UIComponent) {
    "use strict";

    return Controller.extend("com.relacon.purchorders.controller.BaseController", {

        /**
         * Convenience method - get the component router.
         * Usage: this.getRouter()
         */
        getRouter: function () {
            return UIComponent.getRouterFor(this);
        },

        /**
         * Convenience method - get a named (or default) model from the view.
         * Usage: this.getModel()  /  this.getModel("i18n")
         */
        getModel: function (sName) {
            return this.getView().getModel(sName);
        },

        /**
         * Convenience method - set a model on the view.
         * Usage: this.setModel(oModel, "view")
         */
        setModel: function (oModel, sName) {
            return this.getView().setModel(oModel, sName);
        },

        /**
         * Convenience method - get i18n resource bundle.
         * Usage: this.getResourceBundle().getText("myKey")
         */
        getResourceBundle: function () {
            return this.getOwnerComponent().getModel("i18n").getResourceBundle();
        },

        /**
         * Back navigation with browser history support.
         * Usage: set navButtonPress=".onNavBack" in XML view.
         */
        onNavBack: function () {
            var sPreviousHash = History.getInstance().getPreviousHash();
            if (sPreviousHash !== undefined) {
                window.history.go(-1);
            } else {
                this.getRouter().navTo("home", {}, true);
            }
        }
    });
});
```

**Feature controllers extending BaseController:**

```javascript
sap.ui.define([
    "com/relacon/purchorders/controller/BaseController"
], function (BaseController) {
    "use strict";

    return BaseController.extend("com.relacon.purchorders.controller.Main", {

        onInit: function () {
            var oRouter = this.getRouter();
            oRouter.getRoute("main").attachMatched(this._onRouteMatched, this);
        },

        _onRouteMatched: function (oEvent) {
            // handle route match
        }
    });
});
```

**Rules:**
- All public event handlers use `on` prefix: `onPress`, `onSearch`, `onNavBack`.
- All private functions use `_` prefix: `_loadData`, `_onRouteMatched`.
- Never pass anonymous (naked) functions to event handlers — always use named object functions.

---

## Formatters

All formatter functions live in **`webapp/model/formatter.js`** — a single module, globally available.

```javascript
sap.ui.define([], function () {
    "use strict";

    return {

        /**
         * Formats a status code to a semantic ValueState.
         * @param {string} sStatus - "A" | "B" | "C"
         * @returns {string} sap.ui.core.ValueState
         */
        statusToState: function (sStatus) {
            var mStates = {
                "A": "Success",
                "B": "Warning",
                "C": "Error"
            };
            return mStates[sStatus] || "None";
        },

        /**
         * Formats a boolean to a visible/invisible state.
         * @param {boolean} bValue
         * @returns {boolean}
         */
        boolToVisible: function (bValue) {
            return !!bValue;
        }
    };
});
```

**Using formatters in XML Views:**

```xml
<mvc:View
    controllerName="com.relacon.purchorders.controller.Main"
    xmlns:mvc="sap.ui.core.mvc"
    xmlns="sap.m">

    <!-- Reference formatter at view level -->
    <Page title="{i18n>pageTitle}">
        <ObjectStatus
            state="{
                path: 'Status',
                formatter: '.formatter.statusToState'
            }"
        />
    </Page>
</mvc:View>
```

And in the controller, import formatter as a dependency:

```javascript
sap.ui.define([
    "com/relacon/purchorders/controller/BaseController",
    "com/relacon/purchorders/model/formatter"
], function (BaseController, formatter) {
    "use strict";

    return BaseController.extend("com.relacon.purchorders.controller.Main", {
        formatter: formatter,   // expose to XML view via .formatter.*
        ...
    });
});
```

**Rules:**
- Formatters are always one-way — they convert model → view only.
- Never modify model data inside a formatter.
- Never put complex business logic in a formatter — it is display/format conversion only.
- Always import formatter module as a controller dependency and expose it as `this.formatter`.

---

## Routing & Navigation

Always configure routing in `manifest.json`, never by calling `new sap.m.routing.Router()` manually.

**manifest.json routing section:**

```json
"routing": {
    "config": {
        "routerClass": "sap.m.routing.Router",
        "viewType": "XML",
        "viewPath": "com.relacon.purchorders.view",
        "controlId": "app",
        "controlAggregation": "pages",
        "async": true,
        "bypassed": {
            "target": "notFound"
        }
    },
    "routes": [
        {
            "pattern": "",
            "name": "home",
            "target": "home"
        },
        {
            "pattern": "orders/{orderId}",
            "name": "orderDetail",
            "target": "orderDetail"
        }
    ],
    "targets": {
        "home": {
            "viewName": "Main",
            "viewLevel": 1,
            "transition": "slide"
        },
        "orderDetail": {
            "viewName": "Detail",
            "viewLevel": 2,
            "transition": "slide"
        },
        "notFound": {
            "viewName": "NotFound",
            "viewLevel": 3
        }
    }
}
```

**Navigation in controller:**

```javascript
// Navigate to a route with parameters
this.getRouter().navTo("orderDetail", {
    orderId: encodeURIComponent(sOrderId)
});

// Navigate and replace history (no back button)
this.getRouter().navTo("home", {}, true);
```

**Attach route matched in controller:**

```javascript
onInit: function () {
    // Preferred: attach to specific route only
    this.getRouter()
        .getRoute("orderDetail")
        .attachMatched(this._onRouteMatched, this);
},

_onRouteMatched: function (oEvent) {
    var sOrderId = decodeURIComponent(
        oEvent.getParameter("arguments").orderId
    );
    this._loadOrder(sOrderId);
}
```

**Rules:**
- Route names are camelCase: `orderDetail`, `home`, `notFound`.
- Target names match route names exactly.
- `viewLevel` must increase as you drill deeper (controls the back-button animation).
- Always `encodeURIComponent` / `decodeURIComponent` for route parameters.
- Use `attachMatched` on a specific route (not `attachRouteMatched` on the router) to avoid firing on every route change.
- Always call `this.getRouter().initialize()` at the end of `Component.js init()`.
- The `onNavBack` helper in `BaseController` handles back navigation with proper history fallback.

---

## i18n & Resource Model

File location: `webapp/i18n/i18n.properties` (default, English).  
Additional locales: `i18n_de.properties`, `i18n_fr.properties`, etc.

**i18n.properties format:**

```properties
# ── Application ──────────────────────────────────────────────
#XTIT: Browser tab title
appTitle=Purchase Orders

#YDES: Application description shown in Fiori Launchpad
appDescription=Manage purchase orders

# ── Main View ─────────────────────────────────────────────────
#XTIT: Page title on main view
mainPageTitle=Purchase Orders

#XBUT: Confirm button label
btnConfirm=Confirm

#XMSG: Success message after saving
msgSaveSuccess=Order {0} saved successfully.

#XMSG: Error message when order not found
msgOrderNotFound=Order {0} could not be found.
```

**Annotation prefixes (ABAP-style, SAP convention):**

| Prefix | Meaning |
|---|---|
| `#XTIT` | Title |
| `#XBUT` | Button label |
| `#XLBL` | Field label |
| `#XMSG` | Message |
| `#YDES` | Description (longer text) |
| `#XFLD` | Input field placeholder |

**Usage in controllers (via BaseController):**

```javascript
// Simple text
var sTitle = this.getResourceBundle().getText("mainPageTitle");

// Text with placeholder
var sMsg = this.getResourceBundle().getText("msgSaveSuccess", [sOrderId]);
```

**Usage in XML Views:**

```xml
<Page title="{i18n>mainPageTitle}">
    <Button text="{i18n>btnConfirm}" press=".onConfirm"/>
</Page>
```

**Rules:**
- All static UI texts must use i18n — never hard-code UI strings.
- App title and description in manifest.json use `{{appTitle}}` double-brace syntax.
- Use placeholder notation `{0}`, `{1}` for dynamic values — never string concatenation.
- The i18n model must always be set as named model `"i18n"` at component level (via manifest models section).

---

## Models & Data Binding

**Named model convention:**

| Model Name | Type | Purpose |
|---|---|---|
| `""` (default) | ODataModel v2/v4 | Main backend data |
| `"i18n"` | ResourceModel | Translations |
| `"view"` | JSONModel | Local view state (UI flags, loading indicators) |
| `"device"` | JSONModel | `sap.ui.Device` properties |

**View model pattern (local state):**

```javascript
onInit: function () {
    var oViewModel = new sap.ui.model.json.JSONModel({
        busy: false,
        editable: false,
        itemCount: 0
    });
    this.setModel(oViewModel, "view");
}
```

In XML:
```xml
<Page busy="{view>/busy}">
    <Toolbar>
        <Button text="Edit" visible="{= !${view>/editable} }" press=".onEdit"/>
    </Toolbar>
</Page>
```

**Binding mode:**

| Model | Default Binding Mode |
|---|---|
| JSONModel | TwoWay |
| ODataModel v2 | OneWay (must explicitly set TwoWay for editable fields) |
| ResourceModel | OneTime |

**Rules:**
- Never use `sap.ui.getCore().setModel()` for app-level models — set them via `Component.js` or manifest.
- Always use named models — except the default OData model.
- Do not chain more than 3 dots in a single expression (readability rule): break into variables.
- Use `oModel.createKey()` to generate OData key paths — never build key strings manually.

---

## OData Integration

This section covers both **OData v2** (classic Gateway / SEGW services) and **OData v4** (RAP-based services). Choose the model version based on what the backend exposes — they are not interchangeable.

| | OData v2 | OData v4 (RAP) |
|---|---|---|
| Model class | `sap.ui.model.odata.v2.ODataModel` | `sap.ui.model.odata.v4.ODataModel` |
| Service URL pattern | `/sap/opu/odata/sap/ZSERVICE_SRV/` | `/sap/opu/odata4/sap/zservice/default/sap/zservice/0001/` |
| Data access | `oModel.read()`, `oModel.createKey()`, `bindElement` | Context API: `oContext.requestObject()`, `oContext.setProperty()` |
| Batch | Deferred groups, `oModel.submitChanges()` | `$$groupId`, `oModel.submitBatch()` |
| CRUD | Model-level methods | Context-level methods (`oContext.delete()`, `oList.create()`) |
| Draft | Manual implementation | Built-in with RAP draft actions (Edit/Activate/Discard) |
| Test tool | `/n/IWFND/GW_CLIENT` | `/n/IWFND/GW_CLIENT` or Postman with CSRF token |

---

### Manifest: Declaring Data Sources

**OData v2:**
```json
"dataSources": {
    "mainService": {
        "uri": "/sap/opu/odata/sap/ZPURCHORDER_SRV/",
        "type": "OData",
        "settings": { "odataVersion": "2.0" }
    }
}
```

**OData v4 (RAP):**
```json
"dataSources": {
    "mainService": {
        "uri": "/sap/opu/odata4/sap/zpurchorder/default/sap/zpurchorder/0001/",
        "type": "OData",
        "settings": { "odataVersion": "4.0" }
    }
}
```

**Model declaration in `sap.ui5.models` (v4 specific settings):**
```json
"models": {
    "": {
        "dataSource": "mainService",
        "preload": true,
        "settings": {
            "autoExpandSelect": true,
            "operationMode": "Server",
            "synchronizationMode": "None"
        }
    }
}
```
- `autoExpandSelect: true` — model automatically generates `$select` and `$expand` based on bound controls. Recommended for RAP.
- `synchronizationMode: "None"` — required parameter for v4, no exceptions.
- `operationMode: "Server"` — all filtering/sorting done server-side.

---

### Local Proxy (both v2 and v4, in `ui5.yaml`)

```yaml
server:
  customMiddleware:
    - name: fiori-tools-proxy
      afterMiddleware: compression
      configuration:
        backend:
          - path: /sap
            url: http://your-s4-host:8000
```

---

### OData v2: Read & Bind Patterns

**Object page — bind a single entity:**
```javascript
_loadOrder: function (sOrderId) {
    var oModel = this.getModel();
    // Always use createKey() - never build path strings manually
    var sPath = oModel.createKey("/PurchaseOrderSet", { OrderId: sOrderId });

    this.getView().bindElement({
        path: sPath,
        parameters: {
            expand: "to_Items,to_Partner"
        },
        events: {
            dataRequested: function () {
                this.getModel("view").setProperty("/busy", true);
            }.bind(this),
            dataReceived: function (oEvent) {
                this.getModel("view").setProperty("/busy", false);
                if (oEvent.getParameter("error")) {
                    // handle error
                }
            }.bind(this)
        }
    });
}
```

**List — filter and read:**
```javascript
_applyFilter: function (sStatus) {
    var oList = this.byId("orderList");
    var oBinding = oList.getBinding("items");
    var aFilters = [new sap.ui.model.Filter("Status", sap.ui.model.FilterOperator.EQ, sStatus)];
    oBinding.filter(aFilters);
}
```

**Write — update and submit:**
```javascript
onSave: function () {
    var oModel = this.getModel();
    // v2 collects all pending changes and submits in one batch
    oModel.submitChanges({
        success: function () {
            sap.m.MessageToast.show(this.getResourceBundle().getText("msgSaveSuccess"));
        }.bind(this),
        error: function (oError) {
            // handle error
        }
    });
},

onCancel: function () {
    this.getModel().resetChanges();
}
```

**v2 Rules:**
- Always use `oModel.createKey()` to build entity paths — never concatenate strings manually.
- Always handle both `dataRequested` and `dataReceived` (including error parameter) for busy state.
- Use `oModel.submitChanges()` / `oModel.resetChanges()` for transactional save/cancel.
- Never manually set the CSRF token — the v2 model fetches and manages it automatically.
- Test all services in `/n/IWFND/GW_CLIENT` before building the UI layer.

---

### OData v4 / RAP: Read & Bind Patterns

In v4 the **Context** is the central object — not the model. All data access and modifications go through `oContext`.

**Object page — bind a single entity:**
```javascript
_loadOrder: function (sOrderId) {
    // v4: path with literal key value, no createKey() needed
    var sPath = "/PurchaseOrderSet('" + sOrderId + "')";

    this.getView().bindElement({
        path: sPath,
        parameters: {
            $expand: "to_Items($select=ItemNo,Material,Quantity),to_Partner",
            $select: "OrderId,Status,TotalAmount,Currency"
        },
        events: {
            dataReceived: function (oEvent) {
                this.getModel("view").setProperty("/busy", false);
                if (oEvent.getParameter("error")) {
                    // handle error
                }
            }.bind(this)
        }
    });
}
```

**List binding with $expand in XML view (recommended — let autoExpandSelect handle it):**
```xml
<Table
    id="orderTable"
    items="{
        path: '/PurchaseOrderSet',
        parameters: {
            $orderby: 'OrderId desc',
            $$groupId: '$auto'
        }
    }">
    <columns>...</columns>
    <items>
        <ColumnListItem press=".onItemPress" type="Navigation">
            <cells>
                <Text text="{OrderId}"/>
                <Text text="{Status}"/>
            </cells>
        </ColumnListItem>
    </items>
</Table>
```

**Programmatic read with requestContexts:**
```javascript
_readOrders: function () {
    var oModel = this.getModel();
    var oListBinding = oModel.bindList("/PurchaseOrderSet", null, null, null, {
        $select: "OrderId,Status,TotalAmount",
        $$groupId: "$auto"
    });

    oListBinding.requestContexts(0, 50).then(function (aContexts) {
        var aOrders = aContexts.map(function (oCtx) {
            return oCtx.getObject();  // returns plain JS object
        });
        this.getModel("view").setProperty("/orders", aOrders);
    }.bind(this));
}
```

**v4 Rules:**
- Use `oContext.requestObject()` (async, returns Promise) or `oContext.getObject()` (sync, from cache) for data access — never `oModel.getProperty()`.
- Use `oContext.setProperty("FieldName", value)` for direct property changes.
- `$$groupId: "$auto"` sends requests automatically; `$$groupId: "myGroup"` defers until `oModel.submitBatch("myGroup")`.
- Do not chain more than 3 `.` expressions — break into variables for readability.
- `autoExpandSelect: true` in manifest is preferred over manually specifying `$select`/`$expand` on every binding — controls drive the selection automatically.

---

### OData v4 / RAP: Draft Handling (Edit / Activate / Discard)

RAP draft-enabled entities have three standard actions: `EditAction`, `ActivationAction`, `DiscardAction`. These are bound actions on the entity context.

**Edit — create a draft from active entity:**
```javascript
onEdit: function () {
    var oObjectPage = this.byId("objectPage");
    var oActiveContext = oObjectPage.getBindingContext();
    var that = this;

    // Remember active context for cancel scenario
    this._oActiveContext = oActiveContext;

    oActiveContext.getModel()
        .bindContext("com.sap.namespace.EditAction(...)", oActiveContext, {
            $$inheritExpandSelect: true  // reuse same $expand/$select as list
        })
        .invoke("$auto", false, null, /*bReplaceWithRVC*/ true)
        .then(function (oDraftContext) {
            // Switch object page to draft context
            oObjectPage.setBindingContext(oDraftContext);
            that.getModel("view").setProperty("/editable", true);
        })
        .catch(function (oError) {
            // handle error
        });
},
```

**Activate (Save) — turn draft into active entity:**
```javascript
onActivate: function () {
    var oDraftContext = this.byId("objectPage").getBindingContext();
    var that = this;

    oDraftContext.getModel()
        .bindContext("com.sap.namespace.ActivationAction(...)", oDraftContext, {
            $$inheritExpandSelect: true
        })
        .invoke("$auto")
        .then(function (oActiveContext) {
            that.byId("objectPage").setBindingContext(oActiveContext);
            that.getModel("view").setProperty("/editable", false);
            that._oActiveContext = null;
        })
        .catch(function (oError) {
            // handle activation errors (validation messages from RAP)
        });
},
```

**Discard — delete draft, restore active entity:**
```javascript
onDiscard: function () {
    var oDraftContext = this.byId("objectPage").getBindingContext();
    var oActiveContext = this._oActiveContext;
    var that = this;

    // Replace draft row in list with active entity in-situ, then delete draft
    oDraftContext.replaceWith(oActiveContext);

    oDraftContext.delete("$auto").then(function () {
        that.byId("objectPage").setBindingContext(oActiveContext);
        that.getModel("view").setProperty("/editable", false);
        that._oActiveContext = null;
    });
}
```

**Draft Rules:**
- Store the active context in `this._oActiveContext` before calling `EditAction` — needed for `Discard`.
- Always use `$$inheritExpandSelect: true` on draft action bindings — avoids duplicate `$expand` definitions.
- Use `bReplaceWithRVC: true` flag in `invoke()` for `EditAction` so the list row is updated in-place.
- Call `oDraftContext.replaceWith(oActiveContext)` before `delete()` on discard — keeps list UI consistent without a full refresh.
- RAP validation messages come back as OData error responses — always add a `.catch()` on `ActivationAction`.

---

### OData v4 / RAP: Actions and Function Imports

RAP exposes both **bound actions** (on an entity context) and **unbound actions** (on the service root).

**Bound action (e.g., Approve an order):**
```javascript
onApprove: function () {
    var oContext = this.byId("objectPage").getBindingContext();

    oContext.getModel()
        .bindContext("com.sap.namespace.ApproveAction(...)", oContext)
        .invoke("$auto")
        .then(function () {
            // Refresh side-affected fields after action
            oContext.requestSideEffects(["Status", "ApprovalDate"]);
        })
        .catch(function (oError) {
            // show error message
        });
},
```

**Unbound action (e.g., mass processing):**
```javascript
_callUnboundAction: function (aOrderIds) {
    var oModel = this.getModel();
    var oActionBinding = oModel.bindContext("/com.sap.namespace.MassApprove(...)");

    // Set action parameters
    oActionBinding.setParameter("OrderIds", aOrderIds);
    oActionBinding.setParameter("Reason", "Approved in bulk");

    return oActionBinding.invoke("$auto");
},
```

**Action Rules:**
- Bound actions: path format is `"namespace.ActionName(...)"` — the `(...)` is required.
- After an action that causes server-side changes, always call `oContext.requestSideEffects([...])` with the list of affected property paths.
- Unbound actions: bind on model directly with absolute path `/namespace.ActionName(...)`.
- Use `$$patchWithoutSideEffects: true` on bindings when you manage side effects explicitly — avoids double-loading.

---

### OData v4 / RAP: Deep Entity Navigation ($expand)

v4 `$expand` is more powerful than v2 — you can nest `$select`, `$filter`, `$top` inside expand.

**In XML view binding:**
```xml
<Table items="{
    path: '/PurchaseOrderSet',
    parameters: {
        $expand: {
            to_Items: {
                $select: 'ItemNo,Material,Quantity,UoM',
                $orderby: 'ItemNo'
            },
            to_Partner: {
                $select: 'PartnerId,PartnerName'
            }
        },
        $select: 'OrderId,Status,TotalAmount,Currency'
    }
}">
```

**Inline expand on bindElement (controller):**
```javascript
this.getView().bindElement({
    path: "/PurchaseOrderSet('" + sOrderId + "')",
    parameters: {
        $expand: "to_Items($select=ItemNo,Material,Quantity;$orderby=ItemNo),to_Partner($select=PartnerId,PartnerName)",
        $select: "OrderId,Status,TotalAmount,Currency"
    }
});
```

**Expand Rules:**
- With `autoExpandSelect: true` in manifest, you usually do NOT need to specify `$select`/`$expand` manually — UI5 derives them from bound controls. Only override when you need data not shown in the UI (e.g., for logic in the controller).
- Never expand entire entities without `$select` — this over-fetches data and slows RAP performance.
- Nested `$filter` inside `$expand` is only supported on collection navigation properties (one-to-many).
- Test expanded responses in `/n/IWFND/GW_CLIENT` or Postman before binding in UI to confirm RAP behavior.

---

## Fragments

Fragments are reusable UI parts (dialogs, popovers) — they do **not** have their own controller.

**Fragment file (`webapp/fragment/FilterDialog.fragment.xml`):**

```xml
<core:FragmentDefinition
    xmlns="sap.m"
    xmlns:core="sap.ui.core">

    <Dialog
        id="filterDialog"
        title="{i18n>filterDialogTitle}"
        afterClose=".onDialogClose">

        <content>
            <!-- content here -->
        </content>

        <beginButton>
            <Button text="{i18n>btnApply}" press=".onApplyFilter" type="Emphasized"/>
        </beginButton>
        <endButton>
            <Button text="{i18n>btnCancel}" press=".onCancelFilter"/>
        </endButton>
    </Dialog>
</core:FragmentDefinition>
```

**Loading a fragment in controller (lazy, recommended):**

```javascript
onOpenFilter: function () {
    if (!this._oFilterDialog) {
        this._oFilterDialog = this.loadFragment({
            name: "com.relacon.purchorders.fragment.FilterDialog"
        });
    }
    this._oFilterDialog.then(function (oDialog) {
        oDialog.open();
    });
},

onCancelFilter: function () {
    this.byId("filterDialog").close();
}
```

**Rules:**
- Fragments share the host controller — event handlers are defined in the host controller.
- Use `this.loadFragment()` (async, UI5 1.93+) rather than `sap.ui.xmlfragment()`.
- Lazy-load fragments — create them on first use, cache in a private property (`this._oFilterDialog`).
- Fragment IDs are scoped to the view using `this.byId()`.
- Never put business logic in fragments — they are UI layout only.

---

## JS Variable & Function Naming

Follow **Hungarian Notation** prefix convention (standard in UI5 SAP code):

| Prefix | Type | Example |
|---|---|---|
| `s` | string | `sOrderId`, `sTitle` |
| `i` | integer | `iCount`, `iIndex` |
| `f` | float | `fPrice`, `fAmount` |
| `b` | boolean | `bVisible`, `bEditable` |
| `o` | object | `oModel`, `oEvent`, `oView` |
| `a` | array | `aItems`, `aFilters` |
| `fn` | function (var) | `fnCallback`, `fnSuccess` |
| `_` | private | `_loadData`, `_oDialog` |

**Additional naming rules:**
- All event handler functions: `on` prefix, camelCase → `onPress`, `onSearch`, `onRouteMatched`.
- Private helper functions: `_` prefix → `_loadOrder`, `_formatStatus`.
- Do not use `var` for module-level declarations — always inside functions.
- Always `"use strict"` at the top of every `sap.ui.define` callback.
- No global variables — everything scoped inside the AMD module function.

---

## XML View Conventions

```xml
<!-- One attribute per line when element has 3+ attributes -->
<Button
    id="confirmButton"
    text="{i18n>btnConfirm}"
    type="Emphasized"
    press=".onConfirm"/>

<!-- Single attribute - keep on same line -->
<Title text="{i18n>pageTitle}"/>

<!-- Self-closing tag when no aggregations -->
<Input value="{OrderId}"/>

<!-- Empty lines before/after aggregation blocks for readability -->
<Page title="{i18n>pageTitle}">

    <headerContent>
        <Button text="{i18n>btnEdit}" press=".onEdit"/>
    </headerContent>

    <content>
        <List items="{/OrderSet}">
            <items>
                <StandardListItem title="{OrderId}" type="Navigation" press=".onItemPress"/>
            </items>
        </List>
    </content>

</Page>
```

**Rules:**
- Always use XML views — never JS, JSON, or HTML views.
- Namespace prefixes: `xmlns="sap.m"` as default, others prefixed: `xmlns:core="sap.ui.core"`.
- All text-facing properties must use `{i18n>key}` binding.
- Event handlers use `.onHandlerName` (dot prefix means "in this controller").
- Self-close tags with no aggregations: `/>` without space before slash.
- IDs use camelCase: `id="confirmButton"`.

---

## Performance Best Practices

- **Async views**: set `"async": true` in manifest `rootView` and routing targets.
- **Component preloading**: ensure `Component-preload.js` is generated (`ui5 build --all`).
- **Lazy loading**: load fragments and rarely-used modules on demand with `sap.ui.require()`.
- **Growing list**: use `growing="true" growingThreshold="20"` on lists and tables.
- **$batch**: ODataModel batches requests automatically — avoid disabling batch mode unless required.
- **No `sap.ui.getCore()`** for model access in controllers — use `this.getModel()` / `this.getOwnerComponent()`.
- **Avoid `setTimeout`** in UI5 — use lifecycle hooks or model events instead.
- **ESLint**: always run ESLint with the UI5 plugin before committing.

---

## UI5 CLI Tooling Reference

**Node.js**: v20.11.0+ or v22.0.0+ required.

### Project initialization

```bash
npm init --yes
ui5 init
ui5 use sapui5@1.120.0        # always pin version for production
ui5 add sap.ui.core sap.m sap.ui.layout themelib_sap_fiori_3
```

### Development

```bash
ui5 serve                     # start dev server on http://localhost:8080
ui5 serve --open index.html   # open browser automatically
```

### Build

```bash
ui5 build --all --clean-dest  # production build, clean output first
```

### Minimal `ui5.yaml`

```yaml
specVersion: "4.0"
type: application
metadata:
  name: com.relacon.purchorders
framework:
  name: SAPUI5
  version: "1.120.0"
  libraries:
    - name: sap.ui.core
    - name: sap.m
    - name: sap.ui.layout
    - name: themelib_sap_fiori_3
      optional: true
```

### Proxy middleware for local OData (add to ui5.yaml)

```yaml
server:
  customMiddleware:
    - name: fiori-tools-proxy
      afterMiddleware: compression
      configuration:
        backend:
          - path: /sap
            url: http://your-s4-host:8000
```

**Rules:**
- Always use local `@ui5/cli` installation (`--save-dev`), not global.
- Always commit `ui5.yaml` and `package.json` to version control.
- Use `ui5 serve` during development — never `ui5 build` for testing.
- Use `ui5 build --clean-dest --all` for production deployments only.
- All modules must use `sap.ui.define` format — no ES `import/export`.

---

## Quick Reference: What Goes Where

| Concern | Location |
|---|---|
| App bootstrapping, routing init | `Component.js` |
| App descriptor, models, routing config | `manifest.json` |
| Shared controller helpers (router, model, i18n, navBack) | `BaseController.js` |
| Business / feature logic | `<ViewName>.controller.js` |
| Display formatting | `model/formatter.js` |
| Device / view state models | `model/models.js` |
| All UI texts | `i18n/i18n.properties` |
| Reusable UI parts (dialogs, popovers) | `fragment/*.fragment.xml` |
| Global styles | `css/style.css` |
| Mock data for local testing | `localService/mockdata/` |

---


## OData v2 vs v4 Cheat Sheet

| Task | OData v2 | OData v4 (RAP) |
|---|---|---|
| Build entity path | `oModel.createKey("/Set", {Key: val})` | `"/Set('" + val + "')"` |
| Bind object page | `this.getView().bindElement({path, events})` | `this.getView().bindElement({path, parameters: {$expand, $select}})` |
| Read property | `oModel.getProperty(path)` | `oContext.getObject()` / `oContext.requestObject()` |
| Set property | `oModel.setProperty(path, val)` | `oContext.setProperty("Field", val)` |
| Save changes | `oModel.submitChanges()` | `oModel.submitBatch("groupId")` |
| Cancel changes | `oModel.resetChanges()` | `oModel.resetChanges(["groupId"])` |
| Call action/function | `oModel.callFunction("/ActionImport", {...})` | `oModel.bindContext("/ns.Action(...)").invoke()` |
| Bound action | N/A (use function imports) | `oModel.bindContext("ns.Action(...)", oContext).invoke()` |
| Side effects after change | Manual re-read | `oContext.requestSideEffects(["Field1","Field2"])` |
| Draft: start edit | Manual | `oModel.bindContext("ns.EditAction(...)", oCtx).invoke()` |
| Draft: activate/save | Manual | `oModel.bindContext("ns.ActivationAction(...)", oCtx).invoke()` |
| Draft: discard | Manual delete | `oDraftCtx.replaceWith(oActiveCtx)` + `oDraftCtx.delete()` |

*References: SAP UI5 SDK (ui5.sap.com), DSAG UI5 Best Practice Guide, SAP Fiori Design Guidelines, UI5 Tooling v4 documentation.*
