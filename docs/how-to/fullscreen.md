# Fullscreen mode

This guide is a follow up to [Get started](./get-started.md).

Here is some code that can toggle your game's fullscreen mode with borderless as an option.

```csharp
bool _isFullscreen = false;
bool _isBorderless = false;
int _width = 0;
int _height = 0;

public void ToggleFullscreen(GraphicsDeviceManager graphics, GameWindow window) {
    _isFullScreen = !_isFullScreen;

    if (_isFullScreen) {
        SetFullscreen(graphics, window);
    } else {
        UnsetFullscreen(graphics);
    }
}
public void ToggleBorderless(GraphicsDeviceManager graphics, GameWindow window) {
    _isBorderless = !_isBorderless;

    if (_isFullscreen) {
        SetBorderless(graphics);
    } else {
        _isFullscreen = true;
        SetFullscreen(graphics, window);
    }
}

public void SetBorderless(GraphicsDeviceManager graphics) {
    graphics.HardwareModeSwitch = !_isBorderless;
    graphics.ApplyChanges();
}
public void SetFullscreen(GraphicsDeviceManager graphics, GameWindow window) {
    _width = Window.ClientBounds.Width;
    _height = Window.ClientBounds.Height;

    graphics.PreferredBackBufferWidth = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Width;
    graphics.PreferredBackBufferHeight = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Height;
    graphics.HardwareModeSwitch = !_isBorderless;

    graphics.IsFullScreen = true;
    graphics.ApplyChanges();
}
public void UnsetFullscreen(GraphicsDeviceManager graphics) {
    graphics.PreferredBackBufferWidth = _width;
    graphics.PreferredBackBufferHeight = _height;
    graphics.IsFullScreen = false;
    graphics.ApplyChanges();
}
```

## Explanation

Track the current fullscreen state. Initialized to false since the game starts in window mode.

```csharp
bool _isFullscreen = false;
bool _isBorderless = false;
```

---

Going fullscreen means changing the current window's width and height. It's a good idea to save the previous values so that we can restore the window mode. Initialized to 0 but the actual value will be set by SetFullscreen.

```csharp
int _width = 0;
int _height = 0;
```

---

Borderless mode happens when the hardware mode switch is set to false. This method is called when the game is already in fullscreen.

```csharp
public void SetBorderless(GraphicsDeviceManager graphics) {
    graphics.HardwareModeSwitch = !_isBorderless;
    graphics.ApplyChanges();
}
```

---

Going fullscreen we preserve the current window size. Then we apply the size of the monitor that the window is currently in. This method handles the current borderless state.

```csharp
public void SetFullscreen(GraphicsDeviceManager graphics, GameWindow window) {
    _width = Window.ClientBounds.Width;
    _height = Window.ClientBounds.Height;

    graphics.PreferredBackBufferWidth = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Width;
    graphics.PreferredBackBufferHeight = GraphicsAdapter.DefaultAdapter.CurrentDisplayMode.Height;
    graphics.HardwareModeSwitch = !_isBorderless;

    graphics.IsFullScreen = true;
    graphics.ApplyChanges();
}
```

---

To go back to the window mode, we restore the previously saved sizes.

```csharp
public void UnsetFullscreen(GraphicsDeviceManager graphics) {
    graphics.PreferredBackBufferWidth = _width;
    graphics.PreferredBackBufferHeight = _height;
    graphics.IsFullScreen = false;
    graphics.ApplyChanges();
}
```

---

Toggling the fullscreen state is a matter of calling the respective methods based on the state we're tracking.

```csharp
public void ToggleFullscreen(GraphicsDeviceManager graphics, GameWindow window) {
    _isFullScreen = !_isFullScreen;

    if (_isFullScreen) {
        SetFullscreen(graphics, window);
    } else {
        UnsetFullscreen(graphics);
    }
}
```

---

Toggling the borderless mode is a bit trickier. This toggle makes sure that no matter what we're in fullscreen mode. If we're already in fullscreen mode, then it's just a matter of switching the hardware switch.

```csharp
public void ToggleBorderless(GraphicsDeviceManager graphics, GameWindow window) {
    _isBorderless = !_isBorderless;

    if (_isFullscreen) {
        SetBorderless(graphics);
    } else {
        _isFullscreen = true;
        SetFullscreen(graphics, window);
    }
}
```
