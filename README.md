# AquaticSeriesLib Comprehensive Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Core Components](#core-components)
3. [Custom Block System](#custom-block-system)
4. [Packet Object System](#packet-object-system)
5. [Interactable System](#interactable-system)
6. [Workload Management System](#workload-management-system)
7. [Networking System](#networking-system)
8. [Inventory System](#inventory-system)
9. [Utility Functions](#utility-functions)
10. [Conclusion](#conclusion)

## Introduction

AquaticSeriesLib is a comprehensive library for Bukkit/Spigot plugin development. It provides a wide range of utilities, adapters, and custom systems designed to streamline the creation of sophisticated Minecraft plugins. The library includes support for custom blocks, packet-based objects, interactables, workload management, networking, and advanced inventory management.

## Core Components

### AbstractAquaticSeriesLib

The AbstractAquaticSeriesLib class serves as the main entry point for the library. It initializes various features and manages core functionality.

Explanation:
This abstract class is designed to be extended by your plugin's main class. It provides access to key features of the library and handles initialization of various systems. The class supports both Paper and Spigot servers, manages message formatting (supporting both Legacy and MiniMessage formats), and initializes features like interactables, inventories, and networking.

Code example:
```kotlin
class MyPlugin : JavaPlugin() {
    lateinit var lib: AbstractAquaticSeriesLib

    override fun onEnable() {
        lib = object : AbstractAquaticSeriesLib(this, createNMSAdapter(), createFeatures()) {
            // You can override methods here if needed
        }
        // Now you can use lib to access various features
        lib.interactableHandler?.registerInteractable(MyCustomInteractable())
    }

    private fun createNMSAdapter(): NMSAdapter {
        // Create and return your NMS adapter implementation
    }

    private fun createFeatures(): HashMap<Features, IFeature> {
        // Create and return a map of enabled features
        return hashMapOf(
            Features.INTERACTABLES to InteractableHandler(20L),
            Features.INVENTORIES to InventoryHandler,
            Features.NETWORKING to NetworkPacketListener(/* ... */)
        )
    }
}
```

### Config

The Config class provides a convenient way to handle YAML configuration files in your plugin.

Explanation:
This utility class wraps Bukkit's configuration system, offering methods to easily load, access, modify, and save configuration files. It's particularly useful for managing plugin settings and data storage.

Code example:
```kotlin
class MyPluginConfig(plugin: JavaPlugin) {
    private val config = Config(File(plugin.dataFolder, "config.yml"))

    fun load() {
        config.load()
    }

    fun getSomeValue(): String? {
        return config.getConfiguration()?.getString("some.path")
    }

    fun setSomeValue(value: String) {
        config.getConfiguration()?.set("some.path", value)
        config.save()
    }
}

// Usage
val myConfig = MyPluginConfig(plugin)
myConfig.load()
val value = myConfig.getSomeValue()
myConfig.setSomeValue("new value")
```

### Extensions

AquaticSeriesLib provides several extension functions to enhance existing Bukkit classes, making common operations more convenient.

Explanation:
These extensions add functionality to classes like ItemStack, String, and FileConfiguration. They simplify tasks such as setting item display names and lore, converting strings to AquaticString format, and accessing configuration sections.

Code example:
```kotlin
// ItemStack extensions
val sword = ItemStack(Material.DIAMOND_SWORD)
sword.displayName("&bLegendary Sword".toAquatic())
sword.lore(listOf(
    "&7A sword of great power".toAquatic(),
    "&cDamage: +50".toAquatic()
))

// String extension
val formattedMessage = "Welcome to the server, &a%player%&r!".toAquatic()

// FileConfiguration extension
val config: FileConfiguration = // ... obtain configuration
val sectionList = config.getSectionList("my.complex.section")
for (section in sectionList) {
    val name = section.getString("name")
    val value = section.getInt("value")
    // Process each section...
}
```

### Custom Events

AquaticSeriesLib introduces custom events to handle specific scenarios not covered by Bukkit's default event system.

Explanation:
These custom events allow you to respond to specific actions or states in your plugin. The two main custom events are:

1. NMSEntityInteractEvent: Fired when a player interacts with an entity, providing low-level access to the interaction.
2. PlayerChunkLoadEvent: Fired when a chunk is loaded for a specific player, allowing for player-specific chunk processing.

Code example:
```kotlin
class MyEventListener : Listener {
    @EventHandler
    fun onNMSEntityInteract(event: NMSEntityInteractEvent) {
        val player = event.player
        val entityId = event.entityId
        val action = event.action

        when (action) {
            Action.RIGHT_CLICK_AIR, Action.RIGHT_CLICK_BLOCK -> {
                player.sendMessage("You right-clicked entity with ID: $entityId")
            }
            Action.LEFT_CLICK_AIR, Action.LEFT_CLICK_BLOCK -> {
                player.sendMessage("You left-clicked entity with ID: $entityId")
            }
        }
    }

    @EventHandler
    fun onPlayerChunkLoad(event: PlayerChunkLoadEvent) {
        val player = event.player
        val chunk = event.chunk

        player.sendMessage("Chunk at ${chunk.x}, ${chunk.z} loaded for you!")
        
        // Perform player-specific chunk processing
        // For example, spawn custom entities or load player-specific data
    }
}

// Register the listener
plugin.server.pluginManager.registerEvents(MyEventListener(), plugin)
```

## Custom Block System

AquaticSeriesLib's Custom Block System allows you to create and manage custom blocks in your Minecraft server. This system supports vanilla Minecraft blocks as well as custom blocks from popular customization plugins like ItemsAdder and Oraxen.

### AquaticBlock

The AquaticBlock is an abstract base class that defines the structure for all custom block types.

Explanation:
This abstract class provides a common interface for all custom blocks, regardless of their origin (vanilla, ItemsAdder, or Oraxen). It defines a single method, `place(location: Location)`, which is responsible for placing the block in the world.

Code example:
```kotlin
abstract class AquaticBlock {
    abstract fun place(location: Location)
}

class MyCustomBlock : AquaticBlock() {
    override fun place(location: Location) {
        // Custom placement logic
        location.block.type = Material.DIAMOND_BLOCK
        // Additional setup, like setting block data or metadata
    }
}

// Usage
val myBlock = MyCustomBlock()
myBlock.place(player.location)
```

### Block Types

AquaticSeriesLib supports three types of blocks: VanillaBlock, ItemsAdderBlock, and OraxenBlock.

Explanation:
1. VanillaBlock: Represents standard Minecraft blocks.
2. ItemsAdderBlock: Represents custom blocks from the ItemsAdder plugin.
3. OraxenBlock: Represents custom blocks from the Oraxen plugin.

Each of these implementations extends the AquaticBlock class and provides specific logic for placing the respective block type.

Code example:
```kotlin
// VanillaBlock
val diamondBlock = VanillaBlock(Material.DIAMOND_BLOCK)
diamondBlock.place(player.location)

// ItemsAdderBlock
val customIABlock = ItemsAdderBlock("ia_custom_block_id")
customIABlock.place(player.location.add(0, 1, 0))

// OraxenBlock
val customOraxenBlock = OraxenBlock("oraxen_block_id")
customOraxenBlock.place(player.location.add(1, 0, 0))

// Using custom blocks in your plugin
class MyPlugin : JavaPlugin() {
    fun placeCustomBlocks(location: Location) {
        val vanillaBlock = VanillaBlock(Material.EMERALD_BLOCK)
        val iaBlock = ItemsAdderBlock("my_custom_block")
        val oraxenBlock = OraxenBlock("my_oraxen_block")

        vanillaBlock.place(location)
        iaBlock.place(location.add(0, 1, 0))
        oraxenBlock.place(location.add(1, 0, 0))

        // You can also create your own custom blocks
        val myBlock = object : AquaticBlock() {
            override fun place(location: Location) {
                location.block.type = Material.BEDROCK
                // Add custom logic, like setting metadata
            }
        }
        myBlock.place(location.add(0, 0, 1))
    }
}
```

This system allows you to work with different types of blocks in a unified manner, simplifying the process of creating and placing custom blocks in your plugin.

## Packet Object System

The Packet Object System in AquaticSeriesLib provides a powerful way to create and manage custom entities and blocks that exist only for specific players, without affecting the actual game world.

### AbstractPacketObject

AbstractPacketObject is the base class for all packet-based objects in the system.

Explanation:
This abstract class defines the core structure and behavior for packet objects. It includes properties for location and audience (the players who can see the object), methods for spawning and despawning the object, and methods for sending spawn and despawn packets to players.

Code example:
```kotlin
abstract class AbstractPacketObject(
    open val location: Location,
    open val audience: AbstractAudience
) {
    var spawned: Boolean = false

    abstract fun sendDespawnPacket(vararg players: Player)
    abstract fun sendSpawnPacket(vararg players: Player)
    abstract fun spawn()
    abstract fun despawn()
}

class MyCustomPacketObject(
    override val location: Location,
    override val audience: AbstractAudience
) : AbstractPacketObject(location, audience) {
    
    override fun sendDespawnPacket(vararg players: Player) {
        for (player in players) {
            // Send custom despawn packet
            // This might involve using NMS or ProtocolLib
        }
    }

    override fun sendSpawnPacket(vararg players: Player) {
        for (player in players) {
            // Send custom spawn packet
            // This might involve using NMS or ProtocolLib
        }
    }

    override fun spawn() {
        if (!spawned) {
            for (player in audience.currentlyViewing) {
                sendSpawnPacket(player)
            }
            spawned = true
        }
    }

    override fun despawn() {
        if (spawned) {
            for (player in audience.currentlyViewing) {
                sendDespawnPacket(player)
            }
            spawned = false
        }
    }
}

// Usage
val audience = object : AbstractAudience {
    override val currentlyViewing: List<Player> = // ... list of players
}
val packetObject = MyCustomPacketObject(location, audience)
packetObject.spawn()
// Later...
packetObject.despawn()
```

### PacketEntity

PacketEntity is a specific implementation of AbstractPacketObject for creating custom entities.

Explanation:
This class represents a custom entity that exists only for specific players. It includes an entity ID and a custom interaction handler.

Code example:
```kotlin
class MyPacketEntity(
    location: Location,
    entityId: Int,
    audience: AbstractAudience,
    onInteract: Consumer<PacketEntityInteractEvent>
) : PacketEntity(location, entityId, audience, onInteract) {

    override fun sendSpawnPacket(vararg players: Player) {
        for (player in players) {
            // Send custom entity spawn packet
            // This might use NMS or ProtocolLib to send a custom entity spawn packet
        }
    }

    override fun sendDespawnPacket(vararg players: Player) {
        for (player in players) {
            // Send custom entity despawn packet
            // This might use NMS or ProtocolLib to send an entity destroy packet
        }
    }
}

// Usage
val packetEntity = MyPacketEntity(
    location,
    nextEntityId(),
    audience
) { event ->
    val player = event.player
    player.sendMessage("You interacted with a packet entity!")
}
packetEntity.spawn()

// In your packet listener (using ProtocolLib as an example)
protocolManager.addPacketListener(object : PacketAdapter(plugin, ListenerPriority.NORMAL, PacketType.Play.Client.USE_ENTITY) {
    override fun onPacketReceiving(event: PacketEvent) {
        val packet = event.packet
        val entityId = packet.integers.read(0)
        if (entityId == packetEntity.entityId) {
            val player = event.player
            val interactEvent = PacketEntityInteractEvent(player, packetEntity, Action.RIGHT_CLICK_AIR)
            packetEntity.onInteract.accept(interactEvent)
        }
    }
})
```

### PacketBlock

PacketBlock is another implementation of AbstractPacketObject, representing a custom block visible only to specific players.

Explanation:
This class allows you to create custom blocks that exist only for certain players, without changing the actual world data.

Code example:
```kotlin
class MyPacketBlock(
    location: Location,
    blockData: BlockData,
    audience: AbstractAudience,
    onInteract: Consumer<PlayerInteractEvent>
) : PacketBlock(location, blockData, audience, onInteract) {

    override fun sendDespawnPacket(vararg players: Player) {
        for (player in players) {
            player.sendBlockChange(location, location.block.blockData)
        }
    }

    override fun sendSpawnPacket(vararg players: Player) {
        for (player in players) {
            player.sendBlockChange(location, blockData)
        }
    }
}

// Usage
val packetBlock = MyPacketBlock(
    location,
    Material.DIAMOND_BLOCK.createBlockData(),
    audience
) { event ->
    val player = event.player
    player.sendMessage("You interacted with a packet block!")
}
packetBlock.spawn()

// Don't forget to handle block interactions in your plugin
@EventHandler
fun onPlayerInteract(event: PlayerInteractEvent) {
    val block = event.clickedBlock ?: return
    if (block.location == packetBlock.location) {
        packetBlock.onInteract.accept(event)
        event.isCancelled = true
    }
}
```

### FakeObjectHandler

The FakeObjectHandler is a singleton object that manages the registration and handling of packet objects.

Explanation:
This handler keeps track of all packet objects (both entities and blocks) and manages their lifecycle, including spawning, despawning, and chunk loading/unloading. It provides methods to register and unregister packet objects, and handles events like player join/quit and chunk load/unload to ensure packet objects are properly managed.

Code example:
```kotlin
object FakeObjectHandler : IFeature {
    private val registry = FakeObjectRegistry()

    fun registerBlock(packetBlock: PacketBlock) {
        registry.registerBlock(packetBlock)
    }

    fun registerEntity(packetEntity: PacketEntity) {
        registry.registerEntity(packetEntity)
    }

    fun unregisterBlock(location: Location) {
        registry.unregisterBlock(location)
    }

    fun unregisterEntity(location: Location, id: Int) {
        registry.unregisterEntity(location, id)
    }

    override fun initialize(lib: AbstractAquaticSeriesLib) {
        // Register event listeners
        lib.plugin.server.pluginManager.registerEvents(object : Listener {
            @EventHandler
            fun onPlayerJoin(event: PlayerJoinEvent) {
                // Inject packet listener for the player
                lib.nmsAdapter?.packetListenerAdapter()?.inject(event.player)
            }

            @EventHandler
            fun onPlayerQuit(event: PlayerQuitEvent) {
                // Eject packet listener for the player
                lib.nmsAdapter?.packetListenerAdapter()?.eject(event.player)
            }

            @EventHandler
            fun onChunkLoad(event: ChunkLoadEvent) {
                // Spawn packet objects in the loaded chunk
                registry.getObjectsInChunk(event.chunk).forEach { it.spawn() }
            }

            @EventHandler
            fun onChunkUnload(event: ChunkUnloadEvent) {
                // Despawn packet objects in the unloaded chunk
                registry.getObjectsInChunk(event.chunk).forEach { it.despawn() }
            }
        }, lib.plugin)
    }
}

// Usage in your plugin
class MyPlugin : JavaPlugin() {
    override fun onEnable() {
        // Register a packet block
        val packetBlock = MyPacketBlock(location, Material.DIAMOND_BLOCK.createBlockData(), audience, onInteract)
        FakeObjectHandler.registerBlock(packetBlock)

        // Register a packet entity
        val packetEntity = MyPacketEntity(location, nextEntityId(), audience, onInteract)
        FakeObjectHandler.registerEntity(packetEntity)
    }
}
```

## Interactable System

The Interactable System in AquaticSeriesLib provides a powerful framework for creating complex, interactive objects in the game world. These objects can span multiple blocks, persist across server restarts, and have custom behaviors.

### AbstractInteractable

AbstractInteractable is the base class for all interactable objects in the system.

Explanation:
This abstract class defines the core structure and behavior for interactable objects. It includes properties like persistence, unique ID, serializer, and shape. It also provides methods for spawning the interactable and handling chunk load/unload events.

Code example:
```kotlin
abstract class AbstractInteractable {
    abstract val persistent: Boolean
    abstract val id: String
    abstract val serializer: AbstractInteractableSerializer<*>
    abstract val shape: BlockShape

    abstract fun spawn(location: Location): AbstractSpawnedInteractable
    abstract fun onChunkLoad(data: InteractableData, location: Location)
    abstract fun onChunkUnload(data: InteractableData)

    // Utility method to process the interactable's shape
    fun processLayerCells(layers: Map<Int, Map<Int, String>>, location: Location, operation: (Char, Location) -> Unit) {
        // Implementation
    }
}

class MyCustomInteractable : AbstractInteractable() {
    override val persistent = true
    override val id = "my_custom_interactable"
    override val serializer = MyCustomInteractableSerializer()
    override val shape = BlockShape(
        layers = mapOf(
            0 to mapOf(0 to "XXX", 1 to "X X", 2 to "XXX"),
            1 to mapOf(0 to "X X", 1 to " X ", 2 to "X X"),
            2 to mapOf(0 to "XXX", 1 to "XXX", 2 to "XXX")
        ),
        blocks = mapOf('X' to MyCustomBlock())
    )

    override fun spawn(location: Location): AbstractSpawnedInteractable {
        return MySpawnedInteractable(location, this)
    }

    override fun onChunkLoad(data: InteractableData, location: Location) {
        // Implement chunk load logic
        val spawned = spawn(location)
        spawned.load(data)
    }

    override fun onChunkUnload(data: InteractableData) {
        // Implement chunk unload logic
        // This might involve saving the interactable's state
    }
}

// Usage
val myInteractable = MyCustomInteractable()
val spawnedInteractable = myInteractable.spawn(location)
```

### AbstractSpawnedInteractable

AbstractSpawnedInteractable represents an instance of an interactable that has been spawned in the world.

Explanation:
This abstract class defines the properties and methods for a spawned interactable object. It includes the location of the interactable, a reference to its AbstractInteractable definition, a list of associated locations (for multi-block structures), and methods for loading and despawning the interactable.

Code example:
```kotlin
abstract class AbstractSpawnedInteractable {
    abstract val location: Location
    abstract val interactable: AbstractInteractable
    abstract val associatedLocations: List<Location>
    abstract var loaded: Boolean

    abstract fun load(data: InteractableData?)
    abstract fun despawn()
}

class MySpawnedInteractable(
    override val location: Location,
    override val interactable: MyCustomInteractable
) : AbstractSpawnedInteractable() {
    override val associatedLocations = mutableListOf<Location>()
    override var loaded = false

    override fun load(data: InteractableData?) {
        if (loaded) return
        interactable.processLayerCells(interactable.shape.layers, location) { char, loc ->
            val block = interactable.shape.blocks[char]
            if (block != null) {
                block.place(loc)
                associatedLocations.add(loc)
            }
        }
        loaded = true
    }

    override fun despawn() {
        associatedLocations.forEach { it.block.type = Material.AIR }
        loaded = false
    }
}
```

### InteractableHandler

The InteractableHandler is responsible for managing all interactables in the game.

Explanation:
This class handles the lifecycle of interactables, including registration, spawning, loading, and unloading. It also manages chunk loading and unloading events to ensure interactables are properly handled when chunks are loaded or unloaded.

Code example:
```kotlin
class InteractableHandler(private val plugin: JavaPlugin) : IFeature {
    private val registry = mutableMapOf<String, AbstractInteractable>()
    private val spawnedInteractables = mutableMapOf<Chunk, MutableSet<AbstractSpawnedInteractable>>()

    fun registerInteractable(interactable: AbstractInteractable) {
        registry[interactable.id] = interactable
    }

    fun spawnInteractable(id: String, location: Location): AbstractSpawnedInteractable? {
        val interactable = registry[id] ?: return null
        val spawned = interactable.spawn(location)
        spawnedInteractables.getOrPut(location.chunk) { mutableSetOf() }.add(spawned)
        return spawned
    }

    @EventHandler
    fun onChunkLoad(event: ChunkLoadEvent) {
        val chunk = event.chunk
        // Load interactables in this chunk
        // This would involve querying a database or file for saved interactables in this chunk
    }

    @EventHandler
    fun onChunkUnload(event: ChunkUnloadEvent) {
        val chunk = event.chunk
        spawnedInteractables[chunk]?.forEach { it.interactable.onChunkUnload(it.serializeData()) }
        spawnedInteractables.remove(chunk)
    }

    override fun initialize(lib: AbstractAquaticSeriesLib) {
        plugin.server.pluginManager.registerEvents(this, plugin)
    }
}

// Usage in your plugin
class MyPlugin : JavaPlugin() {
    lateinit var interactableHandler: InteractableHandler

    override fun onEnable() {
        interactableHandler = InteractableHandler(this)
        interactableHandler.registerInteractable(MyCustomInteractable())

        // Spawn an interactable
        val location = Location(world, 0.0, 64.0, 0.0)
        interactableHandler.spawnInteractable("my_custom_interactable", location)
    }
}
```

## Workload Management System

The Workload Management System in AquaticSeriesLib provides a way to efficiently manage and execute tasks, particularly useful for operations that need to be performed over time or in specific contexts like chunk loading.

### AbstractWorkload

AbstractWorkload is the base class for all workload types in the system.

Explanation:
This abstract class defines the core structure and behavior for workloads. It includes methods for running the next job in the workload, starting the workload execution, and cancelling all remaining jobs.

Code example:
```kotlin
abstract class AbstractWorkload {
    abstract var isRunning: Boolean
    abstract fun runNext()
    abstract fun run(): CompletableFuture<Void>
    abstract fun cancelAll()
}

class MyCustomWorkload(private val jobs: MutableList<Runnable>, private val delay: Long) : AbstractWorkload() {
    override var isRunning = false
    private val future = CompletableFuture<Void>()

    override fun runNext() {
        if (!isRunning || jobs.isEmpty()) {
            isRunning = false
            future.complete(null)
            return
        }

        val job = jobs.removeAt(0)
        job.run()

        Bukkit.getScheduler().runTaskLater(MyPlugin.instance, this::runNext, delay)
    }

    override fun run(): CompletableFuture<Void> {
        if (isRunning) return future
        isRunning = true
        runNext()
        return future
    }

    override fun cancelAll() {
        jobs.clear()
        isRunning = false
        future.completeExceptionally(CancellationException("Workload cancelled"))
    }
}

// Usage
val workload = MyCustomWorkload(
    mutableListOf(
        Runnable { println("Job 1") },
        Runnable { println("Job 2") },
        Runnable { println("Job 3") }
    ),
    20L // 1 second delay between jobs
)

workload.run().thenRun {
    println("All jobs completed")
}
```

### ChunkWorkload

ChunkWorkload is a specialized workload associated with a specific chunk.

Explanation:
This class extends AbstractWorkload and is designed to handle tasks specific to a chunk. It automatically cancels all remaining jobs when the chunk is unloaded, ensuring that no unnecessary work is performed on unloaded chunks.

Code example:
```kotlin
class ChunkWorkload(
    val chunk: Chunk,
    private val jobs: MutableList<Runnable>,
    private val delay: Long
) : AbstractWorkload() {
    override var isRunning = false
    private val future = CompletableFuture<Void>()

    override fun runNext() {
        if (!isRunning || jobs.isEmpty() || !chunk.isLoaded) {
            isRunning = false
            future.complete(null)
            return
        }

        val job = jobs.removeAt(0)
        job.run()

        Bukkit.getScheduler().runTaskLater(MyPlugin.instance, this::runNext, delay)
    }

    override fun run(): CompletableFuture<Void> {
        if (isRunning) return future
        isRunning = true
        runNext()
        return future
    }

    override fun cancelAll() {
        jobs.clear()
        isRunning = false
        future.completeExceptionally(CancellationException("ChunkWorkload cancelled"))
    }
}

// Usage
val chunkWorkload = ChunkWorkload(
    player.location.chunk,
    mutableListOf(
        Runnable { /* Perform chunk-specific operation */ },
        Runnable { /* Another chunk-specific operation */ }
    ),
    10L // 0.5 second delay between jobs
)

chunkWorkload.run().thenRun {
    println("All chunk operations completed")
}
```

### ChunkWorkloadHandler

ChunkWorkloadHandler is a singleton class that manages ChunkWorkloads and ensures they are properly cancelled when chunks are unloaded.

Explanation:
This handler keeps track of all active ChunkWorkloads and listens for chunk unload events. When a chunk is unloaded, it cancels all workloads associated with that chunk to prevent operations on unloaded chunks.

Code example:
```kotlin
object ChunkWorkloadHandler : Listener {
    private val workloads = mutableMapOf<Chunk, MutableSet<ChunkWorkload>>()

    fun registerWorkload(workload: ChunkWorkload) {
        workloads.getOrPut(workload.chunk) { mutableSetOf() }.add(workload)
    }

    fun unregisterWorkload(workload: ChunkWorkload) {
        workloads[workload.chunk]?.remove(workload)
    }

    @EventHandler
    fun onChunkUnload(event: ChunkUnloadEvent) {
        workloads[event.chunk]?.forEach { it.cancelAll() }
        workloads.remove(event.chunk)
    }

    fun initialize(plugin: JavaPlugin) {
        plugin.server.pluginManager.registerEvents(this, plugin)
    }
}

// Usage in your plugin
class MyPlugin : JavaPlugin() {
    override fun onEnable() {
        ChunkWorkloadHandler.initialize(this)

        // Create and register a chunk workload
        val chunkWorkload = ChunkWorkload(/* ... */)
        ChunkWorkloadHandler.registerWorkload(chunkWorkload)
        chunkWorkload.run()
    }
}
```

## Networking System 

### NetworkPacketListener

NetworkPacketListener is the central class that manages packet registration, handling, and serialization.

Explanation:
This class is responsible for registering custom packet types, handling incoming packets, and managing the serialization and deserialization of network packets. It acts as the core of the networking system, coordinating communication between servers.

Code example:
```kotlin
class NetworkPacketListener(private val serverName: String) : IFeature {
    private val packetHandlers = mutableMapOf<Class<out NetworkPacket>, NetworkPacketHandler<*>>()
    private val serializerModules = mutableListOf<SerializersModule>()
    private val json = Json { serializersModule = SerializersModule {} }

    fun <T : NetworkPacket> registerPacket(
        packetClass: Class<T>,
        handler: NetworkPacketHandler<T>,
        serializerModule: SerializersModule
    ) {
        packetHandlers[packetClass] = handler
        serializerModules.add(serializerModule)
        updateSerializer()
    }

    fun handlePacket(packet: SignedNetworkPacket): CompletableFuture<NetworkResponse> {
        val handler = packetHandlers[packet.packet.javaClass] ?: return CompletableFuture.completedFuture(
            NetworkResponse(NetworkResponse.Status.ERROR, "No handler found for packet type")
        )
        @Suppress("UNCHECKED_CAST")
        return (handler as NetworkPacketHandler<NetworkPacket>).handle(packet)
            .thenApply { NetworkResponse(NetworkResponse.Status.SUCCESS, it) }
    }

    fun serializePacket(packet: SignedNetworkPacket): String {
        return json.encodeToString(SignedNetworkPacket.serializer(), packet)
    }

    fun deserializePacket(jsonString: String): SignedNetworkPacket? {
        return try {
            json.decodeFromString(SignedNetworkPacket.serializer(), jsonString)
        } catch (e: Exception) {
            null
        }
    }

    private fun updateSerializer() {
        json = Json {
            serializersModule = SerializersModule {
                serializerModules.forEach { include(it) }
            }
        }
    }

    override fun initialize(lib: AbstractAquaticSeriesLib) {
        // Initialization logic
    }
}

// Usage
val networkListener = NetworkPacketListener("server1")
networkListener.registerPacket(
    MyCustomPacket::class.java,
    MyCustomPacketHandler(),
    myCustomPacketSerializerModule
)
```

### NetworkAdapter

NetworkAdapter is an abstract class that defines the basic structure for network communication adapters.

Explanation:
This abstract class provides a common interface for different network communication protocols (like Redis or TCP). It defines methods for sending packets and retrieving connected servers.

Code example:
```kotlin
abstract class NetworkAdapter {
    abstract val serverName: String

    abstract fun send(packet: NetworkPacket): CompletableFuture<NetworkResponse>
    abstract fun connectedServers(): List<String>
}

// Implementation for Redis
class RedisNetworkAdapter(
    override val serverName: String,
    private val redisClient: RedisClient,
    private val channel: String,
    private val networkListener: NetworkPacketListener
) : NetworkAdapter() {
    override fun send(packet: NetworkPacket): CompletableFuture<NetworkResponse> {
        val future = CompletableFuture<NetworkResponse>()
        val signedPacket = SignedNetworkPacket(packet, serverName)
        val serializedPacket = networkListener.serializePacket(signedPacket)
        
        redisClient.publish(channel, serializedPacket).subscribe { 
            // Handle successful publish
        }

        return future
    }

    override fun connectedServers(): List<String> {
        // Implement logic to retrieve connected servers from Redis
        return emptyList()
    }
}
```

### Redis Communication (RedisHandler)

RedisHandler implements the NetworkAdapter for Redis-based communication.

Explanation:
This class uses Redis pub/sub mechanisms to facilitate communication between servers. It handles publishing messages to Redis channels and subscribing to receive messages from other servers.

Code example:
```kotlin
class RedisHandler(
    override val serverName: String,
    private val redisSettings: RedisNetworkSettings,
    private val networkListener: NetworkPacketListener
) : NetworkAdapter() {

    private val jedisPool: JedisPool = JedisPool(redisSettings.ip, redisSettings.port)

    init {
        subscribe()
    }

    private fun subscribe() {
        Thread {
            jedisPool.resource.use { jedis ->
                jedis.subscribe(object : JedisPubSub() {
                    override fun onMessage(channel: String, message: String) {
                        val packet = networkListener.deserializePacket(message)
                        if (packet != null) {
                            networkListener.handlePacket(packet)
                        }
                    }
                }, redisSettings.channel)
            }
        }.start()
    }

    override fun send(packet: NetworkPacket): CompletableFuture<NetworkResponse> {
        val future = CompletableFuture<NetworkResponse>()
        val signedPacket = SignedNetworkPacket(packet, serverName)
        val serializedPacket = networkListener.serializePacket(signedPacket)

        jedisPool.resource.use { jedis ->
            jedis.publish(redisSettings.channel, serializedPacket)
        }

        return future
    }

    override fun connectedServers(): List<String> {
        // Implement logic to retrieve connected servers from Redis
        return redisSettings.servers
    }
}

// Usage
val redisSettings = RedisNetworkSettings(
    ip = "localhost",
    port = 6379,
    channel = "minecraft_network",
    servers = listOf("lobby", "game1", "game2")
)
val redisHandler = RedisHandler("server1", redisSettings, networkListener)
```

### TCP Communication (TCPHandler)

TCPHandler implements the NetworkAdapter for TCP-based communication.

Explanation:
This class manages direct socket connections between servers. It handles establishing connections, sending packets over TCP, and receiving incoming packets from other servers.

Code example:
```kotlin
class TCPHandler(
    override val serverName: String,
    private val tcpSettings: TCPNetworkSettings,
    private val networkListener: NetworkPacketListener
) : NetworkAdapter() {

    private val serverSocket = ServerSocket(tcpSettings.port)
    private val clientConnections = mutableMapOf<String, Socket>()

    init {
        startServer()
        connectToServers()
    }

    private fun startServer() {
        Thread {
            while (true) {
                val clientSocket = serverSocket.accept()
                handleClientConnection(clientSocket)
            }
        }.start()
    }

    private fun handleClientConnection(socket: Socket) {
        Thread {
            val reader = BufferedReader(InputStreamReader(socket.getInputStream()))
            while (true) {
                val message = reader.readLine() ?: break
                val packet = networkListener.deserializePacket(message)
                if (packet != null) {
                    networkListener.handlePacket(packet)
                }
            }
        }.start()
    }

    private fun connectToServers() {
        tcpSettings.servers.forEach { (name, info) ->
            if (name != serverName) {
                val socket = Socket(info.ip, info.port)
                clientConnections[name] = socket
            }
        }
    }

    override fun send(packet: NetworkPacket): CompletableFuture<NetworkResponse> {
        val future = CompletableFuture<NetworkResponse>()
        val signedPacket = SignedNetworkPacket(packet, serverName)
        val serializedPacket = networkListener.serializePacket(signedPacket)

        val targetServer = packet.channel
        val socket = clientConnections[targetServer] ?: throw IllegalStateException("No connection to server $targetServer")

        socket.getOutputStream().write(serializedPacket.toByteArray())
        socket.getOutputStream().write("\n".toByteArray())
        socket.getOutputStream().flush()

        return future
    }

    override fun connectedServers(): List<String> {
        return clientConnections.keys.toList()
    }
}

// Usage
val tcpSettings = TCPNetworkSettings(
    port = 25565,
    servers = mapOf(
        "lobby" to ServerInfo("192.168.1.100", 25566),
        "game1" to ServerInfo("192.168.1.101", 25567)
    )
)
val tcpHandler = TCPHandler("server1", tcpSettings, networkListener)
```

### Custom Packets

The library uses a custom packet system for inter-server communication.

Explanation:
Custom packets are defined by extending the NetworkPacket class. Each custom packet should have its own serializer and handler.

Code example:
```kotlin
@Serializable
class MyCustomPacket(val data: String) : NetworkPacket() {
    override val channel: String = "target_server"
}

class MyCustomPacketHandler : NetworkPacketHandler<MyCustomPacket> {
    override fun handle(packet: SignedNetworkPacket): CompletableFuture<String> {
        val myPacket = packet.packet as MyCustomPacket
        println("Received data: ${myPacket.data} from ${packet.sentFrom}")
        return CompletableFuture.completedFuture("Processed")
    }
}

val myCustomPacketSerializerModule = SerializersModule {
    polymorphic(NetworkPacket::class) {
        subclass(MyCustomPacket::class)
    }
}

// Usage
networkListener.registerPacket(
    MyCustomPacket::class.java,
    MyCustomPacketHandler(),
    myCustomPacketSerializerModule
)

// Sending a packet
val packet = MyCustomPacket("Hello, server!")
networkAdapter.send(packet).thenAccept { response ->
    if (response.status == NetworkResponse.Status.SUCCESS) {
        println("Packet sent successfully")
    }
}
```

This comprehensive Networking System allows for flexible and efficient communication between Minecraft servers, supporting both Redis and TCP protocols and allowing for easy creation and handling of custom packets.



## Inventory System

AquaticSeriesLib provides a comprehensive inventory management system that allows for the creation of custom inventories with advanced features such as pagination, personalization, dynamic content, and inventory history.

### CustomInventory

CustomInventory is the foundation for creating custom inventories with advanced features.

Explanation:
This class allows you to create customized inventories with dynamic titles, custom click handlers, and component-based item management. It also supports inventory history for easy navigation between different inventory screens.

Code example:
```kotlin
class CustomInventory(
    var titleHolder: TitleHolder,
    size: Int,
    type: InventoryType = InventoryType.CHEST,
    factory: Consumer<CustomInventory>? = null
) : InventoryHolder {

    private val inventory: Inventory
    val componentHandler = ComponentHandler(this)
    val history = InventoryHistory()

    var onOpen: Consumer<CustomInventoryOpenEvent>? = null
    var onClose: Consumer<CustomInventoryCloseEvent>? = null
    var onClick: Consumer<CustomInventoryClickEvent>? = null

    init {
        inventory = Bukkit.createInventory(this, type, titleHolder.generate(this, null).legacy())
        factory?.accept(this)
    }

    override fun getInventory(): Inventory = inventory

    fun open(player: Player, ignoreHistory: Boolean = false) {
        history.addToHistory(player, player.openInventory.topInventory.holder as? CustomInventory, ignoreHistory)
        player.openInventory(inventory)
    }

    fun setItem(slot: Int, itemStack: ItemStack?) {
        inventory.setItem(slot, itemStack)
    }

    fun onClick(event: CustomInventoryClickEvent) {
        onClick?.accept(event)
        componentHandler.onClick(event)
    }

    // Other methods for handling events and managing the inventory
}

// Usage
val customInventory = CustomInventory(TitleHolder.of(BasicTitleComponent("My Custom Inventory")), 27) { inv ->
    inv.setItem(13, ItemStack(Material.DIAMOND))
    inv.onClick = Consumer { event ->
        if (event.slot == 13) {
            event.player.sendMessage("You clicked the diamond!")
        }
    }
}

player.openInventory(customInventory.inventory)
```

### PaginatedInventory

PaginatedInventory extends CustomInventory to provide pagination support.

Explanation:
This class allows you to create inventories with multiple pages, making it easy to display large amounts of content. It provides methods for navigating between pages and manages page-specific content.

Code example:
```kotlin
class PaginatedInventory(
    title: TitleHolder,
    size: Int,
    val player: Player,
    factory: Consumer<PaginatedInventory>? = null
) : CustomInventory(title, size, InventoryType.CHEST, null) {

    val pages = mutableListOf<InventoryPage>()
    var currentPage = 0

    init {
        factory?.accept(this)
    }

    fun addPage(page: InventoryPage) {
        pages.add(page)
    }

    fun nextPage(): Boolean {
        if (currentPage < pages.size - 1) {
            currentPage++
            updatePage()
            return true
        }
        return false
    }

    fun previousPage(): Boolean {
        if (currentPage > 0) {
            currentPage--
            updatePage()
            return true
        }
        return false
    }

    private fun updatePage() {
        val page = pages[currentPage]
        titleHolder = page.titleHolder
        componentHandler.clear()
        componentHandler.addAll(page.components)
        player.openInventory(inventory)
    }
}

// Usage
val paginatedInventory = PaginatedInventory(
    TitleHolder.of(BasicTitleComponent("My Paginated Inventory")),
    54,
    player
) { inv ->
    val page1 = InventoryPage(TitleHolder.of(BasicTitleComponent("Page 1")))
    page1.addComponent(Button(ItemStack(Material.APPLE), SlotSelection.of(0), Consumer { /* click logic */ }))
    
    val page2 = InventoryPage(TitleHolder.of(BasicTitleComponent("Page 2")))
    page2.addComponent(Button(ItemStack(Material.BANANA), SlotSelection.of(0), Consumer { /* click logic */ }))
    
    inv.addPage(page1)
    inv.addPage(page2)
}

paginatedInventory.open(player)
```

### ComponentHandler

ComponentHandler manages all components within a CustomInventory.

Explanation:
This class is responsible for managing and rendering inventory components. It handles component addition, removal, and click events, as well as managing component priorities.

Code example:
```kotlin
class ComponentHandler(private val inventory: CustomInventory) {
    private val components = mutableMapOf<String, MenuComponent>()
    private val renderedComponents = mutableMapOf<Int, String>()

    fun addComponent(id: String, component: MenuComponent) {
        components[id] = component
        redrawComponent(id)
    }

    fun removeComponent(id: String) {
        components.remove(id)
        renderedComponents.entries.removeIf { it.value == id }
    }

    fun redrawComponents() {
        inventory.inventory.clear()
        renderedComponents.clear()
        components.entries.sortedBy { it.value.priority }.forEach { (id, _) ->
            redrawComponent(id)
        }
    }

    fun onClick(event: CustomInventoryClickEvent) {
        val componentId = renderedComponents[event.slot] ?: return
        val component = components[componentId] ?: return
        component.onClick(ComponentClickEvent(inventory, component, event.originalEvent))
    }

    private fun redrawComponent(id: String) {
        val component = components[id] ?: return
        for (slot in component.slotSelection.slots) {
            if (renderedComponents[slot] == null || components[renderedComponents[slot]]!!.priority <= component.priority) {
                renderedComponents[slot] = id
                inventory.setItem(slot, component.getItemStack())
            }
        }
    }
}

// Usage is typically internal to the CustomInventory class
```

## Utility Functions

AquaticSeriesLib provides several utility functions to simplify common tasks in plugin development.

### Command Registration

An extension function for easy command registration.

Explanation:
This utility function extends the Bukkit Command class to provide a simple way to register commands without manually editing the plugin.yml file.

Code example:
```kotlin
fun Command.register(plugin: JavaPlugin) {
    try {
        val commandMapField = Bukkit.getServer().javaClass.getDeclaredField("commandMap")
        commandMapField.isAccessible = true
        val commandMap = commandMapField.get(Bukkit.getServer()) as CommandMap
        commandMap.register(plugin.name, this)
    } catch (e: Exception) {
        plugin.logger.severe("Failed to register command: ${this.name}")
        e.printStackTrace()
    }
}

// Usage
class MyCommand : Command("mycommand") {
    override fun execute(sender: CommandSender, commandLabel: String, args: Array<out String>): Boolean {
        sender.sendMessage("You executed mycommand!")
        return true
    }
}

// In your plugin's onEnable method
MyCommand().register(this)
```

### String Extensions

Utility extensions for string manipulation and conversion.

Explanation:
These extensions provide convenient methods for converting strings to AquaticString format and applying color codes.

Code example:
```kotlin
fun String.toAquatic(): AquaticString {
    return AquaticString(this)
}

fun String.color(): String {
    return ChatColor.translateAlternateColorCodes('&', this)
}

// Usage
val message = "&aHello, &b%player%&a!".toAquatic()
player.sendMessage(message.legacy()) // Sends a colored message

val coloredString = "&cThis is red".color()
player.sendMessage(coloredString) // Sends "This is red" in red
```

## Conclusion

AquaticSeriesLib is a powerful and flexible library for Bukkit/Spigot plugin development. It provides a wide range of features and utilities that simplify the creation of complex plugins:

1. Custom Block System: Easily create and manage custom blocks, including support for popular customization plugins.
2. Packet Object System: Implement client-side only entities and blocks for enhanced gameplay experiences.
3. Interactable System: Create complex, multi-block structures with custom behaviors and persistence.
4. Workload Management System: Efficiently manage and execute tasks, particularly useful for chunk-based operations.
5. Networking System: Implement robust cross-server communication using either Redis or TCP protocols.
6. Inventory System: Create advanced custom inventories with pagination, dynamic content, and custom components.
7. Utility Functions: Simplify common tasks such as command registration and string manipulation.

By leveraging AquaticSeriesLib, developers can focus on creating unique and engaging plugin content while relying on the library to handle many of the complex underlying systems. Whether you're creating a simple utility plugin or a complex mini-game, AquaticSeriesLib provides the tools and frameworks to streamline your development process and create high-quality Minecraft plugins.
