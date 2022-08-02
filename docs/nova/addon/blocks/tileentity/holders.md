# Holders

Holders are classes that are used to store/handle specific data like energy amount or side config data. Every 
``NetworkedTileEntity`` has an ``energyHolder``, an ``itemHolder`` and a ``fluidHolder`` property. To make them functional,
you need to override them in your TileEntity implementation. All 3 holders have a common property: The ``connectionConfig`` 
which is used to determine what ``NetworkConnectionType`` each ``BlockFace`` has. The connection types ``NONE``, ``INSERT``,
``EXTRACT`` and ``BUFFER`` are available. The ``BUFFER`` connection type is used for faces that have both an insert and 
extract connection. All holder constructors take a lambda to configure the default connection/side config.

## EnergyHolder

Before overriding the ``energyHolder`` property, you need to choose the best implementation for your TileEntity.

* ConsumerEnergyHolder: Used for TileEntities that need and consume energy to operate.
* ProviderEnergyHolder: Used for TileEntities that generate energy.
* BufferEnergyHolder: Used for TileEntities that store energy and transfer it to other tileEntities when needed. E.g. power cells.

These all have different constructors. You can open the example box below to see how each of them is implemented.

??? example "EnergyHolder implementations"

    === "ConsumerEnergyHolder"

        ```kotlin
        override val energyHolder = ConsumerEnergyHolder(
            this, // (1)
            MAX_ENERGY, // (2)
            ENERGY_PER_TICK, // (3)
            null, // (4)
            upgradeHolder // (5)
        ) { createSideConfig(NetworkConnectionType.INSERT, BlockSide.FRONT) } // (6)
        ```
        
        1. The parent TileEntity that the holder is attached to.
        2. A config reloadable value that stores the maximum amount of energy that can be stored (See [Config](../../configs.md) for more info). The
           actual maximum amount is calculated with upgrades.
        3. (optional) Another config reloadable value that stores the amount of energy that is consumed per tick. While this value is not
           required, it is still recommended to set it since it will automatically calculate the actual energy consumption with upgrades.
        4. (optional) A second energy consumption rate if needed.
        5. (optional) A ``UpgradeHolder`` instance. See [UpgradeHolder](upgrades.md) for more info.
        6. A lambda that creates a default side config for the holder. You can use ``createSideConfig`` or ``createExclusiveSideConfig`` if
           you have a simple default config. In this case, a config with insert on all sides except the front will be created.

    === "ProviderEnergyHolder"

        ```kotlin
        override val energyHolder = ProviderEnergyHolder(
            this, // (1)
            MAX_ENERGY, // (2)
            ENERGY_PER_TICK, // (3)
            upgradeHolder // (4)
        ) { createSideConfig(NetworkConnectionType.EXTRACT, FRONT) } // (5)
        ```

        1. The parent TileEntity that the holder is attached to.
        2. A config reloadable value that stores the maximum amount of energy that can be stored (See [Config](../../configs.md) for more info). The
           actual maximum amount is calculated with upgrades.
        3. (optional) Another config reloadable value that stores the amount of energy that is generated per tick. While this value is not
           required, it is still recommended to set it since it will automatically calculate the actual energy generation with upgrades.
        4. (optional) A ``UpgradeHolder`` instance. See [UpgradeHolder](upgrades.md) for more info.
        5. A lambda that creates a default side config for the holder. You can use ``createSideConfig`` or ``createExclusiveSideConfig`` if
           you have a simple default config. In this case, a config with extract on all sides except the front will be created.

    === "BufferEnergyHolder"
        
        ```kotlin
        override val energyHolder = BufferEnergyHolder(
            this, // (1)
            MAX_ENERGY, // (2)
            false // (3)
        ) { createSideConfig(BUFFER) } // (4)
        ```

        1. The parent TileEntity that the holder is attached to.
        2. A config reloadable value that stores the maximum amount of energy that can be stored. (See [Config](../../configs.md) for more info). The
           actual maximum amount is calculated with upgrades.
        3. (optional) A boolean value that determines if the holder should provide an infinite amount of energy. Can be used
           for creative blocks (E.g. a creative power cell).
        4. A lambda that creates a default side config for the holder. You can use ``createSideConfig`` or ``createExclusiveSideConfig`` if
           you have a simple default config. In this case, a config with buffer on all sides will be created.

## ItemHolder

!!! warning "VirtualInventories"

    Make sure to read the [VirtualInventories](../../../../invui/virtualinventory.md) page on the InvUI wiki before proceeding!

``ItemHolders`` are used provide an interface for networks to interact with your TileEntity's inventories. Depending on
what your TileEntity does, you may need more than one inventory (For example, a pulverizer with an input and output inventory).
Unlike ``EnergyHolders``, ``ItemHolders`` only have one default implementation called ``NovaItemHolder``. To create a 
``VirtualInventory`` use the ``getInventory`` functions. For this example, let's create an input and output inventory.

```kotlin
private val inputInv = getInventory("input", 1)
private val outputInv = getInventory("output", 2)
```

If you want to handle inventory changes, you can pass a lambda or method reference to the ``getInventory`` function.

```kotlin
private val inputInv = getInventory("input", 1, ::handleInputUpdate)
// ...
private fun handleInputUpdate(event: ItemUpdateEvent) {
    // ...
}
```

Now we can override the ``itemHolder`` property.

```kotlin
override val itemHolder = NovaItemHolder(
    this, // (1)
    inputInv to NetworkConnectionType.BUFFER, // (2)
    outputInv to NetworkConnectionType.EXTRACT // (3)
) { createSideConfig(NetworkConnectionType.INSERT, BlockSide.FRONT) } // (4)
```

1. The parent TileEntity that the holder is attached to.
2. The default inventory. A pair of ``VirtualInventory`` and ``NetworkConnectionType``. The first element is the input
   inventory and the second element is the default connection type.
3. Vararg of additional inventories.
4. A lambda that creates a default side config for the holder. You can use ``createSideConfig`` or ``createExclusiveSideConfig`` if
   you have a simple default config. In this case, a config with insert on all sides except the front will be created.

??? example "More holder examples"

    === "FluidHolder example: Freezer"

        First, we need an input tank:
        
        ```kotlin
        private val waterTank = getFluidContainer("water", setOf(FluidType.WATER), WATER_CAPACITY, 0)
        ```

        We can then pass this tank to the ``FluidHolder``:
        
        ```kotlin
        override val fluidHolder = NovaFluidHolder(this, waterTank to NetworkConnectionType.BUFFER) { 
            createSideConfig(NetworkConnectionType.INSERT, BlockSide.FRONT) // (1)
        }
        ```

        1. Set all sides to insert except the front.

## Back to the original Solar Panel

For our solar panel, we only have to override the ``energyHolder`` property. It could look something like this:

```kotlin title="SolarPanel.kt"
override val energyHolder = ProviderEnergyHolder(this, MAX_ENERGY, ENERGY_PER_TICK, null) {
    createExclusiveSideConfig(NetworkConnectionType.EXTRACT, BlockSide.BOTTOM)
}
```

## Handling ticks

Now that we've got our energyHolder set up, we can start giving it energy. To do that, we just override the ``handleTick`` method
and check if it's still day time.

```kotlin title="SolarPanel.kt"
override fun handleTick() {
    if(world.time < 13000)
        energyHolder.energy += ENERGY_PER_TICK
}
```

And that's it! We now have a working solar panel.