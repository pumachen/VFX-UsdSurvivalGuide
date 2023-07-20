# Materials
Materials in USD are exposed via the [USDShade](https://openusd.org/dev/api/usd_shade_page_front.html) module.

Shader networks are encoded via the [UsdShade.ConnectableAPI](https://openusd.org/dev/api/class_usd_shade_connectable_a_p_i.html). So we have full access to the node graph as it is fully represented as USD prims. This allows for flexible editing, as it is as simple as editing attributes on your individual material node prims.

USD has support for encoding [MaterialX](https://materialx.org/) node graphs, which allows for render engine agnostic shader creation.

~~~admonish question title="Still under construction!"
This section still needs some more love, we'll likely expand it more in the near future.
~~~

## Material Binding <a name="materialBinding">
One of the most common use cases of relationships is encoding the material binding. Here we simply link from any imageable (renderable) prim to a `UsdShade.Material` (`Material`) prim.

~~~admonish important
Material bindings are a special kind of relationship. Here are a few important things to know:
- When looking up material bindings, USD also looks at parent prims if it can't find a written binding on the prim directly. This means you can create the binding on any parent prim and just as with primvars, it will be inherited downwards to its children.
- The "binding strength" can be adjusted, so that a child prim assignment can also be override from a binding higher up the hierarchy.
- Material bindings can also be written per purpose, if not then they bind to all purposes. (Technically it is not called purpose, the token names are `UsdShade.MaterialBindingAPI.GetMaterialPurposes() -> ['', 'preview', 'full']`). The 'preview' is usually bound to the 'UsdGeom.Tokens.proxy' purpose, the 'full' to the 'UsdGeom.Tokens.render' purpose.
- The material binding can be written in two ways:
    - Direct Binding: A relationship that points directly to a material prim
    - Collection Based Binding: A relationship that points to another collection, that then stores the actual binding paths) and to a material prim to bind.
~~~

Here is an example of a direct binding:
```python
over "asset"
{
    over "GEO"(
        prepend apiSchemas = ["MaterialBindingAPI"]
    )
    {
        rel material:binding = </materials/metal>
        over "plastic_mesh" (
            prepend apiSchemas = ["MaterialBindingAPI"]
        )
        {
            rel material:binding = </asset/materials/plastic>
        }
    }
}
```

And here is an example of a collection based binding. As you can see it is very easy to exclude a certain prim from a single control point, whereas with the direct binding we have to author it on the prim itself.
```python
def "asset" (
    prepend apiSchemas = ["MaterialBindingAPI", "CollectionAPI:material_metal"]
)
{
    rel material:binding:collection:material_metal = [
        </shaderball.collection:material_metal>,
        </materials/metal>,
    ]

    uniform token collection:material_metal:expansionRule = "expandPrims"
    rel collection:material_metal:includes = </asset>
    rel collection:material_metal:excludes = </asset/GEO/plastic_mesh>
}
```

For creating bindings in the high level API, we use the `UsdShade.MaterialBindingAPI` schema.
Here is the link to the official [API docs](https://openusd.org/dev/api/class_usd_shade_material_binding_a_p_i.html).

For more info about the load order (how collection based bindings win over direct bindings), you can read the "Bound Material Resolution" section on the API docs page.

~~~admonish tip title=""
```python
{{#include ../../../../code/core/elements.py:relationshipMaterialBinding}}
```
~~~