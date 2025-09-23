---
description: Retrieve, list, and delete base or custom models across providers with unified tasks and cursor-based pagination
icon: cube
--- 

# Models

returns [`IModelData`](#getmodel--getcustommodel) or [`IModelData[]`](#listmodels--listcustommodels)

Manage model catalogs in a provider-agnostic way. This page covers:

* **GetModel** — retrieve a *base* (provider) model by ID
* **ListModels** — enumerate available *base* models (with cursor pagination)
* **GetCustomModel** — retrieve a *custom* / fine-tuned model by ID
* **ListCustomModels** — enumerate *custom* models (with cursor pagination)
* **DeleteModel** — delete a *custom* model by ID (where supported)

All calls return normalized `IModelData` objects (or arrays).

> Uses `CursorQuery` for pagination and sorting (`limit`, `order`, `after`, `before`).

---

## Quick Start (convenience form)

```csharp
using Glitch9.AIDevKit;
using Glitch9.IO.Networking.RESTApi;

// Retrieve a base model
IModelData gpt4o = await Api.OpenAI.GetModel("gpt-4o").ExecuteAsync();

// List base models (first page)
IModelData[] models = await Api.OpenAI.ListModels().ExecuteAsync();

// Paginate base models
var q = new CursorQuery { Limit = 50, Order = SortOrder.Descending };
IModelData[] page1 = await Api.OpenAI.ListModels(q).ExecuteAsync;
q.After = page1.Length > 0 ? page1[^1].Id : null;           // use last item ID as cursor
IModelData[] page2 = await Api.OpenAI.ListModels(q).ExecuteAsync();

// Retrieve a custom (fine-tuned) model
IModelData custom = await Api.OpenAI.GetCustomModel("ft:gpt-4o:org:2025-09-01").ExecuteAsync();

// List custom models
IModelData[] customList = await Api.OpenAI.ListCustomModels().ExecuteAsync();

// Delete a custom model
bool deleted = await Api.OpenAI.DeleteModel("ft:gpt-4o:org:2025-09-01").ExecuteAsync();
```

---

## GetModel / GetCustomModel

**returns** `IModelData`

```csharp
// Base model (provider catalog)
IModelData m1 = await Api.OpenAI.GetModel("gpt-4o-mini").ExecuteAsync();

// Custom model (your fine-tune / variant)
IModelData m2 = await Api.OpenAI.GetCustomModel("ft:gpt-4o-mini:org:2025-08-12").ExecuteAsync();
```

**Notes**

* Throws if `modelId` is null/empty.
* `GetModel` targets provider base catalogs; `GetCustomModel` targets user/org-scoped models.

---

## ListModels / ListCustomModels

**returns** `IModelData[]`

### Basic listing

```csharp
IModelData[] baseModels   = await Api.OpenAI.ListModels().ExecuteAsync();
IModelData[] customModels = await Api.OpenAI.ListCustomModels().ExecuteAsync();
```

### With `CursorQuery` (pagination & sort)

```csharp
using Glitch9.IO.Networking.RESTApi;

var q = new CursorQuery
{
    Limit = 100,                 // 1..100 (default 20)
    Order = SortOrder.Descending // by created_at
    // After / Before can be set to model IDs for navigation
};

IModelData[] first = await Api.OpenAI.ListModels(q).ExecuteAsync();

// Next page (forward):
q.After = first.Length > 0 ? first[^1].Id : null;
IModelData[] second = await Api.OpenAI.ListModels(q).ExecuteAsync();

// Previous page (backward):
q.Before = first.Length > 0 ? first[0].Id : null;
IModelData[] prev = await Api.OpenAI.ListModels(q).ExecuteAsync();
```

**Tips**

* Prefer **server-side pagination** (`Limit`, `After`, `Before`) over client filtering.
* `Order` accepts `SortOrder.Ascending` or `SortOrder.Descending` (by `created_at`).

---

## DeleteModel

**returns** `bool`

Deletes a **custom** model (if the provider supports deletion).

```csharp
bool ok = await Api.OpenAI.DeleteModel("ft:gpt-4o-mini:org:2025-08-12").ExecuteAsync();
if (!ok) DevLog.Warning("Delete rejected or not supported.");
```

**Notes**

* Irreversible. Gate behind a confirmation UI.
* Some providers restrict delete permissions (owner/admin only) or disallow deletes of base models.

---

## Error Handling & Best Practices

* **Validation**: All tasks guard against null/empty IDs.
* **Auth / scope**: Results are scoped to your credentials and org/project.
* **Pagination**: Cache cursors (`After`/`Before`) if you need stable navigation.
* **Resilience**: Wrap calls in `try/catch`. Surface partial lists gracefully.
* **Parity**: Not all providers expose the exact same fields; `IModelData` normalizes common ones.

---

## API Surface (tasks behind the facade)

* `GetModelTask(Api api, string modelId)` → `IModelData`
* `ListModelsTask(Api api)` + `.SetQuery(CursorQuery)` → `IModelData[]`
* `GetCustomModelTask(Api api, string modelId)` → `IModelData`
* `ListCustomModelsTask(Api api)` + `.SetQuery(CursorQuery)` → `IModelData[]`
* `DeleteModelTask(Api api, string modelId)` → `bool`

> Prefer the **convenience form**:
> `Api.OpenAI.GetModel(id)`, `Api.OpenAI.ListModels(query)`,
> `Api.OpenAI.GetCustomModel(id)`, `Api.OpenAI.ListCustomModels(query)`,
> `Api.OpenAI.DeleteModel(id)` — all followed by `.ExecuteAsync()`.
