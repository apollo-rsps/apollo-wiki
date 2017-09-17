# Getting Started with Kotlin Plugins

Note: This tutorial is for developing Plugins for the kotlin-experiments branch prior to the release of the Kotlin plugin system for Apollo. Once the system is release and made part of the master branch, I will update this Wiki page. @tlf30

__First, you will need a environment to work in.__ 

IDEA by Jet Brains is a good IDE for kotlin development. To get started make sure you have the JDK and then go get IDEA: https://www.jetbrains.com/idea/

When you start IDEA, you will be prompted with a screen to create, import, open, or checkout a project. We want to check out a project from GitHub. 
Enter the URL: https://github.com/apollo-rsps/apollo.git and you can choose the rest to your liking. It will get checked out, then click to open it.

The next window you will see if to import from gradle. We want to uncheck the 'Create separate module per source set' check box. Then click 'OK'.

We then want to checkout the kotlin-experiments branch. To do this, use the navigation bar at the top of IDEA and click 'VCS->Git->Branches->origin/kotlin-experiments->Checkout as new branch'
Name the new branch something along the lines of kotlin-experiments-my-plugin.

Now you have cloned apollo into IDEA!

__Second, we setup the home for our plugin.__

To write your first plugin. Navigate to /game/plugin. Here you will see plugins that are already a part of the project. 
We want to create a directory for our plugin as well. To do this, right click on the plugins folder and click New->Directory. We will name this directory 'myplugin'

Inside our directory we will want to create another directory called 'src'
Right click our src directory and click 'Mark Directory as->Sources Root' it will turn blue.
Also inside our directory we want to create a file named 'meta.toml'
Inside the meta.toml we will want to add some info:
```
plugin {
    name = "myplugin"
    packageName = "org.apollo.game.plugin.myplugin"
    authors = [
        "your name",
    ]
    
}
```

The 'name' field is how other plugins will find our plugin.
The 'packageName' field is where our plugin is located when the code runs. In this case, take note that our directory name is what is at the end of the classpath.
The 'authors' is a array of people who worked on the plugin.
Sometimes you need to use code from another plugin. If this is the case you can use the 'dependencies' field like some
`dependencies = [
        "util:lookup",
    ]`
This imports the lookup plugin from utils.

Finally the config section tells the gradle build system where our plugin code will go.


__Third, create the plugin!__

Now we will create our plugin. To do this, we need to make a code file.

Lets create a new file in our 'src' directory called 'myplugin.plugin.kts'
The .plugin.kts part of the name tells apollo that we are a plugin code file. 

Lets add some functionality to our plugin. In our 'myplugin.plugin.kts' file lets add the following code

```
import org.apollo.game.action.Action
import org.apollo.game.message.impl.InventoryItemMessage
import org.apollo.game.model.Item
import org.apollo.game.model.entity.Entity
import org.apollo.game.model.entity.EntityType
import org.apollo.game.model.entity.GroundItem
import org.apollo.game.model.entity.Player

class DropItemAction(val player: Player, val item: Int): Action<Player>(0, true, player) {
    override fun execute() {
        val region = player.world.regionRepository.fromPosition(player.position)
        if (region.getEntities<Entity>(player.position, EntityType.DYNAMIC_OBJECT, EntityType.STATIC_OBJECT).isEmpty()) {
            val amount = player.inventory.getAmount(item)
            System.out.println(amount)
            player.inventory.remove(item, amount)
            val groundItem = GroundItem.dropped(player.world, player.position, Item(item, amount), player)
            player.world.spawn(groundItem)
        } else {
            player.sendMessage("You cannot drop this here.")
        }
        stop()
    }

}

on {InventoryItemMessage::class}
        .where {option == 5 && interfaceId == 3214}
        .then {
            it.startAction(DropItemAction(it, id))
            terminate()
        }
```


So, this is a lot at first, but lets break it down so it is simple.

The first thing is the `on {...}` lambda function at the end. This is how we catch events 
Here we are catching the InventoryItemMessage, this is called when ever the player performs an action on an item in an inventory. 
The 'where' clause allows us to filter out requests we do not care about. In our where clause, we are filtering out any option that is not option 5 and any interface that is not 3214.
Option 5 is the drop button when the user right clicks an item. Interface 3214 is the user's inventory.

The 'then' clause is where we put out code that we want to run when the message is the one we want.

In this code block, the `it` variable is the player object that called the message. We are telling the player to run an action. After, we tell the server we are done with the message by terminating it. 

The DropItemAction class is where we put our action. This class extends the Action class with the player type to indicate that a player is running the action. When we initialize the action, `0` is how many server ticks to delay the action by. The `true` is if the action should ignore the delay and run immediately.

In the action we override the execute function. This is where the logic of the action occurs. This function will get run every tick until the `stop()` function is called. 

In our execute function, we put in the logic to drop items. 
Important functions to note here: 
`player.sendMessage(...)` displays a message in the players chat box from the server
`player.world.spawn(...)` Spawns an entity into the world

Important objects to note here:
`player.inventory` is the players inventory
`player.world` is a good way to access the world that the player is in
The `region` is a object that stores what is going on in that part of the map. The world consists of many regions. This allows for easier management of the information in the world. 

Now you can build it by running a gradle build. To open gradle. go to View->Tool Windows->Gradle
Then click Execute Gradle Task. Type 'run' into the command. 

You are now running your first plugin!
