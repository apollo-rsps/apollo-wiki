Area actions are actions that are executed when a player enters, moves around in, or leaves a defined area. This is useful for implementing area-specific behaviour (such as enabling player-vs-player combat when a player is in the wilderness).

## Requirements

Any code that defines areas or creates area actions depends on the `areas` plugin, which is distributed with Apollo, and the `plugin.xml` file must specify this.

## Areas

Areas are user-defined parts of the game world that can have actions attached to them, which will be executed when on player movement. Each area must have a name, a boundary, and a set of actions which will be executed.

### Area Syntax

To create an area with a single action, the syntax is as follows:

    area :name => :area_name, :coordinates => [ min_x, min_y, max_x, max_y, height ], :actions => :action_name

If multiple actions are required, the `:actions` value should be an array:

    area :name => :area_name, :coordinates => [ min_x, min_y, max_x, max_y, height ], :actions => [ :action_name, :other_action_name ]

The `height` value is optional; if it is not present, the action will be executed regardless of the height level the player is on.

## Actions

As mentioned above, actions are code blocks that will be executed when a player enters, moves around in, or leaves an area. Each action must have a name, and at least one of these types being listened for.

### Action Syntax

There are three types of movement that can be listened for using an area action: a player entering the area, a player moving around inside the area, or a player leaving the area. To create an action, the syntax is as follows:

    area_action :action_name do

        # more code here

    end

To listen for a player entering the area, an `on_entry` block should be defined:

    on_entry do |player|
        # code here
    end

To listen for a player moving around inside the area, a `while_in` block should be defined:

    while_in do |player|
        # code here
    end

Note that this is only executed if the player moves around inside the area; if the player stands still inside the area, it will not be executed.

To listen for a player exiting the area, an `on_exit` block should be defined:

    on_exit do |player|
        # code here
    end

If the code inside a block is a single statement, the short style block notation can be used (for all of three types):

    on_entry { |player| ... }