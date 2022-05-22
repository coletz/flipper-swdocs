---
sidebar_position: 5
---

# Callbacks

Currently, our program simply does this:

1. Sets up our GameState
2. Allocates a ViewPort
3. Open our gui record
4. Add the ViewPort to the Gui
5. Cleans everything up and exits

<br></br>
<p align="center">
No drawing to the screen, and no input processing.
<br></br>

**Let's change that with callbacks!**
</p>

### Callback Methods

The `gui` service provides us with two nice methods for handling drawing and input.

These are aptly declared: `view_port_draw_callback_set` and `view_port_input_callback_set`

Let's look at their full declarations:

```c
void view_port_draw_callback_set(ViewPort* view_port, ViewPortDrawCallback callback, void* context);
void view_port_input_callback_set(ViewPort* view_port, ViewPortInputCallback callback, void* context);
```
As you might guess, `view_port_draw_callback_set` sets the callback function that is called whenever a new frame is signalled to be drawn.

And `view_port_input_callback_set` sets the callback function that is called whenever input is recieved.


So, what would we like to do with the callbacks?
- Draw: Draw a box from our GameState's values
- Input: Handle input from the keys
  
Let's tackle the draw callback first.

### Draw Callback

Callbacks pose a problem because they run on separate threads than our main app. This means that accessing our GameState could result in a [race condition](https://www.techtarget.com/searchstorage/definition/race-condition) between our callbacks and main loop.

Let's fix that by using [mutex](https://en.wikipedia.org/wiki/Lock_(computer_science)) to wrap our GameState as a blocking resource, only allowing one thread access at a time.

```c
    // --snip--
    int32_t box_mover_app(void* p){
    UNUSED(p);

    GameState* game_state = malloc(sizeof(GameState));
    init_state(game_state);

    ValueMutex state_mutex;
    if(!init_mutex(&state_mutex, game_state, sizeof(GameState))){ // initialize our mutex and checks for error
        free(game_state);
        return -1;
    }

    // --snip--
    return 0;
}
```

Great! Now our GameState can be accessed by multiple threads without having to worry. Let's implement a simple draw callback now.

```c
// --snip--
void draw_callback(Canvas* canvas, void* ctx){
    GameState* game_state = (GameState*)acquire_mutex((ValueMutex*)ctx, 25) // Acquires mutex with a timeout of 25 ticks.
    if(game_state==NULL){ // If we cannot acquire, exit.
        return;
    }

    canvas_draw_dot(canvas, game_state->x, game_state->y); // Draw a pixel on the screen at game_state->x,y

    release_mutex((ValueMutex*)ctx, game_state); // Release our mutex to be used by other threads.
}
// --snip--
int32_t box_mover_app(void* p){
    // --snip--
    
    view_port_draw_calback_set(view_port, draw_callback, &state_mutex);

    // --snip--
    return 0;
}
```

Now our `ViewPort` is set up with a drawing callback! Next, we need to implement 

:::note
At this point, the program does nothing. This is expected behavior.
:::