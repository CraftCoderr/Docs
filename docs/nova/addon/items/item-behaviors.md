# Item Behaviors

Item behaviors are used to add functionality to items. There are some default implementations, but you can also create your own.

As stated in the previous section, for consumable items you can either call ``NovaMaterial#registerFood`` or call
``NovaMaterial#registerItem`` with the ``Consumable`` item behavior. This item behavior lets you specify the food options
like the consumeTime, the nutrition and much more. However, there are more item behaviors that you can use for different
purposes.

???+ example "Default Item Behaviors"

    These are the default item behaviors that Nova provides:

    === "Chargeable"
    
        Allows you to make an item, that stores energy. This should mostly be used with other custom item behaviors, since
        there is no default implementation for consuming energy. The behavior takes a maxEnergy in the constructor via a
        ``ValueReloadable`` instance. This class is used to wrap config reloadable values. Configs will be explained in
        more detail later. For this example, let's set it to a static value of 100000.
    
        ```kotlin
        Chargeable(notReloadable(100000))
        ```

        Or for accessing a config value:
    
        ```kotlin
        val CAPACITY = configReloadable { NovaConfig["example:example_item"].getLong("capacity") }
        val behavior = Chargeable(CAPACITY)
        ```

    === "Consumable"

        Allows you to make a custom consumable item. Example:

        ```kotlin
        val behavior = Consumable(FoodOptions(
            consumeTime = 40,
            nutrition = 5,
            saturationModifier = 0.5f,
            instantHealth = 5.0,
            effects = listOf(PotionEffect(PotionEffectType.SPEED, 20, 1))
        ))
        ```

    === "Wearable"

        Allows you to make an item that can be equipped in a players armor slots. The constructor takes 3 values:
        
        * The ArmorType
        * The armor
        * The armor toughness

        If you need some examples for the ``armor`` and ``armorToughness`` values, you can check out the [Minecraft wiki](https://minecraft.fandom.com/wiki/Armor#Defense_points).

        Example for a chestplate with the same amount of protection as a diamond chestplate:
        
        ```kotlin
        val behavior = Wearable(
            ArmorType.CHESTPLATE,
            armor = 8.0,
            armorToughness = 2.0
        )
        ```